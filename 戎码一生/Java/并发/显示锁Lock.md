### 显示锁

Java并发包中的显式锁接口和类位于包java.util.concurrent.locks下，主要接口和类有：

- 锁接口==Lock==，主要实现类是**ReentrantLock**
- 读写锁接口==ReadWriteLock==，主要实现类是**ReentrantReadWriteLock**

#### Lock

显式锁接口Lock的定义为：

>public interface Lock {
>    void lock();
>    void lockInterruptibly() throws InterruptedException;
>    boolean tryLock();
>    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
>    void unlock();
>    Condition newCondition();
>}

- lock()/unlock()：就是普通的获取锁和释放锁方法，==lock()会阻塞直到成功==。
- lockInterruptibly()：与lock()的不同是，它可以响应中断，如果被其他线程中断了，抛出InterruptedException。
- ==tryLock()==：只是尝试获取锁，==立即返回，不阻塞==，如果获取成功，返回true，否则返回false。
- tryLock(long time, TimeUnit unit) ：先尝试获取锁，如果能成功则立即返回true，否则阻塞等待，但等待的最长时间为指定的参数，在等待的同时响应中断，如果发生了中断，抛出InterruptedException，如果在等待的时间内获得了锁，返回true，否则返回false。
- newCondition：新建一个条件，一个Lock可以关联多个条件。

**相比synchronized，显式锁支持以非阻塞方式获取锁、可以响应中断、可以限时，这使得它灵活的多。**

******

#### 可重入锁ReentrantLock

**基本用法**

Lock接口的主要实现类是ReentrantLock，它的基本用法lock/unlock实现了与synchronized一样的语义，包括：

- 可重入，一个线程在持有一个锁的前提下，可以继续获得该锁
- 可以解决竞态条件问题
- 可以保证内存可见性

ReentrantLock有两个**构造方法**：

>public ReentrantLock()
>
>public ReentrantLock(boolean fair) 

==参数fair表示是否保证公平==，不指定的情况下，==默认为false==，表示不保证公平。所谓公平是指，等待时间最长的线程优先获得锁。保证公平会影响性能，一般也不需要，所以默认不保证，synchronized锁也是不保证公平的。

**使用显式锁，一定要记得调用unlock，应该将lock之后的代码包装到try语句内，在finally语句内释放锁。**

```java
public class Counter {
    private final Lock lock = new ReentrantLock();
    private volatile int count;

    public void incr() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        return count;
    }
}
```

**使用tryLock避免死锁**

==使用tryLock()，可以避免死锁。==在持有一个锁，获取另一个锁，获取不到的时候，可以释放已持有的锁，给其他线程机会获取锁，然后再重试获取所有锁。

