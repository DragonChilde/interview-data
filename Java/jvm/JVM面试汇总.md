

# JVM垃圾回收的时候如何确定垃圾？是否知道什么是GC Roots

## 什么是垃圾

简单来说就是内存中已经不再被使用的空间就是垃圾

## 如何判断一个对象是否可以被回收

### 引用计数法

Java中，引用和对象是有关联的。如果要操作对象则必须用引用进行。

因此，很显然一个简单的办法就是通过引用计数来判断一个对象是否可以回收。简单说，给对象中添加一个引用计数器

每当有一个地方引用它，计数器值加1

每当有一个引用失效，计数器值减1

任何时刻计数器值为零的对象就是不可能再被使用的，那么这个对象就是可回收对象。

那么为什么主流的Java虚拟机里面都没有选用这个方法呢？其中最主要的原因是它很难解决对象之间相互循环引用的问题。

该算法存在但目前无人用了，解决不了循环引用的问题，了解即可。

![](http://120.77.237.175:9080/photos/eight/java/jvm/10.png)

### 枚举根节点做可达性分析

根搜索路径算法

为了解决引用计数法的循环引用个问题，Java使用了可达性分析的方法：

![](http://120.77.237.175:9080/photos/eight/java/jvm/14.png)

所谓 GC Roots 或者说 Tracing Roots的“根集合” 就是一组必须活跃的引用

基本思路就是通过一系列名为 GC Roots的对象作为起始点，从这个被称为GC Roots的对象开始向下搜索，如果一个对象到GC Roots没有任何引用链相连，则说明此对象不可用。也即给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可到达的）对象就被判定为存活，没有被遍历到的对象就被判定为死亡

![](http://120.77.237.175:9080/photos/eight/java/jvm/15.png)

必须从GC Roots对象开始，这个类似于linux的 / 也就是根目录

蓝色部分是从GC Roots出发，能够循环可达

而白色部分，从GC Roots出发，无法到达

### 一句话理解GC Roots

假设我们现在有三个实体，分别是 人，狗，毛衣

然后他们之间的关系是：人 牵着 狗，狗穿着毛衣，他们之间是强连接的关系

有一天人消失了，只剩下狗狗 和 毛衣，这个时候，把人想象成 GC Roots，因为 人 和 狗之间失去了绳子连接，

那么狗可能被回收，也就是被警察抓起来，被送到流浪狗寄养所

假设狗和人有强连接的时候，狗狗就不会被当成是流浪狗

### 那些对象可以当做GC Roots

- 虚拟机栈（栈帧中的局部变量区，也叫做局部变量表）中的引用对象
- 方法区中的类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中的JNI（Native方法）的引用对象

### 代码说明

```java
/**
 * 在Java中，可以作为GC Roots的对象有：
 * - 虚拟机栈（栈帧中的局部变量区，也叫做局部变量表）中的引用对象
 * - 方法区中的类静态属性引用的对象
 * - 方法区中常量引用的对象
 * - 本地方法栈中的JNI（Native方法）的引用对象
 * @title: GCRootDemo
 * @Author Wen
 * @Date: 11/11/2021 下午4:37
 * @Version 1.0
 */
public class GCRootDemo {

    // 方法区中的类静态属性引用的对象
    // private static GCRootDemo2 t2;

    // 方法区中的常量引用，GC Roots 也会以这个为起点，进行遍历
    // private static final GCRootDemo3 t3 = new GCRootDemo3(8);

    public static void m1() {
        // 第一种，虚拟机栈中的引用对象
        GCRootDemo t1 = new GCRootDemo();
        System.gc();
        System.out.println("第一次GC完成");
    }
    public static void main(String[] args) {
        m1();
    }
}
```

------

# JVM参数调优

## 前言

你说你做过JVM调优和参数配置，请问如何盘点查看JVM系统默认值

使用`jps`和`jinfo`进行查看

```
-Xms：初始堆空间
-Xmx：堆最大值
-Xss：栈空间
```

`-Xms` 和 `-Xmx`最好调整一致，防止`JVM`频繁进行收集和回收

[官网链接](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)

## JVM参数类型

- 标配参数（从JDK1.0 - Java12都在，很稳定）
  - -version
  - -help
  - java -showversion
- X参数（了解）
  - -Xint：解释执行
  - -Xcomp：第一次使用就编译成本地代码
  - -Xmixed：混合模式
- XX参数（重点）
  - Boolean类型
    - 公式：-XX:+ 或者-某个属性 + 表示开启，-表示关闭
    - Case：-XX:-PrintGCDetails：表示关闭了GC详情输出
  - key-value类型
    - 公式：-XX:属性key=属性value
    - 不满意初始值，可以通过下列命令调整
    - case：如何：-XX:MetaspaceSize=21807104：查看Java元空间的值

## 查看运行的Java程序，JVM参数是否开启，具体值为多少？

首先我们运行一个`HelloGC`的`java`程序

```java
public class HelloGC {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("hello GC");
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

然后使用下列命令查看它的默认参数

```
jps：查看java的后台进程
jinfo：查看正在运行的java程序
```

具体使用：

```
jps -l得到进程号
```

```
PS H:\Code\demo> jps -l
13136 com.example.demo.HelloGC
19840
12872 sun.tools.jps.Jps
17032 org.jetbrains.jps.cmdline.Launcher
```

查看到`HelloGC`的进程号为：13136

我们使用`jinfo -flag` 然后查看是否开启`PrintGCDetails`这个参数

```
jinfo -flag PrintGCDetails 13136
```

得到的内容为

```
-XX:-PrintGCDetails
```

上面提到了，-号表示关闭，即没有开启`PrintGCDetails`这个参数

下面我们需要在启动`HelloGC`的时候，增加 `PrintGCDetails`这个参数，需要在运行程序的时候配置JVM参数,在VM Options中加入下面的代码，现在+号表示开启

```
-XX:+PrintGCDetails
```

然后在使用jinfo查看我们的配置

```
jps -l
jinfo -flag PrintGCDetails 13136
```

得到的结果为

```
-XX:+PrintGCDetails
```

我们看到原来的-号变成了`+`号，说明我们通过 `VM Options`配置的`JVM`参数已经生效了

使用下列命令，会把`jvm`的全部默认参数输出

```
jinfo -flags 进程号
```

------

##  题外话（坑题）

两个经典参数：-Xms 和 -Xmx，这两个参数 如何解释

这两个参数，还是属于XX参数，因为取了别名

- -Xms 等价于 -XX:InitialHeapSize ：初始化堆内存（默认只会用最大物理内存的64分1）
- -Xmx 等价于 -XX:MaxHeapSize ：最大堆内存（默认只会用最大物理内存的4分1）

## 查看JVM默认参数

- `-XX:+PrintFlagsInitial`

  - 主要是查看初始默认值

  - 公式

    - `java -XX:+PrintFlagsInitial -version`

    - `java -XX:+PrintFlagsInitial`（重要参数）

      ```
      PS H:\Code\demo> java -XX:+PrintFlagsInitial
      [Global flags]
           intx ActiveProcessorCount                      = -1                                  {product}
          uintx AdaptiveSizeDecrementScaleFactor          = 4                                   {product}
          uintx AdaptiveSizeMajorGCDecayTimeScale         = 10                                  {product}
          uintx AdaptiveSizePausePolicy                   = 0                                   {product}
          uintx AdaptiveSizePolicyCollectionCostMargin    = 50                                  {product}
          uintx AdaptiveSizePolicyInitializingSteps       = 20                                  {product}
          uintx AdaptiveSizePolicyOutputInterval          = 0                                   {product}
          uintx AdaptiveSizePolicyWeight                  = 10                                  {product}
          uintx AdaptiveSizeThroughPutPolicy              = 0                                   {product}
          uintx AdaptiveTimeWeight                        = 25                                  {product}
           bool AdjustConcurrency                         = false                               {product}
           bool AggressiveHeap                            = false                               {product}
           bool AggressiveOpts                            = false                               {product}
           intx AliasLevel                                = 3                                   {C2 product}
           bool AlignVector                               = true                                {C2 product}
           intx AllocateInstancePrefetchLines             = 1                                   {product}
           intx AllocatePrefetchDistance                  = -1                                  {product}
           intx AllocatePrefetchInstr                     = 0                                   {product}
           intx AllocatePrefetchLines                     = 3                                   {product}
           intx AllocatePrefetchStepSize                  = 16                                  {product}
           intx AllocatePrefetchStyle                     = 1                                   {product}
           bool AllowJNIEnvProxy                          = false                               {product}
           bool AllowNonVirtualCalls                      = false                               {product}
           bool AllowParallelDefineClass                  = false                               {product}
           bool AllowUserSignalHandlers                   = false                               {product}
      ...
      ```

- `-XX:+PrintFlagsFinal`：表示修改以后，最终的值

 会将`JVM`的各个结果都进行打印

 如果有 := 表示修改过的， = 表示没有修改过的

------

##  工作中常用的JVM基本配置参数

![](http://120.77.237.175:9080/photos/eight/java/jvm/16.png)

###  查看堆内存

查看JVM的初始化堆内存 -Xms 和最大堆内存 Xmx

```java
public class HelloGC {
    public static void main(String[] args) throws InterruptedException {
        // 返回Java虚拟机中内存的总量
        long totalMemory = Runtime.getRuntime().totalMemory();

        // 返回Java虚拟机中试图使用的最大内存量
        long maxMemory = Runtime.getRuntime().maxMemory();

        System.out.println("TOTAL_MEMORY(-Xms) = " + totalMemory + "(字节)、" + (totalMemory / (double)1024 / 1024) + "MB");
        System.out.println("MAX_MEMORY(-Xmx) = " + maxMemory + "(字节)、" + (maxMemory / (double)1024 / 1024) + "MB");
    }
}
```

运行结果为：

```
TOTAL_MEMORY(-Xms) = 257425408(字节)、245.5MB
MAX_MEMORY(-Xmx) = 3804758016(字节)、3628.5MB
Heap
 PSYoungGen      total 76288K, used 6554K [0x000000076af80000, 0x0000000770480000, 0x00000007c0000000)
  eden space 65536K, 10% used [0x000000076af80000,0x000000076b5e68c8,0x000000076ef80000)
  from space 10752K, 0% used [0x000000076fa00000,0x000000076fa00000,0x0000000770480000)
  to   space 10752K, 0% used [0x000000076ef80000,0x000000076ef80000,0x000000076fa00000)
 ParOldGen       total 175104K, used 0K [0x00000006c0e00000, 0x00000006cb900000, 0x000000076af80000)
  object space 175104K, 0% used [0x00000006c0e00000,0x00000006c0e00000,0x00000006cb900000)
 Metaspace       used 3662K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 388K, committed 512K, reserved 1048576K
```

-Xms 初始堆内存为：物理内存的1/64 -Xmx 最大堆内存为：系统物理内存的 1/4

###  打印JVM默认参数

使用 `-XX:+PrintCommandLineFlags` 打印出JVM的默认的简单初始化参数

比如我的机器输出为：

```
PS H:\Code\demo> java -XX:+PrintCommandLineFlags
-XX:InitialHeapSize=267456640 -XX:MaxHeapSize=4279306240 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
```

###  生活常用调优参数

- `-Xms`：初始化堆内存，默认为物理内存的1/64，等价于` -XX:initialHeapSize`
- `-Xmx`：最大堆内存，默认为物理内存的1/4，等价于`-XX:MaxHeapSize`
- `-Xss`：设计单个线程栈的大小，一般默认为512K~1024K，等价于 `-XX:ThreadStackSize`
  - 使用` jinfo -flag ThreadStackSize` 会发现 `-XX:ThreadStackSize = 0`
  - 这个值的大小是取决于平台的
  - Linux/x64:1024KB
  - OS X：1024KB
  - Oracle Solaris：1024KB
  - Windows：取决于虚拟内存的大小
- `-Xmn`：设置年轻代大小
- `-XX:MetaspaceSize`：设置元空间大小
  - 元空间的本质和永久代类似，都是对JVM规范中方法区的实现，不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存，因此，默认情况下，元空间的大小仅受本地内存限制。
  - `-Xms10m -Xmx10m -XX:MetaspaceSize=1024m -XX:+PrintFlagsFinal`
  - 但是默认的元空间大小：只有20多M
  - 为了防止在频繁的实例化对象的时候，让元空间出现OOM，因此可以把元空间设置的大一些
- `-XX:PrintGCDetails`：输出详细GC收集日志信息
  - GC
  - Full GC

GC日志收集流程图

![](http://120.77.237.175:9080/photos/eight/java/jvm/17.png)

我们使用一段代码，制造出垃圾回收的过程

首先我们设置一下程序的启动配置: 设置初始堆内存为10M，最大堆内存为10M

```
-Xms10m -Xmx10m -XX:+PrintGCDetails
```

然后用下列代码，创建一个 非常大空间的`byte`类型数组

```java
public class HelloGC {
    public static void main(String[] args) throws InterruptedException {
        byte [] byteArray = new byte[50 * 1024 * 1024];
    }
}
```

运行后，发现会出现下列错误，这就是OOM：java内存溢出，也就是堆空间不足

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.HelloGC.main(HelloGC.java:23)
```

同时还打印出了GC垃圾回收时候的详情

```
[GC (Allocation Failure) [PSYoungGen: 2048K->496K(2560K)] 2048K->765K(9728K), 0.0007367 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 700K->488K(2560K)] 969K->773K(9728K), 0.0005232 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 488K->488K(2560K)] 773K->781K(9728K), 0.0003931 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 293K->694K(7168K)] 781K->694K(9728K), [Metaspace: 3612K->3612K(1056768K)], 0.0037449 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 694K->694K(9728K), 0.0002155 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 694K->676K(7168K)] 694K->676K(9728K), [Metaspace: 3612K->3612K(1056768K)], 0.0052187 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 2560K, used 98K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 4% used [0x00000000ffd00000,0x00000000ffd18888,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 676K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 9% used [0x00000000ff600000,0x00000000ff6a91e8,0x00000000ffd00000)
 Metaspace       used 3666K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 356K, capacity 388K, committed 512K, reserved 1048576K
```

问题发生的原因：

因为们通过 -Xms10m 和 -Xmx10m 只给Java堆栈设置了10M的空间，但是创建了50M的对象，因此就会出现空间不足，而导致出错

同时在垃圾收集的时候，我们看到有两个对象：GC 和 Full GC

#### GC垃圾收集

GC在新生区

```
[GC (Allocation Failure) [PSYoungGen: 2048K->496K(2560K)] 2048K->765K(9728K), 0.0007367 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
```

GC (Allocation Failure)：表示分配失败，那么就需要触发年轻代空间中的内容被回收

```
[PSYoungGen: 2048K->496K(2560K)] 2048K->765K(9728K), 0.0007367 secs]
```

参数对应的图为：

![](http://120.77.237.175:9080/photos/eight/java/jvm/18.png)

####  Full GC垃圾回收

Full GC大部分发生在养老区

```
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 694K->676K(7168K)] 694K->676K(9728K), [Metaspace: 3612K->3612K(1056768K)], 0.0052187 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

![](http://120.77.237.175:9080/photos/eight/java/jvm/19.png)

规律：

```
[名称： GC前内存占用 -> GC后内存占用 (该区内存总大小)]
```

当我们出现了老年代都扛不住的时候，就会出现OOM异常

------

### -XX:SurvivorRatio

调节新生代中 eden 和 S0、S1的空间比例，默认为 -XX:SuriviorRatio=8，Eden:S0:S1 = 8:1:1

加入设置成 -XX:SurvivorRatio=4，则为 Eden:S0:S1 = 4:1:1

SurvivorRatio值就是设置eden区的比例占多少，S0和S1相同

Java堆从GC的角度还可以细分为：新生代（Eden区，From Survivor区合To Survivor区）和老年代

![](http://120.77.237.175:9080/photos/eight/java/jvm/20.png)

- eden、SurvivorFrom复制到SurvivorTo，年龄 + 1

首先，当Eden区满的时候会触发第一次GC，把还活着的对象拷贝到SurvivorFrom去，当Eden区再次触发GC的时候会扫描Eden区合From区域，对这两个区域进行垃圾回收，经过这次回收后还存活的对象，则直接复制到To区域（如果对象的年龄已经到达老年的标准，则赋值到老年代区），通知把这些对象的年龄 + 1

- 清空eden、SurvivorFrom

然后，清空eden，SurvivorFrom中的对象，也即复制之后有交换，谁空谁是to

- SurvivorTo和SurvivorFrom互换

最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区，部分对象会在From和To区域中复制来复制去，如此交换15次（由JVM参数MaxTenuringThreshold决定，这个参数默认为15），最终如果还是存活，就存入老年代

![](http://120.77.237.175:9080/photos/eight/java/jvm/21.png)

------

### -XX:NewRatio（了解）

配置年轻代new 和老年代old 在堆结构的占比

默认： -XX:NewRatio=2 新生代占1，老年代2，年轻代占整个堆的1/3

-XX:NewRatio=4：新生代占1，老年代占4，年轻代占整个堆的1/5，NewRadio值就是设置老年代的占比，剩下的1个新生代

新生代特别小，会造成频繁的进行GC收集

------

### -XX:MaxTenuringThreshold

设置垃圾最大年龄，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区，部分对象会在From和To区域中复制来复制去，如此交换15次（由JVM参数MaxTenuringThreshold决定，这个参数默认为15），最终如果还是存活，就存入老年代

这里就是调整这个次数的，默认是15，并且设置的值 在 0~15之间

查看默认进入老年代年龄：jinfo -flag MaxTenuringThreshold 17344

-XX:MaxTenuringThreshold=0：设置垃圾最大年龄。如果设置为0的话，则年轻对象不经过Survivor区，直接进入老年代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大的值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概念

------

#  Java中的引用

##  前言

在原来的时候，我们谈到一个类的实例化

```java
Person p = new Person()
```

在等号的左边，就是一个对象的引用，存储在栈中

而等号右边，就是实例化的对象，存储在堆中

其实这样的一个引用关系，就被称为强引用

## 整体架构

![](http://120.77.237.175:9080/photos/eight/java/jvm/22.png)

## 强引用

当内存不足的时候，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收，打死也不回收~！

强引用是我们最常见的普通对象引用，只要还有一个强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到，JVM也不会回收，因此强引用是造成Java内存泄漏的主要原因之一。

对于一个普通的对象，如果没有其它的引用关系，只要超过了引用的作用于或者显示地将相应（强）引用赋值为null，一般可以认为就是可以被垃圾收集的了（当然具体回收时机还是要看垃圾回收策略）

强引用小例子：

```java
public class StrongReferenceDemo {
  public static void main(String[] args) {
    // 这样定义的默认就是强应用
    Object obj1 = new Object();

    // 使用第二个引用，指向刚刚创建的Object对象
    Object obj2 = obj1;

    // 置空
    obj1 = null;

    // 垃圾回收
    System.gc();

    System.out.println(obj1);

    System.out.println(obj2);
  }
}
```

输出结果我们能够发现，即使 obj1 被设置成了null，然后调用gc进行回收，但是也没有回收实例出来的对象，obj2还是能够指向该地址，也就是说垃圾回收器，并没有将该对象进行垃圾回收

```
null
java.lang.Object@2f0e140b
```

------

## 软引用

软引用是一种相对弱化了一些的引用，需要用Java.lang.ref.SoftReference类来实现，可以让对象豁免一些垃圾收集，对于只有软引用的对象来讲：

- 当系统内存充足时，它不会被回收
- 当系统内存不足时，它会被回收

软引用通常在对内存敏感的程序中，比如高速缓存就用到了软引用，内存够用 的时候就保留，不够用就回收

具体使用

```java
public class SoftReferenceDemo {

  /** 内存够用的时候 */
  public static void softRefMemoryEnough() {
    // 创建一个强应用
    Object o1 = new Object();
    // 创建一个软引用
    SoftReference<Object> softReference = new SoftReference<>(o1);
    System.out.println(o1);
    System.out.println(softReference.get());

    o1 = null;
    // 手动GC
    System.gc();

    System.out.println(o1);
    System.out.println(softReference.get());
  }

  public static void main(String[] args) {
    softRefMemoryEnough();
  }
}
```

我们写了两个方法，一个是内存够用的时候，一个是内存不够用的时候

我们首先查看内存够用的时候，首先输出的是 o1 和 软引用的 softReference，我们都能够看到值

然后我们把o1设置为`null`，执行手动`GC`后，我们发现`softReference`的值还存在，说明内存充足的时候，软引用的对象不会被回收

```
java.lang.Object@2f0e140b
java.lang.Object@2f0e140b
[GC (System.gc()) [PSYoungGen: 3932K->808K(76288K)] 3932K->808K(251392K), 0.0317095 secs] [Times: user=0.06 sys=0.00, real=0.03 secs] 
[Full GC (System.gc()) [PSYoungGen: 808K->0K(76288K)] [ParOldGen: 0K->719K(175104K)] 808K->719K(251392K), [Metaspace: 3629K->3629K(1056768K)], 0.0203734 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
null
java.lang.Object@2f0e140b
Heap
 PSYoungGen      total 76288K, used 1966K [0x000000076af80000, 0x0000000770480000, 0x00000007c0000000)
  eden space 65536K, 3% used [0x000000076af80000,0x000000076b16ba70,0x000000076ef80000)
  from space 10752K, 0% used [0x000000076ef80000,0x000000076ef80000,0x000000076fa00000)
  to   space 10752K, 0% used [0x000000076fa00000,0x000000076fa00000,0x0000000770480000)
 ParOldGen       total 175104K, used 719K [0x00000006c0e00000, 0x00000006cb900000, 0x000000076af80000)
  object space 175104K, 0% used [0x00000006c0e00000,0x00000006c0eb3df8,0x00000006cb900000)
 Metaspace       used 3636K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
```

下面我们看当内存不够的时候，我们使用了`JVM`启动参数配置，给初始化堆内存为5M

```
-Xms5m -Xmx5m -XX:+PrintGCDetails
```

但是在创建对象的时候，我们创建了一个30M的大对象

```java
// 创建30M的大对象
byte[] bytes = new byte[30 * 1024 * 1024];
```

```java

/** @title: SoftReferenceDemo @Author Wen @Date: 13/11/2021 上午11:44 @Version 1.0 */
public class SoftReferenceDemo {

  /** JVM配置，故意产生大对象并配置小的内存，让它的内存不够用了导致OOM，看软引用的回收情况 -Xms5m -Xmx5m -XX:+PrintGCDetails */
  public static void softRefMemoryNoEnough() {

    System.out.println("========================");
    // 创建一个强应用
    Object o1 = new Object();
    // 创建一个软引用
    SoftReference<Object> softReference = new SoftReference<>(o1);
    System.out.println(o1);
    System.out.println(softReference.get());

    o1 = null;

    // 模拟OOM自动GC
    try {
      // 创建30M的大对象
      byte[] bytes = new byte[30 * 1024 * 1024];
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      System.out.println(o1);
      System.out.println(softReference.get());
    }
  }

  public static void main(String[] args) {

    softRefMemoryNoEnough();
  }
}
```

这就必然会触发垃圾回收机制，这也是中间出现的垃圾回收过程，最后看结果我们发现，`o1 `和 `softReference`都被回收了，因此说明，软引用在内存不足的时候，会自动回收

```
========================
java.lang.Object@2f0e140b
java.lang.Object@2f0e140b
[GC (Allocation Failure) [PSYoungGen: 694K->504K(1536K)] 972K->817K(5632K), 0.0067946 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 504K->504K(1536K)] 817K->857K(5632K), 0.0018312 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 504K->0K(1536K)] [ParOldGen: 353K->675K(4096K)] 857K->675K(5632K), [Metaspace: 3631K->3631K(1056768K)], 0.0086422 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 675K->675K(5632K), 0.0013035 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 675K->657K(4096K)] 675K->657K(5632K), [Metaspace: 3631K->3631K(1056768K)], 0.0052605 secs] [Times: user=0.03 sys=0.00, real=0.00 secs] 
null
null
Heap
 PSYoungGen      total 1536K, used 52K [0x00000000ffe00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 5% used [0x00000000ffe00000,0x00000000ffe0d020,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 4096K, used 657K [0x00000000ffa00000, 0x00000000ffe00000, 0x00000000ffe00000)
  object space 4096K, 16% used [0x00000000ffa00000,0x00000000ffaa4660,0x00000000ffe00000)
 Metaspace       used 3666K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 356K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.SoftReferenceDemo.softRefMemoryNoEnough(SoftReferenceDemo.java:41)
	at com.example.demo.SoftReferenceDemo.main(SoftReferenceDemo.java:54)
```

------

## 弱引用

不管内存是否够，只要有GC操作就会进行回收

弱引用需要用 `java.lang.ref.WeakReference` 类来实现，它比软引用生存期更短

对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的空间。

```java
public class WeakReferenceDemo {
    public static void main(String[] args) {
        Object o1 = new Object();
        WeakReference<Object> weakReference = new WeakReference<>(o1);
        System.out.println(o1);
        System.out.println(weakReference.get());
        o1 = null;
        System.gc();
        System.out.println(o1);
        System.out.println(weakReference.get());
    }
}
```

我们看结果，能够发现，我们并没有制造出OOM内存溢出，而只是调用了一下GC操作，垃圾回收就把它给收集了

```
java.lang.Object@2f0e140b
java.lang.Object@2f0e140b
[GC (System.gc()) [PSYoungGen: 5243K->808K(76288K)] 5243K->816K(251392K), 0.0008371 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 808K->0K(76288K)] [ParOldGen: 8K->712K(175104K)] 816K->712K(251392K), [Metaspace: 3597K->3597K(1056768K)], 0.0039723 secs] [Times: user=0.05 sys=0.00, real=0.00 secs] 
null
null
Heap
 PSYoungGen      total 76288K, used 1966K [0x000000076af80000, 0x0000000770480000, 0x00000007c0000000)
  eden space 65536K, 3% used [0x000000076af80000,0x000000076b16baa0,0x000000076ef80000)
  from space 10752K, 0% used [0x000000076ef80000,0x000000076ef80000,0x000000076fa00000)
  to   space 10752K, 0% used [0x000000076fa00000,0x000000076fa00000,0x0000000770480000)
 ParOldGen       total 175104K, used 712K [0x00000006c0e00000, 0x00000006cb900000, 0x000000076af80000)
  object space 175104K, 0% used [0x00000006c0e00000,0x00000006c0eb2088,0x00000006cb900000)
 Metaspace       used 3605K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 348K, capacity 388K, committed 512K, reserved 1048576K
```

------

## 软引用和弱引用的使用场景

场景：假如有一个应用需要读取大量的本地图片

- 如果每次读取图片都从硬盘读取则会严重影响性能
- 如果一次性全部加载到内存中，又可能造成内存溢出

此时使用软引用可以解决这个问题

设计思路：使用`HashMap`来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占的空间，从而有效地避免了`OOM`的问题

```java
Map<String, SoftReference<String>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
```

### `WeakHashMap`是什么？

比如一些常常和底层打交道的，mybatis等，底层都应用到了WeakHashMap

WeakHashMap和HashMap类似，只不过它的Key是使用了弱引用的，也就是说，当执行GC的时候，HashMap中的key会进行回收，下面我们使用例子来测试一下

我们使用了两个方法，一个是普通的HashMap方法

我们输入一个Key-Value键值对，然后让它的key置空，然后在查看结果

第二个是使用了`WeakHashMap`，完整代码如下

```java
public class WeakHashMapDemo {

  public static void main(String[] args) {
    myHashMap();
    System.out.println("==========");
    myWeakHashMap();
  }

  private static void myHashMap() {
    Map<Integer, String> map = new HashMap<>();
    Integer key = new Integer(1);
    String value = "HashMap";

    map.put(key, value);
    System.out.println(map);

    key = null;

    System.gc();

    System.out.println(map);
  }

  private static void myWeakHashMap() {
    Map<Integer, String> map = new WeakHashMap<>();
    Integer key = new Integer(1);
    String value = "WeakHashMap";

    map.put(key, value);
    System.out.println(map);

    key = null;

    System.gc();

    System.out.println(map);
  }
}
```

最后输出结果为：

```
{1=HashMap}
{1=HashMap}
==========
{1=WeakHashMap}
{}
```

从这里我们看到，对于普通的`HashMap`来说，`key`置空并不会影响，`HashMap`的键值对，因为这个属于强引用，不会被垃圾回收。

但是`WeakHashMap`，在进行`GC`操作后，弱引用的就会被回收

------

##  虚引用

###  概念

虚引用又称为幽灵引用，需要`java.lang.ref.PhantomReference` 类来实现

顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。

如果一个对象持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和引用队列ReferenceQueue联合使用。

虚引用的主要作用和跟踪对象被垃圾回收的状态，仅仅是提供一种确保对象被finalize以后，做某些事情的机制。

PhantomReference的get方法总是返回null，因此无法访问对象的引用对象。其意义在于说明一个对象已经进入finalization阶段，可以被gc回收，用来实现比finalization机制更灵活的回收操作

换句话说，设置虚引用关联的唯一目的，就是在这个对象被收集器回收的时候，收到一个系统通知或者后续添加进一步的处理，Java技术允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前，做必要的清理工作

这个就相当于Spring AOP里面的后置通知

### 场景

一般用于在回收时候做通知相关操作

## 引用队列 `ReferenceQueue`

软引用，弱引用，虚引用在回收之前，需要在引用队列保存一下

我们在初始化的弱引用或者虚引用的时候，可以传入一个引用队列

```java
Object o1 = new Object();

// 创建引用队列
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

// 创建一个弱引用
WeakReference<Object> weakReference = new WeakReference<>(o1, referenceQueue);
```

那么在进行GC回收的时候，弱引用和虚引用的对象都会被回收，但是在回收之前，它会被送至引用队列中

完整代码如下：

```java
public class PhantomReferenceDemo {

  public static void main(String[] args) {

    Object o1 = new Object();

    // 创建引用队列
    ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

    // 创建一个弱引用
    WeakReference<Object> weakReference = new WeakReference<>(o1, referenceQueue);

    // 创建一个弱引用
    //        PhantomReference<Object> weakReference = new PhantomReference<>(o1, referenceQueue);

    System.out.println(o1);
    System.out.println(weakReference.get());
    // 取队列中的内容
    System.out.println(referenceQueue.poll());

    o1 = null;
    System.gc();
    System.out.println("执行GC操作");

    try {
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    System.out.println(o1);
    System.out.println(weakReference.get());
    // 取队列中的内容
    System.out.println(referenceQueue.poll());
  }
}
```

运行结果

```
java.lang.Object@2f0e140b
java.lang.Object@2f0e140b
null
执行GC操作
null
null
java.lang.ref.WeakReference@7440e464
```

从这里我们能看到，在进行垃圾回收后，我们弱引用对象，也被设置成null，但是在队列中还能够导出该引用的实例，这就说明在回收之前，该弱引用的实例被放置引用队列中了，我们可以通过引用队列进行一些后置操作

------

##  GCRoots和四大引用小总结

- 红色部分在垃圾回收之外，也就是强引用的
- 蓝色部分：属于软引用，在内存不够的时候，才回收
- 虚引用和弱引用：每次垃圾回收的时候，都会被干掉，但是它在干掉之前还会存在引用队列中，我们可以通过引用队列进行一些通知机制

![](http://120.77.237.175:9080/photos/eight/java/jvm/23.png)

------

#  Java内存溢出OOM

##  经典错误

`JVM`中常见的两个错误

`StackoverFlowError `：栈溢出

`OutofMemoryError: java heap space`：堆溢出

除此之外，还有以下的错误

- `java.lang.StackOverflowError`
- `java.lang.OutOfMemoryError：java heap space`
- `java.lang.OutOfMemoryError：GC overhead limit exceeeded`
- `java.lang.OutOfMemoryError：Direct buffer memory`
- `java.lang.OutOfMemoryError：unable to create new native thread`
- `java.lang.OutOfMemoryError：Metaspace`

##  架构

`OutOfMemoryError`和`StackOverflowError`是属于`Error`，不是`Exception`

![](http://120.77.237.175:9080/photos/eight/java/jvm/24.png)

## StackoverFlowError

堆栈溢出，我们有最简单的一个递归调用，就会造成堆栈溢出，也就是深度的方法调用

栈一般是512K，不断的深度调用，直到栈被撑破

```java
public class StackOverflowErrorDemo {

    public static void main(String[] args) {
        stackOverflowError();
    }
    /**
     * 栈一般是512K，不断的深度调用，直到栈被撑破
     * Exception in thread "main" java.lang.StackOverflowError
     */
    private static void stackOverflowError() {
        stackOverflowError();
    }
}
```

运行结果

```
Exception in thread "main" java.lang.StackOverflowError
	at com.example.demo.StackOverflowErrorDemo.stackOverflowError(StackOverflowErrorDemo.java:19)
	at com.example.demo.StackOverflowErrorDemo.stackOverflowError(StackOverflowErrorDemo.java:19)
```

------

## OutOfMemoryError

### java heap space

创建了很多对象，导致堆空间不够存储

```java
public class JavaHeapSpaceDemo {

    public static void main(String[] args) {

        // 堆空间的大小 -Xms10m -Xmx10m
        // 创建一个 80M的字节数组
        byte [] bytes = new byte[80 * 1024 * 1024];
    }
}
```

我们创建一个80M的数组，会直接出现`Java heap space`

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.JavaHeapSpaceDemo.main(JavaHeapSpaceDemo.java:15)
```

### GC overhead limit exceeded

GC回收时间过长时会抛出OutOfMemoryError，过长的定义是，超过了98%的时间用来做GC，并且回收了不到2%的堆内存

连续多次GC都只回收了不到2%的极端情况下，才会抛出。假设不抛出GC overhead limit 错误会造成什么情况呢？

那就是GC清理的这点内存很快会再次被填满，迫使GC再次执行，这样就形成了恶性循环，CPU的使用率一直都是100%，而GC却没有任何成果。

![](http://120.77.237.175:9080/photos/eight/java/jvm/25.png)

代码演示：

为了更快的达到效果，我们首先需要设置JVM启动参数

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
```

这个异常出现的步骤就是，我们不断的像list中插入String对象，直到启动GC回收

```java
public class GCOverheadLimitDemo {
    public static void main(String[] args) {
        int i = 0;
        List<String> list = new ArrayList<>();
        try {
            while(true) {
                list.add(String.valueOf(++i).intern());
            }
        } catch (Exception e) {
            System.out.println("***************i:" + i);
            e.printStackTrace();
            throw e;
        } finally {

        }

    }
}
```

运行结果

```
[GC (Allocation Failure) [PSYoungGen: 2046K->504K(2560K)] 2046K->693K(9728K), 0.0023071 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 671K->504K(2560K)] 861K->733K(9728K), 0.0043996 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 504K->488K(2560K)] 733K->757K(9728K), 0.0019376 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 269K->687K(7168K)] 757K->687K(9728K), [Metaspace: 3571K->3571K(1056768K)], 0.0108548 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 687K->687K(9728K), 0.0002809 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 687K->669K(7168K)] 687K->669K(9728K), [Metaspace: 3571K->3571K(1056768K)], 0.0056066 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 2560K, used 101K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 4% used [0x00000000ffd00000,0x00000000ffd195f8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 669K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 9% used [0x00000000ff600000,0x00000000ff6a7568,0x00000000ffd00000)
 Metaspace       used 3631K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 351K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.example.demo.JavaHeapSpaceDemo.main(JavaHeapSpaceDemo.java:15)
```

我们能够看到 多次`Full GC`，并没有清理出空间，在多次执行`GC`操作后，就抛出异常` GC overhead limit`

------

### Direct buffer memory

Netty + NIO：这是由于NIO引起的

写NIO程序的时候经常会使用ByteBuffer来读取或写入数据，这是一种基于通道(Channel) 与 缓冲区(Buffer)的I/O方式，它可以使用Native 函数库直接分配堆外内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

ByteBuffer.allocate(capability)：第一种方式是分配JVM堆内存，属于GC管辖范围，由于需要拷贝所以速度相对较慢

ByteBuffer.allocteDirect(capability)：第二种方式是分配OS本地内存，不属于GC管辖范围，由于不需要内存的拷贝，所以速度相对较快

但如果不断分配本地内存，堆内存很少使用，那么JVM就不需要执行GC，DirectByteBuffer对象就不会被回收，这时候怼内存充足，但本地内存可能已经使用光了，再次尝试分配本地内存就会出现OutOfMemoryError，那么程序就奔溃了。

一句话说：本地内存不足，但是堆内存充足的时候，就会出现这个问题

我们使用 `-XX:MaxDirectMemorySize=5m` 配置能使用的堆外物理内存为5M

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
```

然后我们申请一个6M的空间

```
// 只设置了5M的物理内存使用，但是却分配 6M的空间
ByteBuffer bb = ByteBuffer.allocateDirect(6 * 1024 * 1024);
```

这个时候，运行就会出现问题了

```java
public class OOMEDirectBufferMemoryDemo {

    /**
     * -Xms5m -Xmx5m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
     *
     * @param args
     * @throws InterruptedException
     */
    public static void main(String[] args) throws InterruptedException {
        System.out.println(String.format("配置的maxDirectMemory: %.2f MB",//
                sun.misc.VM.maxDirectMemory() / 1024.0 / 1024));

        TimeUnit.SECONDS.sleep(3);

        ByteBuffer bb = ByteBuffer.allocateDirect(6 * 1024 * 1024);
    }

}
```

这个时候，运行就会出现问题了

```
[GC (Allocation Failure) [PSYoungGen: 2048K->504K(2560K)] 2048K->710K(9728K), 0.0051132 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
配置的maxDirectMemory: 5.00 MB
[GC (System.gc()) [PSYoungGen: 2313K->488K(2560K)] 2520K->1321K(9728K), 0.0026175 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 488K->0K(2560K)] [ParOldGen: 833K->1165K(7168K)] 1321K->1165K(9728K), [Metaspace: 4645K->4645K(1056768K)], 0.0201696 secs] [Times: user=0.06 sys=0.00, real=0.02 secs] 
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
	at java.nio.Bits.reserveMemory(Bits.java:695)
	at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
	at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:311)
	at com.example.demo.OOMEDirectBufferMemoryDemo.main(OOMEDirectBufferMemoryDemo.java:26)
Heap
 PSYoungGen      total 2560K, used 53K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 2% used [0x00000000ffd00000,0x00000000ffd0d6d8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 1165K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 16% used [0x00000000ff600000,0x00000000ff723710,0x00000000ffd00000)
 Metaspace       used 4680K, capacity 4808K, committed 4864K, reserved 1056768K
  class space    used 463K, capacity 496K, committed 512K, reserved 1048576K
```

------

### unable to create new native thread

不能够创建更多的新的线程了，也就是说创建线程的上限达到了

在高并发场景的时候，会应用到

高并发请求服务器时，经常会出现如下异常`java.lang.OutOfMemoryError:unable to create new native thread`，准确说该`native thread`异常与对应的平台有关

导致原因：

- 应用创建了太多线程，一个应用进程创建多个线程，超过系统承载极限
- 服务器并不允许你的应用程序创建这么多线程，linux系统默认运行单个进程可以创建的线程为1024个，如果应用创建超过这个数量，就会报 `java.lang.OutOfMemoryError:unable to create new native thread`

解决方法：

1. 想办法降低你应用程序创建线程的数量，分析应用是否真的需要创建这么多线程，如果不是，改代码将线程数降到最低
2. 对于有的应用，确实需要创建很多线程，远超过`linux`系统默认1024个线程限制，可以通过修改`linux`服务器配置，扩大`linux`默认限制

```java
public class UnableCreateNewThreadDemo {
    public static void main(String[] args) {
        for (int i = 0; ; i++) {
            System.out.println("************** i = " + i);
            new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

上面程序在`Linux OS（CentOS）`运行，会出现下列的错误，线程数大概在900多个

```
Exception in thread "main" java.lang.OutOfMemoryError: unable to cerate new native thread
```

#### 上限调整

非root用户登录Linux系统（CentOS）测试

服务器级别调参调优

查看系统线程限制数目

```
ulimit -u
```

修改系统线程限制数目

```
vim /etc/security/limits.d/90-nproc.conf
```

打开后发现除了`root`，其他账户都限制在1024个

![](http://120.77.237.175:9080/photos/eight/java/jvm/26.png)

假如我们想要张三这个用卢运行，希望他生成的线程多一些，我们可以如下配置

![](http://120.77.237.175:9080/photos/eight/java/jvm/27.png)

------

### Metaspace

使用`java -XX:+PrintFlagsInitial`命令查看本机的初始化参数，`-XX:MetaspaceSize`为21810376B（大约20.8M）

Java 8及之后的版本使用`Metaspace`来替代永久代。

元空间内存不足，`Matespace`元空间应用的是本地内存

`-XX:MetaspaceSize` 的处理化大小为20M

#### 元空间是什么

元空间就是我们的方法区，存放的是类模板，类信息，常量池等

Metaspace是方法区HotSpot中的实现，它与持久代最大的区别在于：Metaspace并不在虚拟内存中，而是使用本地内存，也即在java8中，class metadata（the virtual machines internal presentation of Java class），被存储在叫做Matespace的native memory

永久代（java8后背元空间Metaspace取代了）存放了以下信息：

- 虚拟机加载的类信息
- 常量池
- 静态变量
- 即时编译后的代码

模拟Metaspace空间溢出，我们不断生成类 往元空间里灌输，类占据的空间总会超过Metaspace指定的空间大小

首先添加`CGLib`依赖

```xml
		<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>3.3.0</version>
		</dependency>
```

#### 代码

在模拟异常生成时候，因为初始化的元空间为20M，因此我们使用JVM参数调整元空间的大小，为了更好的效果

```
-XX:MetaspaceSize=8m -XX:MaxMetaspaceSize=8m
```

代码如下：

```java
package com.example.demo;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

/** @title: MetaspaceOutOfMemoryDemo @Author Wen @Date: 14/11/2021 下午5:22 @Version 1.0 */
public class MetaspaceOutOfMemoryDemo {

  // 静态类
  static class OOMTest {}

  public static void main(final String[] args) {
    // 模拟计数多少次以后发生异常
    int i = 0;
    try {
      while (true) {
        i++;
        // 使用Spring的动态字节码技术
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(OOMTest.class);
        enhancer.setUseCache(false);
        enhancer.setCallback(
            new MethodInterceptor() {

              @Override
              public Object intercept(
                  Object o, Method method, Object[] objects, MethodProxy methodProxy)
                  throws Throwable {
                return methodProxy.invokeSuper(o, args);
              }
            });
      }
    } catch (Exception e) {
      System.out.println("发生异常的次数:" + i);
      e.printStackTrace();
    } finally {

    }
  }
}
```

会出现以下错误：

```
发生异常的次数: 201
java.lang.OutOfMemoryError:Metaspace
```

------

# 垃圾收集器

## GC垃圾回收算法和垃圾收集器关系

> 天上飞的理念，要有落地的实现（垃圾收集器就是GC垃圾回收算法的实现）
>
> GC算法是内存回收的方法论，垃圾收集器就是算法的落地实现

GC算法主要有以下几种

- 引用计数（几乎不用，无法解决循环引用的问题）
- 复制拷贝（用于新生代）
- 标记清除（用于老年代）
- 标记整理（用于老年代）

因为目前为止还没有完美的收集器出现，更没有万能的收集器，只是针对具体应用最合适的收集器，进行分代收集（那个代用什么收集器）

## 四种主要的垃圾收集器

- Serial：串行回收 `-XX:+UseSeriallGC`
- Parallel：并行回收 `-XX:+UseParallelGC`
- CMS：并发标记清除
- G1
- ZGC：（java 11 出现的）

![](http://120.77.237.175:9080/photos/eight/java/jvm/28.png)

### Serial

串行垃圾回收器，它为单线程环境设计且值使用一个线程进行垃圾收集，会暂停所有的用户线程，只有当垃圾回收完成时，才会重新唤醒主线程继续执行。所以不适合服务器环境

![](http://120.77.237.175:9080/photos/eight/java/jvm/29.png)

### Parallel

并行垃圾收集器，多个垃圾收集线程并行工作，此时用户线程也是阻塞的，适用于科学计算 / 大数据处理等弱交互场景，也就是说Serial 和 Parallel其实是类似的，不过是多了几个线程进行垃圾收集，但是主线程都会被暂停，但是并行垃圾收集器处理时间，肯定比串行的垃圾收集器要更短

![](http://120.77.237.175:9080/photos/eight/java/jvm/30.png)

###  CMS

并发标记清除，用户线程和垃圾收集线程同时执行（不一定是并行，可能是交替执行），不需要停顿用户线程，互联网公司都在使用，适用于响应时间有要求的场景。并发是可以有交互的，也就是说可以一边进行收集，一边执行应用程序。

![](http://120.77.237.175:9080/photos/eight/java/jvm/31.png)

###  G1

G1垃圾回收器将堆内存分割成不同区域，然后并发的进行垃圾回收

![](http://120.77.237.175:9080/photos/eight/java/jvm/32.png)

## 垃圾收集器总结

- 串行垃级回收器(Serial) - 它为单线程环境设计且值使用一个线程进行垃圾收集，会暂停所有的用户线程，只有当垃圾回收完成时，才会重新唤醒主线程继续执行。所以不适合服务器环境。
- 并行垃圾回收器(Parallel) - 多个垃圾收集线程并行工作，此时用户线程也是阻塞的，适用于科学计算 / 大数据处理等弱交互场景，也就是说Serial 和 Parallel其实是类似的，不过是多了几个线程进行垃圾收集，但是主线程都会被暂停，但是并行垃圾收集器处理时间，肯定比串行的垃圾收集器要更短。
  并发垃圾回收器(CMS) - 用户线程和垃圾收集线程同时执行（不一定是并行，可能是交替执行），不需要停顿用户线程，互联网公司都在使用，适用于响应时间有要求的场景。
- G1垃圾回收器 - G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收。
- ZGC（Java 11的，了解）

> 注意：并行垃圾回收在单核CPU下可能会更慢

![](http://120.77.237.175:9080/photos/eight/java/jvm/33.png)

## 查看默认垃圾收集器

使用下面JVM命令，查看配置的初始参数

```
java -XX:+PrintCommandLineFlags -version
```

然后运行一个程序后，能够看到它的一些初始配置信息

```
PS H:\Code\demo> java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=267456640 -XX:MaxHeapSize=4279306240 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
openjdk version "1.8.0_302"
OpenJDK Runtime Environment (build 1.8.0_302-b08)
OpenJDK 64-Bit Server VM (build 25.302-b08, mixed mode)
```

移动到最后一句，就能看到 `-XX:+UseParallelGC` 说明使用的是并行垃圾回收

或者

```
jps -l
```

得出Java程序号

```
jinfo -flags (Java程序号)
```

## 默认垃圾收集器有哪些

Java中一共有7大垃圾收集器

- 年轻代GC
  - `UserSerialGC`：串行垃圾收集器
  - `UserParallelGC`：并行垃圾收集器
  - `UseParNewGC`：年轻代的并行垃圾回收器
- 老年代GC
  - `UseConcMarkSweepGC`：（CMS）并发标记清除
  - `UseParallelOldGC`：老年代的并行垃圾回收器
  - `UserSerialOldGC`：串行老年代垃圾收集器（已经被移除）
- 老嫩通吃
  - `UseG1GC：G1`垃圾收集器

底层源码

![](http://120.77.237.175:9080/photos/eight/java/jvm/34.png)

## 各垃圾收集器的使用范围

垃圾收集器就来具体实现这些GC算法并实现内存回收。

不同厂商、不同版本的虚拟机实现差别很大，`HotSpot`中包含的收集器如下图所示：

![](http://120.77.237.175:9080/photos/eight/java/jvm/35.png)

新生代使用的：

- Serial Copying： UserSerialGC，串行垃圾回收器
- Parallel Scavenge：UserParallelGC，并行垃圾收集器
- ParNew：UserParNewGC，新生代并行垃圾收集器

老年区使用的：

- Serial Old：UseSerialOldGC，老年代串行垃圾收集器
- Parallel Compacting（Parallel Old）：UseParallelOldGC，老年代并行垃圾收集器
- CMS：UseConcMarkSwepp，并行标记清除垃圾收集器

各区都能使用的：

G1：UseG1GC，G1垃圾收集器

垃圾收集器就来具体实现这些GC算法并实现内存回收，不同厂商，不同版本的虚拟机实现差别很大，HotSpot中包含的收集器如下图所示：

![](http://120.77.237.175:9080/photos/eight/java/jvm/36.png)

## 部分参数说明

- DefNew：Default New Generation
- Tenured：Old
- ParNew：Parallel New Generation
- PSYoungGen：Parallel Scavenge
- ParOldGen：Parallel Old Generation

## Java中的Server和Client模式

使用范围：一般使用Server模式，Client模式基本不会使用

操作系统

- 32位的Window操作系统，不论硬件如何都默认使用Client的JVM模式

- 32位的其它操作系统，2G内存同时有2个cpu以上用Server模式，低于该配置还是Client模式

- 64位只有Server模式

  ```
  PS H:\Code\demo> java -version
  openjdk version "1.8.0_302"
  OpenJDK Runtime Environment (build 1.8.0_302-b08)
  OpenJDK 64-Bit Server VM (build 25.302-b08, mixed mode)
  ```

------

## 新生代下的垃圾收集器

### 串行GC(Serial)

串行GC（Serial）（Serial Copying）

是一个单线程单线程的收集器，在进行垃圾收集时候，必须暂停其他所有的工作线程直到它收集结束。

![](http://120.77.237.175:9080/photos/eight/java/jvm/37.png)

串行收集器是最古老，最稳定以及效率高的收集器，只使用一个线程去回收但其在垃圾收集过程中可能会产生较长的停顿(Stop-The-World 状态)。 虽然在收集垃圾过程中需要暂停所有其它的工作线程，但是它简单高效，对于限定单个CPU环境来说，没有线程交互的开销可以获得最高的单线程垃圾收集效率，因此Serial垃圾收集器依然是Java虚拟机运行在Client模式下默认的新生代垃圾收集器

对应JVM参数是：`-XX:+UseSerialGC`

开启后会使用：Serial(Young区用) + Serial Old(Old区用) 的收集器组合

表示：新生代、老年代都会使用串行回收收集器，新生代使用复制算法，老年代使用标记-整理算法

```java
public class GCDemo {

  public static void main(String[] args) throws InterruptedException {

    Random rand = new Random(System.nanoTime());

    try {
      String str = "Hello, World";
      while (true) {
        str += str + rand.nextInt(Integer.MAX_VALUE) + rand.nextInt(Integer.MAX_VALUE);
      }
    } catch (Throwable e) {
      e.printStackTrace();
    }
  }
}
```

VM参数：（启用UseSerialGC）

```
-Xms10m -Xmx10m -XX:PrintGCDetails -XX:+PrintConmandLineFlags -XX:+UseSerialGC
```

输出结果：

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseSerialGC 
[GC (Allocation Failure) [DefNew: 2712K->319K(3072K), 0.0029588 secs] 2712K->843K(9920K), 0.0039647 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 2749K->235K(3072K), 0.0021740 secs] 3273K->1431K(9920K), 0.0021994 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 2185K->1K(3072K), 0.0011462 secs] 3382K->2137K(9920K), 0.0011775 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 953K->1K(3072K), 0.0009992 secs] 3089K->3077K(9920K), 0.0010199 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 1946K->2K(3072K), 0.0014423 secs] 5021K->4956K(9920K), 0.0014643 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 1942K->5K(3072K), 0.0018968 secs] 6897K->6838K(9920K), 0.0019274 secs] [Times: user=0.00 sys=0.02, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 1909K->1909K(3072K), 0.0000137 secs][Tenured: 6833K->3570K(6848K), 0.0029570 secs] 8743K->3570K(9920K), [Metaspace: 3615K->3615K(1056768K)], 0.0030092 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [Tenured: 3570K->3520K(6848K), 0.0021671 secs] 3570K->3520K(9920K), [Metaspace: 3615K->3615K(1056768K)], 0.0021908 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 3072K, used 97K [0x00000000ff600000, 0x00000000ff950000, 0x00000000ff950000)
  eden space 2752K,   3% used [0x00000000ff600000, 0x00000000ff6187e0, 0x00000000ff8b0000)
  from space 320K,   0% used [0x00000000ff8b0000, 0x00000000ff8b0000, 0x00000000ff900000)
  to   space 320K,   0% used [0x00000000ff900000, 0x00000000ff900000, 0x00000000ff950000)
 tenured generation   total 6848K, used 3520K [0x00000000ff950000, 0x0000000100000000, 0x0000000100000000)
   the space 6848K,  51% used [0x00000000ff950000, 0x00000000ffcc0348, 0x00000000ffcc0400, 0x0000000100000000)
 Metaspace       used 3672K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 355K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
```

- DefNew：Default New Generation
- Tenured：Old

------

### 并行GC(ParNew)

并行收集器，使用多线程进行垃圾回收，在垃圾收集，会Stop-the-World暂停其他所有的工作线程直到它收集结束

![](http://120.77.237.175:9080/photos/eight/java/jvm/38.png)

`ParNew`收集器其实就是Serial收集器新生代的并行多线程版本，最常见的应用场景时配合老年代的CMS GC工作，其余的行为和Serial收集器完全一样，Par`New垃圾收集器在垃圾收集过程中同样也要暂停所有其他的工作线程。它是很多Java虚拟机运行在Server模式下新生代的默认垃圾收集器。

常见对应JVM参数：`-XX:+UseParNewGC` 启动`ParNew`收集器，只影响新生代的收集，不影响老年代

开启上述参数后，会使用：`ParNew`（Young区用） +` Serial Old`的收集器组合，新生代使用复制算法，老年代采用标记-整理算法

```
-Xms10m -Xmx10m -XX:PrintGCDetails -XX:+PrintConmandLineFlags -XX:+UseParNewGC
```

但是会出现警告，即 `ParNew `和` Serial Old `这样搭配，Java8已经不再被推荐

![](http://120.77.237.175:9080/photos/eight/java/jvm/39.png)

> 备注：` -XX:ParallelGCThreads `限制线程数量，默认开启和CPU数目相同的线程数

复用上一节的`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParNewGC
```

输出结果：

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC 
[GC (Allocation Failure) [ParNew: 2718K->319K(3072K), 0.0012317 secs] 2718K->787K(9920K), 0.0012753 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2824K->186K(3072K), 0.0007338 secs] 3292K->1724K(9920K), 0.0007566 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2246K->46K(3072K), 0.0008461 secs] 3784K->3075K(9920K), 0.0008695 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 1053K->12K(3072K), 0.0010759 secs] 4082K->5030K(9920K), 0.0011004 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2013K->2013K(3072K), 0.0000119 secs][Tenured: 5018K->3272K(6848K), 0.0021217 secs] 7031K->3272K(9920K), [Metaspace: 3538K->3538K(1056768K)], 0.0021817 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2054K->6K(3072K), 0.0004696 secs] 5327K->5268K(9920K), 0.0004879 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2016K->2016K(3072K), 0.0000241 secs][Tenured: 5261K->3772K(6848K), 0.0018193 secs] 7278K->3772K(9920K), [Metaspace: 3580K->3580K(1056768K)], 0.0018767 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [Tenured: 3772K->3615K(6848K), 0.0017243 secs] 3772K->3615K(9920K), [Metaspace: 3580K->3580K(1056768K)], 0.0017466 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 par new generation   total 3072K, used 169K [0x00000000ff600000, 0x00000000ff950000, 0x00000000ff950000)
  eden space 2752K,   6% used [0x00000000ff600000, 0x00000000ff62b180, 0x00000000ff8b0000)
  from space 320K,   0% used [0x00000000ff900000, 0x00000000ff900000, 0x00000000ff950000)
  to   space 320K,   0% used [0x00000000ff8b0000, 0x00000000ff8b0000, 0x00000000ff900000)
 tenured generation   total 6848K, used 3615K [0x00000000ff950000, 0x0000000100000000, 0x0000000100000000)
   the space 6848K,  52% used [0x00000000ff950000, 0x00000000ffcd7c00, 0x00000000ffcd7c00, 0x0000000100000000)
 Metaspace       used 3667K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 354K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
OpenJDK 64-Bit Server VM warning: Using the ParNew young collector with the Serial old collector is deprecated and will likely be removed in a future release
```

------

### 行回收GC（Parallel）/ （Parallel Scavenge）

因为`Serial `和 `ParNew`都不推荐使用了，因此现在新生代默认使用的是`Parallel Scavenge`，也就是新生代和老年代都是使用并行

![](http://120.77.237.175:9080/photos/eight/java/jvm/40.png)

Parallel Scavenge收集器类似ParNew也是一个新生代垃圾收集器，使用复制算法，也是一个并行的多线程的垃圾收集器，俗称吞吐量优先收集器。一句话：串行收集器在新生代和老年代的并行化

它关注的重点是：

可控制的吞吐量（Thoughput = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间) ），也即比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99%。高吞吐量意味着高效利用CPU时间，它多用于在后台运算而不需要太多交互的任务。

自适应调节策略也是ParallelScavenge收集器与ParNew收集器的一个重要区别。（自适应调节策略：虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间( -XX:MaxGCPauseMills)）或最大的吞吐量。

常用JVM参数：-XX:+UseParallelGC 或 -XX:+UseParallelOldGC（可互相激活）使用Parallel Scanvenge收集器

开启该参数后：新生代使用复制算法，老年代使用标记-整理算法

多说一句：`-XX:ParallelGCThreads=数字N` 表示启动多少个GC线程

- cpu>8 N= 5/8
- cpu<8 N=实际个数

复用上一节`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelGC
```

输出结果：

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
[GC (Allocation Failure) [PSYoungGen: 2006K->504K(2560K)] 2006K->733K(9728K), 0.0127633 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 2258K->499K(2560K)] 2488K->1114K(9728K), 0.0013287 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2059K->344K(2560K)] 2674K->2215K(9728K), 0.0009860 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1897K->280K(2560K)] 3768K->2653K(9728K), 0.0010016 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1856K->312K(2560K)] 6239K->4694K(9728K), 0.0007040 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2358K->0K(2560K)] [ParOldGen: 6392K->3697K(7168K)] 8750K->3697K(9728K), [Metaspace: 3649K->3649K(1056768K)], 0.0045953 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 3697K->3697K(8704K), 0.0002182 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 3697K->3677K(7168K)] 3697K->3677K(8704K), [Metaspace: 3649K->3649K(1056768K)], 0.0050724 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 1536K, used 53K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 5% used [0x00000000ffd00000,0x00000000ffd0d560,0x00000000ffe00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
 ParOldGen       total 7168K, used 3677K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 51% used [0x00000000ff600000,0x00000000ff997650,0x00000000ffd00000)
 Metaspace       used 3684K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 357K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
```

------

## 老年代下的垃圾收集器

### 串行GC（Serial Old） / (Serial MSC)

Serial Old是Serial垃圾收集器老年代版本，它同样是一个单线程的收集器，使用标记-整理算法，这个收集器也主要是运行在Client默认的Java虚拟机中默认的老年代垃圾收集器

在Server模式下，主要有两个用途（了解，版本已经到8及以后）

- 在JDK1.5之前版本中与新生代的Parallel Scavenge收集器搭配使用（Parallel Scavenge + Serial Old）
- 作为老年代版中使用CMS收集器的后备垃圾收集方案。

复用上一节`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseSerialOldGC
```

输出结果：

```
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
Unrecognized VM option 'UseSerialOldGC'
Did you mean '(+/-)UseSerialGC'?
```

> 在Java8中，`-XX:+UseSerialOldGC`不起作用。

------

###  并行GC（Parallel Old）/ （Parallel MSC）

Parallel Old收集器是Parallel Scavenge的老年代版本，使用多线程的标记-整理算法，Parallel Old收集器在JDK1.6才开始提供。

在JDK1.6之前，新生代使用ParallelScavenge收集器只能搭配老年代的Serial Old收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量。在JDK1.6以前(Parallel Scavenge + Serial Old)

Parallel Old正是为了在老年代同样提供吞吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，JDK1.8后可以考虑新生代Parallel Scavenge和老年代Parallel Old 收集器的搭配策略。在JDK1.8及后（Parallel Scavenge + Parallel Old）

JVM常用参数：

```
-XX +UseParallelOldGC：使用Parallel Old收集器，设置该参数后，新生代Parallel+老年代 Parallel Old
```

![](http://120.77.237.175:9080/photos/eight/java/jvm/41.png)

复用上一节`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelOldGC
```

输出结果：

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelOldGC 
[GC (Allocation Failure) [PSYoungGen: 2048K->488K(2560K)] 2048K->717K(9728K), 0.0008003 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2272K->483K(2560K)] 2502K->1135K(9728K), 0.0009760 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2002K->424K(2560K)] 2654K->2310K(9728K), 0.0007343 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2435K->480K(2560K)] 4321K->3847K(9728K), 0.0008047 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 2490K->0K(2560K)] [ParOldGen: 5341K->1670K(7168K)] 7832K->1670K(9728K), [Metaspace: 3649K->3649K(1056768K)], 0.0038275 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 987K->0K(2560K)] 4632K->3645K(9728K), 0.0006856 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 3645K->3645K(8704K), 0.0002953 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 3645K->3644K(7168K)] 3645K->3644K(8704K), [Metaspace: 3649K->3649K(1056768K)], 0.0061134 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] 3644K->3644K(9216K), 0.0002163 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2048K)] [ParOldGen: 3644K->3625K(7168K)] 3644K->3625K(9216K), [Metaspace: 3649K->3649K(1056768K)], 0.0058929 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 2048K, used 62K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 6% used [0x00000000ffd00000,0x00000000ffd0fac0,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 7168K, used 3625K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 50% used [0x00000000ff600000,0x00000000ff98a440,0x00000000ffd00000)
 Metaspace       used 3685K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 357K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
```

------

###  并发标记清除GC（CMS）

CMS收集器（Concurrent Mark Sweep：并发标记清除）是一种以最短回收停顿时间为目标的收集器

适合应用在互联网或者B/S系统的服务器上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短。

CMS非常适合堆内存大，CPU核数多的服务器端应用，也是G1出现之前大型应用的首选收集器。

![](http://120.77.237.175:9080/photos/eight/java/jvm/42.png)

oncurrent Mark Sweep：并发标记清除，并发收集低停顿，并发指的是与用户线程一起执行

开启该收集器的JVM参数：` -XX:+UseConcMarkSweepGC` 开启该参数后，会自动将 `-XX:+UseParNewGC`打开，开启该参数后，使用`ParNew`(young 区用）+ `CMS`（Old 区用） + `Serial Old` 的收集器组合，`Serial Old`将作为CMS出错的后备收集器

####  四个步骤

- 初始标记（CMS initial mark）
  - 只是标记一个GC Roots 能直接关联的对象，速度很快，仍然需要暂停所有的工作线程
- 并发标记（CMS concurrent mark）和用户线程一起
  - 进行GC Roots跟踪过程，和用户线程一起工作，不需要暂停工作线程。主要标记过程，标记全部对象
- 重新标记（CMS remark）
  - 为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程，由于并发标记时，用户线程依然运行，因此在正式清理前，在做修正
- 并发清除（CMS concurrent sweep）和用户线程一起
  - 清除GC Roots不可达对象，和用户线程一起工作，不需要暂停工作线程。基于标记结果，直接清理对象，由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户现在一起并发工作，所以总体上来看CMS收集器的内存回收和用户线程是一起并发地执行。

![](http://120.77.237.175:9080/photos/eight/java/jvm/43.png)

优点：并发收集低停顿

缺点：并发执行，对CPU资源压力大，采用的标记清除算法会导致大量碎片

由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说，CMS必须在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW方式进行一次GC，从而造成较大的停顿时间

标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩，CMS也提供了参数 -`XX:CMSFullGCSBeForeCompaction`（默认0，即每次都进行内存整理）来指定多少次CMS收集之后，进行一次压缩的Full GC

复用上一节`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseConcMarkSweepGC
```

输出结果：

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:MaxNewSize=3497984 -XX:MaxTenuringThreshold=6 -XX:NewSize=3497984 -XX:OldPLABSize=16 -XX:OldSize=6987776 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC 
[GC (Allocation Failure) [ParNew: 2729K->320K(3072K), 0.0026719 secs] 2729K->869K(9920K), 0.0027313 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2923K->142K(3072K), 0.0010835 secs] 3473K->1633K(9920K), 0.0011226 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2207K->40K(3072K), 0.0008423 secs] 3698K->2528K(9920K), 0.0008819 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 1050K->15K(3072K), 0.0008183 secs] 3537K->3500K(9920K), 0.0008454 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (CMS Initial Mark) [1 CMS-initial-mark: 3484K(6848K)] 5505K(9920K), 0.0004676 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark-start]
[GC (Allocation Failure) [ParNew: 2021K->10K(3072K), 0.0012643 secs] 5505K->5487K(9920K), 0.0012912 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark: 0.001/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2059K->2059K(3072K), 0.0000164 secs][CMS (concurrent mode failure): 5477K->2702K(6848K), 0.0029678 secs] 7536K->2702K(9920K), [Metaspace: 3633K->3633K(1056768K)], 0.0030271 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2030K->2030K(3072K), 0.0000155 secs][CMS: 6688K->5670K(6848K), 0.0029352 secs] 8719K->5670K(9920K), [Metaspace: 3649K->3649K(1056768K)], 0.0029943 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
[GC (CMS Initial Mark) [1 CMS-initial-mark: 5670K(6848K)] 7663K(9920K), 0.0000895 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 2043K->2043K(3072K), 0.0000138 secs][CMS (concurrent mode failure): 5670K->2680K(6848K), 0.0021920 secs] 7713K->2680K(9920K), [Metaspace: 3649K->3649K(1056768K)], 0.0022429 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [ParNew: 1993K->1993K(3072K), 0.0000152 secs][CMS: 6667K->6667K(6848K), 0.0032905 secs] 8660K->6667K(9920K), [Metaspace: 3649K->3649K(1056768K)], 0.0034663 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [CMS: 6667K->6647K(6848K), 0.0034422 secs] 6667K->6647K(9920K), [Metaspace: 3649K->3649K(1056768K)], 0.0034750 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (CMS Initial Mark) [1 CMS-initial-mark: 6647K(6848K)] 6647K(9920K), 0.0001224 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (CMS Final Remark) [YG occupancy: 51 K (3072 K)][Rescan (parallel) , 0.0001365 secs][weak refs processing, 0.0000058 secs][class unloading, 0.0002029 secs][scrub symbol table, 0.0004628 secs][scrub string table, 0.0000805 secs][1 CMS-remark: 6647K(6848K)] 6699K(9920K), 0.0009486 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 par new generation   total 3072K, used 106K [0x00000000ff600000, 0x00000000ff950000, 0x00000000ff950000)
  eden space 2752K,   3% used [0x00000000ff600000, 0x00000000ff61aad8, 0x00000000ff8b0000)
  from space 320K,   0% used [0x00000000ff900000, 0x00000000ff900000, 0x00000000ff950000)
  to   space 320K,   0% used [0x00000000ff8b0000, 0x00000000ff8b0000, 0x00000000ff900000)
 concurrent mark-sweep generation total 6848K, used 6646K [0x00000000ff950000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 3684K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 357K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
```

------

##  为什么新生代采用复制算法，老年代采用标整算法

### 新生代使用复制算法

因为新生代对象的生存时间比较短，80%的都要回收的对象，采用标记-清除算法则内存碎片化比较严重，采用复制算法可以灵活高效，且便与整理空间。

### 老年代采用标记整理

标记整理算法主要是为了解决标记清除算法存在内存碎片的问题，又解决了复制算法两个Survivor区的问题，因为老年代的空间比较大，不可能采用复制算法，特别占用内存空间

## 垃圾收集器如何选择

### 组合的选择

- 单CPU或者小内存，单机程序
  - -XX:+UseSerialGC
- 多CPU，需要最大的吞吐量，如后台计算型应用
  - -XX:+UseParallelGC（这两个相互激活）
  - -XX:+UseParallelOldGC
- 多CPU，追求低停顿时间，需要快速响应如互联网应用
  - -XX:+UseConcMarkSweepGC
  - -XX:+ParNewGC

|                         |                          |            |                                                              |            |
| :---------------------: | :----------------------: | :--------: | :----------------------------------------------------------: | :--------: |
|          参数           |     新生代垃圾收集器     | 新生代算法 |                       老年代垃圾收集器                       | 老年代算法 |
|    -XX:+UseSerialGC     |         SerialGC         |    复制    |                         SerialOldGC                          |  标记整理  |
|    -XX:+UseParNewGC     |          ParNew          |    复制    |                         SerialOldGC                          |  标记整理  |
|   -XX:+UseParallelGC    |   Parallel [Scavenge]    |    复制    |                         Parallel Old                         |  标记整理  |
| -XX:+UseConcMarkSweepGC |          ParNew          |    复制    | CMS + Serial Old的收集器组合，Serial Old作为CMS出错的后备收集器 |  标记清除  |
|      -XX:+UseG1GC       | G1整体上采用标记整理算法 |  局部复制  |                                                              |            |

------

##  G1垃圾收集器

###  开启G1垃圾收集器

复用上一节`GCDemo`

VM参数：

```
-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseG1GC
```

输出结果

```
-XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation 
[GC pause (G1 Evacuation Pause) (young), 0.0010802 secs]
   [Parallel Time: 0.8 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 190.6, Avg: 190.6, Max: 190.6, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.3, Max: 0.5, Diff: 0.4, Sum: 1.1]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.2, Avg: 0.5, Max: 0.6, Diff: 0.3, Sum: 1.8]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.5, Max: 2, Diff: 1, Sum: 6]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [GC Worker Total (ms): Min: 0.7, Avg: 0.8, Max: 0.8, Diff: 0.1, Sum: 3.1]
      [GC Worker End (ms): Min: 191.3, Avg: 191.3, Max: 191.4, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.1 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 4096.0K(4096.0K)->0.0B(4096.0K) Survivors: 0.0B->1024.0K Heap: 4096.0K(10240.0K)->1135.0K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC pause (G1 Humongous Allocation) (young) (initial-mark), 0.0012504 secs]
   [Parallel Time: 1.1 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 193.6, Avg: 193.6, Max: 193.6, Diff: 0.0]
      [Ext Root Scanning (ms): Min: 0.2, Avg: 0.2, Max: 0.2, Diff: 0.0, Sum: 0.8]
      [Update RS (ms): Min: 0.2, Avg: 0.4, Max: 0.6, Diff: 0.4, Sum: 1.8]
         [Processed Buffers: Min: 1, Avg: 5.0, Max: 17, Diff: 16, Sum: 20]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.2, Avg: 0.4, Max: 0.6, Diff: 0.4, Sum: 1.4]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 2, Avg: 2.3, Max: 3, Diff: 1, Sum: 9]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 1.0, Avg: 1.0, Max: 1.0, Diff: 0.0, Sum: 4.1]
      [GC Worker End (ms): Min: 194.6, Avg: 194.6, Max: 194.6, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 3072.0K(4096.0K)->0.0B(4096.0K) Survivors: 1024.0K->1024.0K Heap: 6132.9K(10240.0K)->2392.1K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC concurrent-root-region-scan-start]
