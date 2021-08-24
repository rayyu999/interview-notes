# 并发

## 并发编程注意事项

> 获取单例对象需要保证线程安全，其中的方法也要保证线程安全。

- 单例对象会被多线程共享，因此要保证它是线程安全的，它其中的方法都要保证是线程安全的。
- 工具类、资源驱动类、单例工厂类都要注意这个问题。

> 创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。
> 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。

- 使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者 “过度切换”的问题。

> 线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

- FixedThreadPool 和 SingleThreadPool：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
- CachedThreadPool 和 ScheduledThreadPool：允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

> SimpleDateFormat 是线程不安全的类，一般不要定义为static变量，如果定义为 static，必须加锁，或者使用 DateUtils 工具类。

- 如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar， DateTimeFormatter 代替 Simpledateformatter。

> 高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁; 能锁区块，就不要锁整个方法体; 能用对象锁，就不要用类锁。


对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造 成死锁。

- 线程一需要对表 A、B、C 依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是 A、B、C，否则可能出现死锁。

> 并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加 锁，要么在数据库层使用乐观锁，使用 version 作为更新依据。

- 如果每次访问冲突概率小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次 数不得小于 3 次。

> 多线程并行处理定时任务时，Timer 运行多个 TimerTask 时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用 ScheduledExecutorService 则没有这个问题。
>
> 使用 CountDownLatch 进行异步转同步操作，每个线程退出前必须调用 countDown 方法，线程执行代码注意 catch 异常，确保 countDown 方法可以执行，避免主线程无法执行至 await 方法，直到超时才返回结果。

- 请在try…finally语句里执行countDown方法，与关闭资源类似。

> 避免 Random 实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一 seed 导致的性能下降。

- Random 实例包括 java.util.Random 的实例或者 Math.random()实例。
- 在 JDK7 之后，可以直接使用 API ThreadLocalRandom。
- 在 JDK7 之前，可以把Random放在ThreadLocal里，只在本线程中使用。

> 通过双重检查锁(double-checked locking)实现延迟初始化的优 化问题隐患，只要不是特别老的JDK版本(1.4以下)，双检锁是没问题的。
>
> volatile 解决多线程内存不可见问题。
> 对于一写多读，是可以解决变量同步问题，但是如果多写，是无法解决线程安全问题的。

- 如果是 count++操作，使用如下类实现:

- - AtomicInteger count = new AtomicInteger();
  - count.addAndGet(1);

- 如果是 JDK8，推荐使用 LongAdder 对象，比 AtomicLong 性能更好(减少乐观锁的重试次数)。

> HashMap 在容量不够进行 resize 时由于高并发可能出现死链，导致 CPU 飙升，在开发过程中注意规避此风险。

- 开发程序的时候要预估使用量，根据使用量来设置初始值。

> ThreadLocal 无法解决共享对象的更新问题，ThreadLocal 对象建议使用 static 修饰。这个变量是针对一个线程内所有操作共有的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。

