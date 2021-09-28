# Redis缓存淘汰策略

> **Redis 缓存淘汰策略相关的面试题**

1. 生产上你们的redis内存设置多少？
2. 如何配置、修改redis的内存大小？
3. 如果内存满了你怎么办？
4. redis 清理内存的方式？定期删除和惰性删除了解过吗
5. redis 的缓存淘汰策略
6. redis 的 LRU 淘汰机制了解过吗？可否手写一个 LRU 算法

------

## redis 内存满了怎么办

> **redis 默认内存多少？在哪里查看? 如何设置修改?**

1. **如何查看 redis 最大占用内存**

   在 redis.conf 配置文件中有一个，输入 `:set nu` 显示行号，大约在 800 多行有一个 `maxmemory` 字段，用预设值 redis 的最大占用内存

   ![](http://120.77.237.175:9080/photos/eight/redis/10.jpg)

2. **redis 会占用物理机器多少内存？**

   如果不设置最大内存大小或者设置最大内存大小为 0，在 64 位操作系统下不限制内存大小，在32位操作系统下最多使用 3GB 内存

3. **一般生产上如何配置 redis 的内存**

   一般推荐Redis设置内存为最大物理内存的四分之三，也就是 0.75

4. **如何修改 redis 内存设置**

   1. 通过修改文件配置（永久生效）：修改 maxmemory 字段，单位为字节

      ![](http://120.77.237.175:9080/photos/eight/redis/11.jpg)

   2. 通过命令修改（重启失效）：`config set maxmemory 104857600` 设置 redis 最大占用内存为 100MB，`config get maxmemory` 获取 redis 最大占用内存

      ```sh
      localhost:6379> config set maxmemory 104857600
      OK
      localhost:6379> config get maxmemory
      1) "maxmemory"
      2) "104857600"
      ```

5. **通过命令查看 redis 内存使用情况?**

   通过 info 指令可以查看 redis 内存使用情况：`used_memory_human` 表示实际已经占用的内存，`maxmemory` 表示 redis 最大占用内存

   ```
   # Memory
   used_memory:885008
   used_memory_human:864.27K		#表示实际已经占用的内存
   used_memory_rss:12435456
   used_memory_rss_human:11.86M
   used_memory_peak:4064232
   used_memory_peak_human:3.88M
   used_memory_peak_perc:21.78%
   used_memory_overhead:835288
   used_memory_startup:810000
   used_memory_dataset:49720
   used_memory_dataset_perc:66.29%
   allocator_allocated:1222320
   allocator_active:1589248
   allocator_resident:4304896
   total_system_memory:13304016896
   total_system_memory_human:12.39G
   used_memory_lua:45056
   used_memory_lua_human:44.00K
   used_memory_scripts:1264
   used_memory_scripts_human:1.23K
   number_of_cached_scripts:4
   maxmemory:104857600	#最大占用内存
   maxmemory_human:100.00M
   ```

------

> **如果把 redis 内存打满了会发生什么? 如果 redis 内存使用超出了设置的最大值会怎样?**

修改配置，故意把最大内存设置为 1byte，再通过 `set k1 v1` 命令下 redis 中写入数据，redis 将会报错：(error) OOM command not allowed when used memory > ‘maxmemory’

```sh
localhost:6379> config set maxmemory 1
OK
localhost:6379> set k1 v1
(error) OOM command not allowed when used memory > 'maxmemory'.
```

> 结纶

如果设置了 `maxmemory` 的选项，假如 redis 内存使用达到上限，并且 key 都没有加上过期时间，就会导致数据写爆 redis 内存。为了避免类似情况，于是引出下一部分的内存淘汰策略

------

## redis 缓存淘汰策略

> **redis 如何删除设置了过期时间的 key**

1. **redis过期键的删除策略**

   如果一个键是过期的，那它到了过期时间之后是不是马上就从内存中被被删除呢？那么过期后到底什么时候被删除呢？redis 如何操作的呢？

   通过查看 redis 配置文件可知，默认淘汰策略是【noeviction（Don’t evict anything, just return an error on write operations.）】，如果 redis 内存被写爆了，直接返回 error

   ```
   # allkeys-random -> Remove a random key, any key.
   # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
   # noeviction -> Don't evict anything, just return an error on write operations.
   #
   # LRU means Least Recently Used
   # LFU means Least Frequently Used
   #
   # Both LRU, LFU and volatile-ttl are implemented using approximated
   # randomized algorithms.
   #
   # Note: with any of the above policies, when there are no suitable keys for
   # eviction, Redis will return an error on write operations that require
   # more memory. These are usually commands that create new keys, add data or
   # modify existing keys. A few examples are: SET, INCR, HSET, LPUSH, SUNIONSTORE,
   # SORT (due to the STORE argument), and EXEC (if the transaction includes any
   # command that requires memory).
   #
   # The default is:
   #
   # maxmemory-policy noeviction
   ```

------

> **redis 对于过期 key 的三种不同删除策略**

1. **定时删除**

   立即删除能保证内存中数据的最大新鲜度，因为它保证过期键值会在过期后马上被删除，其所占用的内存也会随之释放。但是立即删除对 CPU 是最不友好的。因为删除操作会占用 CPU 的时间，如果刚好碰上了 CPU 很忙的时候，比如正在做交集或排序等计算的时候，就会给 CPU 造成额外的压力，让 CPU 心累，时时需要删除，忙死。。。。。。

   这会产生大量的性能消耗，同时也会影响数据的读取操作。

   总结：定时删除对 CPU 不友好，但对 memory 友好，用处理器性能换取存储空间（拿时间换空间）

2. **惰性删除**

   惰性删除的策略刚好和定时删除相反，惰性删除在数据到达过期时间后不做处理，等下次访问该数据时发现已过期，并将其删除，并返回不存在。

   使用惰性删除访问数据的特点：访问一个数据，如果发现其在过期时间之内，则返回改数据；如果发现已经过了过期时间，则将其删除，并返回不存在

   如果一个键已经过期，而这个键又仍然保留在数据库中，那么只要这个过期键不被删除，它所占用的内存就不会释放。因此惰性删除策略的缺点是：它对内存是最不友好的。

   在使用惰性删除策略时，如果数据库中有非常多的过期键，而这些过期键又恰好没有被访问到的话，那么它们也许永远也不会被删除（除非用户手动执行 FLUSHDB），我们甚至可以将这种情况看作是一种内存泄漏——无用的垃圾数据占用了大量的内存，而服务器却不会自己去释放它们，这对于运行状态非常依赖于内存的 redis 服务器来说，肯定不是一个好消息

   总结：惰性删除对 memory 不友好，但对 CPU 友好，用存储空间换取处理器性能（拿空间换时间）

3. **定期删除**

   **折中方案：定期删除**

   上面两种删除策略都走极端，因此引出我们的定期删除策略。

   定期删除策略是前两种策略的折中：定期删除策略每隔一段时间执行一次删除过期键操作，并通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。其做法为：周期性轮询 redis 库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度。

   - 定期删除的特点

     - 特点1：CPU 性能占用设置有峰值，检测频度可自定义设置
     - 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理
     - 总结：周期性抽查存储空间（随机抽查，重点抽查)

   - 定期删除的举例

     - redis 默认每间隔 100ms 检查是否有过期的 key，如果有过期 key 则删除。注意：redis 不是每隔100ms 将所有的 key 检查一次而是随机抽取进行检查（如果每隔 100ms，全部 key 进行检查，redis 直接进去ICU）。因此，如果只采用定期删除策略，会导致很多 key 到时间没有删除。

   - 定期删除的难点

     - 定期删除策略的难点是确定删除操作执行的时长和频率：redis 不可能时时刻刻遍历所有被设置了生存时间的 key，来检测数据是否已经到达过期时间，然后对它进行删除。

       如果删除操作执行得太频繁，或者执行的时间太长，定期删除策略就会退化成定时删除策略，以至于将 CPU 时间过多地消耗在删除过期键上面。

       如果删除操作执行得太少，或者执行的时间太短，定期删除策略又会和惰性删除束略一样，出现浪费内存的情况。

       因此，如果采用定期删除策略的话，服务器必须根据情况，合理地设置删除操作的执行时长和执行频率。

   **总结**

   惰性删除和定期删除都存在数据没有被抽到的情况，如果这些数据已经到了过期时间，没有任何作用，这会导致大量过期的 key 堆积在内存中，导致 redis 内存空间紧张或者很快耗尽

   因此必须要有一个更好的兜底方案，接下来引出 redis 内存淘汰策略

   ------

   

   > **redis 6.0.8 版本的内存淘汰策略有哪些？**

   **8 种内存淘汰策略**

   1. `noeviction`：不会驱逐任何key
   2. `allkeys-lru`：对所有key使用LRU算法进行删除
   3. `volatile-lru`：对所有设置了过期时间的key使用LRU算法进行删除
   4. `allkeys-random`：对所有key随机删除
   5. `volatile-random`：对所有设置了过期时间的key随机删除
   6. `volatile-ttl`：删除马上要过期的key
   7. `allkeys-lfu`：对所有key使用LFU算法进行删除
   8. `volatile-lfu`：对所有设置了过期时间的key使用LFU算法进行删除

   ------

   > **如何配置 redis 的内存淘汰策略**

   1. 通过修改文件配置（永久生效）：配置 `maxmemory-policy` 字段

      ```
      # is reached. You can select one from the following behaviors:
      #
      # volatile-lru -> Evict using approximated LRU, only keys with an expire set.
      # allkeys-lru -> Evict any key using approximated LRU.
      # volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
      # allkeys-lfu -> Evict any key using approximated LFU.
      # volatile-random -> Remove a random key having an expire set.
      # allkeys-random -> Remove a random key, any key.
      # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
      # noeviction -> Don't evict anything, just return an error on write operations.
      #
      # LRU means Least Recently Used
      # LFU means Least Frequently Used
      #
      # Both LRU, LFU and volatile-ttl are implemented using approximated
      # randomized algorithms.
      #
      # Note: with any of the above policies, when there are no suitable keys for
      # eviction, Redis will return an error on write operations that require
      # more memory. These are usually commands that create new keys, add data or
      # modify existing keys. A few examples are: SET, INCR, HSET, LPUSH, SUNIONSTORE,
      # SORT (due to the STORE argument), and EXEC (if the transaction includes any
      # command that requires memory).
      #
      # The default is:
      #
      # maxmemory-policy noeviction
      maxmemory-policy allkeys-lru
      ```

   2. 通过命令修改（重启失效）：`config set maxmemory-policy allkeys-lru` 命令设置内存淘汰策略，`config get maxmemory-policy` 命令获取当前采用的内存淘汰策略

      ```sh
      localhost:6379> config set maxmemory-policy allkeys-lru
      OK
      localhost:6379> config get maxmemory-policy
      1) "maxmemory-policy"
      2) "allkeys-lru"
      ```

   ------

## redis LRU 算法

LRU 是 Least Recently Used 的缩写，即最近最少使用，是一种常用的页面置换算法，每次选择最近最久未使用的页面予以淘汰

------

### LRU 算法题来源

[146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)

------

运用你所掌握的数据结构，设计和实现一个 [LRU (最近最少使用) 缓存机制](https://baike.baidu.com/item/LRU) 。

实现 `LRUCache`类：

1. `LRUCache(int capacity)`以正整数作为容量 `capacity`初始化 LRU 缓存
2. `int get(int key)` 如果关键字 `key`存在于缓存中，则返回关键字的值，否则返回 -1 。
3. `void put(int key, int value) `如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

------------------------------------------------
```
进阶：你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

------

### LRU 算法设计思想

查找和插入的时间复杂度为 `O(1)`，HashMap 没得跑了，但是 HashMap 是无序的集合，怎么样将其改造为有序集合呢？答案就是在各个 Node 节点之间增加 `prev` 指针和 `next` 指针，构成双向链表

![](http://120.77.237.175:9080/photos/eight/redis/13.jpg)

LRU 的算法核心是哈希链表，本质就是 `HashMap+DoubleLinkedList` 时间复杂度是O(1)，哈希表+双向链表的结合体，下面这幅动图完美诠释了 `HashMap+DoubleLinkedList` 的工作原理

![](http://120.77.237.175:9080/photos/eight/redis/14.gif)

------

### LRU 算法编码实现

#### 借助JDK自带的**LinkedHashMap**实现

`LinkedHashMap` 的注释中写明了： `LinkedHashMap` 非常适合用来构建 LRU 缓存

```
 * <p>A special {@link #LinkedHashMap(int,float,boolean) constructor} is
 * provided to create a linked hash map whose order of iteration is the order
 * in which its entries were last accessed, from least-recently accessed to
 * most-recently (<i>access-order</i>).  This kind of map is well-suited to
 * building LRU caches.  Invoking the {@code put}, {@code putIfAbsent},
 * {@code get}, {@code getOrDefault}, {@code compute}, {@code computeIfAbsent},
 * {@code computeIfPresent}, or {@code merge} methods results
 * in an access to the corresponding entry (assuming it exists after the
 * invocation completes). The {@code replace} methods only result in an access
 * of the entry if the value is replaced.  The {@code putAll} method generates one
 * entry access for each mapping in the specified map, in the order that
 * key-value mappings are provided by the specified map's entry set iterator.
 * <i>No other methods generate entry accesses.</i>  In particular, operations
 * on collection-views do <i>not</i> affect the order of iteration of the
 * backing map.
```

------

```java
package com.interview.demo.leetcode;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @title: LRUCacheDemo
 * @Author Wen
 * @Date: 28/9/2021 10:14 AM
 * @Version 1.0
 */
public class LRUCacheDemo {


    LinkedHashMap<Integer, Integer> LRUCache;


    //缓存容量
    private int capacity;

    public LRUCacheDemo(int capacity) {

        LRUCache = new LinkedHashMap<Integer, Integer>(capacity, 0.75F, true) {

            /**
             * 用于判断是否需要删除最近最久未使用的节点
             *
             * @param eldest
             * @return
             */
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
                return super.size() > capacity;
            }
        };
    }

    public int get(int key) {
        return LRUCache.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        LRUCache.put(key, value);
    }

    @Override
    public String toString() {
        return LRUCache.toString();
    }

    public static void main(String[] args) {

        LRUCacheDemo lruCacheDemo = new LRUCacheDemo(3);
        lruCacheDemo.put(1, 1);
        lruCacheDemo.put(2, 2);
        lruCacheDemo.put(3, 3);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[1, 2, 3]

        lruCacheDemo.put(4, 4);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[2, 3, 4]

        lruCacheDemo.put(3, 3);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[2, 4, 3]
        lruCacheDemo.put(3, 3);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[2, 4, 3]
        lruCacheDemo.put(3, 3);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[2, 4, 3]

        lruCacheDemo.put(5, 5);
        System.out.println(lruCacheDemo.LRUCache.keySet());  //[4, 3, 5]

    }
}
```

> **为何要重写 `boolean removeEldestEntry(Map.Entry<K, V> eldest)` 方法**

先来看看 `LinkedHashMap` 中的 `boolean removeEldestEntry(Map.Entry<K, V> eldest)` 方法，直接 `return false`，缓存爆就爆，反正就是不会删除 `EldestEntry`

```java
    /** @return   <tt>true</tt> if the eldest entry should be removed
     *           from the map; <tt>false</tt> if it should be retained.
     */
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```

`boolean removeEldestEntry(Map.Entry<K, V> eldest)` 方法在` void afterNodeInsertion(boolean evict)` 方法中被调用，只有当 `boolean removeEldestEntry(Map.Entry<K, V> eldest) `方法返回 true 时，才能够删除 `EldestEntry`

```java
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```

因此我们重写之后的判断条件为：如果 `LinkedHashMap` 中存储的元素个数已经大于缓存容量 `capacity`，则返回 `true`，表示允许删除 `EldestEntry`；否则返回 `false`，表示无需删除 `EldestEntry`

------

**举例说明构造函数中的 `accessOrder` 的含义**

**构造函数中的 `accessOrder` 字段**

在 `LRUCacheDemo` 的构造方法中，我们调用了 `LinkedHashMap` 的构造方法，其中有一个字段为 `accessOrder`

```java
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

`accessOrder = true` 和 `accessOrder = false` 的情况

当 `accessOrder = true` 时，每次使用 key 时（put 或者 get 时），都将 key 对应的数据移动到队尾（右边），表示最近经常使用；当 accessOrder = false 时，key 的顺序为插入双向链表时的顺序

`accessOrder = true`时

```
[1, 2, 3]
[2, 3, 4]
----区别如下---
[2, 4, 3]
[2, 4, 3]
[2, 4, 3]
[4, 3, 5]
```

`accessOrder = false`时

```
[1, 2, 3]
[2, 3, 4]
----区别如下---
[2, 3, 4]
[2, 3, 4]
[2, 3, 4]
[3, 4, 5]
```

------

**`LinkedHashMap` 的 `put()` 方法**

`LinkedHashMap` 的 `put()` 方法其实就是 `HashMap` 的 `put()` 方法，`LinkedHashMap`` 就是 `HashMap`？？？其实并不是。。。

> 下面是`HashMap`源码部分

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);		//主要是这里
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);	//主要是这里
        return null;
    }
```

在 `putval()` 方法的调用了两个方法：`afterNodeAccess(e)` 方法和 `afterNodeInsertion(evict)` 方法，这两个方法就是专门针对于 `LinkedHashMap` 写的方法

在 HashMap 中这些方法均为空实现的方法，没有任何代码逻辑，需要推迟到子类 `LinkedHashMap` 中去实现，等等，听着好像有点耳熟，这不就是模板方法设计模式嘛~

```java
    // Callbacks to allow LinkedHashMap post-actions
    void afterNodeAccess(Node<K,V> p) { }
    void afterNodeInsertion(boolean evict) { }
    void afterNodeRemoval(Node<K,V> p) { }
```

在 `LinkedHashMap`的 `void afterNodeAccess(Node<K,V> e)`方法中：如果设置了` accessOrder = true` 时，则每次使用 `key`时（put 或者 get 时），都将 `key`对应的数据移动到队尾（右边），表示这是最近经常使用的节点

```java
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {	//accessOrder为true时才会进入判断
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

在 `LinkedHashMap `的 `void afterNodeInsertion(boolean evict)` 方法中：如果头指针不为空并且当前需要删除老节点，则执行 `removeNode(hash(key), key, null, false, true)` 方法删除 `EldestEntry`（若 `accessOrder = true` 时，`EldestEntry `表示最近最少使用的数据，若 `accessOrder = false` 时，`EldestEntry`表示最先插入链表的节点）

```java
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```



#### 完全手写

   图解双向队列

   1. `DoubleLinkedList` 双向链表的初始化

      ![](http://120.77.237.175:9080/photos/eight/redis/15.jpg)

   2. 双向链表插入节点

      ![](http://120.77.237.175:9080/photos/eight/redis/16.jpg)

   3. 双向链表删除节点

      ![](http://120.77.237.175:9080/photos/eight/redis/17.jpg)

------

1. 先定义 Node 类作为数据的承载体

   ```java
       // 1. 构造一个Node节点作为数据载体
       class Node<K, V> {
           private K key;
           private V value;
           private Node prev;
           private Node next;
   
           public Node() {
               this.prev = this.next = null;
           }
   
           public Node(K key, V value) {
               this.key = key;
               this.value = value;
               this.prev = this.next = null;
           }
       }
   ```

2. 定义双向链表，里面存放的就是 Node 对象，Node 节点之间通过 `prev` 和 `next` 指针连接起来

   ```
     // 2. 构建一个虚拟的双向链表,里面存放的就是Node
       class DoubleLinkedList<K, V> {
           Node<K, V> head;
           Node<K, V> tail;
   
   
           public DoubleLinkedList() {
               this.head = new Node<>();
               this.tail = new Node<>();
   
               this.head.next = this.tail;
               this.tail.prev = this.head;
           }
   
   
           // 3. 添加到节点头部
           public void addHead(Node<K, V> node) {
   
   
               node.prev = head;
               node.next = head.next;
               head.next.prev = node;
               head.next = node;
           }
   
           // 4. 删除节点
           public void removeNode(Node<K, V> node) {
   
               node.next.prev = node.prev;
               node.prev.next = node.next;
               node.prev = null;
               node.next = null;
           }
   
           // 5. 获得最后一个节点
           public Node<K, V> getLast() {
   
               if (tail.prev == head) {
                   return null;
               }
               return tail.prev;
           }
       }
   ```

3. 通过 `HashMap` 和 `DoubleLinkedList` 构建 `LinkedHashMap`，我们这里可是将最近最常使用的节点放在了双向链表的头部（和 `LinkedHashMap` 不同哦）

   ```java
       private int cacheSize;
       Map<Integer, Node<Integer, Integer>> map;
       DoubleLinkedList<Integer, Integer> doubleLinkedList;
   
       public LRUCacheDemo2(int cacheSize) {
           this.cacheSize = cacheSize; //缓存大小
           map = new HashMap<>();  //根据HashMap进行查找
           doubleLinkedList = new DoubleLinkedList<>();
       }
   
       public int get(int key) {
           if (!map.containsKey(key)) {
               return -1;
           }
   
           Node<Integer, Integer> node = map.get(key);
           doubleLinkedList.removeNode(node);
           doubleLinkedList.addHead(node);
   
           return node.value;
       }
   
       public void put(int key, int value) {
           if (map.containsKey(key)) { //判断当前key是否存在,存在则更新
               Node<Integer, Integer> node = map.get(key);
               node.value = value;
               map.put(key, node);
               doubleLinkedList.removeNode(node);
               doubleLinkedList.addHead(node);
           } else {
   
               if (map.size() == cacheSize) {  //判断当前map是否已经满了
                   Node<Integer, Integer> last = doubleLinkedList.getLast();
                   map.remove(last.key);
                   doubleLinkedList.removeNode(last);
   
               }
   
               //新增Node节点
               Node<Integer, Integer> node = new Node<>(key, value);
               map.put(key, node);
               doubleLinkedList.addHead(node);
           }
       }
   ```

4. 测试

   ```java
       public static void main(String[] args) {
           LRUCacheDemo2 lruCacheDemo2 = new LRUCacheDemo2(3);
           lruCacheDemo2.put(1, 1);
           lruCacheDemo2.put(2, 2);
           lruCacheDemo2.put(3, 3);
           System.out.println(lruCacheDemo2.map.keySet());
   
           lruCacheDemo2.put(4, 4);
           System.out.println(lruCacheDemo2.map.keySet());
   
           lruCacheDemo2.put(3, 3);
           System.out.println(lruCacheDemo2.map.keySet());
           lruCacheDemo2.put(3, 3);
           System.out.println(lruCacheDemo2.map.keySet());
           lruCacheDemo2.put(3, 3);
           System.out.println(lruCacheDemo2.map.keySet());
   
           lruCacheDemo2.put(5, 5);
           System.out.println(lruCacheDemo2.map.keySet());
   
           lruCacheDemo2.put(6, 6);
           System.out.println(lruCacheDemo2.map.keySet());
       }
   ```

   结果如下:这个顺序是不对滴，因为这是 `HashMap` 中 `key` 的顺序，并不是 `DoubleLinkedList` 中 `key` 的顺序，但至少可以说明最近最少使用的数据已经被删除了

   ```
   [1, 2, 3]
   [2, 3, 4]
   [2, 3, 4]
   [2, 3, 4]
   [2, 3, 4]
   [3, 4, 5]
   [3, 5, 6]
   ```

完整代码

```java
package com.interview.demo.leetcode;

import java.util.HashMap;
import java.util.Map;

/**
 * @title: LRUCacheDemo2
 * @Author Wen
 * @Date: 28/9/2021 1:04 PM
 * @Version 1.0
 */
public class LRUCacheDemo2 {

    // 1. 构造一个Node节点作为数据载体
    class Node<K, V> {
        private K key;
        private V value;
        private Node prev;
        private Node next;

        public Node() {
            this.prev = this.next = null;
        }

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            this.prev = this.next = null;
        }
    }

    // 2. 构建一个虚拟的双向链表,里面存放的就是Node
    class DoubleLinkedList<K, V> {
        Node<K, V> head;
        Node<K, V> tail;


        public DoubleLinkedList() {
            this.head = new Node<>();
            this.tail = new Node<>();

            this.head.next = this.tail;
            this.tail.prev = this.head;
        }


        // 3. 添加到节点头部
        public void addHead(Node<K, V> node) {


            node.prev = head;
            node.next = head.next;
            head.next.prev = node;
            head.next = node;
        }

        // 4. 删除节点
        public void removeNode(Node<K, V> node) {

            node.next.prev = node.prev;
            node.prev.next = node.next;
            node.prev = null;
            node.next = null;
        }

        // 5. 获得最后一个节点
        public Node<K, V> getLast() {

            if (tail.prev == head) {
                return null;
            }
            return tail.prev;
        }
    }

    private int cacheSize;
    Map<Integer, Node<Integer, Integer>> map;
    DoubleLinkedList<Integer, Integer> doubleLinkedList;

    public LRUCacheDemo2(int cacheSize) {
        this.cacheSize = cacheSize; //缓存大小
        map = new HashMap<>();  //根据HashMap进行查找
        doubleLinkedList = new DoubleLinkedList<>();
    }

    public int get(int key) {
        if (!map.containsKey(key)) {
            return -1;
        }

        Node<Integer, Integer> node = map.get(key);
        doubleLinkedList.removeNode(node);
        doubleLinkedList.addHead(node);

        return node.value;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) { //判断当前key是否存在,存在则更新
            Node<Integer, Integer> node = map.get(key);
            node.value = value;
            map.put(key, node);
            doubleLinkedList.removeNode(node);
            doubleLinkedList.addHead(node);
        } else {

            if (map.size() == cacheSize) {  //判断当前map是否已经满了
                Node<Integer, Integer> last = doubleLinkedList.getLast();
                map.remove(last.key);
                doubleLinkedList.removeNode(last);

            }

            //新增Node节点
            Node<Integer, Integer> node = new Node<>(key, value);
            map.put(key, node);
            doubleLinkedList.addHead(node);
        }
    }

    public static void main(String[] args) {
        LRUCacheDemo2 lruCacheDemo2 = new LRUCacheDemo2(3);
        lruCacheDemo2.put(1, 1);
        lruCacheDemo2.put(2, 2);
        lruCacheDemo2.put(3, 3);
        System.out.println(lruCacheDemo2.map.keySet());

        lruCacheDemo2.put(4, 4);
        System.out.println(lruCacheDemo2.map.keySet());

        lruCacheDemo2.put(3, 3);
        System.out.println(lruCacheDemo2.map.keySet());
        lruCacheDemo2.put(3, 3);
        System.out.println(lruCacheDemo2.map.keySet());
        lruCacheDemo2.put(3, 3);
        System.out.println(lruCacheDemo2.map.keySet());

        lruCacheDemo2.put(5, 5);
        System.out.println(lruCacheDemo2.map.keySet());

        lruCacheDemo2.put(6, 6);
        System.out.println(lruCacheDemo2.map.keySet());
    }
}
```

