# AbstractQueuedSynchronizer

## AQS理论初步认识

`AbstractQueuedSynchronizer `抽象队列同步器。

一般我们说的 AQS 指的是` java.util.concurrent.locks` 包下的 `AbstractQueuedSynchronizer`，但其实还有另外三种抽象队列同步器：`AbstractOwnableSynchronizer`、`AbstractQueuedLongSynchronizer` 和 `AbstractQueuedSynchronizer`
```java
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    	...
    }
```

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/01.jpg)

AQS 是用来构建锁或者其它同步器组件的重量级基础框架及整个JUC体系的基石， 通过内置的FIFO队列来完成资源获取线程的排队工作，并通过一个int类变量（state）表示持有锁的状态

CLH：Craig、Landin and Hagersten 队列，是一个双向链表，AQS中的队列是CLH变体的虚拟双向队列FIFO

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/02.jpg)

------------------------------------------------
## AQS 是 JUC 的基石

> **和AQS有关的并发编程类**

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/03.jpg)

> **进一步理解锁和同步器的关系**

锁，面向锁的使用者。定义了程序员和锁交互的使用层API，隐藏了实现细节，你调用即可，可以理解为用户层面的 API。

同步器，面向锁的实现者。比如Java并发大神Douglee，提出统一规范并简化了锁的实现，屏蔽了同步状态管理、阻塞线程排队和通知、唤醒机制等，Java 中有那么多的锁，就能简化锁的实现啦。

------

## AQS 能干嘛

> **AQS：加锁会导致阻塞**

有阻塞就需要排队，实现排队必然需要有某种形式的队列来进行管理

抢到资源的线程直接使用办理业务，抢占不到资源的线程的必然涉及一种排队等候机制，抢占资源失败的线程继续去等待（类似办理窗口都满了，暂时没有受理窗口的顾客只能去候客区排队等候），仍然保留获取锁的可能且获取锁流程仍在继续（候客区的顾客也在等着叫号，轮到了再去受理窗口办理业务）。

既然说到了排队等候机制，那么就一定会有某种队列形成，这样的队列是什么数据结构呢？如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中，这个队列就是AQS的抽象表现。它将请求共享资源的线程封装成队列的结点（Node） ，通过CAS、自旋以及`LockSuport.park()`的方式，维护state变量的状态，使并发达到同步的效果。

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/02.jpg)

------

## AQS 初步认识

```
package java.util.concurrent.locks;
import java.util.concurrent.TimeUnit;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Date;
import sun.misc.Unsafe;

/**
 * Provides a framework for implementing blocking locks and related
 * synchronizers (semaphores, events, etc) that rely on
 * first-in-first-out (FIFO) wait queues.  This class is designed to
 * be a useful basis for most kinds of synchronizers that rely on a
 * single atomic {@code int} value to represent state. Subclasses
 * must define the protected methods that change this state, and which
 * define what that state means in terms of this object being acquired
 * or released.  Given these, the other methods in this class carry
 * out all queuing and blocking mechanics. Subclasses can maintain
 * other state fields, but only the atomically updated {@code int}
 * value manipulated using methods {@link #getState}, {@link
 * #setState} and {@link #compareAndSetState} is tracked with respect
 * to synchronization.
 /
 
 上面翻译如下:提供一个框架，用于实现依赖先进先出(FIFO)等待队列的阻塞锁和相关同步器(信号量、事件等)。这个类被设计为大多数依赖单个原子{@code int}值来表示状态的同步器的有用基础。子类必须定义受保护的方法来改变这种状态，并定义这种状态对于被获取或释放的对象意味着什么。有了这些条件，该类中的其他方法将执行所有排队和阻塞机制。子类可以维护其他状态字段，但只有原子更新的{@code int}值使用方法{@link #getState}， {@link #setState}和{@link #compareAndSetState}被跟踪与同步有关
```

有阻塞就需要排队，实现排队必然需要队列

