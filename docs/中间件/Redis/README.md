# Redis



## 命令

### SETNX

```shell
SETNX key value
```

如果指定的 key 不存在，则创建并为其设置值，然后返回状态码 1；如果指定的 key 存在，则直接返回 0。如果返回值为 1，代表获得该锁；此时其他进程再次尝试创建时，由于 key 已经存在，则都会返回 0 ，代表锁已经被占用。

使用该命令能实现分布式锁。



## 过期键清除策略

过期键清除策略有三种，分别是定时删除、定期删除和惰性删除。Redis过期键采用的是**定期删除+惰性删除**二者结合的方式进行删除的。

* **定时删除：**在设置键的过期时间的同时，创建一个定时器，让定时器在键的过期时间来临时，立即执行对键的删除操作。

* **定期删除：**每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。

* **惰性删除：**指使用的时候，发现Key过期了，此时再进行删除。

![](https://images.yingwai.top/picgo/20210726213612.png ':size=60%')



## 内存淘汰

Redis支持内存淘汰，配置参数`maxmemory_policy`决定了内存淘汰策略的策略。这个参数一共有8个枚举值。

![](https://images.yingwai.top/picgo/20210726213710.png ':size=65%')

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。

4.0 版本后增加以下两种：

7. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰

8. **allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key

### 内存淘汰算法

Redis用的是近似LRU算法，LRU算法需要一个双向链表来记录数据的最近被访问顺序，但是出于节省内存的考虑，Redis的LRU算法并非完整的实现。

Redis通过对少量键进行取样，然后和目前维持的淘汰池综合比较，回收其中的最久未被访问的键。通过调整每次回收时的采样数量`maxmemory-samples`，可以实现调整算法的精度。
