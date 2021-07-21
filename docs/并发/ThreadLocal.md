# ThreadLocal

![](http://images.yingwai.top/picgo/20210109174109.png)



## 使用场景

ThreadLocal 是线程的局部变量，这些变量只能在这个线程内被读写，在其他线程内是无法访问的。 ThreadLocal 定义的通常是与线程关联的私有静态字段（例如，用户ID或事务ID）。

变量有局部的还有全局的，局部变量没什么好说的，一涉及到全局，那自然就会出现多线程的安全问题，要保证多线程安全访问，不出现脏读脏写，那就要涉及到线程同步了。而 ThreadLocal 相当于提供了介于局部变量与全局变量中间的这样一种线程内部的全局变量。

使用场景：当我们只想在本身的线程内使用的变量，可以用 ThreadLocal 来实现，并且这些变量是和线程的生命周期密切相关的，线程结束，变量也就销毁了。

所以说 ThreadLocal 不是为了解决线程间的共享变量问题的，如果是多线程都需要访问的数据，那需要用全局变量加同步机制。

举几个例子说明一下：

1. 比如线程中处理一个非常复杂的业务，可能方法有很多，那么，使用 ThreadLocal 可以代替一些参数的显式传递；

2. 比如用来存储用户 Session。Session 的特性很适合 ThreadLocal ，因为 Session 之前当前会话周期内有效，会话结束便销毁。我们先笼统但不正确的分析一次 web 请求的过程：

   - 用户在浏览器中访问 web 页面；
   - 浏览器向服务器发起请求；
   - 服务器上的服务处理程序（例如tomcat）接收请求，并开启一个线程处理请求，期间会使用到 Session ；
   - 最后服务器将请求结果返回给客户端浏览器。

   从这个简单的访问过程我们看到正好这个 Session 是在处理一个用户会话过程中产生并使用的，如果单纯的理解一个用户的一次会话对应服务端一个独立的处理线程，那用 ThreadLocal 在存储 Session ,简直是再合适不过了。但是例如 tomcat 这类的服务器软件都是采用了线程池技术的，并不是严格意义上的一个会话对应一个线程。并不是说这种情况就不适合 ThreadLocal 了，而是要在每次请求进来时先清理掉之前的 Session ，一般可以用拦截器、过滤器来实现。

3. 在一些多线程的情况下，如果用线程同步的方式，当并发比较高的时候会影响性能，可以改为 ThreadLocal 的方式，例如高性能序列化框架 Kyro 就要用 ThreadLocal 来保证高性能和线程安全；

4. 还有像线程内上下文管理器、数据库连接等可以用到 ThreadLocal；



## 使用方式

ThreadLocal 的使用非常简单，最核心的操作就是四个：创建、创建并赋初始值、赋值、取值。

1. 创建

   ```java
   ThreadLocal<String> mLocal = new ThreadLocal<>();
   ```

2. 创建并赋初值。下面代码表示创建了一个 String 类型的 ThreadLocal 并且重写了 `initialValue` 方法，并返回初始字符串，之后调用 get() 方法获取的值便是` initialValue` 方法返回的值。

   ```java
   private static ThreadLocal<String> mLocal = new ThreadLocal<String>(){
               @Override
               protected String initialValue(){
                   return "init value";
               }
           };
   System.out.println(mLocal.get());
   ```

3. 设置值

   ```java
   mLocal.set("hello");
   ```

4. 取值

   ```java
   mLocal.get()
   ```



## 实现原理

首先 ThreadLocal 是一个泛型类，保证可以接受任何类型的对象。

因为一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 `ThreadLocalMap` 的静态内部类。而我们使用的 get()、set() 方法其实都是调用了这个 `ThreadLocalMap` 类对应的 get()、set() 方法。例如下面的 set 方法：

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

调用 ThreadLocal 的 set 方法时，首先获取到了当前线程，然后获取当前线程维护的 `ThreadLocalMap` 对象，最后在`ThreadLocalMap` 实例中添加上。如果 `ThreadLocalMap` 实例不存在则初始化并赋初始值。

这里看到 set 方法的第一个参数是 `this` ，`this`即指的是当前的 ThreadLocal 对象，会看上看的代码就是指的 mLocal 这个对象。而在 `ThreadLocalMap` 的 set 方法中会根据当前 ThreadLocal 对象实例，做一些操作和判断，最终实现赋值操作（具体参考源码）。

所以说，最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 ThreadLocal 上，ThreadLocal 可以理解为只是一个中间工具，传递了变量值。



## 内存泄露问题

实际上 `ThreadLocalMap` 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 `ThreadLocalMap` 中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

`ThreadLocalMap` 实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下。



- 使用 ThreadLocal 的时候，最好要声明为静态的；
- 使用完 ThreadLocal ，最好手动调用 remove() 方法，例如上面说到的 Session 的例子，如果不在拦截器或过滤器中处理，不仅可能出现内存泄漏问题，而且会影响业务逻辑。