1. `AQS`使用一个`volatile`的int类型的成员变量来表示同步状态，通过内置的 `FIFO`队列来完成资源获取的排队工作将每条要去抢占资源的线程封装成 一个`Node`节点来实现锁的分配，通过`CAS`完成对State值的修改。
2. `Node `节点是啥？答：你有见过 HashMap 的 Node 节点吗？JDK 用` static class Node<K,V> implements Map.Entry<K,V>` { 来封装我们传入的 KV 键值对。这里也是一样的道理，`JDK `使用 `Node`来封装（管理）`Thread`
3. 可以将 `Node `和 `Thread `类比于候客区的椅子和等待用餐的顾客

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/04.jpg)

------

1. `AQS `内部体系框架

   ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/05.jpg)

2. `AQS`的`int`变量

   `AQS`的同步状态`State`成员变量，类似于银行办理业务的受理窗口状态：零就是没人，自由状态可以办理；大于等于1，有人占用窗口，等着去

   ```java
   /**
    * The synchronization state.
    */
   private volatile int state;
   ```

3. `AQS`的`CLH`队列

   CLH队列（三个大牛的名字组成），为一个双向队列，类似于银行侯客区的等待顾客

   ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/06.jpg)

4. 内部类`Node`（`Node`类在`AQS`类内部）

   ```java
           /**
            * Status field, taking on only the values:
            *   SIGNAL:     The successor of this node is (or will soon be)
            *               blocked (via park), so the current node must
            *               unpark its successor when it releases or
            *               cancels. To avoid races, acquire methods must
            *               first indicate they need a signal,
            *               then retry the atomic acquire, and then,
            *               on failure, block.
            *   CANCELLED:  This node is cancelled due to timeout or interrupt.
            *               Nodes never leave this state. In particular,
            *               a thread with cancelled node never again blocks.
            *   CONDITION:  This node is currently on a condition queue.
            *               It will not be used as a sync queue node
            *               until transferred, at which time the status
            *               will be set to 0. (Use of this value here has
            *               nothing to do with the other uses of the
            *               field, but simplifies mechanics.)
            *   PROPAGATE:  A releaseShared should be propagated to other
            *               nodes. This is set (for head node only) in
            *               doReleaseShared to ensure propagation
            *               continues, even if other operations have
            *               since intervened.
            *   0:          None of the above
            *
            * The values are arranged numerically to simplify use.
            * Non-negative values mean that a node doesn't need to
            * signal. So, most code doesn't need to check for particular
            * values, just for sign.
            *
            * The field is initialized to 0 for normal sync nodes, and
            * CONDITION for condition nodes.  It is modified using CAS
            * (or when possible, unconditional volatile writes).
            */
           volatile int waitStatus;
   ```

   Node类的内部结构

   ```java
   static final class Node{
       //表示线程以共享的模式等待锁
       static final Node SHARED = new Node();
       
       //表示线程正在以独占的方式等待锁
       static final Node EXCLUSIVE = null;
       
       //线程被取消了
       static final int CANCELLED = 1;
       
       //后继线程需要唤醒
       static final int SIGNAL = -1;
       
       //等待condition唤醒
       static final int CONDITION = -2;
       
       //共享式同步状态获取将会无条件地传播下去
       static final int PROPAGATE = -3;
       
        //当前节点在队列中的状态（重点）
       //说人话：
       //等候区其它顾客(其它线程)的等待状态
       //队列中每个排队的个体就是一个Node
       //初始为0，状态上面的几种
       volatile int waitStatus;
       
       // 前置节点（重点）
       volatile Node prev;
       
       // 后继节点（重点）
       volatile Node next;
   
       // 表示处于该节点的线程
     	volatile Thread thread;
       
     	//指向下一个处于CONDITION状态的节点
    		Node nextWaiter;
     
     	//返回前驱节点，没有的话抛出npe
     	final Node predecessor() throws NullPointerException {
               Node p = prev;
               if (p == null)
                   throw new NullPointerException();
               else
                   return p;
        }
     
     	//...
   }
   ```

5. 总结

   有阻塞就需要排队，实现排队必然需要队列，通过state 变量 + CLH双端 Node 队列实现

------

> **AQS同步队列的基本结构**

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/07.jpg)

------

> **AQS底层是怎么排队的？**

通过调用 `LockSupport.pork()` 来进行排队

------

## 从 `ReentrantLock`进入 SQS

### `ReentrantLock`锁

