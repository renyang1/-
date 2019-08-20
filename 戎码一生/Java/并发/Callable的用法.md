#### 问题：

我们通常使用继承Thread、实现Runnable接口这两种方式来创建线程，但该种方式有以下缺	 

点：无返回值、不会抛出异常。

解决：

从JDK1.5开始就提供了Callable和Future，Callable与Runnable一样表示要执行的异步任务，但	

Callable有结果，会抛出异常。Future表示异步任务的结果。即Callable用于产生结果，Runnable

用于获取结果。



#### Callable:

Callable位于java.util.concurrent包下，它也是一个接口，在它里面也只声明了一个方法，只不过	

这个方法叫做call()，这是一个泛型接口，call()函数返回的类型就是传递进来的V类型。

~~~java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
~~~

Callable一般情况下配合ExecutorService来使用，常有以下两种用法：

1. 将Callable对象直接作为调用ExecutorService中submit()方法的参数，得到方法的返回值  

​    Future对象，利用submit()方法返回的Future对象可以得到异步结果。

2. 先利用Callable对象构造出FutureTask对象，再将FutureTask对象作为调用ExecutorService中

​     execute()方法的参数，该方法无返回值。但可以利用之前创建的FutureTask对象得到异步结	

​     果。

#### Executor

Executor表示最简单的执行服务，其定义为：

```java
public interface Executor {
    void execute(Runnable command);
}
```

就是可以执行一个Runnable，没有返回结果。接口没有限定任务如何执行，可

能是创建一个新线程，可能是复用线程池中的某个线程，也可能是在调用者线

程中执行。

#### ExecutorService

ExecutorService扩展了Executor，定义了更多服务，基本方法有：

~~~java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    //... 其他方法
    void execute(Runnable command);//继承自Executor接口
}
~~~

这三个submit都表示提交一个任务，==返回值类型都是Future==，返回后，只是表示任务已提交，不代表已执行，通过Future可以查询异步任务的状态、获取最终结果、取消任务等。我们知道，对于Callable，任务最终有个返回值，而对于Runnable是没有返回值的，第二个提交Runnable的方法==可以同时提供一个结果，在异步任务结束时返回==，而对于第三个方法，异步任务的最终返回值为null。

#### Future

我们来看Future接口的定义：

~~~java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, 
        ExecutionException, TimeoutException;
}
~~~

get用于返回异步任务最终的结果，如果任务还未执行完成，会==阻塞等待==，另一个get方法可以限定阻塞等待的时间，如果==超时任务还未结束，会抛出TimeoutException==。

cancel用于取消异步任务，如果任务已完成、或已经取消、或由于某种原因不能取消，cancel返回false，否则返回true。如果任务还未开始，则不再运行。但如果任务已经在运行，则不一定能取消，参数mayInterruptIfRunning表示，如果任务正在执行，是否调用interrupt方法中断线程，如果为false就不会，如果为true，就会==尝试==中断线程，但我们知道，中断不一定能取消线程。

isDone和isCancelled用于查询任务状态。isCancelled表示任务是否被取消，只要cancel方法返回了true，随后的isCancelled方法都会返回true，即使执行任务的线程还未真正结束。isDone表示任务是否结束，不管什么原因都算，可能是任务正常结束、可能是任务抛出了异常、也可能是任务被取消。

我们再来看下get方法，任务最终大概有三个结果：

1. 正常完成，get方法会返回其执行结果，如果==任务是Runnable且没有提供结果，返回null==
2. 任务执行抛出了异常，get方法会将异常包装为ExecutionException重新抛出，通过异常的	

​    getCause方法可以获取原异常

1. 任务被取消了，get方法会抛出异常CancellationException

如果调用get方法的线程被中断了，get方法会抛出InterruptedException。



#### FutureTask

ExecutorService最基本的方法是submit，它是如何实现的呢？我们来看AbstractExecutorService	

的代码：

~~~java
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
~~~

它调用newTaskFor生成了一个RunnableFuture，RunnableFuture是一个接口，既扩展了Runnable，又扩展了Future，没有定义新方法，作为Runnable，它表示要执行的任务，传递给execute方法进行执行，作为Future，它又表示任务执行的异步结果。这可能令人混淆，我们来看具体代码：

~~~java
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
~~~

就是创建了一个FutureTask对象,所以最终其实是将FutureTask对象做为参数，调用	

ExecutorService的execute()方法。

demo1: 先创建FutureTask对象，利用该对象获取异步结果信息

~~~java
/**
 * Description:先创建FutureTask对象，利用该对象获取异步结果信息
 *
 * @auther: renyang
 * @param:
 * @return:
 * @date: 2019/3/4 10:11
 */
