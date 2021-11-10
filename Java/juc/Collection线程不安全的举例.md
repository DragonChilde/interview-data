##  Collection线程不安全的举例

当我们执行下面语句的时候，底层进行了什么操作

```java
new ArrayList<Integer>();
```

底层创建了一个空的数组，伴随着初始值为10

当执行`add`方法后，如果超过了10，那么会进行扩容，扩容的大小为原值的一半，也就是5个，使用下列方法扩容

```java
Arrays.copyOf(elementData, netCapacity)
```

##  单线程环境下

单线程环境的`ArrayList`是不会有问题的

```java
public class ArrayListNotSafeDemo {
    public static void main(String[] args) {

        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");

        for(String element : list) {
            System.out.println(element);
        }
    }
}
```

## 多线程环境

为什么`ArrayList`是线程不安全的？因为在进行写操作的时候，方法上为了保证并发性，是没有添加`synchronized`修饰，所以并发写的时候，就会出现问题

```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

当我们同时启动30个线程去操作List的时候

```java
public class ArrayListNotSafeDemo {

    public static void main(String[] args) {

        List<String> list = new ArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}
```

这个时候出现了错误，也就是`java.util.ConcurrentModificationException`

```
Exception in thread "13" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:911)
	at java.util.ArrayList$Itr.next(ArrayList.java:861)
	at java.util.AbstractCollection.toString(AbstractCollection.java:461)
	at java.lang.String.valueOf(String.java:2994)
	at java.io.PrintStream.println(PrintStream.java:821)
	at com.example.demo.ArrayListNotSafeDemo.lambda$main$0(ArrayListNotSafeDemo.java:22)
	at java.lang.Thread.run(Thread.java:748)
```

这个异常是并发修改的异常

可看出`toString()`，`Itr.next()`，`Itr.checkForComodification()`后抛出异常，那么看看它们`next()`，`checkForComodification()`源码：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    
    ...
    
	private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;//modCount在AbstractList类声明

        Itr() {}

        ...

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
			...
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();//<---异常在此抛出
        }
    }
    
    
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;//添加时，修改了modCount的值

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
	...
}
```

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
	
    ...

    protected transient int modCount = 0;
    
    ...
}

```

modCount具体详细说明如下：

> The number of times this list has been structurally modified. Structural modifications are those that change the size of the list, or otherwise perturb it in such a fashion that iterations in progress may yield incorrect results.
>
> This field is used by the iterator and list iterator implementation returned by the iterator and listIterator methods. If the value of this field changes unexpectedly, the iterator (or list iterator) will throw a ConcurrentModificationException in response to the next, remove, previous, set or add operations. This provides fail-fast behavior, rather than non-deterministic behavior in the face of concurrent modification during iteration.
>
> 此列表已在结构上修改的次数。结构修改是改变列表大小的人，或以其他方式扰乱这种时尚迭代可以产生不正确的结果。该字段由迭代器和列表迭代器和ListIterator方法返回的迭代器实现使用。如果此字段的值意外地发生变化，迭代器（或列表迭代器）将响应下一个，删除，上一个，设置或添加操作抛出同时映射扫描消息。这提供了失败的行为，而不是在迭代期间并发修改面前的非确定性行为。此列表已在结构上修改的次数。结构修改是改变列表大小的人，或以其他方式扰乱这种时尚迭代可以产生不正确的结果。该字段由迭代器和列表迭代器和ListIterator方法返回的迭代器实现使用。如果此字段的值意外地发生变化，迭代器（或列表迭代器）将响应下一个，删除，上一个，设置或添加操作抛出同时映射扫描消息。这提供了失败的行为，而不是在迭代期间并发修改的非确定性行为。
>
> [Link](https://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html#modCount)

综上所述，假设线程A将通过迭代器next()获取下一元素时，从而将其打印出来。但之前，其他某线程添加新元素至list，结构发生了改变，`modCount`自增。当线程A运行到`checkForComodification()`，`expectedModCount`是`modCount`之前自增的值，判定`modCount != expectedModCount`为真，继而抛出`ConcurrentModificationException`。
##  解决方案

### 方案一：`Vector`

第一种方法，就是不用`ArrayList`这种不安全的List实现类，而采用`Vector`，线程安全的

关于`Vector`如何实现线程安全的，而是在方法上加了锁，即`synchronized`

```java
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }
```

这样就每次只能够一个线程进行操作，所以不会出现线程不安全的问题，但是因为加锁了，导致并发性基于下降

###   方案二：`Collections.synchronized()`

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

采用`Collections`集合工具类，在`ArrayList`外面包装一层 同步 机制

###  方案三：采用JUC里面的方法(推荐)

`CopyOnWriteArrayList`：写时复制，主要是一种读写分离的思想

写时复制，`CopyOnWrite`容器即写时复制的容器，往一个容器中添加元素的时候，不直接往当前容器`Object[]`添加，而是先将`Object[]`进行`copy`，复制出一个新的容器``object[] newElements`，然后新的容器`Object[] newElements`里添加原始，添加元素完后，在将原容器的引用指向新的容器 `setArray(newElements)`；