`ReentrantLock` 类是 `Lock` 接口的实现类，基本都是通过【聚合】了一个【队列同步器】的子类完成线程访问控制的

------

> **`ReentrantLock `的原理**

`ReentrantLock` 实现了 Lock 接口，在 `ReentrantLock` 内部聚合了一个 `AbstractQueuedSynchronizer` 的实现类

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/08.jpg)

------

### 公平锁 & 非公平锁

> **通过 `ReentrantLock` 的源码来讲解公平锁和非公平锁**

在 `ReentrantLock` 内定义了静态内部类，分别为 `NoFairSync`（非公平锁）和 `FairSync`（公平锁）

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/09.jpg)

`ReentrantLock` 的构造函数：不传参数表示创建非公平锁；参数为 true 表示创建公平锁；参数为 false 表示创建非公平锁

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/10.jpg)

捞一眼 `lock()` 方法的执行流程：以 `NonfairSync` 为例

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/11.png)

在 `ReentrantLock` 中，`NoFairSync` 和 `FairSync` 中 `tryAcquire()` 方法的区别，可以明显看出公平锁与非公平锁的`lock()`方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件:
`hasQueuedPredecessors()`

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/12.jpg)

`hasQueuedPredecessors()` 方法是公平锁加锁时判断等待队列中是否存在有效节点的方法

```java
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

------

> **公平锁与非公平锁的总结**

对比公平锁和非公平锁的`tryAcqure()`方法的实现代码， 其实差别就在于非公平锁获取锁时比公平锁中少了一个判断`!hasQueuedPredecessors()`，`hasQueuedPredecessors()`中判断了是否需要排队，导致公平锁和非公平锁的差异如下:

1. 公平锁：公平锁讲究先来先到，线程在获取锁时，如果这个锁的等待队列中已经有线程在等待，那么当前线程就会进入等待队列中；
2. 非公平锁：不管是否有等待队列，如果可以获取锁，则立刻占有锁对象。也就是说队列的第一个排队线程在`unpark()`，之后还是需要竞争锁(存在线程竞争的情况下)

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/13.jpg)

而 `acquire()` 方法最终都会调用 `tryAcquire()` 方法

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

在 `NonfairSync` 和 `FairSync` 中均重写了其父类 `AbstractQueuedSynchronizer` 中的 `tryAcquire()` 方法

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/14.png)

------

### 从非公平锁的 lock() 入手

源码解读比较困难，我们这里举个栗子，假设 A、B、C 三个人都要去银行窗口办理业务，但是银行窗口只有一个个，我们使用 `lock.lock()` 模拟这种情况

```java
package com.interview.demo.aqs;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @title: AQSDemo
 * @Author Wen
 * @Date: 29/9/2021 10:13 AM
 * @Version 1.0
 */
