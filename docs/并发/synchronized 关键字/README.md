# `synchronized` 关键字

![](https://images.yingwai.top/picgo/20210901162458.jpg)

图片来源：https://www.zhihu.com/question/53826114

## 实现原理

`synchronized` 不论是修饰方法还是代码块，都是通过持有修饰对象的锁来实现同步，`synchronized` 的锁对象是存在锁对象的对象头的 MarkWord 中的。对象在内存中的布局如下：

![](https://images.yingwai.top/picgo/20210730180343.png)

已知对象是存放在堆内存中的，对象大致可以分为三个部分，分别是对象头、实例变量和填充字节：

- 对象头主要是由 MarkWord 和 KlassPoint（类型指针）组成，其中 KlassPoint 是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，MarkWord 用于存储对象自身的运行时数据。如果对象是数组对象，那么对象头占用3个字宽（Word），如果对象是非数组对象，那么对象头占用2个字宽。（1 word = 2 Byte = 16 bit）；
- 实例变量存储的是对象的属性信息，包括父类的属性信息，按照4字节对齐；
- 填充字符，因为虚拟机要求对象字节必须是8字节的整数倍，填充字符就是用于凑齐这个整数倍的。

### 在 JVM 中的实现原理

重量级锁对应的锁标志位是10，存储了指向重量级监视器锁的指针，在 Hotspot 中，对象的监视器（monitor）锁对象由 ObjectMonitor 对象实现（C++），其跟同步相关的数据结构如下：

```c
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



## [与 ReentrantLock 的区别](https://segmentfault.com/a/1190000039091031)

### 使用方式

Synchronized可以修饰实例方法，静态方法，代码块。**自动释放锁。**

ReentrantLock一般需要try catch finally语句，在try中获取锁，在finally释放锁。需要**手动释放锁。**

### 实现方式

Synchronized是重量级锁。重量级锁需要将线程从内核态和用户态来回**切换**。如：A线程切换到B线程，A线程需要保存当前现场，B线程切换也需要保存现场。这样做的**缺点是耗费系统资源**。

ReentrantLock是轻量级锁。**采用cas+volatile管理线程**，不需要线程切换切换，获取锁线程觉得自己肯定能成功，这是一种**乐观的思想**（可能失败）。

用一个形象例子来说明：比如您在看我这篇文章时，觉得“重量级锁”概念不是很明白，就立刻去翻看关于“重量级锁”的其他文章，过会儿回头再继续往下面看， 这种行为我们称为切换。保存现场的意思就是你大脑需要记住你跳跃的点然后继续阅读，如果文章篇幅大，你的大脑可能需要记忆越多的东西，会越耗费脑神经。同理，在轻量级锁中，你觉得“重量级锁”概念不是很明白，他不会立刻去翻看其他文章，他会坚持会儿继续看，如果实在不明白再去翻资料了。需要注意的是：这是两种不一样的思维方式，**前者是被动阻塞悲观锁，状态是block，后者是主动的阻塞乐观锁，状态是wait。**

### 公平和非公平

Synchronized只有非公平锁。

ReentrantLock提供公平和非公平两种锁，默认是非公平的。公平锁通过构造函数传递true表示。

用一个形象例子来说明：排队打饭，Synchronized允许插队，如果ReentrantLock是公平锁，就不许插队了。

### 可重入锁

Synchronized和ReentrantLock都是可重入的，Synchronized是本地方法是C++实现，而ReentrantLock是JUC包用Java实现。

用一个形象例子来说明：如下图：一个房中房，房里外各有一把锁，但只有唯一的钥匙可以开，拥有钥匙的人可以先进入门1，再进入门2，其中进入门2就是叫锁可重入了。

在ReentrantLock中，重入次数用整形state表示。进入1次递增1次，出来1次递减1次。

![](https://images.yingwai.top/picgo/20210901103750.png)

### 可中断的

Synchronized是不可中断的。

**ReentrantLock提供可中断和不可中断两种方式。**其中**lockInterruptibly**方法表示可中断，lock方法表示不可中断。

用一个形象例子来说明：你和女朋友一起去做核酸，女朋友排在前面，所以女朋友进门先做，你在门外排队等待过程中突然接到领导电话要回去修改bug，你现在有两种选择，1.不和女朋友打招呼，立即回去修改bug，2.等待女朋友做完核酸，进去和女朋友打个招呼，然后回去修改bug。这两种情况最终都会导致一个结果，你无法完成核酸，在这两种情况中，虽然你都被领导中断了，但第一种情况你立即反馈领导叫可中断，第二种情况是你为了不做单身狗，打个招呼再去修改bug，需要注意的是“打招呼”需要提前获取锁，也就是需要等待你女朋友做完核酸检测。

### 条件队列

Synchronized只有一个等待队列。

ReentrantLock中一把锁可以对应多个条件队列。通过newCondition表示。

用一个形象例子来说明：母鸡下蛋和捡蛋人对应生产者和消费者，母鸡产蛋后，捡蛋人需要被母鸡通知，母鸡产蛋过程中，其中捡蛋人就会入条件队列（等待队列）。捡蛋人捡蛋完成后，捡蛋人需要通知母鸡继续产蛋，捡蛋人捡蛋过程中，母鸡也需要加入条件队列等待。

**注意：**有几个概念需要说明下。同步队列，条件队列和等待队列。

* **同步队列**：多线程同时竞争一把锁失败被挂起的线程。

* **条件队列**：正在执行的线程调用await/wait，从同步队列加入的线程会进入条件队列。正在执行线程调用signal/signalAll/notify/notifyAll，会将条件队列一个线程或多个线程加入到同步队列。

* **等待队列**：和条件队列一个概念。

