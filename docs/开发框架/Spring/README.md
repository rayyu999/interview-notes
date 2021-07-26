# Spring



## IoC

Spring的核心思想之一：**Inversion of Control，控制反转 IoC**。控制反转意思是**对象的创建交给外部容器完成**：

- Spring使用控制反转来实现对象不用在程序中写死
- 控制反转解决对象处理问题【把对象交给别人创建】

对象的对象之间的依赖关系Spring是通过**依赖注入，dependency injection**：

- Spring使用依赖注入来实现对象之间的依赖关系
- 在创建完对象之后，对象的关系处理就是依赖注入

> IoC的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。第一，资源集中管理，实现资源的可配置和易管理。第二，降低了使用资源双方的依赖程度，也就是我们说的耦合度。也就是说，甲方要达成某种目的不需要直接依赖乙方，它只需要达到的目的告诉第三方机构就可以了，比如甲方需要一双袜子，而乙方它卖一双袜子，它要把袜子卖出去，并不需要自己去直接找到一个卖家来完成袜子的卖出。它也只需要找第三方，告诉别人我要卖一双袜子。
>
> 在以上的描述中，诞生了两个专业词汇，依赖注入和控制反转所谓的依赖注入，则是，甲方开放接口，在它需要的时候，能够讲乙方传递进来(注入)所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

参考链接：

* [Spring IoC 有什么好处呢？](https://www.zhihu.com/question/23277575/answer/169698662)

### Spring IoC 的初始化过程

![](https://images.yingwai.top/picgo/20210107165847.png)



### Spring 如何解决循环依赖？

* [高频面试题：Spring 如何解决循环依赖？](https://zhuanlan.zhihu.com/p/84267654)
* [Spring 是如何解决循环依赖的？](https://segmentfault.com/a/1190000039091691)



## AOP

AOP（Aspect-Oriented Programming：面向切面编程）能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![](https://images.yingwai.top/picgo/20210107170016.jpg)

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

**参考链接：**

* [代理模式](https://snailclimb.gitee.io/javaguide/#/docs/java/basis/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3)
* [@Aspect 注解使用详解](https://blog.csdn.net/fz13768884254/article/details/83538709)

### Spring AOP 和 AspectJ AOP 区别

|     Spring AOP     |              AspectJ AOP              |
| :----------------: | :-----------------------------------: |
|     运行时增强     |              编译时增强               |
| 基于代理(Proxying) | 基于字节码操作(Bytecode Manipulation) |

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。



## Spring 事务

### Spring 事务中的事务传播行为

#### 支持当前事务的情况

- `TransactionDefinition.PROPAGATION_REQUIRED`： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- `TransactionDefinition.PROPAGATION_SUPPORTS`： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `TransactionDefinition.PROPAGATION_MANDATORY`： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

#### 不支持当前事务的情况

- `TransactionDefinition.PROPAGATION_REQUIRES_NEW`： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `TransactionDefinition.PROPAGATION_NEVER`： 以非事务方式运行，如果当前存在事务，则抛出异常。

#### 其他情况

- `TransactionDefinition.PROPAGATION_NESTED`： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。

