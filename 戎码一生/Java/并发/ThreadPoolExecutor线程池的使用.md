### ThreadPoolExecutor线程池的使用

#### 一： 创建线程池对象

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(2, 20, 0L
                , TimeUnit.MILLISECONDS, new LinkedBlockingDeque<>(1024)
                , Executors.defaultThreadFactory()
                , new ThreadPoolExecutor.AbortPolicy());	
```

==构造函数==：

```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
```

######  构造函数参数含义：

**corePoolSize**:指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到         						workQueue任务队列中去；

**maximumPoolSize**:指定了线程池中的最大线程数量，这个参数会==根据使用的workQueue任务队列的类型==，决定线程池会开辟的最大线程数量；

**keepAliveTime**:当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；

**unit**:keepAliveTime的单位

**workQueue**:任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；

**threadFactory**:线程工厂，用于创建线程，一般用默认即可；

**handler**:拒绝策略；当任务太多来不及处理时，如何拒绝任务；

#### 二：workQueue任务队列

##### 1.==SynchronousQueue==（直接提交队列）

​	SynchronousQueue是一个特殊的BlockingQueue，它==没有容量==，每执行一个插入操作就会阻塞，需要再执行一个删除操作才会被唤醒，反之每一个删除操作也都要等待对应的插入操作。

~~~java
public static void useSynchronousQueue() {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(1, 2, 1000
                , TimeUnit.MILLISECONDS, new SynchronousQueue<>()
                , Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
        for (int i = 0; i < 10; i++) {
            pool.execute(()-> System.out.println(Thread.currentThread().getName()));
        }
        // 关闭线程池
        pool.shutdown();
    }
~~~

输出结果：

~~~
pool-1-thread-1
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task com.ryang.thread.threadpool.ThreadPoolExecutorDemo$$Lambda$1/558638686@7699a589 rejected from java.util.concurrent.ThreadPoolExecutor@58372a00[Running, pool size = 2, active threads = 2, queued tasks = 0, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2047)
	at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:823)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1369)
	at com.ryang.thread.threadpool.ThreadPoolExecutorDemo.useSynchronousQueue(ThreadPoolExecutorDemo.java:35)
	at com.ryang.thread.threadpool.ThreadPoolExecutorDemo.main(ThreadPoolExecutorDemo.java:25)
pool-1-thread-2
~~~

​	可以看到，当任务队列为SynchronousQueue，创建的线程数大于maximumPoolSize时，直接执行了拒绝策略抛出异常。

​	使用SynchronousQueue队列，提交的任务不会被保存，总是会马上提交执行。如果用于执行任务的线程数量小于maximumPoolSize，则尝试创建新的进程，如果达到maximumPoolSize设置的最大值，则根据你设置的handler执行拒绝策略。因此这种方式你提交的==任务不会被缓存起来，而是会被马上执行==，在这种情况下，你需要==对并发量有个准确的评估==，才能设置合适的maximumPoolSize数量，否则很容易就会执行拒绝策略；

##### 2. ==ArrayBlockingQueue==（**有界的任务队列**）

​	使用ArrayBlockingQueue有界任务队列，若有新的任务需要执行时，线程池会创建新的线程，直到创建的线程数量达到corePoolSize时，则会将新的任务加入到等待队列中。若等待队列已满，即超过ArrayBlockingQueue初始化的容量，则继续创建线程，直到线程数量达到maximumPoolSize设置的最大线程数量，若大于maximumPoolSize，则执行拒绝策略。在这种情况下，线程数量的上限与有界任务队列的状态有直接关系，==如果有界队列初始容量较大或者没有达到超负荷的状态，线程数将一直维持在corePoolSize以下==，反之当任务队列已满时，则会以maximumPoolSize为最大线程数上限。

##### 3. ==LinkedBlockingQueue==（无界的任务队列）

​	使用无界任务队列，线程池的任务队列可以无限制的添加新的任务，而==线程池创建的最大线程数量就是你corePoolSize设置的数量==，也就是说在这种情况下==maximumPoolSize这个参数是无效的==，哪怕你的任务队列中缓存了很多未执行的任务，当线程池的线程数达到corePoolSize后，就不会再增加了；若后续有新的任务加入，则直接进入队列等待，当使用这种任务队列模式时，一定要注意你任务提交与处理之间的协调与控制，不然会出现队列中的任务由于无法及时处理导致一直增长，直到最后资源耗尽的问题。

##### 4. ==PriorityBlockingQueue==（优先任务队列）

​	PriorityBlockingQueue它其实是一个==特殊的无界队列==，它其中无论添加了多少个任务，==线程池创建的线程数也不会超过corePoolSize==的数量，只不过其他队列一般是按照==先进先出==的规则处理任务，而PriorityBlockingQueue队列可以自定义规则根据任务的==优先级顺序==先后执行。

#### 三：拒绝策略

**1、AbortPolicy策略**：该策略会直接抛出异常，阻止系统正常工作；

**2、CallerRunsPolicy策略**：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中												 运行；

**3、DiscardOledestPolicy策略**：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交；

**4、DiscardPolicy策略**：该策略会默默丢弃无法处理的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失；

​	以上内置的策略均实现了RejectedExecutionHandler接口，当然也==可以自己扩展RejectedExecutionHandler接口，定义自己的拒绝策略==。

#### 四：ThreadFactory自定义线程创建

​	线程池中线程就是通过ThreadPoolExecutor中的ThreadFactory，线程工厂创建的。那么通过自定义ThreadFactory，==可以按需要对线程池中创建的线程进行一些特殊的设置==，如命名、优先级等。

```java
/**
     * Description:使用自定义的ThreadFactory,ThreadFactory是一个函数式接口，使用lambda表达式
     * @auther: renyang
     * @param:
     * @return:
     * @date: 2019/8/13 19:33
     */
    public static void useMyThreadFactory() {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(2, 5, 1000
                , TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(5)
                , runnable -> new Thread(runnable, "threadPool" + runnable.hashCode())
                , new ThreadPoolExecutor.AbortPolicy());
        for (int i = 0; i < 7; i++) {
            pool.execute(() -> System.out.println(Thread.currentThread().getName()));
        }
        pool.shutdown();
    }
