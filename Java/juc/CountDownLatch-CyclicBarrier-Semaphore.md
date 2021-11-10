# CountDownLatch

##  概念

让一些线程阻塞直到另一些线程完成一系列操作才被唤醒

`CountDownLatch`主要有两个方法，当一个或多个线程调用`await`方法时，调用线程就会被阻塞。其它线程调用`CountDown`方法会将计数器减1（调用`CountDown`方法的线程不会被阻塞），当计数器的值变成零时，因调用`await`方法被阻塞的线程会被唤醒，继续执行

## 场景

现在有这样一个场景，假设一个自习室里有7个人，其中有一个是班长，班长的主要职责就是在其它6个同学走了后，关灯，锁教室门，然后走人，因此班长是需要最后一个走的，那么有什么方法能够控制班长这个线程是最后一个执行，而其它线程是随机执行的

## 解决方案

这个时候就用到了`CountDownLatch`，计数器了。我们一共创建6个线程，然后计数器的值也设置成6

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

最后我们需要通过`CountDownLatch`的`await`方法来控制班长主线程的执行，这里`countDownLatch.await()`可以想成是一道墙，只有当计数器的值为0的时候，墙才会消失，主线程才能继续往下执行

```java
countDownLatch.await();

System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
```

不加`CountDownLatch`的执行结果，我们发现`main`线程提前已经执行完成了

```
0	 上完自习，离开教室
1	 上完自习，离开教室
2	 上完自习，离开教室
main	 班长最后关门
3	 上完自习，离开教室
4	 上完自习，离开教室
5	 上完自习，离开教室
6	 上完自习，离开教室
```

引入`CountDownLatch`后的执行结果，我们能够控制住`main`方法的执行，这样能够保证前提任务的执行

```
0	 上完自习，离开教室
2	 上完自习，离开教室
1	 上完自习，离开教室
3	 上完自习，离开教室
4	 上完自习，离开教室
5	 上完自习，离开教室
6	 上完自习，离开教室
main	 班长最后关门
```

##  完整代码

```java
/** @title: CountDownLatchDemo @Author Wen @Date: 4/11/2021 上午10:40 @Version 1.0 */
public class CountDownLatchDemo {

  public static void main(String[] args) throws InterruptedException {

    // 计数器
    CountDownLatch countDownLatch = new CountDownLatch(6);

    for (int i = 0; i <= 6; i++) {
      new Thread(
              () -> {
                System.out.println(Thread.currentThread().getName() + "\t 上完自习，离开教室");
                countDownLatch.countDown();
              },
              String.valueOf(i))
          .start();
    }

    countDownLatch.await();

    System.out.println(Thread.currentThread().getName() + "\t 班长最后关门");
  }
}
```

**温习枚举**

枚举 + `CountDownLatch`,程序演示秦国统一六国

```java
/** @title: CountryEnum @Author Wen @Date: 4/11/2021 上午11:14 @Version 1.0 */
public enum CountryEnum {
  ONE(1, "齐"), TWO(2, "楚"), THREE(3, "燕"), FOUR(4, "赵"), FIVE(5, "魏"), SIX(6, "韩");

  public Integer returnCode;
  public String MessageCode;

  CountryEnum(Integer returnCode, String messageCode) {
    this.returnCode = returnCode;
    MessageCode = messageCode;
  }

  public Integer getReturnCode() {
    return returnCode;
  }

  public String getMessageCode() {
    return MessageCode;
  }

  public static CountryEnum getValue(int index) {
    CountryEnum[] values = CountryEnum.values();
    for (CountryEnum value : values) {
      if (index == value.returnCode) {
        return value;
      }
    }
    return null;
  }
}
```

```java
public class UnifySixCountriesDemo {

  public static void main(String[] args) {
    /** 初始化倒计时数倒为6* */
    CountDownLatch countDownLatch = new CountDownLatch(6);
    for (int i = 1; i <= 6; i++) {
      new Thread(
              () -> {
                System.out.println(Thread.currentThread().getName() + "灭");
                /** 每执行一次递减* */
                countDownLatch.countDown();
              },
              CountryEnum.getValue(i).getMessageCode())
          .start();
    }

    try {
      // 线程阻塞,当倒计时为0时被唤醒继续执行下去
      countDownLatch.await();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    System.out.println(Thread.currentThread().getName() + " 秦国统一中原。");
  }
}
```

以上代码输出如下

```
齐灭
楚灭
燕灭
赵灭
魏灭
韩灭
main 秦国统一中原。
```

------

# CyclicBarrier

## 概念

和`CountDownLatch`相反，需要集齐七颗龙珠，召唤神龙。也就是做加法，开始是0，加到某个值的时候就执行

`CyclicBarrier`的字面意思就是可循环（`cyclic`）使用的屏障（`Barrier`）。它要求做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过`CyclicBarrier`的`await`方法

## 案例

集齐7个龙珠，召唤神龙的`Demo`，我们需要首先创建`CyclicBarrier`

```java
/**
* 定义一个循环屏障，参数1：需要累加的值，参数2 需要执行的方法
*/
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
/** @title: CyclicBarrierDemo @Author Wen @Date: 4/11/2021 上午11:28 @Version 1.0 */
public class CyclicBarrierDemo {

  public static void main(String[] args) {
    /** 一般初始化使用下面方法* */
    // CyclicBarrier(int parties, Runnable barrierAction)
    CyclicBarrier cyclicBarrier =
        new CyclicBarrier(
            7,
            () -> {
              System.out.println("集齐7颗龙珠召唤神龙！");
            });

    for (int i = 1; i <= 7; i++) {
      new Thread(
              () -> {
                System.out.println(Thread.currentThread().getName() + "颗龙珠集齐！");
                try {
                  cyclicBarrier.await();
                } catch (InterruptedException e) {
                  e.printStackTrace();
                } catch (BrokenBarrierException e) {
                  e.printStackTrace();
                }
              },
              String.valueOf(i))
          .start();
    }
  }
}
```