public class AQSDemo {
    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        // 带入一个银行办理业务的案例来模拟我们的AQS如何进行线程的管理和通知唤醒机制
        // 3个线程模拟3个来银行网点，受理窗口办理业务的顾客
        // A顾客就是第一个顾客，此时受理窗口没有任何人，A可以直接去办理
        new Thread(
                () -> {
                    lock.lock();
                    try {
                        System.out.println("-----A thread come in");

                        try {
                            TimeUnit.MICROSECONDS.sleep(20);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    } finally {
                        lock.unlock();
                    }
                }, "A"
        ).start();

        // 第二个顾客，第二个线程---》由于受理业务的窗口只有一个(只能一个线程持有锁)，此时B只能等待，
        // 进入候客区
        new Thread(

                () -> {
                    lock.lock();
                    try {
                        System.out.println("-----B thread come in");
                    } finally {
                        lock.unlock();
                    }
                }, "B"
        ).start();

        // 第三个顾客，第三个线程---》由于受理业务的窗口只有一个(只能一个线程持有锁)，此时C只能等待，
        // 进入候客区
        new Thread(
                () -> {
                    lock.lock();
                    try {
                        System.out.println("-----C thread come in");
                    } finally {
                        lock.unlock();
                    }
                },"C"
        ).start();
    }
}
```

------

> **先来看看线程 A（客户 A）的执行流程**

之前已经讲到过，`new ReentrantLock()` 不传参默认是非公平锁，调用 `lock.lock()` 方法最终都会执行 `NonfairSync`重写后的 `lock()` 方法

- 第一次执行` lock()`方法

  由于第一次执行 `lock()` 方法，`state`变量的值等于 0，表示 `lock`锁没有被占用，此时执行` compareAndSetState(0, 1) `CAS 判断，可得 state == expected == 0，因此 CAS 成功，将 state 的值修改为 1

  ```java
     /**
     当前执行的是非公平锁的Lock
     **/
     static final class NonfairSync extends Sync {
          private static final long serialVersionUID = 7316153563782823691L;
  
          /**
           * Performs lock.  Try immediate barge, backing up to normal
           * acquire on failure.
           */
          final void lock() {
              if (compareAndSetState(0, 1))
                  setExclusiveOwnerThread(Thread.currentThread());
              else
                  acquire(1);
          }
      }
  ```

  再来复习下 CAS：通过 `Unsafe `提供的 `compareAndSwapXxx()`方法保证修改操作的原子性（通过 CPU 原语保证），如果变量的值等于期望值，则修改变量的值为 `update`，并返回 `true`；若不等，则返回 false。this 代表当前对象，`stateOffset `表示 `state `变量在该对象中的偏移量

  ```java
      protected final boolean compareAndSetState(int expect, int update) {
          // See below for intrinsics setup to support this
          return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
      }
  ```

  再来看看 `setExclusiveOwnerThread()` 方法做了啥：将拥有 lock 锁的线程修改为线程 A

  ```java
      protected final void setExclusiveOwnerThread(Thread thread) {
          exclusiveOwnerThread = thread;
      }
  ```

  > **再来看看线程 B（客户 B）的执行流程**

- **第二次执行 lock() 方法**

  由于第二次执行 `lock()` 方法，`state` 变量的值等于 1，表示 `lock`锁被占用，此时执行 `compareAndSetState(0, 1)` CAS 判断，可得 `state != expected`，因此 CAS 失败，进入 `acquire()` 方法

  ```java
  final void lock() {
    if (compareAndSetState(0, 1))
      setExclusiveOwnerThread(Thread.currentThread());
    else
      acquire(1);	//CAS失败,进入这里
  }
  ```

  `acquire()` 方法主要包含如下几个方法，下面我们一个一个来讲解

  ```java
      public final void acquire(int arg) {
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }
  ```

  ------

  **`tryAcquire(arg)` 方法的执行流程**

  先来看看 `tryAcquire()` 方法，诶，怎么抛了个异常？别着急，仔细一看是 `AbstractQueuedSynchronizer` 抽象队列同步器中定义的方法，既然抛出了异常，就证明父类强制要求子类去实现

  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/15.jpg)

  Ctrl + Alt + B 找到子类中的实现

  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/16.jpg)

  这里以非公平锁 `NonfairSync` 为例，在 `tryAcquire()` 方法中调用了 `nonfairTryAcquire()` 方法，注意，这里传入的参数都是1

  ```java
          protected final boolean tryAcquire(int acquires) {
              return nonfairTryAcquire(acquires);
          }
  ```

  ------

  `nonfairTryAcquire(acquires)` 正常的执行流程：

  在` nonfairTryAcquire() `方法中，大多数情况都是如下的执行流程：线程 B 执行 `int c = getState()` 时，获取到 `state` 变量的值为 1，表示 `lock `锁正在被占用；于是执行 `if (c == 0) `{ 发现条件不成立，接着执行下一个判断条件 `else if (current == getExclusiveOwnerThread())` {，current 线程为线程 B，而 `getExclusiveOwnerThread()` 方法返回正在占用 `lock `锁的线程，为线程 A，因此 `tryAcquire() `方法最后会 `return false`，表示并没有抢占到 `lock `锁

  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/17.jpg)

  **补充**：`getExclusiveOwnerThread()` 方法返回正在占用 lock 锁的线程（排他锁，`exclusive`）

  ```java
      protected final void setExclusiveOwnerThread(Thread thread) {
          exclusiveOwnerThread = thread;
      }
  ```

  ------

  `nonfairTryAcquire(acquires) `比较特殊的执行流程：

  第一种情况是，走到` int c = getState()` 语句时，此时线程 A 恰好执行完成，让出了 `lock `锁，那么 `state `变量的值为 0，当然发生这种情况的概率很小，那么线程 B 执行 CAS 操作成功后，将占用 `lock `锁的线程修改为自己，然后返回 true，表示抢占锁成功。其实这里还有一种情况，需要留到 `unlock()` 方法才能说清楚

  第二种情况为可重入锁的表现，假设 A 线程又再次抢占 lock 锁（当然示例代码里面并没有体现出来），这时 `current == getExclusiveOwnerThread()` 条件成立，将 `state `变量的值加上 `acquire`，这种情况下也应该 return true，表示线程 A 正在占用 `lock `锁。因此，state 变量的值是可以大于 1 的

  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/18.jpg)

  > 继续往下走,执行`addWaiter(Node.EXCLUSIVE)`方法

  在 `tryAcquire()` 方法返回 `false` 之后，进行 `!` 操作后为 `true`，那么会继续执行 `addWaiter()` 方法

  ```java
      public final void acquire(int arg) {
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }
  ```

  ------

  来看看 `addWaiter()` 方法做了些什么？

  之前讲过，`Node`节点用于封装用户线程，这里将当前正在执行的线程通过 `Node`封装起来（当前线程正是抢占 `lock`锁没有抢占到的线程）

  判断 `tail`尾指针是否为空，双端队列此时还没有元素呢~肯定为空呀，那么执行 `enq(node) `方法，将封装了线程 B 的 `Node `节点入队
  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/19.jpg)

  ------

  **`enq(node)` 方法：构建双端同步队列**

  也许看到这里的代码有点蒙，需要有些前置知识，在双端同步队列中，第一个节点为虚节点（也叫哨兵节点），其实并不存储任何信息，只是占位。 真正的第一个有数据的节点，是从第二个节点开始的。

  ```java
       private Node enq(final Node node) {
          for (;;) {
              Node t = tail;
              if (t == null) { // Must initialize
                  if (compareAndSetHead(new Node()))
                      tail = head;
              } else {
                  node.prev = t;
                  if (compareAndSetTail(t, node)) {
                      t.next = node;
                      return t;
                  }
              }
          }
      }
  ```

  第一次执行 `for`循环：现在解释起来就不费劲了，当线程 B 进来时，双端同步队列为空，此时肯定要先构建一个哨兵节点。此时` tail == null`，因此进入` if(t == null)` { 的分支，头指针指向哨兵节点，此时队列中只有一个节点，尾节点即是头结点，因此尾指针也指向该哨兵节点

  ![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/20.jpg)

第二次执行 `for`循环：现在该将装着线程 B 的节点放入双端同步队列中，此时 `tail`指向了哨兵节点，并不等于 `null`，因此 `if (t == null) `不成立，进入 `else `分支。以尾插法的方式，先将 `node`（装着线程 B 的节点）的 `prev `指向之前的 `tail`，再将 node 设置为尾节点（执行` compareAndSetTail(t, node)`），最后将` t.next` 指向 `node`，最后执行 `return t`结束 `for `循环

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/21.jpg)

**补充**：`compareAndSetTail(t, node)` 方法的实现

```java
    private final boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }
