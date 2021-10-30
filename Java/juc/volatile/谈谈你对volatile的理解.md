# 谈谈你对volatile的理解

Volatile在日常的单线程环境是应用不到的

- Volatile是Java虚拟机提供的轻量级的同步机制（三大特性）
  - 保证可见性
  - 不保证原子性
  - 禁止指令重排

------

# JMM是什么

JMM是Java内存模型，也就是Java Memory Model，简称JMM，本身是一种抽象的概念，实际上并不存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式

JMM关于同步的规定：

- 线程解锁前，必须把共享变量的值刷新回主内存
- 线程加锁前，必须读取主内存的最新值，到自己的工作内存
- 加锁和解锁是同一把锁

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，`但线程对变量的操作（读取赋值等）必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存`，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的变量副本拷贝，因此不同的线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成，其简要访问过程：

![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/03.png)

数据传输速率：硬盘 < 内存 < < cache < CPU

上面提到了两个概念：主内存 和 工作内存

- 主内存：就是计算机的内存，也就是经常提到的8G内存，16G内存

- 工作内存：但我们实例化 new student，那么 age = 25 也是存储在主内存中

  当同时有三个线程同时访问 student中的age变量时，那么每个线程都会拷贝一份，到各自的工作内存，从而实现了变量的拷贝

  ![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/04.png)

> 即：JMM内存模型的可见性，指的是当主内存区域中的值被某个线程写入更改后，其它线程会马上知晓更改后的值，并重新得到更改后的值。

------

# 缓存一致性

为什么这里主线程中某个值被更改后，其它线程能马上知晓呢？其实这里是用到了总线嗅探技术

在说嗅探技术之前，首先谈谈缓存一致性的问题，就是当多个处理器运算任务都涉及到同一块主内存区域的时候，将可能导致各自的缓存数据不一。

为了解决缓存一致性的问题，需要各个处理器访问缓存时都遵循一些协议，在读写时要根据协议进行操作，这类协议主要有MSI、MESI等等。

## MESI

当CPU写数据时，如果发现操作的变量是共享变量，即在其它CPU中也存在该变量的副本，会发出信号通知其它CPU将该内存变量的缓存行设置为无效，因此当其它CPU读取这个变量的时，发现自己缓存该变量的缓存行是无效的，那么它就会从内存中重新读取。

## 总线嗅探

那么是如何发现数据是否失效呢？

这里是用到了总线嗅探技术，就是每个处理器通过嗅探在总线上传播的数据来检查自己缓存值是否过期了，当处理器发现自己的缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置为无效状态，当处理器对这个数据进行修改操作的时候，会重新从内存中把数据读取到处理器缓存中。

## 总线风暴

总线嗅探技术有哪些缺点？

由于Volatile的MESI缓存一致性协议，需要不断的从主内存嗅探和CAS循环，无效的交互会导致总线带宽达到峰值。因此不要大量使用volatile关键字，至于什么时候使用volatile、什么时候用锁以及Syschonized都是需要根据实际场景的。

------

------

# JMM的特性

JMM的三大特性，volatile只保证了两个，即可见性和有序性，不满足原子性

- 可见性
- 原子性
- 有序性

# 可见性代码验证

但我们对于成员变量没有添加任何修饰时，是无法感知其它线程修改后的值

