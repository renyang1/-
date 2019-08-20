### 并发相关API

#### Object类API

1. **notify**

   两个重载方法：

   ~~~java
   public final void notify()
   public final void notifyAll()
   ~~~

   notify做的事情就是从条件队列中选一个线程，将其从队列中移除并唤醒，notifyAll和notify的区别是，它会移除条件队列中所有的线程并全部唤醒。

2. **wait**

   三个重载方法：

   ~~~java
   public final void wait() throws InterruptedException
   public final void wait(long timeout) throws InterruptedException
   public final void wait(long timeout, int nanos) throws InterruptedException
   ~~~

   一个带时间参数，单位是毫秒，表示最多等待这么长时间，==参数为0表示无限期等待==。不带时间参数，表示无限期等待，实际就是调用wait(0)。在等待期间都可以被中断，如果被中断，会抛出InterruptedException。==wait方法被调用后会释放锁。==

   **wait实际上做了什么呢？它在等待什么？**

   ==每个对象==都有一把**锁**和**等待队列**，一个线程在进入synchronized代码块时，会尝试获取锁，获取不到的话会把当前线程加入等待队列中，其实，除了用于锁的等待队列，每个对象还有另一个等待队列，表示**条件队列**，该队列用于线程间的协作。调用wait就会把当前线程放到条件队列上并阻塞，表示当前线程执行不下去了，它需要等待一个条件，这个条件它自己改变不了，需要其他线程改变。

#### Thread类API

1. **id和name**

   每个线程都有一个id和name，id是一个递增的整数，每创建一个线程就加一，name的默认值是"Thread-"后跟一个编号，name可以在Thread的构造方法中进行指定，也可以通过setName方法进行设置，给Thread设置一个友好的名字，可以方便调试。

2. **优先级**

   >public final void setPriority(int newPriority)
   >
   >public final int getPriority()

   ==优先级从1到10，默认为5==

3. **状态**

   > public State getState()

4. **是否daemo线程**

   >public final void setDaemon(boolean on)
   >
   >public final boolean isDaemon()

5. **sleep方法**

   >public static native void sleep(long millis) throws InterruptedException;

   ==静态方法==，sleep方法会让当前线程睡眠指定的时间，单位是毫秒。睡眠期间，==该线程会让出CPU，但不会释放持有的锁==。睡眠的时间不一定是确切的给定毫秒数，可能有一定的偏差，偏差与系统定时器和操作系统调度器的准确度和精度有关。睡眠期间，线程可以被中断，如果被中断，sleep会抛出InterruptedException。

6. **yield方法**

   > public static native void yield();

   ==静态方法==，调用该方法是告诉操作系统的调度器，我现在不着急占用CPU，你可以先让其他线程运行。不过，这对调度器也==仅仅是建议==，调度器如何处理是不一定的，它可能完全忽略该调用。

7. **join方法**

   > public final void join() throws InterruptedException
   >
   > public final synchronized void join(long millis) throws InterruptedException

   ==让调用join的线程等待该线程结束==，在等待线程结束的过程中，这个等待可能被中断，如果被中断，会抛出InterruptedException。

   ~~~java
   public static void main(String[] args) throws InterruptedException {
       Thread thread = new HelloThread();
       thread.start();
       thread.join();
   }    
   ~~~

   以上代码调用join的线程为main线程，即目的是为了让main线程在子线程thread结束后再退出。

#### 