```

**注意**：哨兵节点和 `nodeB` 节点的 `waitStatus` 均为 0，表示在等待队列中

------

**`acquireQueued()` 方法的执行**

执行完 `addWaiter()` 方法之后，就该执行 `acquireQueued()` 方法了，这个方法有点东西，我们放到后面再去讲它

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

> **最后来看看线程 C（客户 C）的执行流程**

线程 C 和线程 B 的执行流程很类似，都是执行 `acquire()` 中的方法

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

但是在 `addWaiter()` 方法中，执行流程有些区别。此时 `tail != null`，因此在 `addWaiter()` 方法中就已经将 `nodeC` 添加至队尾了

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/22.jpg)

执行完 `addWaiter()` 方法后，就已经将 `nodeC` 挂在了双端同步队列的队尾，不需要再执行 `enq(node)` 方法

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/23.jpg)

> **补前面的坑：`acquireQueued()` 方法的执行逻辑**

先来看看 `acquireQueued()` 方法的源代码，其实这样直接看代码有点懵逼，我们接下来举例来理解。注意看：两个 `if` 判断中的代码都放在 `for( ; ; )` 中执行，这样可以实现自旋的操作

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/24.jpg)

------

**线程 B 的执行流程**

线程 B 执行` addWaiter()` 方法之后，就进入了 `acquireQueued()` 方法中，此时传入的参数为封装了线程 B 的 `nodeB`节点，`nodeB`的前驱结点为哨兵节点，因此 `final Node p = node.predecessor()` 执行完后，p 将指向哨兵节点。哨兵节点满足 `p == head`，但是线程 B 执行 `tryAcquire(arg) `方法尝试抢占 `lock`锁时还是会失败，因此会执行下面 `if`判断中的 `shouldParkAfterFailedAcquire(p, node) `方法，该方法的代码如下：

```java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