[GC concurrent-root-region-scan-end, 0.0001040 secs]
[GC concurrent-mark-start]
[GC concurrent-mark-end, 0.0008597 secs]
[GC remark [Finalize Marking, 0.0006409 secs] [GC ref-proc, 0.0002145 secs] [Unloading, 0.0009478 secs], 0.0018816 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC cleanup 5419K->5419K(10240K), 0.0001460 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC pause (G1 Humongous Allocation) (young), 0.0007097 secs]
   [Parallel Time: 0.5 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 199.0, Avg: 199.1, Max: 199.5, Diff: 0.5]
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 0.4]
      [Update RS (ms): Min: 0.0, Avg: 0.3, Max: 0.3, Diff: 0.3, Sum: 1.0]
         [Processed Buffers: Min: 0, Avg: 9.8, Max: 13, Diff: 13, Sum: 39]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.0, Avg: 0.4, Max: 0.5, Diff: 0.5, Sum: 1.5]
      [GC Worker End (ms): Min: 199.5, Avg: 199.5, Max: 199.5, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.1 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(4096.0K)->0.0B(4096.0K) Survivors: 1024.0K->0.0B Heap: 8369.2K(10240.0K)->3836.0K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC pause (G1 Humongous Allocation) (young) (initial-mark), 0.0006342 secs]
   [Parallel Time: 0.4 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 200.4, Avg: 200.4, Max: 200.4, Diff: 0.0]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.1, Max: 0.2, Diff: 0.0, Sum: 0.6]
      [Update RS (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.0, Sum: 1.1]
         [Processed Buffers: Min: 8, Avg: 8.0, Max: 8, Diff: 0, Sum: 32]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.4, Avg: 0.4, Max: 0.4, Diff: 0.0, Sum: 1.7]
      [GC Worker End (ms): Min: 200.8, Avg: 200.8, Max: 200.8, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(4096.0K)->0.0B(1024.0K) Survivors: 0.0B->1024.0K Heap: 7846.5K(10240.0K)->5811.1K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC concurrent-root-region-scan-start]
