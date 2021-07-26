# 场景应用



## 缓存



## 分布式锁

在系统中修改已有数据时，需要先读取，然后进行修改保存，此时很容易遇到并发问题。由于修改和保存不是原子操作，在并发场景下，部分对数据的操作可能会丢失。在单服务器系统我们常用本地锁来避免并发带来的问题，然而，当服务采用集群方式部署时，本地锁无法在多个服务器之间生效，这时候保证数据的一致性就需要分布式锁来实现。

锁是计算机领域一个非常常见的概念，分布式锁也依赖存储组件，针对请求量的不同，可以选择Etcd、MySQL、Redis等。前两者可靠性更强，Redis性能更高。

![](https://images.yingwai.top/picgo/20210726221237.png)

### 实现

想要实现分布式锁，必须要求 Redis 有「互斥」的能力，我们可以使用 SETNX 命令，这个命令表示**SET** if **N**ot e**X**ists，即如果 key 不存在，才会设置它的值，否则什么也不做。

两个客户端进程可以执行这个命令，达到互斥，就可以实现一个分布式锁：

```shell
SETNX lock 1
(integer) 1		// 客户端1加锁成功

SETNX lock 1
(integer) 0		// 客户端2申请加锁，因为后到达，加锁失败
```

操作完成后还要及时释放锁：

```shell
DEL lock 		// 释放锁
(integer) 1
```

![](https://images.yingwai.top/picgo/20210726222219.jpg)



**参考链接：**

* [分布式锁的实现之 redis 篇](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)
* [深度剖析：Redis分布式锁到底安全吗？看完这篇文章彻底懂了！](http://kaito-kidd.com/2021/06/08/is-redis-distributed-lock-really-safe/)

