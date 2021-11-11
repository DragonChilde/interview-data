#  JVM体系结构

##  概览

![](http://120.77.237.175:9080/photos/eight/java/jvm/08.png)

java gc 主要回收的是 方法区 和 堆中的内容

![](http://120.77.237.175:9080/photos/eight/java/jvm/09.png)

##  类加载器

- 类加载器是什么
- 双亲委派机制
- Java类加载的沙箱安全机制

## 常见的垃圾回收算法

- 引用计数

  ![](http://120.77.237.175:9080/photos/eight/java/jvm/10.png)

  在双端循环，互相引用的时候，容易报错，目前很少使用这种方式了

- 复制

  复制算法在年轻代的时候，进行使用，复制时候有交换

    ![](http://120.77.237.175:9080/photos/eight/java/jvm/11.png)

    ![](http://120.77.237.175:9080/photos/eight/java/jvm/12.png)

  优点：没有产生内存碎片

- 标记清除

  先标记，后清除，缺点是会产生内存碎片，用于老年代多一些

    ![](http://120.77.237.175:9080/photos/eight/java/jvm/13.png)

- 标记整理

  标记清除整理

  ![](http://120.77.237.175:9080/photos/eight/java/jvm/07.png)