```

结果：

~~~
threadPool1989780873
threadPool1989780873
threadPool1989780873
threadPool1989780873
threadPool1989780873
threadPool1989780873
threadPool1480010240
~~~

#### 五：ThreadPoolExecutor扩展

ThreadPoolExecutor扩展主要是围绕beforeExecute()、afterExecute()和terminated()三个接口实现的，

**1、beforeExecute：线程池中任务运行前执行**

**2、afterExecute：线程池中任务运行完毕后执行**

**3、terminated：线程池退出后执行**

通过这三个接口我们可以监控每个任务的开始和结束时间，或者其他一些功能。

~~~java
 public static void expandThreadPoolExecutor() {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(2, 5, 1000
                , TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(5)
                , runnable -> new Thread(runnable, "threadPool" + runnable.hashCode())
                , new ThreadPoolExecutor.AbortPolicy()){
            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println("准备执行："+ ((ThreadTask)r).getTaskName());
            }

            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println("执行完毕："+((ThreadTask)r).getTaskName());
            }

            @Override
            protected void terminated() {
                System.out.println("线程池退出");
            }
        };
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadPoolExecutorDemo().new ThreadTask("Task"+i));
        }
        pool.shutdown();
    }

    class ThreadTask implements Runnable{
        // todo: 当线程有属性时，怎么用lambda表达式表示？
        private String taskName;
        public String getTaskName() {
            return taskName;
        }
        // todo:这里怎么用lambda表达式写？
        public void setTaskName(String taskName) {
            this.taskName = taskName;
        }
        public ThreadTask(String name) {
            this.setTaskName(name);
        }
        public void run() {
            //输出执行线程的名称
            System.out.println("TaskName"+this.getTaskName()+"---				 						ThreadName:"+Thread.currentThread().getName());
        }
    }
~~~