以上代码输出如下

```
1颗龙珠集齐！
2颗龙珠集齐！
3颗龙珠集齐！
4颗龙珠集齐！
5颗龙珠集齐！
6颗龙珠集齐！
7颗龙珠集齐！
集齐7颗龙珠召唤神龙！
```

来自《Java编程思想》的例子，展现`CyclicBarrier`的可循环性：

```java
/** @title: Horse @Author Wen @Date: 4/11/2021 上午11:32 @Version 1.0 */
public class Horse implements Runnable {
  private static int counter = 0;
  private final int id = counter++;
  private int strides = 0;
  private static Random rand = new Random(47);
  private static CyclicBarrier barrier;

  public Horse(CyclicBarrier b) {
    barrier = b;
  }

  public synchronized int getStrides() {
    return strides;
  }

  public void run() {
    try {
      while (!Thread.interrupted()) { // 没有中断，就不断循环
        synchronized (this) {
          // 模拟马单位时间的移动距离
          strides += rand.nextInt(3); // Produces 0, 1 or 2
        }
        barrier.await(); // <---等待其他马到齐到循环屏障
      }
    } catch (InterruptedException e) {
      // A legitimate way to exit
    } catch (BrokenBarrierException e) {
      // This one we want to know about
      throw new RuntimeException(e);
    }
  }

  public String toString() {
    return "Horse " + id + " ";
  }

  public String tracks() {
    StringBuilder s = new StringBuilder();
    for (int i = 0; i < getStrides(); i++) s.append("*");
    s.append(id);
    return s.toString();
  }
}

class HorseRace {
  static final int FINISH_LINE = 75;
  private List<Horse> horses = new ArrayList<Horse>();
  private ExecutorService exec = Executors.newCachedThreadPool();
  private CyclicBarrier barrier;

  public HorseRace(int nHorses, final int pause) {
    // 初始化循环屏障
    barrier =
        new CyclicBarrier(
            nHorses,
            new Runnable() {
              // 循环多次执行的任务
              public void run() {

                // The fence on the racetrack
                StringBuilder s = new StringBuilder();
                for (int i = 0; i < FINISH_LINE; i++) s.append("=");
                System.out.println(s);

                // 打印马移动距离
                for (Horse horse : horses) System.out.println(horse.tracks());

                // 判断有没有马到终点了
                for (Horse horse : horses)
                  if (horse.getStrides() >= FINISH_LINE) {
                    System.out.println(horse + "won!");
                    exec.shutdownNow(); // 有只马跑赢了，所有任务都结束了
                    return;
                  }

                try {
                  TimeUnit.MILLISECONDS.sleep(pause);
                } catch (InterruptedException e) {
                  System.out.println("barrier-action sleep interrupted");
                }
              }
            });
    // 开跑！
    for (int i = 0; i < nHorses; i++) {
      Horse horse = new Horse(barrier);
      horses.add(horse);
      exec.execute(horse);
    }
  }

  public static void main(String[] args) {
    int nHorses = 7;
    int pause = 200;
    new HorseRace(nHorses, pause);
  }
}
```

------

# Semaphore：信号量

## 概念

信号量主要用于两个目的

- 一个是用于共享资源的互斥使用
- 另一个用于并发线程数的控制

## 代码

我们模拟一个抢车位的场景，假设一共有6个车，3个停车位

那么我们首先需要定义信号量为3，也就是3个停车位

```java
/**
* 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位
*/
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
try {
	TimeUnit.SECONDS.sleep(3);
} catch (InterruptedException e) {
	e.printStackTrace();
}
```

最后车辆离开，释放信号量

```java
// 释放停车位
semaphore.release();
```

完整代码

```java
/** @title: SemaphoreDemo @Author Wen @Date: 4/11/2021 下午2:58 @Version 1.0 */
public class SemaphoreDemo {

  public static void main(String[] args) {
    /** 初始化一个信号量为3，默认是false 非公平锁， 模拟3个停车位 */
    Semaphore semaphore = new Semaphore(3);

    // 模拟6部车
    for (int i = 1; i <= 6; i++) {
      new Thread(
              () -> {
                try {
                  // 代表一辆车，已经占用了该车位
                  semaphore.acquire(); // 抢占
                  System.out.println(Thread.currentThread().getName() + "号抢到了车位");
                  // 每个车停3秒
                  TimeUnit.SECONDS.sleep(3);
                  System.out.println(Thread.currentThread().getName() + "号停了3秒");

                } catch (InterruptedException e) {
                  e.printStackTrace();
                } finally {
                    // 释放停车位
                  semaphore.release();
                  System.out.println(Thread.currentThread().getName() + "号出来");
                }
              },
              String.valueOf(i))
          .start();
    }
  }
}
```

运行结果

```
2号抢到了车位
1号抢到了车位
3号抢到了车位
1号停了3秒
3号停了3秒
3号出来
4号抢到了车位
2号停了3秒
2号出来
6号抢到了车位
5号抢到了车位
1号出来
5号停了3秒
4号停了3秒
4号出来
6号停了3秒
6号出来
5号出来
```

看运行结果能够发现， 2 1 3车辆首先抢占到了停车位，然后等待3秒后，离开，然后后面 4 5 6 又抢到了车位