这样做的好处是可以对`copyOnWrite`容器进行并发的读 ，而不需要加锁，因为当前容器不需要添加任何元素。所以`CopyOnWrite`容器也是一种读写分离的思想，读和写不同的容器

就是写的时候，把`ArrayList`扩容一个出来，然后把值填写上去，在通知其他的线程，`ArrayList`的引用指向扩容后的

查看底层`add`方法源码

```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

首先需要加锁

```java
final ReentrantLock lock = this.lock;
lock.lock();
```

然后在末尾扩容一个单位

```java
Object[] elements = getArray();
int len = elements.length;
Object[] newElements = Arrays.copyOf(elements, len + 1);
```

然后在把扩容后的空间，填写上需要add的内容

```
newElements[len] = e;
```

最后把内容`set`到`Array`中

------

## `HashSet`非线程安全

### `CopyOnWriteArraySet`

底层还是使用`CopyOnWriteArrayList`进行实例化

```java
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```

`CopyOnWriteArraySet`源码一览：

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }

    public CopyOnWriteArraySet(Collection<? extends E> c) {
        if (c.getClass() == CopyOnWriteArraySet.class) {
            @SuppressWarnings("unchecked") CopyOnWriteArraySet<E> cc =
                (CopyOnWriteArraySet<E>)c;
            al = new CopyOnWriteArrayList<E>(cc.al);
        }
        else {
            al = new CopyOnWriteArrayList<E>();
            al.addAllAbsent(c);
        }
    }
 
    //可看出CopyOnWriteArraySet包装了一个CopyOnWriteArrayList
    
    ...
    
    public boolean add(E e) {
        return al.addIfAbsent(e);
    }
    
    public boolean addIfAbsent(E e) {
        Object[] snapshot = getArray();
        return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
            addIfAbsent(e, snapshot);
    }
    
    //暴力查找
    private static int indexOf(Object o, Object[] elements,
                               int index, int fence) {
        if (o == null) {
            for (int i = index; i < fence; i++)
                if (elements[i] == null)
                    return i;
        } else {
            for (int i = index; i < fence; i++)
                if (o.equals(elements[i]))
                    return i;
        }
        return -1;
    }

    private boolean addIfAbsent(E e, Object[] snapshot) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {//还要检查多一次元素存在性，生怕别的线程已经插入了
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i] && eq(e, current[i]))
                        return false;
                if (indexOf(e, current, common, len) >= 0)
                        return false;
            }
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    
    ...
        
}
```

------

### `HashSet`

`HashSet`也是非线性安全的。（`HashSet`内部是包装了一个`HashMap`的）

```java
public class SetNotSafeDemo {

    public static void main(String[] args) {

        Set<String> set = new HashSet<>();
        
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }

}
```

以上代码会输出以下错误

```
Exception in thread "28" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$KeyIterator.next(HashMap.java:1469)
	at java.util.AbstractCollection.toString(AbstractCollection.java:461)
	at java.lang.String.valueOf(String.java:2994)
	at java.io.PrintStream.println(PrintStream.java:821)
	at com.example.demo.SetNotSafeDemo.lambda$main$0(SetNotSafeDemo.java:24)
	at java.lang.Thread.run(Thread.java:748)
```

同理`HashSet`的底层结构就是`HashMap`

```java
	public HashSet() {
        map = new HashMap<>();
    }
```

但是为什么调用 `HashSet.add()`的方法，只需要传递一个元素，而`HashMap`是需要传递`key-value`键值对？

首先我们查看`hashSet`的`add`方法

```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

我们能发现调用`add`的时候，存储一个值进入`map`中，只是作为`key`进行存储，而`value`存储的是一个`Object`类型的常量，也就是说`HashSet`只关心`key`，而不关心`value`

### 解决方法

1. `Collections.synchronizedSet(new HashSet<>())`
2. `CopyOnWriteArraySet<>()`（推荐）

------

## `HashMap`线程不安全

同理`HashMap`在多线程环境下，也是不安全的

```java
public class MapNotSafeDemo {

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }, String.valueOf(i)).start();
        }

    }

}
```

以上会报如下异常

```
Exception in thread "13" Exception in thread "17" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1479)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1477)
	at java.util.AbstractMap.toString(AbstractMap.java:554)
	at java.lang.String.valueOf(String.java:2994)
	at java.io.PrintStream.println(PrintStream.java:821)
	at com.example.demo.MapNotSafeDemo.lambda$main$0(MapNotSafeDemo.java:23)
	at java.lang.Thread.run(Thread.java:748)
```

###  解决方法

1. `HashTable`

2. 使用`Collections.synchronizedMap(new HashMap<>());`

3. 使用 `ConcurrentHashMap`(推荐)

   ```java
   Map<String, String> map = new ConcurrentHashMap<>();
   ```

   