public static void firstWay() {

    long waitTime = 1000L;
    // 创建FutureTask对象
    FutureTask<String> futureTask1 = new FutureTask<>(() -> {
        long waitTime1 = waitTime;
        Thread.sleep(waitTime1);
        return Thread.currentThread().getName();
    });

    FutureTask<String> futureTask2 = new FutureTask<>(() -> {
        long waitTime2 = waitTime + 1000L;
        Thread.sleep(waitTime2);
        return Thread.currentThread().getName();
    });

    // 创建线程池并返回ExecutorService对象实例
    ExecutorService executor = Executors.newFixedThreadPool(2);
    // 执行任务1
    executor.execute(futureTask1);
    // 执行任务2
    executor.execute(futureTask2);

    // 判断任务是否执行完成
    while (true) {
        try {
            if (futureTask1.isDone() && futureTask2.isDone()) {// 两个任务均执行完成

                System.out.println("All Finished");
                executor.shutdown();
                return;
            }

            if (!futureTask1.isDone()) {// 任务1没有完成，等待任务1完成
                // 该方法会阻塞等待，直到任务完成
                String thread1Name = futureTask1.get();
                System.out.println(thread1Name);
            }

            System.out.println("waiting for futureTask2 to complete");
            // 超时等待，200s没返回则抛出TimeOutException
            String thread2Name = futureTask2.get(200L, TimeUnit.MILLISECONDS);

            if (thread2Name != null) {
                System.out.println(thread2Name);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
~~~



 * Description:先创建FutureTask对象，利用该对象获取异步结果信息
 *
 * @auther: renyang
 * @param:
 * @return:
 * @date: 2019/3/4 10:11
 */
public static void firstWay() {

    long waitTime = 1000L;
    // 创建FutureTask对象
    FutureTask<String> futureTask1 = new FutureTask<>(() -> {
        long waitTime1 = waitTime;
        Thread.sleep(waitTime1);
        return Thread.currentThread().getName();
    });

    FutureTask<String> futureTask2 = new FutureTask<>(() -> {
        long waitTime2 = waitTime + 1000L;
        Thread.sleep(waitTime2);
        return Thread.currentThread().getName();
    });

    // 创建线程池并返回ExecutorService对象实例
    ExecutorService executor = Executors.newFixedThreadPool(2);
    // 执行任务1
    executor.execute(futureTask1);
    // 执行任务2
    executor.execute(futureTask2);

    // 判断任务是否执行完成
    while (true) {
        try {
            if (futureTask1.isDone() && futureTask2.isDone()) {// 两个任务均执行完成

    ​            System.out.println("All Finished");
    ​            executor.shutdown();
    ​            return;
    ​        }

    ​        if (!futureTask1.isDone()) {// 任务1没有完成，等待任务1完成
    ​            // 该方法会阻塞等待，直到任务完成
    ​            String thread1Name = futureTask1.get();
    ​            System.out.println(thread1Name);
    ​        }

    ​        System.out.println("waiting for futureTask2 to complete");
    ​        // 超时等待，200s没返回则抛出TimeOutException
    ​        String thread2Name = futureTask2.get(200L, TimeUnit.MILLISECONDS);

    ​        if (thread2Name != null) {
    ​            System.out.println(thread2Name);
    ​        }
    ​    } catch (Exception e) {
    ​        e.printStackTrace();
    ​    }
    }
}



demo2: 通过ExecutorService的submit()方法执行任务，不创建TaskFuture对象

~~~java
/**
 * Description:通过ExecutorService的submit()方法执行任务，
 *             不创建TaskFuture对象
 * @auther: renyang
 * @param:
 * @return:
 * @date: 2019/3/4 14:19
 */
public static void sencondWay() {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<String> future = executor.submit(()->{
        return Thread.currentThread().getName();
    });

    try {
        String threadName = future.get();
        System.out.println(threadName);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    executor.shutdown();
}

demo3: 利用Future获取Runnable异步任务结果
/**
 * Description:利用Future获取Runnable异步任务结果
 * @auther: renyang
 * @param:
 * @return:
 * @date: 2019/3/4 18:09
 */
public static void runnableTest() {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    // submit参数为Runnable对象实例
    Future future = executor.submit(()-> {
        System.out.println("start...");
        try {
            Thread.sleep(2000L);
            System.out.println(Thread.currentThread().getName());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });

    boolean isDone = future.isDone();
    System.out.println(isDone);
    try {
        // 获取Runnable异步任务返回值
        Object object = future.get();
        System.out.println(object);// null
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    executor.shutdown();
}
~~~

