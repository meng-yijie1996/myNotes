## Java锁之公平锁和非公平锁

### 概念

#### 公平锁

是指多个线程按照申请锁的**顺序**来获取锁，类似于排队买饭，先来后到，先来先服务，就是公平的，也就是**队列**。

#### 非公平锁

是指多个线程获取锁的顺序，并不是按照申请锁的顺序，有可能申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级翻转，或者饥饿的线程（也就是某个线程一直得不到锁）。

### 如何创建

```java
// 创建一个可重入锁，true 表示公平锁，false 表示非公平锁。默认非公平锁
Lock lock = new ReentrantLock(true);

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

#### 源码

```java
// Sync object for fair locks
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;
    final void lock() {
        acquire(1);
    }
    // Fair version of tryAcquire.  Don't grant access unless recursive call or no waiters or is first.
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

```java
// Sync object for non-fair locks
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;
    // Performs lock.  Try immediate barge, backing up to normal acquire on failure.
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

### 两者区别

**公平锁**：就是很公平，在并发环境中，每个线程在获取锁时会先**查看**此锁维护的**等待队列**，如果为空，或者当前线程是等待队列中的第一个，就占用锁，否者就会加入到等待队列中，以后按照FIFO的规则从队列中取到自己。

**非公平锁：** 非公平锁比较粗鲁，上来就**直接**尝试占有锁，如果尝试失败，就再采用类似公平锁那种方式。

### 题外话

Java ReenttrantLock通过构造函数指定该锁是否公平，默认是非公平锁，因为非公平锁的优点在于吞吐量比公平锁大。

synchronized也是一种非公平锁。



## 可重入锁和递归锁ReentrantLock

### 概念

可重入锁**就是**递归锁

指的是同一线程外层函数获得锁之后，内层递归函数仍然能获取到该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。

**总结**：线程可以进入任何一个它已经拥有的锁所同步的代码块。

ReentrantLock / Synchronized 就是一个典型的可重入锁。

### 代码

可重入锁就是，在一个method1方法中加入一把锁，方法2也加锁了，那么他们拥有的是同一把锁

```java
public synchronized void method1() {method2();}
public synchronized void method2() {}
```

也就是说我们只需要进入method1后，那么它也能直接进入method2方法，因为他们所拥有的锁，是同一把。

### 作用

可重入锁的最大作用就是**避免死锁**。

### 可重入锁验证

#### 证明Synchronized

```java
class Phone {
    public synchronized void sendSMS() throws Exception{
        System.out.println(Thread.currentThread().getName() + "\t invoked sendSMS()");
        sendEmail();
    }
    public synchronized void sendEmail() throws Exception{
        System.out.println(Thread.currentThread().getId() + "\t invoked sendEmail()");
    }
}
public class ReenterLockDemo {
    public static void main(String[] args) {
        Phone phone = new Phone();
        // 两个线程操作资源类
        new Thread(() -> {
            try {phone.sendSMS();} catch (Exception e) {e.printStackTrace();}
        }, "t1").start();
        new Thread(() -> {
            try {phone.sendSMS();} catch (Exception e) {e.printStackTrace();}
        }, "t2").start();
    }
}
```

在这里，我们编写了一个资源类phone，拥有两个加了synchronized的同步方法，分别是sendSMS 和 sendEmail，我们在sendSMS方法中，调用sendEmail。最后在主线程同时开启了两个线程进行测试，最后得到的结果为：

```
t1	 invoked sendSMS()      t1线程在外层方法获取锁的时候
t1	 invoked sendEmail()    t1在进入内层方法会自动获取锁
t2	 invoked sendSMS()      t2线程在外层方法获取锁的时候
t2	 invoked sendEmail()    t2在进入内层方法会自动获取锁
```

这就说明当 t1 线程进入sendSMS的时候，拥有了一把锁，同时t2线程无法进入，直到t1线程拿着锁，执行了sendEmail 方法后，才释放锁，这样t2才能够进入

#### 证明ReentrantLock

```java
class Phone implements Runnable{
    Lock lock = new ReentrantLock();
    // get进去的时候，就加锁，调用get方法的时候，能否访问另外一个加锁的set方法
    public void getLock() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t get Lock");
            setLock();
        } finally {
            lock.unlock();
        }
    }
    public void setLock() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t set Lock");
        } finally {
            lock.unlock();
        }
    }
    @Override
    public void run() {
        getLock();
    }
}
public class ReenterLockDemo {
    public static void main(String[] args) {
        Phone phone = new Phone();
        // 因为Phone实现了Runnable接口
        Thread t3 = new Thread(phone, "t3");
        Thread t4 = new Thread(phone, "t4");
        t3.start();
        t4.start();
    }
}
```

现在我们使用ReentrantLock进行验证，首先资源类实现了Runnable接口，重写Run方法，里面调用get方法，get方法在进入的时候，就加了锁。

然后在方法里面，又调用另外一个加了锁的setLock方法

最后输出结果我们能发现，结果和加synchronized方法是一致的，都是在外层的方法获取锁之后，线程能够直接进入里层

```
t3	 get Lock
t3	 set Lock
t4	 get Lock
t4	 set Lock
```

**当我们在getLock方法加两把锁会是什么情况呢？** (阿里面试)

```java
    public void getLock() {
        lock.lock();
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t get Lock");
            setLock();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }
