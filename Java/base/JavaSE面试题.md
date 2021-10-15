## 自增变量

代码的执行结果是什么呢？

```java
public static void main(String[] args) {
        int i = 1;
        i = i++;
        int j = i++;
        int k = i + ++i * i++;
        System.out.println("i = " + i);
        System.out.println("j = " + j);
        System.out.println("k = " + k);
    }

```

1. 首先,先计算=右边,赋值运算是最后计算,先计算i++

   ![](http://120.77.237.175:9080/photos/eight/java/base/04.jpg)

   - 因为++在i后面,所以先把i的值压到操作栈里面

     ![](http://120.77.237.175:9080/photos/eight/java/base/05.jpg)

   - 因为自增的原故,会把局部变量由1变为2

     ![](http://120.77.237.175:9080/photos/eight/java/base/06.jpg)

   - 这个时候还没完事的,这时只是计算=右边的结果,最后还要赋值计算,把操作数栈的结果赋值给i变量,把局部变量的2覆盖掉,因此,i是曾经变为2的

     ![](http://120.77.237.175:9080/photos/eight/java/base/08.jpg)

2. 现在轮到j,同理,如下,先计算=后面的,赋值运算符最后计算

   ![](http://120.77.237.175:9080/photos/eight/java/base/09.jpg)

   - 因为++在i后面,所以先把i的值压到操作栈里面

     ![](http://120.77.237.175:9080/photos/eight/java/base/10.jpg)

   - 因为自增的原故,会把局部变量由1变为2

     ![](http://120.77.237.175:9080/photos/eight/java/base/11.jpg)

   - 最后,=后面的运算都执行完,执行赋值运算,把操作数栈的值赋值给j

     ![](http://120.77.237.175:9080/photos/eight/java/base/12.jpg)
   
3. 现在轮到k,同理,如下,先计算=后面的,赋值运算符最后计算

   ![](http://120.77.237.175:9080/photos/eight/java/base/13.jpg)

   - 先计算=后面的,赋值运算最后计算,虽然在运算符中乘号的优先级比较大,但在操作数栈里是先把数压到栈里面的,因为先把i压进栈里,当前i的值为2压到栈里

     ![](http://120.77.237.175:9080/photos/eight/java/base/15.jpg)

   - 然后再计算++i,先计算++自增,把局部变量i自增变为3

     ![](http://120.77.237.175:9080/photos/eight/java/base/16.jpg)

   - 然后再把++i的结果压到操作栈里3

     ![](http://120.77.237.175:9080/photos/eight/java/base/17.jpg)

   - 然后再到i++,先把i的值压到操作数栈里

     ![](http://120.77.237.175:9080/photos/eight/java/base/18.jpg)

   - 再进行++自增运算

     ![](http://120.77.237.175:9080/photos/eight/java/base/19.jpg)

   - 这时候再运算符优先级把栈顶的两个数执行相乘运算

     ![](http://120.77.237.175:9080/photos/eight/java/base/20.jpg)
   
     ![](http://120.77.237.175:9080/photos/eight/java/base/21.jpg)
   
   - 执行完相乘运算,因为=后面的运算还没结束,不能运行赋值运算,需要把相乘后面结果继续压到操作数栈里
   
     ![](http://120.77.237.175:9080/photos/eight/java/base/22.jpg)
   
   - 执行操作栈里的相加运算
   
     ![](http://120.77.237.175:9080/photos/eight/java/base/23.jpg)
   
   - 最后执行赋值运算,把结果赋值给k
   
     ![](http://120.77.237.175:9080/photos/eight/java/base/24.jpg)

> 运算符优先级++ > *


最后执行结果为

```
i = 4
j = 1
k = 11
```

**总结**

1.  赋值 =，最后计算
2. = 右边的从左到右加载值依次压入操作数栈
3. 实际先算哪个，看运算符优先级
4. 自增、自减操作都是直接修改变量的值，不经过操作数栈
5. 最后赋值之前，临时结果也是存储在操作数栈**