哨兵节点的 `waitStatus == 0`，因此执行 CAS 操作将哨兵节点的 `waitStatus` 改为 `Node.SIGNAL(-1)`

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/25.jpg)

注意：`compareAndSetWaitStatus(pred, ws, Node.SIGNAL) `调用` unsafe.compareAndSwapInt(node, waitStatusOffset, expect, update); `实现，虽然` compareAndSwapInt()` 方法内无自旋，但是在 `acquireQueued()` 方法中的 `for( ; ; )` 能保证此自选操作成功（另一种情况就是线程 B 抢占到 lock 锁）

```java
    private static final boolean compareAndSetWaitStatus(Node node,
                                                         int expect,
                                                         int update) {
        return unsafe.compareAndSwapInt(node, waitStatusOffset,
                                        expect, update);
    }
```

执行完上述操作，将哨兵节点的 `waitStatus` 设置为了 -1

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/26.jpg)

执行完毕将退出 `if` 判断，又会重新进入 `for( ; ; )` 循环，此时执行 `shouldParkAfterFailedAcquire(p, node)` 方法时会返回 `true`，因此此时会接着执行 `parkAndCheckInterrupt()` 方法

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/27.jpg)

线程 B 调用 `park()` 方法后被挂起，程序不会然续向下执行，程序就在这儿排队等待

```java
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

------

**线程 C 的执行流程**

前面的执行过程一样,但当`acquireQueued`这时会把B节点的`waitStatus`为改成-1

```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();	//把C节点的前一节点赋值给P
                if (p == head && tryAcquire(arg)) {	//B节点不为头节点直接跳过判断进入shouldParkAfterFailedAcquire
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

执行到 `shouldParkAfterFailedAcquire(p, node)`这时, 这时根据上面分析的过程,我们清楚现在线程C节点Node是尾节点,因此,会把C的前一个节点,就是B的节点状态0变更为-1

```java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;	//把B的节点状态赋值给ws,ws当前状态为0
        if (ws == Node.SIGNAL)	//直接跳过
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {	//直接跳过
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {	//直接进行CAS 比较赋值
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);		//pred是B线程,ws当前为0,变更为-1
        }
        return false;
    }
```

线程 C 最终也会执行到 `LockSupport.park(this);` 处，然后被挂起，进入等待区

------

总结：

如果前驱节点的 `waitstatus`是 `SIGNAL`状态（-1），即` shouldParkAfterFailedAcquire()` 方法会返回 `true`，程序会继续向下执行 `parkAndCheckInterrupt() `方法，用于将当前线程挂起

根据 `park()` 方法 API 描述，程序在下面三种情况会继续向下执行：

1. 被 `unpark`
2. 被中断`（interrupt）`
3. 其他不合逻辑的返回才会然续向下执行

因上述三种情况程序执行至此，返回当前线程的中断状态，并清空中断状态。如果程序由于被中断，该方法会返回 true

------

### 总算要 unlock() 

> **线程 A 执行 `unlock()` 方法**

A 线程终于要 `unlock()` 

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/28.jpg)

`unlock()` 方法调用了 `sync.release(1)` 方法

```java
    public void unlock() {
        sync.release(1);
    }
```

------

