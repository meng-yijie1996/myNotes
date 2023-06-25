## CountDownLatch

`java.util.concurrent.CountDownLatch`

### 概念

让一些线程阻塞直到另一些线程完成一系列操作才被唤醒。

CountDownLatch主要有两个方法：

- 当一个或多个线程调用**await**方法时，调用线程就会被阻塞。
- 其它线程调用**countDown**方法会将计数器减1（调用CountDown方法的线程不会被阻塞），当计数器的值变成零时，因调用await方法被阻塞的线程会被唤醒，继续执行。

### 场景

现在有这样一个场景，假设一个自习室里有7个人，其中有一个是班长，班长的主要职责就是在其它6个同学走了后，关灯，锁教室门，然后走人，因此班长是需要最后一个走的，那么有什么方法能够控制班长这个线程是最后一个执行，而其它线程是随机执行的。

### 解决方案

使用CountDownLatch计数器：一共创建6个线程，然后计数器的值也设置成6。

```java
// 计数器
CountDownLatch countDownLatch = new CountDownLatch(6);
```

然后每次学生线程执行完，就让计数器的值减1

```java
for (int i = 0; i <= 6; i++) {
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + "\t 上完自习，离开教室");
        countDownLatch.countDown();
    }, String.valueOf(i)).start();
}
```

最后我们需要通过CountDownLatch的await方法来控制班长主线程的执行，这里 countDownLatch.await()可以想成是一道墙，只有当计数器的值为0的时候，墙才会消失，主线程才能继续往下执行

```java
countDownLatch.await();
System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
```

不加CountDownLatch的执行结果，我们发现main线程提前已经执行完成了

```
1	 上完自习，离开教室
0	 上完自习，离开教室
main	 班长最后关门
2	 上完自习，离开教室
...
```

引入CountDownLatch后的执行结果，我们能够控制住main方法的执行，这样能够保证前提任务的执行

```
0	 上完自习，离开教室
...
3	 上完自习，离开教室
main	 班长最后关门
```

### 完整代码

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 计数器
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 上完自习，离开教室");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
    }
}
```

### 枚举在实际中的使用案例

目的：为非阻塞线程指定对应的操作。

枚举说明：

- since jdk1.5；
- 可以当作数据库来用；
- 使用中，可以一个key对应多个value，即ONE(1, "AAA", "BBB", ...)

案例：

```java
public enum CountryEnum {
    ONE(1,"齐"),TWO(2,"楚"),THREE(3,"燕"),FOUR(4,"赵"),FIVE(5,"魏"),SIX(6,"韩");
    private Integer code;
    private String message;
    public Integer getCode() {return code;}
    public String getMessage() {return message;}
    CountryEnum(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
    public static CountryEnum forEach_CountryEnum(int index){
        CountryEnum[] values = CountryEnum.values();
        for (CountryEnum value : values) {
            if (index == value.getCode()){
                return value;
            }
        }
        return null;
    }
}
```



## CyclicBarrier

`java.util.concurrent.CyclicBarrier`

### 概念

和CountDownLatch相反，需要集齐七颗龙珠，召唤神龙。也就是做加法，开始是0，加到某个值的时候就执行

CyclicBarrier的字面意思就是可循环（cyclic）使用的屏障（Barrier）。它要求做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

线程进入屏障通过CyclicBarrier的**await**方法

### 案例

集齐7个龙珠，召唤神龙的Demo，我们需要首先创建CyclicBarrier

```java
// 定义一个循环屏障，参数1：需要累加的值，参数2 需要执行的方法
CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
	System.out.println("召唤神龙");
});
```

然后同时编写七个线程，进行龙珠收集，但一个线程收集到了的时候，我们需要让他执行await方法，等待到7个线程全部执行完毕后，我们就执行原来定义好的方法

```java
        for (int i = 0; i < 7; i++) {
            final Integer tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 收集到 第" + tempInt + "颗龙珠");

                try {
                    // 先到的被阻塞，等全部线程完成后，才能执行方法
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
```

完整代码

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {System.out.println("召唤神龙");});
        for (int i = 0; i < 7; i++) {
            final Integer tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 收集到 第" + tempInt + "颗龙珠");
                try {
                    // 先到的被阻塞，等全部线程完成后，才能执行方法
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```



## Semaphore：信号量

``

### 概念

信号量主要用于两个目的

- 一个是用于共享资源的互斥使用；
- 另一个用于并发线程数的控制；

与synchronized区别：

- synchronized是多个线程争夺一份资源；
- semaphore是多个线程争夺多份资源；
- 当semaphore初始资源数设为1时，他可以替代synchronized。

### 代码

我们模拟一个**抢车位**的场景，假设一共有6个车，3个停车位

那么我们首先需要定义信号量为3，也就是3个停车位

```java
// 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位
Semaphore semaphore = new Semaphore(3, false);
```

然后我们模拟6辆车同时并发抢占停车位，但第一个车辆抢占到停车位后，信号量需要减1

```java
// 代表一辆车，已经占用了该车位
semaphore.acquire(); // 抢占
```

同时车辆假设需要等待3秒后，释放信号量

```java
// 每个车停3秒
try {TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}
```

最后车辆离开，释放信号量

```java
// 释放停车位
semaphore.release();
```

完整代码

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        // 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位
        Semaphore semaphore = new Semaphore(3, false);
        // 模拟6部车
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    // 代表一辆车，已经占用了该车位
                    semaphore.acquire(); // 抢占
                    System.out.println(Thread.currentThread().getName() + "\t 抢到车位");
                    // 每个车停3秒
                    try {TimeUnit.SECONDS.sleep(3);} catch (InterruptedException e) {e.printStackTrace();}
                    System.out.println(Thread.currentThread().getName() + "\t 离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放停车位
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

运行结果

```
0	 抢到车位
2	 抢到车位
1	 抢到车位
2	 离开车位
1	 离开车位
3	 抢到车位
0	 离开车位
4	 抢到车位
5	 抢到车位
4	 离开车位
3	 离开车位
5	 离开车位
```

看运行结果能够发现，0 2 1 车辆首先抢占到了停车位，然后等待3秒后，离开，然后后面 3 4 5 又抢到了车位

- 实际业务场景：
- Semaphore保证同一时间下最大并发数，可用于流量控制；
- CountDownLatch可用多个线程对excel多个worksheet并发读取最后进行某些操作，如汇总所有行的总数量；
- CyclicBarrier可自动重置，实际应用暂未找到，但是可以用一个例子来比喻，它可用于重复性的多线程分工后整合的工作，如车辆的加工和生产，等所有零件各自生产完，最后才一起进行组装。