```java
package com.example.demo;

import java.util.concurrent.TimeUnit;

/** @title: VolatileDemo @Author Wen @Date: 29/10/2021 下午8:51 @Version 1.0 */
public class VolatileDemo {
  public static void main(String[] args) {
      
    Memory m = new Memory();
      
    new Thread(
            () -> {
              System.out.println(Thread.currentThread().getName() + " start!");
              try {
                // 线程睡眠3秒，假设在进行运算
                System.out.println(Thread.currentThread().getName() + " sleep 3 sec");
                TimeUnit.SECONDS.sleep(3);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }
              // 修改number的值
              m.setNumber();
              // 输出修改后的值
              System.out.println(
                  Thread.currentThread().getName() + " set memory number is " + m.number);
            },
            "AAA")
        .start();

    System.out.println("main thread start!");
      
    while (m.number == 0) {
      // main线程就一直在这里等待循环，直到number的值不等于零
    }
    // 按道理这个值是不可能打印出来的，因为主线程运行的时候，number的值为0，所以一直在循环
    // 如果能输出这句话，说明AAA线程在睡眠3秒后，更新的number的值，重新写入到主内存，并被main线程感知到了
    System.out.println(Thread.currentThread().getName() + " " + m.number);
  }
}


/**
 * 假设是主物理内存
 */
class Memory {
    /**
     * 1 验证volatile的可见性 1.1 加入int number=0，number变量之前根本没有添加volatile关键字修饰,没有可见性 1.2 添加了volatile，可以解决可见性问题
     */
   int number = 0;

  public void setNumber() {
    this.number = 30;
  }
}

```

上面结果为

```
main thread start!
AAA start!
AAA sleep 3 sec
AAA set memory number is 30
```

最后线程没有停止，并行没有输出 主线程获取到的值 ，说明没有用`volatile`修饰的变量，是没有可见性

当我们修改`Memory`类中的成员变量时，并且添加`volatile`关键字修饰

```java
/**
 * 假设是主物理内存
 */
class Memory {
    /**
     * volatile 修饰的关键字，是为了增加 主线程和线程之间的可见性，只要有一个线程修改了内存中的值，其它线程也能马上感知
     */ 
  volatile int number = 0;

  public void setNumber() {
    this.number = 30;
  }
}
```

最后输出的结果为：

```
main thread start!
AAA start!
AAA sleep 3 sec
AAA set memory number is 30
main	 mission is over
```

主线程也执行完毕了，说明`volatile`修饰的变量，是具备`JVM`轻量级同步机制的，能够感知其它线程的修改后的值。

------

# Volatile不保证原子性

各个线程对主内存中共享变量的操作都是各个线程各自拷贝到自己的工作内存进行操作后在写回到主内存中的。

这就可能存在一个线程AAA修改了共享变量X的值，但是还未写入主内存时，另外一个线程BBB又对主内存中同一共享变量X进行操作，但此时A线程工作内存中共享变量X对线程B来说是不可见，这种工作内存与主内存同步延迟现象就造成了可见性问题。

##  原子性

不可分割，完整性，也就是说某个线程正在做某个具体业务时，中间不可以被加塞或者被分割，需要具体完成，要么同时成功，要么同时失败。

数据库也经常提到事务具备原子性

## 代码测试

为了测试`volatile`是否保证原子性，我们创建了20个线程，然后每个线程分别循环1000次，来调用`number++`的方法

```java
package com.example.demo;

/** @title: VolatileAtomicDemo @Author Wen @Date: 29/10/2021 下午9:16 @Version 1.0 */
public class VolatileAtomicDemo {

  /**
   * 2 验证volatile不保证原子性
   *
   * <p>2.1 原子性是不可分割，完整性，也即某个线程正在做某个具体业务时，中间不可以被加塞或者分割。 需要整体完成，要么同时成功，要么同时失败。
   *
   * <p>2.2 volatile不可以保证原子性演示
   *
   * <p>3 如何保证原子性一致 1加sync 2使用我们的JUC下AtomicInteger
   */
  public static void main(String[] args) {
      
    Memory memory = new Memory();
      
    // 创建10个线程，线程里面进行1000次循环
    for (int i = 1; i <= 20; i++) {
      new Thread(
              () -> {
                for (int j = 1; j <= 1000; j++) {
                   memory.addNumberPlus();
                }
              },
              String.valueOf(i))
          .start();
    }

    // 需要等待上面20个线程都计算完成后，在用main线程取得最终的结果值
    // 这里判断线程数是否大于2，为什么是2？因为默认是有两个线程的，一个main线程，一个gc线程
    while (Thread.activeCount() > 2) {
      // yield表示不执行
      Thread.yield();
    }
      
    // 查看最终的值
    // 假设volatile保证原子性，那么输出的值应该为：  20 * 1000 = 20000
    System.out.println(
        Thread.currentThread().getName() + "\t finally number value: " + memory.number);
  }
}

/** 假设是主物理内存 */
class Memory {
  /** volatile 修饰的关键字，是为了增加 主线程和线程之间的可见性，只要有一个线程修改了内存中的值，其它线程也能马上感知 */
  volatile int number = 0;

  public void setNumber() {
    this.number = 30;
  }

    // 当加了volatile关键字后，验证原子性不一致的情况
  public void addNumberPlus() {
    this.number++;
  }

}

```

