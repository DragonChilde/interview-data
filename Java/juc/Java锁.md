# 公平锁和非公平锁

## 概念

### 公平锁

是指多个线程按照申请锁的顺序来获取锁，类似于排队买饭，先来后到，先来先服务，就是公平的，也就是队列

### 非公平锁

是指多个线程获取锁的顺序，并不是按照申请锁的顺序，有可能申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级翻转，或者饥饿的线程（也就是某个线程一直得不到锁）

并发包中`ReentrantLock`的创建可以指定构造函数的`boolean`类型来得到公平锁或非公平锁，默认是非公平锁。

> The constructor for this class accepts an optional fairness parameter. When set true, under contention, locks favor granting access to the longest-waiting thread. Otherwise this lock does not guarantee any particular access order. Programs using fair locks accessed by many threads may display lower overall throughput (i.e., are slower; often much slower) than those using the default setting, but have smaller variances in times to obtain locks and guarantee lack of starvation.
>
> Note however, that fairness of locks does not guarantee fairness of thread scheduling. Thus, one of many threads using a fair lock may obtain it multiple times in succession while other active threads are not progressing and not currently holding the lock. Also note that the untimed tryLock() method does not honor the fairness setting. It will succeed if the lock is available even if other threads are waiting.
>
> 此类的构造函数接受可选的公平性参数。当设置为true时，在争用下，锁有利于向等待时间最长的线程授予访问权限。否则，此锁不保证任何特定的访问顺序。与使用默认设置的程序相比，使用由许多线程访问的公平锁的程序可能显示出较低的总体吞吐量（即，较慢；通常要慢得多），但是在获得锁和保证没有饥饿的时间上差异较小。
>
> 但是请注意，锁的公平性并不能保证线程调度的公平性。因此，使用公平锁的多个线程中的一个线程可以连续多次获得公平锁，而其他活动线程则没有进行并且当前没有持有该锁。还要注意，不计时的 tryLock()方法不支持公平性设置。如果锁可用，即使其他线程正在等待，它也会成功。
>
> [Link](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/ReentrantLock.html#ReentrantLock--)

> reentrant
> 英 [riːˈɛntrənt] 美 [ˌriˈɛntrənt]
> a. 可重入;可重入的;重入;可再入的;重进入

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

## 如何创建

并发包中`ReentrantLock`的创建可以指定析构函数的`boolean`类型来得到公平锁或者非公平锁，默认是非公平锁

```java
/**
* 创建一个可重入锁，true 表示公平锁，false 表示非公平锁。默认非公平锁
*/
Lock lock = new ReentrantLock(true);
```

## 两者区别

**公平锁**：就是很公平，在并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程是等待队列中的第一个，就占用锁，否者就会加入到等待队列中，以后安装FIFO的规则从队列中取到自己

**非公平锁：** 非公平锁比较粗鲁，上来就直接尝试占有锁，如果尝试失败，就再采用类似公平锁那种方式。

## 题外话

**`Java ReenttrantLock`通过构造函数指定该锁是否公平，默认是非公平锁，因为非公平锁的优点在于吞吐量比公平锁大**

**对于synchronized而言，也是一种非公平锁**

------

# 可重入锁和递归锁`ReentrantLock`

## 概念

可重入锁就是递归锁

指的是同一线程外层函数获得锁之后，内层递归函数仍然能获取到该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁

也就是说：**线程可以进入任何一个它已经拥有的锁所同步的代码块**

`ReentrantLock `/ `Synchronized `就是一个典型的可重入锁

## 代码

可重入锁就是，在一个method1方法中加入一把锁，方法2也加锁了，那么他们拥有的是同一把锁

```java
public synchronized void method1() {
	method2();
}

public synchronized void method2() {

}
```

也就是说我们只需要进入method1后，那么它也能直接进入method2方法，因为他们所拥有的锁，是同一把。

## 作用

**可重入锁的最大作用就是避免死锁**

## 可重入锁验证

### 证明`Synchronized`

```java
/**
 * 可重入锁（也叫递归锁） 指的是同一线程外层函数获得锁之后，内层递归函数仍然能获取到该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁
 *
 * <p>也就是说：`线程可以进入任何一个它已经拥有的锁所同步的代码块` @Author Wen @Date: 2/11/2021 上午11:03 @Version 1.0
 */
public class RenenterLockDemo {

  public static void main(String[] args) {
    Phone phone = new Phone();

    // 两个线程操作资源列
    new Thread(
            () -> {
              try {
                phone.sendSMS();
              } catch (Exception e) {
                e.printStackTrace();
              }
            },
            "t1")
        .start();

    new Thread(
            () -> {
              try {
                phone.sendSMS();
              } catch (Exception e) {
                e.printStackTrace();
              }
            },
            "t2")
        .start();
  }
}

class Phone {
  /**
   * 发送短信
   *
   * @throws Exception
   */
  public synchronized void sendSMS() throws Exception {

    System.out.println(Thread.currentThread().getName() + "\t sendSMS");
    sendMail();
  }

  /**
   * 发邮件
   *
   * @throws Exception
   */
  public synchronized void sendMail() throws Exception {
    System.out.println(Thread.currentThread().getName() + "\t sendMail");
  }
}
```

在这里，我们编写了一个资源类`phone`，拥有两个加了`synchronized`的同步方法，分别是`sendSMS `和 `sendEmail`，我们在`sendSMS`方法中，调用`sendEmail`。最后在主线程同时开启了两个线程进行测试，最后得到的结果为：

```
t1	 sendSMS
t1	 sendMail
t2	 sendSMS
t2	 sendMail
```

这就说明当 t1 线程进入`sendSMS`的时候，拥有了一把锁，同时`t2`线程无法进入，直到`t1`线程拿着锁，执行了`sendEmail `方法后，才释放锁，这样t2才能够进入

```
t1	 sendSMS	 t1线程在外层方法获取锁的时候
t1	 sendMail	 t1在进入内层方法会自动获取锁
t2	 sendSMS	 t2线程在外层方法获取锁的时候
t2	 sendMail	 t2在进入内层方法会自动获取锁
```

------

### 证明`ReentrantLock`

```java
/**
 * @title: ReenterLockDemo
 * @Author Wen
 * @Date: 2/11/2021 上午11:14
 * @Version 1.0
 */
public class ReenterLockDemo {
    public static void main(String[] args) {
        Phone2 phone2 = new Phone2();

        /**
         * 因为Phone2实现了Runnable接口
         */
        Thread t3 = new Thread(phone2, "t3");
        Thread t4 = new Thread(phone2, "t4");
        t3.start();
        t4.start();
    }
}

/**
 * 资源类
 */
class Phone2 implements Runnable{

    Lock lock = new ReentrantLock();

    /**
     * set进去的时候，就加锁，调用set方法的时候，能否访问另外一个加锁的set方法
     */
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
```

现在我们使用`ReentrantLock`进行验证，首先资源类实现了`Runnable`接口，重写`Run`方法，里面调用`get`方法，get`方法`在进入的时候，就加了锁

```java
    public void getLock() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t get Lock");
            setLock();
        } finally {
            lock.unlock();
        }
    }
```

然后在方法里面，又调用另外一个加了锁的`setLock`方法

```java
    public void setLock() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t set Lock");
        } finally {
            lock.unlock();
        }
    }
```

最后输出结果我们能发现，结果和加`synchronized`方法是一致的，都是在外层的方法获取锁之后，线程能够直接进入里层

```
t3	 get Lock
t3	 set Lock
t4	 get Lock
t4	 set Lock
```

**当我们在`getLock`方法加两把锁会是什么情况呢？** (阿里面试)

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

最后得到的结果也是一样的，因为里面不管有几把锁，其它他们都是同一把锁，也就是说用同一个钥匙都能够打开

**当我们在`getLock`方法加两把锁，但是只解一把锁会出现什么情况呢？**

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

得到如下结果

```
t3	 get Lock
t3	 set Lock
```

也就是说程序直接卡死，线程不能出来，也就说明我们申请几把锁，最后需要解除几把锁

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
Exception in thread "t3" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at com.example.demo.Phone2.getLock(ReenterLockDemo.java:32)
	at com.example.demo.Phone2.run(ReenterLockDemo.java:47)
	at java.lang.Thread.run(Thread.java:748)
Exception in thread "t4" java.lang.IllegalMonitorStateException
	at java.util.concurrent.locks.ReentrantLock$Sync.tryRelease(ReentrantLock.java:151)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.release(AbstractQueuedSynchronizer.java:1261)
	at java.util.concurrent.locks.ReentrantLock.unlock(ReentrantLock.java:457)
	at com.example.demo.Phone2.getLock(ReenterLockDemo.java:32)
	at com.example.demo.Phone2.run(ReenterLockDemo.java:47)
	at java.lang.Thread.run(Thread.java:748)
```

------

# 自旋锁

自旋锁：spinlock，是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU

原来提到的比较并交换，底层使用的就是自旋，自旋就是多次尝试，多次访问，不会阻塞的状态就是自旋。

![](http://120.77.237.175:9080/photos/eight/java/juc/lock/01.png)

> 提到了互斥同步对性能最大的影响阻塞的实现，挂起线程和恢复线程的操作都需要转入内核态完成，这些操作给系统的并发性能带来了很大的压力。同时，虚拟机的开发团队也注意到在许多应用上，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复线程并不值得。如果物理机器有一个以上的处理器，能让两个或以上的线程同时并行执行，我们就可以让后面请求锁的那个线程 “稍等一下”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。
>
> 《深入理解JVM.2nd》Page 398

## 优缺点

优点：循环比较获取直到成功为止，没有类似于wait的阻塞

缺点：当不断自旋的线程越来越多的时候，会因为执行while循环不断的消耗CPU资源

## 手写自旋锁

通过CAS操作完成自旋锁，A线程先进来调用myLock方法自己持有锁5秒，B随后进来发现当前有线程持有锁，不是null，所以只能通过自旋等待，直到A释放锁后B随后抢到

```java
package com.example.demo;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;

/**
 * 手写一个自旋锁 循环比较获取直到成功为止，没有类似于wait的阻塞
 * 通过CAS操作完成自旋锁，A线程先进来调用myLock方法自己持有锁5秒，B随后进来发现当前有线程持有锁，不是null，所以只能通过自旋等待，直到A释放锁后B随后抢到
 *
 * @title: SpinLockDemo @Author Wen @Date: 4/11/2021 上午8:46 @Version 1.0
 */
public class SpinLockDemo {

  public static void main(String[] args) {
    SpinLock spinLock = new SpinLock();
    // 启动t1线程，开始操作
    new Thread(
            () -> {
              // 开始占有锁
              spinLock.myLock();
              try {
                TimeUnit.SECONDS.sleep(5);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              // 开始释放锁
              spinLock.unMyLock();
            },
            "t1")
        .start();

    // 让main线程暂停1秒，使得t1线程，先执行
    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    // 1秒后，启动t2线程，开始占用这个锁
    new Thread(
            () -> {
              // 开始占有锁
              spinLock.myLock();
              try {
                TimeUnit.MILLISECONDS.sleep(1);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              // 开始释放锁
              spinLock.unMyLock();
            },
            "t2")
        .start();
  }
}

class SpinLock {
  // 现在的泛型装的是Thread，原子引用线程
  AtomicReference<Thread> atomicReference = new AtomicReference();

  public void myLock() {
    // 获取当前进来的线程
    Thread thread = Thread.currentThread();

    System.out.println(thread.getName() + "\t come in!");
    // 开始自旋，期望值是null，更新值是当前线程，如果是null，则更新为当前线程，否者自旋
    while (!atomicReference.compareAndSet(null, thread)) {}
  }

  /** 解锁 */
  public void unMyLock() {
    // 获取当前进来的线程
    Thread thread = Thread.currentThread();
    // 自己用完了后，把atomicReference变成null
    atomicReference.compareAndSet(thread, null);
    System.out.println(thread.getName() + "\t invoke unmylock");
  }
}
```

最后输出结果如下

```
t1 come in！
.....一秒后.....
t2 come in！
.....五秒后.....
t1 invoked unmylock
t2 invoked unmylock
```

首先输出的是 t1 come in

然后1秒后，t2线程启动，发现锁被t1占有，所有不断的执行 `compareAndSet`方法，来进行比较，直到t1释放锁后，也就是5秒后，t2成功获取到锁，然后释放

------

# 独占锁/共享锁/互斥锁 (读写锁)

##  概念

独占锁：指该锁一次只能被一个线程所持有。对ReentrantLock和Synchronized而言都是独占锁

共享锁：指该锁可以被多个线程锁持有

对`ReentrantReadWriteLock`其读锁是共享，其写锁是独占

写的时候只能一个人写，但是读的时候，可以多个人同时读

## 为什么会有写锁和读锁

原来我们使用`ReentrantLock`创建锁的时候，是独占锁，也就是说一次只能一个线程访问，但是有一个读写分离场景，读的时候想同时进行，因此原来独占锁的并发性就没这么好了，因为读锁并不会造成数据不一致的问题，因此可以多个人共享读

**多个线程 同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行，但是如果一个线程想去写共享资源，就不应该再有其它线程可以对该资源进行读或写**

- 读-读：能共存
- 读-写：不能共存\
- 写-写：不能共存

## 代码实现

实现一个读写缓存的操作，假设开始没有加锁的时候，会出现什么情况

```java
/** @title: ReadWriteLockDemo @Author Wen @Date: 4/11/2021 上午9:03 @Version 1.0 */
public class ReadWriteLockDemo {

  public static void main(String[] args) {
    MyCache myCache = new MyCache();
    // 线程操作资源类，5个线程写
    for (int i = 0; i < 5; i++) {
      // lambda表达式内部必须是final
      final int tempI = i;
      new Thread(
              () -> {
                myCache.put(tempI + "", tempI + "");
              },
              String.valueOf(i))
          .start();
    }

    // 线程操作资源类， 5个线程读
    for (int i = 0; i < 5; i++) {
      // lambda表达式内部必须是final
      final int tempI = i;
      new Thread(
              () -> {
                myCache.get(tempI + "");
              },
              String.valueOf(i))
          .start();
    }
  }
}

/** 资源类 */
class MyCache {
    /**
     * 缓存中的东西，必须保持可见性，因此使用volatile修饰
     */
  volatile HashMap<String, Object> map = new HashMap<>();

  /**
   * 定义写操作 满足：原子 + 独占
   *
   * @param key
   * @param value
   */
  public void put(String key, Object value) {
    System.out.println(Thread.currentThread().getName() + "\t 开始写入 " + key);
    try {
      // 模拟网络拥堵，延迟0.3秒
      TimeUnit.MILLISECONDS.sleep(300);

      map.put(key, value);
      System.out.println(Thread.currentThread().getName() + "\t 写入完毕");
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

  public void get(String key) {
    System.out.println(Thread.currentThread().getName() + "\t 开始读取 " + key);
    try {
      // 模拟网络拥堵，延迟0.3秒
      TimeUnit.MILLISECONDS.sleep(300);
      Object result = map.get(key);
      System.out.println(Thread.currentThread().getName() + "\t 读取完成 " + result);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

上面代码执行结果如下

```
0	 开始写入 0
1	 开始写入 1
2	 开始写入 2
3	 开始写入 3
4	 开始写入 4
0	 开始读取 0
1	 开始读取 1
2	 开始读取 2
3	 开始读取 3
4	 开始读取 4
0	 读取完成 null
4	 写入完毕
1	 读取完成 null
3	 写入完毕
0	 写入完毕
2	 写入完毕
1	 写入完毕
4	 读取完成 4
2	 读取完成 2
3	 读取完成 3
```

我们可以看到，在写入的时候，写操作都没其它线程打断了，这就造成了，还没写完，其它线程又开始写，这样就造成数据不一致

##  解决方法

上面的代码是没有加锁的，这样就会造成线程在进行写入操作的时候，被其它线程频繁打断，从而不具备原子性，这个时候，我们就需要用到读写锁来解决了

```java
/**
* 创建一个读写锁
* 它是一个读写融为一体的锁，在使用的时候，需要转换
*/
private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
```

当我们在进行写操作的时候，就需要转换成写锁

```java
// 创建一个写锁
rwLock.writeLock().lock();

// 写锁 释放
rwLock.writeLock().unlock();
```

当们在进行读操作的时候，在转换成读锁

```java
// 创建一个读锁
rwLock.readLock().lock();

// 读锁 释放
rwLock.readLock().unlock();
```

这里的读锁和写锁的区别在于，写锁一次只能一个线程进入，执行写操作，而读锁是多个线程能够同时进入，进行读取的操作

完整代码：

```java
/** @title: ReadWriteLockDemo @Author Wen @Date: 4/11/2021 上午9:03 @Version 1.0 */
public class ReadWriteLockDemo {

  public static void main(String[] args) {
    MyCache myCache = new MyCache();
    // 线程操作资源类，5个线程写
    for (int i = 0; i < 5; i++) {
      // lambda表达式内部必须是final
      final int tempI = i;
      new Thread(
              () -> {
                myCache.put(tempI + "", tempI + "");
              },
              String.valueOf(i))
          .start();
    }

    // 线程操作资源类， 5个线程读
    for (int i = 0; i < 5; i++) {
      // lambda表达式内部必须是final
      final int tempI = i;
      new Thread(
              () -> {
                myCache.get(tempI + "");
              },
              String.valueOf(i))
          .start();
    }
  }
}

/** 资源类 */
class MyCache {
  volatile HashMap<String, Object> map = new HashMap<>();
  /**
   * ReentrantLock和Synchronized因为锁太重，不利于并发量大的（杀鸡何需牛刀）会把所有都线程操作都独占，假
   * 如在缓存里是可以保证读-写，写-写分开，但读-读是不能共享的（保证一致性，但并发性下降了）
   */
  /** 创建一个读写锁 它是一个读写融为一体的锁，在使用的时候，需要转换 */
  ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

  /**
   * 定义写操作 满足：原子 + 独占
   *
   * @param key
   * @param value
   */
  public void put(String key, Object value) {

    // 创建一个写锁
    rwLock.writeLock().lock();
    try {

      System.out.println(Thread.currentThread().getName() + "\t 开始写入 " + key);
      // 模拟网络拥堵，延迟0.3秒
      TimeUnit.MILLISECONDS.sleep(300);

      map.put(key, value);
      System.out.println(Thread.currentThread().getName() + "\t 写入完毕");
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      // 写锁 释放
      rwLock.writeLock().unlock();
    }
  }

  public void get(String key) {

    // 读锁
    rwLock.readLock().lock();
    try {
      System.out.println(Thread.currentThread().getName() + "\t 开始读取 " + key);
      // 模拟网络拥堵，延迟0.3秒
      TimeUnit.MILLISECONDS.sleep(300);

      Object result = map.get(key);
      System.out.println(Thread.currentThread().getName() + "\t 读取完成 " + result);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      // 读锁释放
      rwLock.readLock().unlock();
    }
  }
}
```

运行结果如下

```
0	 开始写入 0
0	 写入完毕
1	 开始写入 1
1	 写入完毕
2	 开始写入 2
2	 写入完毕
3	 开始写入 3
3	 写入完毕
4	 开始写入 4
4	 写入完毕
0	 开始读取 0
4	 开始读取 4
3	 开始读取 3
2	 开始读取 2
1	 开始读取 1
4	 读取完成 4
3	 读取完成 3
2	 读取完成 2
1	 读取完成 1
0	 读取完成 0
```

从运行结果我们可以看出，写入操作是一个一个线程进行执行的，并且中间不会被打断，而读操作的时候，是同时5个线程进入，然后并发读取操作

------

# 为什么Synchronized无法禁止指令重排，却能保证有序性

##  前言

首先我们要分析下这道题，这简单的一个问题，其实里面还是包含了很多信息的，要想回答好这个问题，面试者至少要知道一下概念：

- Java内存模型
- 并发编程有序性问题
- 指令重排
- synchronized锁
- 可重入锁
- 排它锁
- as-if-serial语义
- 单线程&多线程

## 标准解答

为了进一步提升计算机各方面能力，在硬件层面做了很多优化，如处理器优化和指令重排等，但是这些技术的引入就会导致有序性问题。

> 先解释什么是有序性问题，也知道是什么原因导致的有序性问题

我们也知道，最好的解决有序性问题的办法，就是禁止处理器优化和指令重排，就像volatile中使用内存屏障一样。

> 表明你知道啥是指令重排，也知道他的实现原理

但是，虽然很多硬件都会为了优化做一些重排，但是在Java中，不管怎么排序，都不能影响单线程程序的执行结果。这就是as-if-serial语义，所有硬件优化的前提都是必须遵守as-if-serial语义。

as-if-serial语义把**单线程**程序保护了起来，遵守as-if-serial语义的编译器，runtime 和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的。as-if-serial语义使单线程程序员无需担心重排序会 干扰他们，也无需担心内存可见性问题。

> 重点！解释下什么是as-if-serial语义，因为这是这道题的第一个关键词，答上来就对了一半了

再说下synchronized，他是Java提供的锁，可以通过他对Java中的对象加锁，并且他是一种排他的、可重入的锁。

所以，当某个线程执行到一段被synchronized修饰的代码之前，会先进行加锁，执行完之后再进行解锁。在加锁之后，解锁之前，其他线程是无法再次获得锁的，只有这条加锁线程可以重复获得该锁。

> 介绍synchronized的原理，这是本题的第二个关键点，到这里基本就可以拿满分了。

synchronized通过排他锁的方式就保证了同一时间内，被synchronized修饰的代码是单线程执行的。所以呢，这就满足了as-if-serial语义的一个关键前提，那就是**单线程**，因为有as-if-serial语义保证，单线程的有序性就天然存在了。

## 来源

https://mp.weixin.qq.com/s/Pd6dOXaMQFUHfAUnOhnwtw