[GC concurrent-root-region-scan-end, 0.0000819 secs]
[GC concurrent-mark-start]
[GC pause (G1 Humongous Allocation) (young), 0.0003908 secs]
   [Parallel Time: 0.2 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 202.1, Avg: 202.1, Max: 202.1, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.1, Max: 0.2, Diff: 0.0, Sum: 0.6]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.5, Max: 1, Diff: 1, Sum: 2]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.3, Max: 2, Diff: 1, Sum: 5]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.1, Avg: 0.2, Max: 0.2, Diff: 0.1, Sum: 0.6]
      [GC Worker End (ms): Min: 202.2, Avg: 202.2, Max: 202.2, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.1 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(1024.0K)->0.0B(2048.0K) Survivors: 1024.0K->0.0B Heap: 7797.1K(10240.0K)->7782.4K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure)  7782K->2649K(10240K), 0.0022011 secs]
   [Eden: 0.0B(2048.0K)->0.0B(5120.0K) Survivors: 0.0B->0.0B Heap: 7782.4K(10240.0K)->2650.0K(10240.0K)], [Metaspace: 3649K->3649K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC concurrent-mark-abort]
[GC pause (G1 Humongous Allocation) (young) (initial-mark), 0.0007441 secs]
   [Parallel Time: 0.5 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 205.4, Avg: 205.4, Max: 205.5, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.2, Max: 0.3, Diff: 0.3, Sum: 0.7]
      [Update RS (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 0.4]
         [Processed Buffers: Min: 0, Avg: 4.3, Max: 7, Diff: 7, Sum: 17]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.2]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.1, Sum: 1.3]
      [GC Worker End (ms): Min: 205.7, Avg: 205.7, Max: 205.8, Diff: 0.1]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.1 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 0.0B(5120.0K)->0.0B(3072.0K) Survivors: 0.0B->0.0B Heap: 4616.7K(10240.0K)->4616.7K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC concurrent-root-region-scan-start]