最终结果我们会发现，`number`输出的值并没有20000，而且是每次运行的结果都不一致的，这说明了`volatile`修饰的变量不保证原子性

多次运行后，结果各不相同

```
main	 finally number value: 19842
```

```
main	 finally number value: 19806
```

```
main	 finally number value: 19966
```

```
main	 finally number value: 20000
```

------

------

## 为什么出现数值丢失

![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/05.png)

各自线程在写入主内存的时候，出现了数据的丢失，而引起的数值缺失的问题

下面我们将一个简单的`number++`操作，转换为字节码文件一探究竟

```java
public class T1 {
    volatile int n = 0;
    public void add() {
        n++;
    }
}
```

转换后的字节码文件

```java
public class com.example.demo.T1 {
  volatile int n;

  public com.example.demo.T1();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_0
       6: putfield      #2                  // Field n:I
       9: return

  public void add();
    Code:
       0: aload_0
       1: dup
       2: getfield      #2                  // Field n:I
       5: iconst_1
       6: iadd
       7: putfield      #2                  // Field n:I
      10: return
}
```

下面我们就针对 `add() `这个方法的字节码文件进行分析

我们能够发现` n++`这条命令，被拆分成了3个指令

- 执行`getfield` 从主内存拿到原始n
- 执行`iadd` 进行加1操作
- 执行`putfileld` 把累加后的值写回主内存

假设我们没有加 `synchronized`那么第一步就可能存在着，三个线程同时通过`getfield`命令，拿到主存中的 n值，然后三个线程，各自在自己的工作内存中进行加1操作，但他们并发进行 `iadd` 命令的时候，因为只能一个进行写，所以其它操作会被挂起，假设1线程，先进行了写操作，在写完后，`volatile`的可见性，应该需要告诉其它两个线程，主内存的值已经被修改了，但是因为太快了，其它两个线程，陆续执行 `iadd`命令，进行写入操作，这就造成了其他线程没有接受到主内存n的改变，从而覆盖了原来的值，出现写丢失，这样也就让最终的结果少于20000

------

##  如何解决

因此这也说明，在多线程环境下 `number ++` 在多线程环境下是非线程安全的，解决的方法有哪些呢？

- 在方法上加入 `synchronized`

  ```java
   public synchronized void addPlusPlus() {
          number ++;
  }
  ```

  运行结果：

  ```
  main	 finally number value: 20000
  ```

我们能够发现引入`synchronized`关键字后，保证了该方法每次只能够一个线程进行访问和操作，最终输出的结果也就为20000

------

##  其它解决方法

上面的方法引入`synchronized`，虽然能够保证原子性，但是为了解决`number++`，而引入重量级的同步机制，有种 杀鸡焉用牛刀

除了引用`synchronized`关键字外，还可以使用`JUC`下面的原子包装类，即刚刚的`int`类型的`number`，可以使用`AtomicInteger`来代替

