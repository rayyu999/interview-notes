# `synchronized` 关键字



## 实现原理

`synchronized` 不论是修饰方法还是代码块，都是通过持有修饰对象的锁来实现同步，`synchronized` 的锁对象是存在锁对象的对象头的 MarkWord 中的。对象在内存中的布局如下：

![](https://images.yingwai.top/picgo/20210730180343.png)

已知对象是存放在堆内存中的，对象大致可以分为三个部分，分别是对象头、实例变量和填充字节：

- 对象头主要是由 MarkWord 和 KlassPoint（类型指针）组成，其中 KlassPoint 是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，MarkWord 用于存储对象自身的运行时数据。如果对象是数组对象，那么对象头占用3个字宽（Word），如果对象是非数组对象，那么对象头占用2个字宽。（1 word = 2 Byte = 16 bit）；
- 实例变量存储的是对象的属性信息，包括父类的属性信息，按照4字节对齐；
- 填充字符，因为虚拟机要求对象字节必须是8字节的整数倍，填充字符就是用于凑齐这个整数倍的。

### 在 JVM 中的实现原理

重量级锁对应的锁标志位是10，存储了指向重量级监视器锁的指针，在 Hotspot 中，对象的监视器（monitor）锁对象由 ObjectMonitor 对象实现（C++），其跟同步相关的数据结构如下：

```c++
ObjectMonitor() {
    _count        = 0; //用来记录该对象被线程获取锁的次数
    _waiters      = 0;
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
}
```

线程在获取锁的几个状态的转换：

![](https://images.yingwai.top/picgo/20210730180747.jpg)

线程的生命周期存在5个状态，start、running、waiting、blocking 和 dead。

对于一个 `synchronized` 修饰的方法（代码块）来说：

1. 当多个线程同时访问该方法，那么这些线程会先被放进`_EntryList`队列，此时线程处于 blocking 状态；
2. 当一个线程获取到了实例对象的监视器（monitor）锁，那么就可以进入 running 状态，执行方法，此时，ObjectMonitor 对象的`_owner`指向当前线程，`_count`加1表示当前对象锁被一个线程获取；
3. 当 running 状态的线程调用`wait()`方法，那么当前线程释放 monitor 对象，进入 waiting 状态，ObjectMonitor 对象的`_owner`变为null，`_count`减1，同时线程进入`_WaitSet`队列，直到有线程调用`notify()`方法唤醒该线程，则该线程重新获取 monitor 对象进入`_Owner`区；
4. 如果当前线程执行完毕，那么也释放 monitor 对象，进入waiting状态，ObjectMonitor 对象的`_owner`变为null，`_count`减1。



## 锁升级过程

![](https://images.yingwai.top/picgo/20210730180857.png)
