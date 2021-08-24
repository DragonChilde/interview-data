# **讲讲java jmm volatile的实现原理**

volatile关键字是Java虚拟机提供的的**最轻量级的同步机制**，它作为一个修饰符，用来修饰变量。它保证变量对所有线程可见性，禁止指令重排，但是不保证原子性。

**volatile是如何保证可见性的呢？我们先来看下java内存模型（jmm）**

- Java虚拟机规范试图定义一种Java内存模型,来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台上都能达到一致的内存访问效果。

- 为了更好的执行性能，java内存模型并没有限制执行引擎使用处理器的特定寄存器或缓存来和主内存打交道，也没有限制编译器进行调整代码顺序优化。所以Java内存模型会存在缓存一致性问题和指令重排序问题的。

- Java内存模型规定所有的变量都是存在主内存当中，每个线程都有自己的工作内存。这里的变量包括实例变量和静态变量，但是不包括局部变量，因为局部变量是线程私有的。

- 线程的工作内存保存了被该线程使用的变量的主内存副本，线程对变量的所有操作都必须在工作内存中进行，而不能直接操作操作主内存。并且每个线程不能访问其他线程的工作内存。

  ![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/01.jpg)

volatile变量，保证新值能立即同步回主内存，以及每次使用前立即从主内存刷新，所以我们说**volatile保证了多线程操作变量的可见性**。

指令重排是指在程序执行过程中,为了提高性能, 编译器和CPU可能会对指令进行重新排序。volatile是如何禁止指令重排的？在Java语言中，有一个先行发生原则（happens-before）

- **程序次序规则**：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。
- **管程锁定规则**：一个unLock操作先行发生于后面对同一个锁额lock操作
- **volatile变量规则**：对一个变量的写操作先行发生于后面对这个变量的读操作
- **线程启动规则**：Thread对象的start()方法先行发生于此线程的每个一个动作
- **线程终止规则**：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- **线程中断规则**：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- **对象终结规则**：一个对象的初始化完成先行发生于他的finalize()方法的开始
- **传递性**：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C

实际上volatile保证可见性和禁止指令重排都跟**内存屏障**有关。我们来看一段volatile使用的demo代码

```java
public class Singleton {  
    private volatile static Singleton instance;  
    private Singleton (){}  
    public static Singleton getInstance() {  
    if (instance == null) {  
        synchronized (Singleton.class) {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        }  
    }  
    return instance;  
    }  
}  
```

编译后，对比有volatile关键字和没有volatile关键字时所生成的汇编代码，发现有volatile关键字修饰时，会多出一个**lock addl $0x0,(%esp)**，即多出一个lock前缀指令，lock指令相当于一个「内存屏障」

lock指令相当于一个**内存屏障**，它保证以下这几点：

- 1.重排序时不能把后面的指令重排序到内存屏障之前的位置
- 2.将本处理器的缓存写入内存
- 3.如果是写入动作，会导致其他处理器中对应的缓存无效。

第2点和第3点就是保证volatile保证可见性的体现嘛，**第1点就是禁止指令重排列的体现**。内存屏障又是什么呢？

内存屏障四大分类：（Load 代表读取指令，Store代表写入指令）

| 内存屏障类型   | 抽象场景                   | 描述                                                         |
| :------------- | :------------------------- | :----------------------------------------------------------- |
| LoadLoad屏障   | Load1; LoadLoad; Load2     | 在Load2要读取的数据被访问前，保证Load1要读取的数据被读取完毕。 |
| StoreStore屏障 | Store1; StoreStore; Store2 | 在Store2写入执行前，保证Store1的写入操作对其它处理器可见     |
| LoadStore屏障  | Load1; LoadStore; Store2   | 在Store2被写入前，保证Load1要读取的数据被读取完毕。          |
| StoreLoad屏障  | Store1; StoreLoad; Load2   | 在Load2读取操作执行前，保证Store1的写入对所有处理器可见。    |

为了实现volatile的内存语义，Java内存模型采取以下的保守策略

- 在每个volatile写操作的前面插入一个StoreStore屏障。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
- 在每个volatile读操作的后面插入一个LoadStore屏障。

有些小伙伴，可能对这个还是有点疑惑，内存屏障这玩意太抽象了。我们照着代码看下吧：

![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/02.jpg)