```java
package com.example.demo;

import java.util.concurrent.atomic.AtomicInteger;

/** @title: VolatileAtomicDemo @Author Wen @Date: 29/10/2021 下午9:16 @Version 1.0 */
public class VolatileAtomicDemo {

  /**
   * 2 验证volatile不保证原子性
   *
   * <p>2.1 原子性是不可分割，完整性，也即某个线程正在做某个具体业务时，中间不可以被加塞或者分割。 需要整体完成，要么同时成功，要么同时失败。
   *
   * <p>2.2 volatile不可以保证原子性演示
   *
   * <p>3 如何保证原子性一致 1加sync 2使用我们的JUC下AtomicInteger
   */
  public static void main(String[] args) {
    Memory memory = new Memory();
    // 创建10个线程，线程里面进行1000次循环
    for (int i = 1; i <= 20; i++) {
      new Thread(
              () -> {
                for (int j = 1; j <= 1000; j++) {
                  memory.addNumberPlus();
                  memory.addAtomicInteger();
                }
              },
              String.valueOf(i))
          .start();
    }

    // 需要等待上面20个线程都计算完成后，在用main线程取得最终的结果值
    // 这里判断线程数是否大于2，为什么是2？因为默认是有两个线程的，一个main线程，一个gc线程
    while (Thread.activeCount() > 2) {
      // yield表示不执行
      Thread.yield();
    }

    // 查看最终的值
    // 假设volatile保证原子性，那么输出的值应该为：  20 * 1000 = 20000
    System.out.println(Thread.currentThread().getName() + " final get number is " + memory.number);
    System.out.println(
        Thread.currentThread().getName() + "\t finally atomicNumber value: " + memory.atomicInteger);
  }
}

/** 假设是主物理内存 */
class Memory {
  /** volatile 修饰的关键字，是为了增加 主线程和线程之间的可见性，只要有一个线程修改了内存中的值，其它线程也能马上感知 */
  volatile int number = 0;

  public void setNumber() {
    this.number = 30;
  }

  /**
     * 注意，此时number 前面是加了volatile修饰
     */
  public synchronized void addNumberPlus() {
    this.number++;
  }

  // 使用java.util.concurrent.atomic.Atomic***保证原子性一致
    /**
     *  创建一个原子Integer包装类，默认为0
      */
  AtomicInteger atomicInteger = new AtomicInteger();

  public void addAtomicInteger() {
      // 相当于 atomicInter ++
    atomicInteger.getAndIncrement();
  }
}
```

下面的结果，一个是引入`synchronized`，一个是使用了原子包装类`AtomicInteger`

```
main final get number is 20000
main	 finally atomicNumber value: 20000
```

