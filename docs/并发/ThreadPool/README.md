# ThreadPool

## 线程池的优势

线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

它的主要特点为：线程复用；控制最大并发数；管理线程。

1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
2. 提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## 线程池的使用

### 架构说明

Java中的线程池是通过 `Executor` 框架实现的，该框架中用到了 `Executor`、`Executors`、`ExecutorService`、`ThreadPoolExecutor` 这几个类：

![http://images.yingwai.top/picgo/20201209164831.bmp](http://images.yingwai.top/picgo/20201209164831.bmp)

### 编码实现

- `Executors.newFixedThreadPool(int nThreads)`：执行长期任务性能好，创建一个线程池，一池有n个固定的线程，有固定线程数的线程；
- `Executors.newSingleThreadExecutor()`：一个任务一个任务的执行，一池一线程；
- `Executors.newCachedThreadPool()`：执行很多短期异步任务，线程池根据需要创建新线程，但在先前构建的线程可用时将重用它们。可扩容，遇强则强。

```java
public class MyThreadPoolDemo {

    public static void main(String[] args) {
        //List list = new ArrayList();
        //List list = Arrays.asList("a","b");
        //固定数的线程池，一池五线程

//       ExecutorService threadPool =  Executors.newFixedThreadPool(5); //一个银行网点，5个受理业务的窗口
//       ExecutorService threadPool =  Executors.newSingleThreadExecutor(); //一个银行网点，1个受理业务的窗口
       ExecutorService threadPool =  Executors.newCachedThreadPool(); //一个银行网点，可扩展受理业务的窗口

        //10个顾客请求
        try {
            for (int i = 1; i <=10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName()+"\\t 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }
}
```

## ThreadPoolExecutor 底层原理

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                 0L, TimeUnit.MILLISECONDS,
                                 new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
                (new ThreadPoolExecutor(1, 1,
                            0L, TimeUnit.MILLISECONDS,
                            new LinkedBlockingQueue<Runnable>()));
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integre.MAX_VALUE,
                                 60L, TimeUnit.SECONDS,
                                 new SynchronousQueue<Runnable>());
}
```

### 线程池的几个重要参数

![http://images.yingwai.top/picgo/20201209171047.png](http://images.yingwai.top/picgo/20201209171047.png)

- `corePoolSize`：线程池中的常驻核心线程数
- `maximumPoolSize`：线程池中能够容纳同时执行的最大线程数，此值必须大于1
- `keepAliveTime`：多余的空闲线程的存活时间。当前池中线程数量超过`corePoolSize`时，当空闲时间达到`keepAliveTime`时，多余线程会被销毁直到只剩下`corePoolSize`个线程为止
- `unit`：`keepAliveTime`的单位
- `workQueue`：任务队列，被提交但尚未被执行的任务
- `threadFactory`：表示生成线程池中工作线程的线程工厂，用于创建线程，一般默认的即可
- `handler`：拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程数（`maximumPoolSize`）时如何来拒绝请求执行的`runnable`的策略

### 线程池的主要处理流程

![http://images.yingwai.top/picgo/20201210104107.bmp](http://images.yingwai.top/picgo/20201210104107.bmp)

`Executor` 接口定义了执行任务的 `execute()`，实现流程还要看是哪个线程池实现的，这里用比较典型的 `ThreadPoolExecutor` 举例。`ThreadPoolExecutor` 执行 `execute()` 方法分下面4种情况：

1. 如果当前运行的线程少于 `corePoolSize`，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。
2. 如果运行的线程等于或多于 `corePoolSize`，则将任务加入 `BlockingQueue`。
3. 如果无法将任务加入 `BlockingQueue`（队列已满），则创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。
4. 如果创建新线程将使当前运行的线程超出 `maximumPoolSize`，任务将被拒绝，并调用  `RejectedExecutionHandler.rejectedExecution()` 方法。

![](https://images.yingwai.top/picgo/20210824095725.png)

## 线程池的拒绝策略

等待队列已经排满了，再也塞不下新任务了同时，线程池中的max线程也达到了，无法继续为新任务服务。这个是时候我们就需要拒绝策略机制合理的处理这个问题。

### JDK 内置的拒绝策略

- **AbortPolicy（默认）**：直接抛出 `RejectedExecutionException` 异常阻止系统正常运行
- **CallerRunsPolicy**：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。
- **DiscardOldestPolicy**：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务。
- **DiscardPolicy**：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

以上内置拒绝策略均实现了 `RejectedExecutionHandle` 接口。

## 生产中如何设置合理参数

### 工作中单一的/固定数的/可变的三种创建线程池的方法哪个用的多？

一个都不用：

![http://images.yingwai.top/picgo/20201210104649.bmp](http://images.yingwai.top/picgo/20201210104649.bmp)

任务是**CPU密集型**的：一般定义 `corePoolSize` 为电脑cpu逻辑处理器数量+1（原因是即使当计算（CPU）密集型的线程偶尔由于页缺失故障或者其他原因而暂停时，这个“额外”的线程也能确保 CPU 的时钟周期不会被浪费）；

任务是**I/O密集型**的：一般定义 `corePoolSize` 为电脑 CPU 逻辑处理器数量的两倍。这是因为I/O密集型的情况下，常常会有比较多的线程因为等待 Socket 就绪而被阻塞，设置为两倍的目的就是当比较多的线程被阻塞时能更好地利用 CPU 资源。

> Java 如何看cpu逻辑处理器数量：`Runtime.getRuntime().availableProcessors();`

* [如何设置线程池参数？美团给出了一个让面试官虎躯一震的回答。](https://www.cnblogs.com/thisiswhy/p/12690630.html)

* [线程池中各个参数如何合理设置_riemann_的博客-CSDN博客_线程池参数设置原则](https://blog.csdn.net/riemann_/article/details/104704197)
