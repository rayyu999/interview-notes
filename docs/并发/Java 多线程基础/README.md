# Java 多线程基础

## 创建一个线程的方式

Java 中创建线程主要有四种方式。

### 继承`Thread`类创建线程类

1. 定义`Thread`类的子类，并重写该类的`run()`方法，该`run()`方法的方法体就代表了线程要完成的任务。因此把`run()`方法称为执行体；
2. 创建`Thread`子类的实例，即创建了线程对象；
3. 调用线程对象的`start()`方法来启动该线程。

```java
public class FirstThreadTest extends Thread {
	int i = 0;
	// 重写run()方法
	public void run() {
		for (; i < 100; i++){
		System.out.println(getName() + " " + i);
		}
	}
	
    public static void main(String[] args) {
		for (int i = 0; i < 100; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
			if (i == 20) {
				new FirstThreadTest().start();
				new FirstThreadTest().start();
			}
		}
	}
}
```

上述代码中`Thread.currentThread()`方法返回当前正在执行的线程对象，`getName()`方法返回调用该方法的线程的名字。

### 通过`Runnable`接口创建线程类

1. 定义`Runnable`接口的实现类，并重写该接口的`run()`方法，该`run()`方法的方法体同样是该线程的线程执行体；
2. 创建`Runnable`实现类的实例，并依此实例作为`Thread`的`target`来创建`Thread`对象，该`Thread`对象才是真正的线程对象；
3. 调用线程对象的`start()`方法来启动该线程。

```java
public class RunnableThreadTest implements Runnable {
	private int i;
	public void run() {
		for (i = 0; i < 100; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
		}
	}
    
	public static void main(String[] args){
		for (int i = 0; i < 100; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
			if (i == 20) {
				RunnableThreadTest rtt = new RunnableThreadTest();
				new Thread(rtt, "新线程1").start();
				new Thread(rtt, "新线程2").start();
			}
		}
	}
}
```

### 通过`Callable`和`Future`创建线程

1. 创建`Callable`接口的实现类，并实现`call()`方法，该`call()`方法将作为线程执行体，并且有返回值；
2. 创建`Callable`实现类的实例，使用`FutureTask`类来包装Callable对象，该`FutureTask`对象封装了该`Callable`对象的`call()`方法的返回值；
3. 使用`FutureTask`对象作为`Thread`对象的`target`创建并启动新线程；
4. 调用`FutureTask`对象的`get()`方法来获得子线程执行结束后的返回值。

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
 
public class CallableThreadTest implements Callable<Integer> {
 
	public static void main(String[] args) {
		CallableThreadTest ctt = new CallableThreadTest();
		FutureTask<Integer> ft = new FutureTask<>(ctt);
		for (int i = 0;i < 100; i++) {
			System.out.println(Thread.currentThread().getName() + "的循环变量i的值" + i);
			if (i == 20) {
				new Thread(ft, "有返回值的线程").start();
			}
		}
		try {
			System.out.println("子线程的返回值：" + ft.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
 
	}
 
	@Override
	public Integer call() throws Exception {
		int i = 0;
		for (; i < 100; i++) {
			System.out.println(Thread.currentThread().getName() + " " + i);
		}
		return i;
	}
}
```

### 使用线程池

1. 创建线程池对象；
2. 创建 `Runnable` 或 `Callable` 接口子类对象；
3. 提交 `Runnable` 或 `Callable` 接口子类对象；
4. 关闭线程池。

```java
public class ThreadPoolDemo {
	public static void main(String[] args) {
		//创建线程池对象
		ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
		//创建Runnable实例对象
		MyRunnable r = new MyRunnable();
        MyCallable c = new MyCallable();
		
		//自己创建线程对象的方式
		//Thread t = new Thread(r);
		//t.start(); ---> 调用MyRunnable中的run()
		
		//从线程池中获取线程对象,然后调用MyRunnable中的run()或MyCallable中的call()
		service.submit(r);
        // 执行完成后通过ft.get()可以获得call()方法的返回值
        FutureTask<Integer> ft = service.submit(c);
		//再获取个线程对象，调用MyRunnable中的run()
		service.submit(r);
        // 也可以使用execute()提交MyRunnable
		service.execute(r);
		//注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。将使用完的线程又归还到了线程池中

		//关闭线程池
		service.shutdown();
	}
}
```

### 四种方式比较

|           继承`Thread`类           |                 实现`Runnable`接口                 |                 实现`Callable`接口                 |                          使用线程池                          |
| :--------------------------------: | :------------------------------------------------: | :------------------------------------------------: | :----------------------------------------------------------: |
|             只能单继承             |                     可以多实现                     |                     可以多实现                     |                              -                               |
| 多个线程不能共享同一个`target`对象 | 多个线程可以共享同一个`target`对象、处理同一份资源 | 多个线程可以共享同一个`target`对象、处理同一份资源 |                              -                               |
|              无返回值              |                      无返回值                      |                      有返回值                      | 使用 `submit(Callable task)` 时返回执行结果或异常；<br />使用 `submit(Runnable task)` 时返回`null`或异常； |
|                 -                  |                         -                          |                         -                          |          可以解决线程生命周期开销问题和资源不足问题          |



**参考链接：**

* [线程池的 submit 和 execute 的区别](https://blog.csdn.net/guhong5153/article/details/71247266)
* [Java多线程之多个线程访问共享对象和数据的方式](https://blog.csdn.net/John8169/article/details/53193096)