[字节码指令表](https://segmentfault.com/a/1190000008722128)

------

#  Volatile禁止指令重排

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令重排，一般分为以下三种：

```
源代码 -> 编译器优化的重排 -> 指令并行的重排 -> 内存系统的重排 -> 最终执行指令
```

单线程环境里面确保最终执行结果和代码顺序的结果一致

处理器在进行重排序时，必须要考虑指令之间的数据依赖性

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测。

##  指令重排 - example 1

```java
public void mySort{
	int x = 11;	//语句1
    int y = 12;	//语句2
    × = × + 5;	//语句3
    y = x * x;	//语句4
}
```

按照正常单线程环境，执行顺序是 1 2 3 4

但是在多线程环境下，可能出现以下的顺序：

- 2 1 3 4
- 1 3 2 4

上述的过程就可以当做是指令的重排，即内部执行顺序，和我们的代码顺序不一样

> 问题：请问语句4可以重排后变成第一个条吗？答：不能。

但是指令重排也是有限制的，即不会出现下面的顺序

- 4 3 2 1

因为处理器在进行重排时候，必须考虑到指令之间的数据依赖性

因为步骤 4：需要依赖于 y的申明，以及x的申明，故因为存在数据依赖，无法首先执行

###  例子

int a,b,x,y = 0

| 线程1        | 线程2  |
| ------------ | ------ |
| x = a;       | y = b; |
| b = 1;       | a = 2; |
| x = 0; y = 0 |        |

因为上面的代码，不存在数据的依赖性，因此编译器可能对数据进行重排

| 线程1        | 线程2  |
| ------------ | ------ |
| b = 1;       | a = 2; |
| x = a;       | y = b; |
| x = 2; y = 1 |        |

这样造成的结果，和最开始的就不一致了，这就是导致重排后，结果和最开始的不一样，因此为了防止这种结果出现，`volatile`就规定禁止指令重排，为了保证数据的一致性

## 指令重排 - example 2

比如下面这段代码

```java
public class ReSortSeqDemo{
	int a = 0;
	boolean flag = false;
    
	public void method01(){
		a = 1;//语句1
		flag = true;//语句2
	}
    
    public void method02(){
        if(flag){
            a = a + 5; //语句3
        }
        System.out.println("retValue: " + a);//可能是6或1或5或0
    }
    
}
```

我们按照正常的顺序，分别调用`method01() `和 `method02() `那么，最终输出就是` a = 6`

但是如果在多线程环境下，因为方法1 和 方法2，他们之间不能存在数据依赖的问题，因此原先的顺序可能是

```java
a = 1;
flag = true;

a = a + 5;
System.out.println("reValue:" + a);
```

但是在经过编译器，指令，或者内存的重排后，可能会出现这样的情况

```java
flag = true;

a = a + 5;
System.out.println("reValue:" + a);

a = 1;
```

也就是先执行 `flag = true`后，另外一个线程马上调用方法2，满足 `flag`的判断，最终让a + 5，结果为5，这样同样出现了数据不一致的问题

为什么会出现这个结果：多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测。

这样就需要通过`volatile`来修饰，来保证线程安全性

##  Volatile针对指令重排做了啥

`Volatile`实现禁止指令重排优化，从而避免了多线程环境下程序出现乱序执行的现象

首先了解一个概念，内存屏障（Memory Barrier）又称内存栅栏，是一个CPU指令，它的作用有两个：

- 保证特定操作的顺序
- 保证某些变量的内存可见性（利用该特性实现`volatile`的内存可见性）

由于编译器和处理器都能执行指令重排的优化，如果在指令间插入一条Memory Barrier则会告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重排序，也就是说 通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。 内存屏障另外一个作用是刷新出各种CPU的缓存数，因此任何CPU上的线程都能读取到这些数据的最新版本。

![](http://120.77.237.175:9080/photos/eight/java/juc/volatile/06.png)

也就是过在`Volatile`的写 和 读的时候，加入屏障，防止出现指令重排的

## 线程安全获得保证

工作内存与主内存同步延迟现象导致的可见性问题

- 可通过`synchronized`或`volatile`关键字解决，他们都可以使一个线程修改后的变量立即对其它线程可见

对于指令重排导致的可见性问题和有序性问题

- 可以使用`volatile`关键字解决，因为`volatile`关键字的另一个作用就是禁止重排序优化

------

# Volatile的应用

##  单线程下的单例模式代码

```java
public class SingletonDemo {

    private static SingletonDemo instance = null;

    private SingletonDemo () {
        System.out.println(Thread.currentThread().getName() + "\t 我是构造方法SingletonDemo");
    }

    public static SingletonDemo getInstance() {
        if(instance == null) {
            instance = new SingletonDemo();
        }
        return instance;
    }

    public static void main(String[] args) {
        // 这里的 == 是比较内存地址
        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    }
}
```

最后输出的结果

```
main	 我是构造方法SingletonDemo
true
true
true
true
```

但是在多线程的环境下，我们的单例模式是否还是同一个对象了

```javascript
public class SingletonDemo {

    private static SingletonDemo instance = null;

    private SingletonDemo () {
        System.out.println(Thread.currentThread().getName() + "\t 我是构造方法SingletonDemo");
    }

    public static SingletonDemo getInstance() {
        if(instance == null) {
            instance = new SingletonDemo();
        }
        return instance;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                SingletonDemo.getInstance();
            }, String.valueOf(i)).start();
        }
    }
}
```

从下面的结果我们可以看出，我们通过`SingletonDemo.getInstance() `获取到的对象，并不是同一个，而是被下面几个线程都进行了创建，那么在多线程环境下，单例模式如何保证呢？

```
0	 我是构造方法SingletonDemo
2	 我是构造方法SingletonDemo
1	 我是构造方法SingletonDemo
```

###  解决方法1

引入`synchronized`关键字

```java
    public synchronized static SingletonDemo getInstance() {
        if(instance == null) {
            instance = new SingletonDemo();
        }
        return instance;
    }
```

输出结果

```
0	 我是构造方法SingletonDemo
```

我们能够发现，通过引入`Synchronized`关键字，能够解决高并发环境下的单例模式问题

但是`synchronized`属于重量级的同步机制，它只允许一个线程同时访问获取实例的方法，但是为了保证数据一致性，而减低了并发性，因此采用的比较少

###  解决方法2

通过引入`DCL Double Check Lock `双端检锁机制

就是在进来和出去的时候，进行检测

```java
    public static SingletonDemo getInstance() {
        if(instance == null) {
            // 同步代码段的时候，进行检测
            synchronized (SingletonDemo.class) {
                if(instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
```

最后输出的结果为：

```
0	 我是构造方法SingletonDemo
```

从输出结果来看，确实能够保证单例模式的正确性，但是上面的方法还是存在问题的

DCL（双端检锁）机制不一定是线程安全的，原因是有指令重排的存在，加入`volatile`可以禁止指令重排

原因是在某一个线程执行到第一次检测的时候，读取到 `instance `不为`null`，`instance`的引用对象可能没有完成实例化。因为 `instance = new SingletonDemo()；`可以分为以下三步进行完成：

- `memory = allocate(); `// 1、分配对象内存空间
- `instance(memory); `// 2、初始化对象
- `instance = memory;` // 3、设置`instance`指向刚刚分配的内存地址，此时`instance != null`

但是我们通过上面的三个步骤，能够发现，步骤2 和 步骤3之间不存在 数据依赖关系，而且无论重排前 还是重排后，程序的执行结果在单线程中并没有改变，因此这种重排优化是允许的。

- `memory = allocate();` // 1、分配对象内存空间
- `instance = memory;` // 3、设置`instance`指向刚刚分配的内存地址，此时`instance != null`，但是对象还没有初始化完成
- `instance(memory);` // 2、初始化对象

这样就会造成什么问题呢？

也就是当我们执行到重排后的步骤2，试图获取`instance`的时候，会得到`null`，因为对象的初始化还没有完成，而是在重排后的步骤3才完成，因此执行单例模式的代码时候，就会重新在创建一个`instance`实例

指令重排只会保证串行语义的执行一致性（单线程），但并不会关系多线程间的语义一致性

所以当一条线程访问`instance`不为`null`时，由于`instance`实例未必已初始化完成，这就造成了线程安全的问题

所以需要引入`volatile`，来保证出现指令重排的问题，从而保证单例模式的线程安全性

```java
private static volatile SingletonDemo instance = null;
```

###  最终代码

```java
package com.example.demo;

/** @title: SingletonDemo @Author Wen @Date: 30/10/2021 下午3:12 @Version 1.0 */
public class SingletonDemo {

  private static volatile SingletonDemo instance = null;

  private SingletonDemo() {
    System.out.println(Thread.currentThread().getName() + "\t 我是构造方法SingletonDemo");
  }

  /** DCL(Double Check Lock)双端检锁机制 */
  public static SingletonDemo getInstance() {
    if (instance == null) {
      // a 双重检查加锁多线程情况下会出现某个线程虽然这里已经为空，但是另外一个线程已经执行到d处
      synchronized (SingletonDemo.class) {
        // c不加volitale关键字的话有可能会出现尚未完全初始化就获取到的情况。原因是内存模型允许无序写入
        if (instance == null) {
          // d 此时才开始初始化
          instance = new SingletonDemo();
        }
      }
    }
    return instance;
  }

  public static void main(String[] args) {
    //        // 这里的 == 是比较内存地址
    //        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    //        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    //        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    //        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
    for (int i = 0; i < 10; i++) {
      new Thread(
              () -> {
                SingletonDemo.getInstance();
              },
              String.valueOf(i))
          .start();
    }
  }
}

```