```

最后得到的结果也是一样的，因为里面不管有几把锁，其它他们都是同一把锁，也就是说用同一个钥匙都能够打开.

**当我们在getLock方法加两把锁，但是只解一把锁会出现什么情况呢？**

```java
public void getLock() {
    lock.lock();
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName() + "\t get Lock");
        setLock();
    } finally {
        lock.unlock();
    }
}
```

得到结果

```
t3	 get Lock
t3	 set Lock
```

也就是说程序直接卡死，线程不能出来，也就说明我们**申请几把**锁，最后需要**解除几把**锁

**当我们只加一把锁，但是用两把锁来解锁的时候，又会出现什么情况呢？**

```java
    public void getLock() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t get Lock");
            setLock();
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }
```

这个时候，运行程序会直接报错

```
t3	 get Lock
t3	 set Lock
t4	 get Lock
t4	 set Lock
Exception in thread "t3" Exception in thread "t4" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at com.moxi.interview.study.thread.Phone.getLock(ReenterLockDemo.java:52)
	at com.moxi.interview.study.thread.Phone.run(ReenterLockDemo.java:67)
	at java.lang.Thread.run(Thread.java:745)
java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at com.moxi.interview.study.thread.Phone.getLock(ReenterLockDemo.java:52)
	at com.moxi.interview.study.thread.Phone.run(ReenterLockDemo.java:67)
	at java.lang.Thread.run(Thread.java:745)
```

```java
public void unlock() {
    sync.release(1);
}
```

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```



## Java锁之自旋锁

自旋锁：spinlock，是指尝试获取锁的线程**不会立即阻塞**，而是采用**循环**的方式去**尝试获取锁**，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

原来提到的比较并交换，底层使用的就是自旋，自旋就是多次尝试，多次访问，不会阻塞的状态就是**自旋**。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

### 优缺点

优点：循环比较获取直到成功为止，没有类似于wait的阻塞。

缺点：当不断自旋的线程越来越多的时候，会因为执行while循环不断的消耗CPU资源。

### 手写自旋锁

通过CAS操作完成自旋锁，A线程先进来调用myLock方法自己持有锁5秒，B随后进来发现当前有线程持有锁，不是null，所以只能通过自旋等待，直到A释放锁后B随后抢到

```java
// 手写一个自旋锁：循环比较获取直到成功为止，没有类似于wait的阻塞
public class SpinLockDemo {
    // 现在的泛型装的是Thread，原子引用线程。当前初始值为null
    AtomicReference<Thread>  atomicReference = new AtomicReference<>();
    public void myLock() {
        // 获取当前进来的线程
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "\t come in ");
        // 开始自旋，期望值是null，更新值是当前线程，如果是null，则更新为当前线程，否者自旋
        while(!atomicReference.compareAndSet(null, thread)) {
        }
    }
    // 解锁
    public void myUnLock() {
        // 获取当前进来的线程
        Thread thread = Thread.currentThread();
        // 自己用完了后，把atomicReference变成null
        atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName() + "\t invoked myUnlock()");
    }
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        // 启动t1线程，开始操作
        new Thread(() -> {
            // 开始占有锁
            spinLockDemo.myLock();
            try {TimeUnit.SECONDS.sleep(5);} catch (InterruptedException e) {e.printStackTrace();}
            // 开始释放锁
            spinLockDemo.myUnLock();
        }, "t1").start();
        // 让main线程暂停1秒，使得t1线程，先执行
        try {TimeUnit.SECONDS.sleep(1);} catch (InterruptedException e) {e.printStackTrace();}
        // 1秒后，启动t2线程，开始占用这个锁
        new Thread(() -> {
            // 开始占有锁
            spinLockDemo.myLock();
            // 开始释放锁
            spinLockDemo.myUnLock();
        }, "t2").start();
    }
}
```

最后输出结果

```
t1 come in
.....一秒后.....
t2 come in
.....五秒后.....
t1 invoked myUnlock()
t2 invoked myUnlock()
```

首先输出的是 t1 come in，然后1秒后，t2线程启动，发现锁被t1占有，所有不断的执行 compareAndSet方法，来进行比较，直到t1释放锁后，也就是5秒后，t2成功获取到锁，然后释放。



## 独占锁（写锁） / 共享锁（读锁） / 互斥锁

### 概念

独占锁：指该锁一次只能被一个线程所持有。对ReentrantLock和Synchronized而言都是独占锁。

共享锁：指该锁可以被多个线程锁持有。

对ReentrantReadWriteLock其读锁是共享，其写锁是独占。

写的时候只能一个人写，但是读的时候，可以多个人同时读。

读锁的共享锁可保证并发读是非常高效的，读写、写读、写写是互斥的。

### 为什么会有写锁和读锁？

原来我们使用ReentrantLock创建锁的时候，是独占锁，也就是说一次只能一个线程访问，但是有一个**读写分离场景**，读的时候想同时进行，因此原来独占锁的并发性就没这么好了，因为读锁并不会造成数据不一致的问题，因此可以多个人共享读。