`release()` 方法的执行流程

其实主要就是看看 `tryRelease(arg) `方法和 `unparkSuccessor(h)` 方法的执行流程，这里先大概说以下，能有个印象：线程 A 即将让出 `lock` 锁，因此 `tryRelease()`执行后将返回 `true`，表示礼让成功，`head `指针指向哨兵节点，并且 if 条件满足，可执行 `unparkSuccessor(h)` 方法

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

------

**`tryRelease(arg)` 方法的执行逻辑**

又是 `AbstractQueuedSynchronizer` 类中定义的方法，又是抛了个异常

```java
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }
```

查看其具体实现

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/29.jpg)

线程 A 只加锁过一次，因此 `state` 的值为 1，参数 `release` 的值也为 1，因此 `c == 0`。将 `free` 设置为 `true`，表示当前 `lock` 锁已被释放，将排他锁占有的线程设置为 `null`，表示没有任何线程占用 `lock` 锁

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

------

`unparkSuccessor(h)` 方法的执行逻辑

在 `release()` 方法中获取到的头结点 h 为哨兵节点，`h.waitStatus == -1`，因此执行 CAS操作将哨兵节点的 `waitStatus` 设置为 0，并将哨兵节点的下一个节点`（s = node.next = nodeB）`获取出来，并唤醒 `nodeB` 中封装的线程`（if (s == null || s.waitStatus > 0) `不成立，只有 `if (s != null) `成立）

```java
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

------

执行完上述操作后，当前占用 `lock` 锁的线程为 `null`，哨兵节点的 `waitStatus` 设置为 0，`state` 的值为 0（表示当前没有任何线程占用 `lock` 锁）

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/30.jpg)

> **杀个回马枪：继续来看 B 线程被唤醒之后的执行逻辑**

再次回到 `lock()` 方法的执行流程中来，线程 B 被 `unpark()` 之后将不再阻塞，继续执行下面的程序，线程 B 正常被唤醒，因此 `Thread.interrupted()` 的值为 `false`，表示线程 B 未被中断

```java
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

回到上一层方法中，此时 `lock` 锁未被占用，线程 B 执行 `tryAcquire(arg)` 方法能够抢到 `lock` 锁，并且将 `state` 变量的值设置为 1，表示该 `lock` 锁已经被占用

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/31.jpg)

接着来研究下 `setHead(node)` 方法：传入的节点为 `nodeB`，头指针指向 `nodeB` 节点；将 `nodeB` 中封装的线程置为 `null`（因为已经获得锁了）；`nodeB` 不再指向其前驱节点（哨兵节点）。这一切都是为了将 `nodeB` 作为新的哨兵节点

```java
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
```

执行完 `setHead(node)` 方法的状态如下图所示

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/32.jpg)

将 `p.next` 设置为 `null`，这是原来的哨兵节点就是完全孤立的一个节点，此时 `nodeB` 作为新的哨兵节点

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/33.png)

线程 C 也是类似的执行流程

> 注意,上图有一个误区,就是节点NodeB的waitStatus,应该为-1,当把指针指到NodeB为头节点时,才会对其waitStatus改为0,下面这段代码已经说明一切,

```java
    public final boolean release(int arg) {	//释放锁
        if (tryRelease(arg)) {	//尝试释放锁,利用ReentrantLock.Sync重写此方法进行释放
            Node h = head;	//把头节点赋值给当前节点
            if (h != null && h.waitStatus != 0)	//头节点不为NULL,并且其waitStatus不为0
                unparkSuccessor(h);	//更改头节点状态,关释放当前阻塞的当前线程
            return true;
        }
        return false;
    }
```

------

## AQS 总结

**第一个考点**：我相信你应该看过源码了，那么AQS里面有个变量叫State，它的值有几种？

**答**：3个状态：没占用是0，占用了是1，大于1是可重入锁

------

**第二个考点**：如果锁正在被占用，AB两个线程进来了以后，请问这个总共有多少个Node节点？

**答**：答案是3个，分别是哨兵节点、nodeA、nodeB

------

> **AQS 源码解读案例图示**

![](http://120.77.237.175:9080/photos/eight/java/juc/aqs/34.png)
