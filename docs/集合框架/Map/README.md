# Map



## HashMap 和 HashTable 的区别

1. **线程是否安全：** `HashMap` 是非线程安全的，`HashTable` 是线程安全的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰；
2. **效率：** 因为线程安全的问题，`HashMap` 要比 `HashTable` 效率高一点。另外，`HashTable` 基本被淘汰，不要在代码中使用它；
3. **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；`HashTable` 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
4. **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`HashTable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 `HashTable` 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小，后面会介绍到为什么是 2 的幂次方。
5. **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。`HashTable` 没有这样的机制。

**`HashMap` 中带有初始容量的构造函数：**

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

 public HashMap(int initialCapacity) {
      this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

下面这个方法保证了 `HashMap` 总是使用 2 的幂作为哈希表的大小。

```java
	/**
     * Returns a power of two size for the given target capacity.
     */
	static final int tableSizeFor(int cap) {
		int n = cap - 1;
		n |= n >>> 1;
		n |= n >>> 2;
		n |= n >>> 4;
		n |= n >>> 8;
        n |= n >>> 16;
		return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
	}
```

## HashMap 和 HashSet 的区别

`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

|               HashMap                |                           HashSet                            |
| :----------------------------------: | :----------------------------------------------------------: |
|          实现了 `Map` 接口           |                       实现 `Set` 接口                        |
|              存储键值对              |                          仅存储对象                          |
|    调用 `put()` 向 map 中添加元素    |              调用 `add()` 方法向 set 中添加元素              |
| HashMap 使用键（Key）计算 `hashcode` | HashSet 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以 `equals()` 方法用来判断对象的相等性 |



## HashMap 和 TreeMap 的区别

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

![](https://images.yingwai.top/picgo/20210831111303.png ':size=60%')

实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。

实现`SortMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。**相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。**

## HashMap 的底层实现

### JDK1.8 之前

JDK1.8 之前 `HashMap` 底层是 **数组和链表** 结合在一起使用也就是 **链表散列**。**HashMap 通过 key 的 hashCode 经过扰动函数处理过后得到 hash 值，然后通过 (n - 1) & hash 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。**

**所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。**

* [JDK 源码中 HashMap 的 hash 方法原理是什么？](https://www.zhihu.com/question/20733617)

**JDK 1.8 HashMap 的 hash 方法源码:**

JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。

```java
static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

JDK1.7：

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

所谓 **“拉链法”** 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

### JDK1.8 之后

相比于之前的版本， JDK1.8 之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

![](https://images.yingwai.top/picgo/20210831111359.png ':size=60%')

> TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

### HashMap 的长度为什么是2的幂次方

我们希望理想的情况是：任意一个Key落在数组中的位置是足够散列的，这样可以减少Hash碰撞的概率。

假设计算出的Hash值是足够散列的，由于Hash值是一个int类型的值，大部分情况下HashMap数组是不会那么长的。所以在有限的数组长度内，当然是取Hash值的低几位算是比较理想的散列方式。

> 我们已经知道hashMap的底层数据结构是数组+链表的结构，所以hashMap每次保存的数据都是分散保存在数组的各个index位置，而**为了存取效率达到最高，就要要求hashMap每次保存的时候尽量平均分散到数组的各个位置**，因为如果每个index位置都只有一个元素的话，不存在链表数据，这样可以避免每次存取都要遍历链表造成额外的时间成本开销；所以我们就想到通过hashMap对数组长度取余运算（`hashCode%size`），这样可以达到最大的平均分配；但是hashMap的作者想到了如果通过位运算来计算取余的话效率会比10进制的取余运算来的快，因为我们知道计算机的底层运算都是转化位二进制运算的；所以为了位运算的取余效果能到10进制的效果，作者推算出了如果容量为2的N次方的话，那么`hash&(length-1) == hash%length`；length转为二进制位一个1+N个0，length-1转为二进制位一个0+N个1；则任意的hash值和length-1做位运算，结构都是后面N个位的的二进制，因为length-1的二进制前面都是0，位运算后都是0，相当于舍弃了高位，保留了后面的N位，后面的N位刚好在0-length之间，也就是等于hash%length取余。

* [关于hashMap的容量为什么是2的幂次方的最详细解析_eleven的博客-CSDN博客](https://blog.csdn.net/LLF_1241352445/article/details/81321991)

设计算法首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方。**

```java
	00100100 10100101 11000100 00100101    // Hash值
&   00000000 00000000 00000000 00001111    // 16 - 1 = 15
----------------------------------
    00000000 00000000 00000000 00000101    // 高位全部归零，只保留末四位。
```

## 扩容机制

* [HashMap工作原理和扩容机制_Spground-CSDN博客_hashmap什么时候扩容](https://blog.csdn.net/u014532901/article/details/78936283)

* [JDK 1.8 中 HashMap 扩容](https://www.jianshu.com/p/bdfd5f98cc31)

## HashMap 多线程操作导致死循环问题

* [疫苗：Java HashMap的死循环 | 酷 壳 - CoolShell](https://coolshell.cn/articles/9606.html)

## HashMap 的几种常见遍历方式

* [HashMap 的 7 种遍历方式与性能分析！(强烈推荐)](https://mp.weixin.qq.com/s/Zz6mofCtmYpABDL1ap04ow)

## ConcurrentHashMap 和 HashTable 的区别

`ConcurrentHashMap` 和 `HashTable` 的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`HashTable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- 实现线程安全的方式（重要）：
  1. **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 对 `synchronized` 锁做了很多优化）** 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；
  2. **`HashTable`(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

**两者的对比图：**

**HashTable:**

![](https://images.yingwai.top/picgo/20210831111506.png ':size=70%')

**JDK1.7 的 ConcurrentHashMap：**

![](https://images.yingwai.top/picgo/20210831111521.jpg ':size=70%')

**JDK1.8 的 ConcurrentHashMap：**

![http://images.yingwai.top/picgo/20210106144801.png](http://images.yingwai.top/picgo/20210106144801.png ':size=70%')

JDK1.8 的 `ConcurrentHashMap` 不再是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。

## ConcurrentHashMap 线程安全的具体实现方式/底层具体实现

### JDK1.7

首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。

Segment 实现了 `ReentrantLock`,所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组。`Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁。

### JDK1.8

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）

`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。