多个线程同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行，但是如果一个线程想去写共享资源，就不应该再有其它线程可以对该资源进行读或写。

读-读：能共存

读-写：不能共存

写-写：不能共存

写操作需要满足：原子 + 独占，整个过程必须是一个完整的统一体，中间不许被分割打断。如案例代码中的正在写入和写入完成。

### 代码实现

实现一个读写缓存的操作，假设开始没有加锁的时候，会出现什么情况

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "\t 正在写入：" + key);
        try {TimeUnit.MILLISECONDS.sleep(300);} catch (InterruptedException e) {e.printStackTrace();}
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "\t 写入完成");
    }
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "\t 正在读取:");
        try {TimeUnit.MILLISECONDS.sleep(300);} catch (InterruptedException e) {e.printStackTrace();}
        Object value = map.get(key);
        System.out.println(Thread.currentThread().getName() + "\t 读取完成：" + value);
    }
}
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 0; i < 5; i++) {
            // lambda表达式内部必须是final
            final int tempInt = i;
            new Thread(() -> {myCache.put(tempInt + "", tempInt +  "");}, String.valueOf(i)).start();
        }
        for (int i = 0; i < 5; i++) {
            final int tempInt = i;
            new Thread(() -> {myCache.get(tempInt + "");}, String.valueOf(i)).start();
        }
    }
}
```

我们分别创建5个线程写入缓存，5个线程读取缓存。

运行结果：

```
0	 正在写入：0
4	 正在写入：4
3	 正在写入：3
1	 正在写入：1
2	 正在写入：2
0	 正在读取:
1	 正在读取:
2	 正在读取:
3	 正在读取:
4	 正在读取:
2	 写入完成
4	 写入完成
4	 读取完成：null
0	 写入完成
3	 读取完成：null
0	 读取完成：null
1	 写入完成
3	 写入完成
1	 读取完成：null
2	 读取完成：null
```

我们可以看到，在写入的时候，写操作都被其它线程打断了，这就造成了，还没写完，其它线程又开始写，这样就造成数据不一致。



### 解决方法

上面的代码是没有加锁的，这样就会造成线程在进行写入操作的时候，被其它线程频繁打断，从而不具备原子性，这个时候，我们就需要用到读写锁来解决了。

这里的读锁和写锁的区别在于，写锁一次只能一个线程进入，执行写操作，而读锁是多个线程能够同时进入，进行读取的操作。

完整代码：

```java
/**
 * 读写锁
 * 多个线程 同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行
 * 但是，如果一个线程想去写共享资源，就不应该再有其它线程可以对该资源进行读或写
 */
// 资源类
class MyCache {
    // 缓存中的东西，必须保持可见性，因此使用volatile修饰
    private volatile Map<String, Object> map = new HashMap<>();
    // 创建一个读写锁：它是一个读写融为一体的锁，在使用的时候，需要转换
    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    // 定义写操作。满足：原子 + 独占
    public void put(String key, Object value) {
        // 进行写操作的时候，就需要转换成写锁
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在写入：" + key);
            // 模拟网络拥堵，延迟0.3秒
            try {TimeUnit.MILLISECONDS.sleep(300);} catch (InterruptedException e) {e.printStackTrace();}
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t 写入完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 写锁 释放
            rwLock.writeLock().unlock();
        }
    }
    // 读取
    public void get(String key) {
        // 进行读操作的时候，再转换成读锁
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在读取:");
            // 模拟网络拥堵，延迟0.3秒
            try {TimeUnit.MILLISECONDS.sleep(300);} catch (InterruptedException e) {e.printStackTrace();}
            Object value = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t 读取完成：" + value);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 读锁释放
            rwLock.readLock().unlock();
        }
    }
    // 清空缓存
    public void clean() {
        map.clear();
    }
}
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        // 线程操作资源类，5个线程写
        for (int i = 1; i <= 5; i++) {
            // lambda表达式内部必须是final
            final int tempInt = i;
            new Thread(() -> {
                myCache.put(tempInt + "", tempInt +  "");
            }, String.valueOf(i)).start();
        }
        // 线程操作资源类， 5个线程读
        for (int i = 1; i <= 5; i++) {
            // lambda表达式内部必须是final
            final int tempInt = i;
            new Thread(() -> {
                myCache.get(tempInt + "");
            }, String.valueOf(i)).start();
        }
    }
}
```

运行结果：

```
1	 正在写入：1
1	 写入完成
2	 正在写入：2
2	 写入完成
3	 正在写入：3
3	 写入完成
4	 正在写入：4
4	 写入完成
5	 正在写入：5
5	 写入完成
2	 正在读取:
3	 正在读取:
1	 正在读取:
4	 正在读取:
5	 正在读取:
2	 读取完成：2
1	 读取完成：1
4	 读取完成：4
3	 读取完成：3
5	 读取完成：5
```

结论：

- 写入操作是一个一个线程进行执行的，并且中间不会被打断；
- 读操作是同时5个线程进入，然后并发读取操作。



## 锁的总结

