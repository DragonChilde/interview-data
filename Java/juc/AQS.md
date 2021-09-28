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

