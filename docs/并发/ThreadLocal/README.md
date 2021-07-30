# ThreadLocal

ThreadLocal 是一个关于创建线程局部变量的类，其作用主要是做数据隔离，填充的数据只属于当前线程，变量的数据对别的线程而言是相对隔离的，在多线程环境下防止自己的变量被其它线程篡改。归纳下来有两类用途：

* **保存线程上下文信息，在任意需要的地方可以获取**，比如：
  * 每个请求怎么把一串后续关联起来，就可以用 ThreadLocal 进行 set，在后续的任意需要记录日志的方法里面进行get获取到请求id，从而把整个请求串起来；
  * Spring 的事务管理，用 ThreadLocal 存储 Connection，从而各个 DAO 可以获取同一 Connection，可以进行事务回滚，提交等操作。
* [线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失](https://juejin.cn/post/6844903870997479437)



## 用法

* 创建，支持泛型：

  ```java
  ThreadLocal<String> mLocal = new ThreadLocal<>();
  ```

* 创建并赋初值（阿里规范建议使用 `static` 修饰），下面代码表示创建了一个 String 类型的 ThreadLocal 并且重写了 initialValue 方法，并返回初始字符串，之后调用 get() 方法获取的值便是initialValue 方法返回的值：

  ```java
  private static ThreadLocal<String> mLocal = new ThreadLocal<String>(){
              @Override
              protected String initialValue(){
                  return "init value";
              }
          };
  System.out.println(mLocal.get());
  ```

* 设置值：

  ```java
  mLocal.set("hello");
  ```

* 取值：

  ```java
  mLocal.get()
  ```



### 使用 `static` 修饰

ThreadLocal 一般会采用 static 修饰，这样做既有好处也有坏处。好处是它一定程度上可以避免错误，至少它可以避免重复创建TSO（Thread Specific Object，即 ThreadLocal所关联的对象）所导致的浪费；坏处是这样做可能正好形成内存泄露所需的条件。



## 原理

首先 ThreadLocal 是一个泛型类，保证可以接受任何类型的对象。

因为一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap，而是 ThreadLocal 实现的一个叫做 `ThreadLocalMap` 的静态内部类。而我们使用的 `get()`、`set()` 方法其实都是调用了这个 `ThreadLocalMap` 类对应的 `get()`、`set()` 方法。最终的变量是放在了当前线程的 ThreadLocalMap 中，并不是存在 ThreadLocal 上，ThreadLocal 可以理解为只是一个中间工具，传递了变量值。

![](https://images.yingwai.top/picgo/20210730112815.jpg)

### [源码解析](https://www.jianshu.com/p/c64f06f0823b)

#### ThreadLocal

ThreadLocal 类定义如下：

```java
public class ThreadLocal<T> {

    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
        new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
    
    public ThreadLocal() {
    }
}
```

ThreadLocal 通过 `threadLocalHashCode` 来标识每一个 ThreadLocal 的唯一性。`threadLocalHashCode` 通过 CAS 操作进行更新，每次 hash 操作的增量为 `0x61c88647`。

`set()` 方法：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

通过 `Thread.currentThread()` 方法获取了当前的线程引用，并传给了 `getMap(Thread)` 方法获取一个 ThreadLocalMap 的实例。继续跟进 `getMap(Thread)` 方法：

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

可以看到 `getMap(Thread)` 方法直接返回 Thread 实例的成员变量 `threadLocals`。它的定义在 Thread 内部，访问级别为 package 级别：

```java
public class Thread implements Runnable {
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
    
}
```

到了这里可以看出，每个 Thread 里面都有一个 `ThreadLocal.ThreadLocalMap` 成员变量，也就是说每个线程通过 `ThreadLocal.ThreadLocalMap` 与 ThreadLocal 相绑定，这样可以确保每个线程访问到的 thread-local variable 都是本线程的。

获取了 ThreadLocalMap 实例以后，如果它不为空则调用 `ThreadLocal.ThreadLocalMap` 的 set 方法设值；若为空则调用 ThreadLocal 的 `createMap` 方法 new 一个 ThreadLocalMap 实例并赋给 `Thread.threadLocals`。

ThreadLocal 的 `createMap()` 方法的源码如下：

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

ThreadLocal 的 `get()` 方法，源码如下：

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```

通过 `Thread.currentThread()` 方法获取了当前的线程引用，并传给了 `getMap(Thread)` 方法获取一个 ThreadLocalMap 的实例。继续跟进 `setInitialValue()` 方法：

```java
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

首先调用 `initialValue()` 方法来初始化，然后 通过 `Thread.currentThread()` 方法获取了当前的线程引用，并传给了 `getMap(Thread)` 方法获取一个 ThreadLocalMap 的实例，并将 初始化值存到 ThreadLocalMap 中。

`initialValue()` 源码如下：

```java
protected T initialValue() {
    return null;
}
```

#### ThreadLocalMap

ThreadLocalMap 是 ThreadLocal 的静态内部类，源码如下：

```java
public class ThreadLocal<T> {

    static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The initial capacity -- MUST be a power of two.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;

        /**
         * The number of entries in the table.
         */
        private int size = 0;

        /**
         * The next size value at which to resize.
         */
        private int threshold; // Default to 0

        ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
    }
    
}
```

其中 `INITIAL_CAPACITY` 代表这个 Map 的初始容量；`table` 是一个 `Entry` 类型的数组，用于存储数据；`size` 代表表中的存储数目；`threshold` 代表需要扩容时对应 size 的阈值。

Entry 类是 ThreadLocalMap 的静态内部类，用于存储数据。它继承了 `WeakReference<ThreadLocal<?>>`，即每个 Entry 对象都有一个 ThreadLocal 的**弱引用**（作为 key），这是为了防止内存泄露。一旦线程结束，key 变为一个不可达的对象，这个 Entry 就可以被 GC 了。

接下来是 ThreadLocalMap 的 `set()` 方法的实现：

```java
private void set(ThreadLocal key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

ThreadLocal 的 `get()` 方法会调用 ThreadLocalMap 的 `getEntry(ThreadLocal key)`，其源码如下：

```java
private Entry getEntry(ThreadLocal key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```



## 内存泄露问题

实际上 `ThreadLocalMap` 中使用的 `key` 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

![](https://images.yingwai.top/picgo/20210730114619.png)

如果一个 ThreadLocal 没有外部强引用来引用它，那么系统 GC 的时候，这个 ThreadLocal 势必会被回收，这样一来， ThreadLocalMap 中就会出现 `key` 为 `null` 的 `Entry`，就没有办法访问这些 `key` 为 `null` 的 `Entry` 的 `value`，如果当前线程再迟迟不结束的话，这些 `key` 为 `null` 的 `Entry` 的 `value` 就会一直存在一条强引用链：`Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value` 永远无法回收，造成内存泄漏。

`ThreadLocalMap` 实现中已经考虑了这种情况，**在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 `key` 为 `null` 的记录。**但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用 `static` 的 `ThreadLocal`，延长了`ThreadLocal` 的生命周期，可能导致的内存泄漏；
- 使用完 ThreadLocal ，最好手动调用 `remove()` 方法，例如上面说到的 Session 的例子，如果不在拦截器或过滤器中处理，不仅可能出现内存泄漏问题，而且会影响业务逻辑。

### 为什么使用弱引用

分两种情况讨论：

- **key 使用强引用**：引用的 `ThreadLocal` 的对象被回收了，但是 `ThreadLocalMap` 还持有 `ThreadLocal` 的强引用，如果没有手动删除，`ThreadLocal` 不会被回收，导致 `Entry` 内存泄漏。
- **key 使用弱引用**：引用的 `ThreadLocal` 的对象被回收了，由于 `ThreadLocalMap` 持有 `ThreadLocal` 的弱引用，即使没有手动删除，`ThreadLocal` 也会被回收。`value` 在下一次`ThreadLocalMap` 调用 `set()`,`get()`，`remove()` 的时候会被清除。

比较两种情况，可以发现：由于`ThreadLocalMap`的生命周期跟 `Thread` 一样长，如果都没有手动删除对应 `key`，都会导致内存泄漏，但是使用弱引用可以多一层保障：**弱引用 `ThreadLocal` 不会内存泄漏，对应的 `value` 在下一次 `ThreadLocalMap` 调用 `set()`,`get()`,`remove()` 的时候会被清除**。

因此，`ThreadLocal` 内存泄漏的根源是：由于 `ThreadLocalMap` 的生命周期跟 `Thread` 一样长，如果没有手动删除对应 `key` 就会导致内存泄漏，而不是因为弱引用。