[GC concurrent-root-region-scan-end, 0.0000029 secs]
[GC concurrent-mark-start]
[GC concurrent-mark-end, 0.0009108 secs]
[GC remark [Finalize Marking, 0.0001478 secs] [GC ref-proc, 0.0001794 secs] [Unloading, 0.0003800 secs], 0.0007922 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC cleanup 8550K->8550K(10240K), 0.0001458 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC pause (G1 Humongous Allocation) (young), 0.0005612 secs]
   [Parallel Time: 0.4 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 209.4, Avg: 209.4, Max: 209.5, Diff: 0.0]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.1, Max: 0.1, Diff: 0.0, Sum: 0.5]
      [Update RS (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.0, Sum: 1.0]
         [Processed Buffers: Min: 7, Avg: 8.0, Max: 9, Diff: 2, Sum: 32]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.4, Avg: 0.4, Max: 0.4, Diff: 0.0, Sum: 1.6]
      [GC Worker End (ms): Min: 209.8, Avg: 209.8, Max: 209.9, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.1 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 0.0B(3072.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 8550.1K(10240.0K)->6583.4K(10240.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure)  6583K->6583K(10240K), 0.0017485 secs]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 6583.4K(10240.0K)->6583.3K(10240.0K)], [Metaspace: 3649K->3649K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure)  6583K->6563K(10240K), 0.0018443 secs]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 6583.3K(10240.0K)->6563.8K(10240.0K)], [Metaspace: 3649K->3649K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
	at java.lang.StringBuilder.append(StringBuilder.java:208)
	at com.example.demo.GCDemo.main(GCDemo.java:15)
Heap
 garbage-first heap   total 10240K, used 6563K [0x00000000ff600000, 0x00000000ff700050, 0x0000000100000000)
  region size 1024K, 1 young (1024K), 0 survivors (0K)
 Metaspace       used 3684K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 357K, capacity 388K, committed 512K, reserved 1048576K
```

###  以前收集器的特点

- 年轻代和老年代是各自独立且连续的内存块
- 年轻代收集使用单eden + S0 + S1 进行复制算法
- 老年代收集必须扫描珍整个老年代区域
- 都是以尽可能少而快速地执行GC为设计原则

###  G1是什么

G1：Garbage-First 收集器，是一款面向服务端应用的收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能满足垃圾收集暂停时间的要求。另外，它还具有一下特征：

- 像CMS收集器一样，能与应用程序并发执行
- 整理空闲空间更快
- 需要更多的时间来预测GC停顿时间
- 不希望牺牲大量的吞吐量性能
- 不需要更大的Java Heap

G1收集器设计目标是取代CMS收集器，它同CMS相比，在以下方面表现的更出色

- G1是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。
- G1的Stop The World（STW）更可控，G1在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

CMS垃圾收集器虽然减少了暂停应用程序的运行时间，但是它还存在着内存碎片问题。于是，为了去除内存碎片问题，同时又保留CMS垃圾收集器低暂停时间的优点，JAVA7发布了一个新的垃圾收集器-G1垃圾收集器

G1是在2012奶奶才在JDK1.7中可用，Oracle官方计划在JDK9中将G1变成默认的垃圾收集器以替代CMS，它是一款面向服务端应用的收集器，主要应用在多CPU和大内存服务器环境下，极大减少垃圾收集的停顿时间，全面提升服务器的性能，逐步替换Java8以前的CMS收集器

主要改变时：Eden，Survivor和Tenured等内存区域不再是连续了，而是变成一个个大小一样的region，每个region从1M到32M不等。一个region有可能属于Eden，Survivor或者Tenured内存区域。

###  特点

- G1能充分利用多CPU，多核环境硬件优势，尽量缩短STW
- G1整体上采用标记-整理算法，局部是通过复制算法，不会产生内存碎片
- 宏观上看G1之中不再区分年轻代和老年代。把内存划分成多个独立的子区域（Region），可以近似理解为一个围棋的棋盘
- G1收集器里面将整个内存区域都混合在一起了，但其本身依然在小范围内要进行年轻代和老年代的区分，保留了新生代和老年代，但他们不再是物理隔离的，而是通过一部分Region的集合且不需要Region是连续的，也就是说依然会采取不同的GC方式来处理不同的区域
- G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代和老年代的区别，也不需要完全独立的Survivor（to space）堆做复制准备，G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换。

### 底层原理

Region区域化垃圾收集器，化整为零，打破了原来新生区和老年区的壁垒，避免了全内存扫描，只需要按照区域来进行扫描即可。

区域化内存划片Region，整体遍为了一些列不连续的内存区域，避免了全内存区的GC操作。

核心思想是将整个堆内存区域分成大小相同的子区域（Region），在JVM启动时会自动设置子区域大小

在堆的使用上，G1并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数`-XX:G1HeapRegionSize=n` 可指定分区大小（1MB~32MB，且必须是2的幂），默认将整堆划分为2048个分区。

大小范围在1MB~32MB，最多能设置2048个区域，也即能够支持的最大内存为：32MB*2048 = 64G内存

Region区域化垃圾收集器

### Region区域化垃圾收集器

G1将新生代、老年代的物理空间划分取消了

![](http://120.77.237.175:9080/photos/eight/java/jvm/44.png)

同时对内存进行了区域划分

![](http://120.77.237.175:9080/photos/eight/java/jvm/45.png)

G1算法将堆划分为若干个区域（Reign），它仍然属于分代收集器，这些Region的一部分包含新生代，新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或者Survivor空间

这些Region的一部分包含老年代，G1收集器通过将对象从一个区域复制到另外一个区域，完成了清理工作。这就意味着，在正常的处理过程中，G1完成了堆的压缩（至少是部分堆的压缩），这样也就不会有CMS内存碎片的问题存在了。

在G1中，还有一种特殊的区域，叫做Humongous（巨大的）区域，如果一个对象占用了空间超过了分区容量50%以上，G1收集器就认为这是一个巨型对象，这些巨型对象默认直接分配在老年代，但是如果他是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响，为了解决这个问题，G1划分了一个Humongous区，它用来专门存放巨型对象。如果一个H区装不下一个巨型对象，那么G1会寻找连续的H区来存储，为了能找到连续的H区，有时候不得不启动Full GC。

###  回收步骤

针对Eden区进行收集，Eden区耗尽后会被触发，主要是小区域收集 + 形成连续的内存块，避免内碎片

- Eden区的数据移动到Survivor区，加入出现Survivor区空间不够，Eden区数据会晋升到Old区
- Survivor区的数据移动到新的Survivor区，部分数据晋升到Old区
- 最后Eden区收拾干净了，GC结束，用户的应用程序继续执行

![](http://120.77.237.175:9080/photos/eight/java/jvm/46.png)

回收完成后

![](http://120.77.237.175:9080/photos/eight/java/jvm/47.png)

小区域收集 + 形成连续的内存块，最后在收集完成后，就会形成连续的内存空间，这样就解决了内存碎片的问题

### 四步过程

- 初始标记：只标记GC Roots能直接关联到的对象
- 并发标记：进行GC Roots Tracing（链路扫描）的过程
- 最终标记：修正并发标记期间，因为程序运行导致标记发生变化的那一部分对象
- 筛选回收：根据时间来进行价值最大化回收

![](http://120.77.237.175:9080/photos/eight/java/jvm/48.png)

------

## G1参数配置及和CMS的比较

- XX:+UseG1GC
- XX:G1HeapRegionSize=n：设置的G1区域的大小。值是2的幂，范围是1MB到32MB。目标是根据最小的Java堆大小划分出约2048个区域
- XX:MaxGCPauseMillis=n：最大GC停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间
- XX:InitiatingHeapOccupancyPercent=n：堆占用了多少的时候就触发GC，默认为45。
- XX:ConcGCThreads=n：并发GC使用的线程数。
- XX:G1ReservePercent=n：设置作为空闲空间的预留内存百分比，以降低目标空间溢出的风险，默认值是10%。

开发人员仅仅需要申明以下参数即可

三步归纳：开始G1+设置最大内存+设置最大停顿时间

`-XX:+UseG1GC -Xmx32G -XX:MaxGCPauseMillis=100`

`-XX:MaxGCPauseMillis=n`：最大GC停顿时间单位毫秒，这是个软目标，JVM尽可能停顿小于这个时间

###  G1和CMS比较

- G1不会产生内碎片
- 是可以精准控制停顿。该收集器是把整个堆（新生代、老年代）划分成多个固定大小的区域，每次根据允许停顿的时间去收集垃圾最多的区域。

------

## SpringBoot结合JVMGC

启动微服务时候，就可以带上JVM和GC的参数

- IDEA开发完微服务工程
- maven进行clean package
- 要求微服务启动的时候，同时配置我们的JVM/GC的调优参数
  - 我们就可以根据具体的业务配置我们启动的JVM参数
- 公式：`java -server jvm的各种参数 -jar 第1步上面的jar/war包名`。

例如：

```
java -Xms1024m -Xmx1024 -XX:UseG1GC -jar   xxx.jar
```
