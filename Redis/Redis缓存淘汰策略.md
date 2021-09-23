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

