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



## Spring Bean

### Spring 中的 bean 的作用域有哪些？

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

[Spring学习（十五）Spring Bean 的5种作用域介绍_wangxin1248的博客-CSDN博客_spring中bean的作用域](https://blog.csdn.net/icarus_wang/article/details/51586776)

### Spring 中的单例 bean 的线程安全问题

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### `@Component` 和 `@Bean` 的区别

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

### 将一个类声明为 Spring 的 bean

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

### `@Autowired` 和 `@Resource` 的区别

- `@Autowird` 属于spring框架，默认使用类型(byType)进行注入

- `@Resource`是 JavaEE 自带的注解,根据ID进行注入的。`@Resource`的作用相当于`@Autowired`，只不过 `@Autowired` 按byType自动注入，而 `@Resource` 默认按byName自动注入罢了。

  `@Resource` 有两个属性是比较重要的，分是name和type，Spring将 `@Resource` 注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。

  `@Resource` 装配顺序：

  1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
  2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
  3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
  4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配

### Spring 中的 bean 生命周期

[Spring Bean的生命周期（非常详细）](https://www.cnblogs.com/zrtqsk/p/3735273.html)

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 与上面的类似，如果实现了其他 `.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![http://images.yingwai.top/picgo/20210108142928.jpg](http://images.yingwai.top/picgo/20210108142928.jpg ':size=70%')

![http://images.yingwai.top/picgo/20210108143539.png](http://images.yingwai.top/picgo/20210108143539.png)



## 循环依赖

有一个 A 对象，创建 A 的时候发现 A 对象依赖 B，然后去创建 B 对象的时候，又发现 B 对象依赖 C，然后去创建 C 对象的时候，又发现 C 对象依赖 A。这就是对象之间的循环依赖，而当程序执行这样的逻辑时，通常会陷入死循环。

![](https://images.yingwai.top/picgo/202108252227683.png)

> Spring IoC 容器会在运行时检测到**构造函数注入**循环引用，并抛出`BeanCurrentlyInCreationException`。
>
> 所以要避免构造函数注入，可以使用 setter 注入替代。

根据官方文档说明，Spring 会自动解决基于 setter 注入的循环依赖。

当然在咱们工作中现在都使用 `@Autowired` 注解来注入属性。

> `@Autowired` 是通过反射进行赋值。

### [Spring 是如何解决循环依赖的？](https://segmentfault.com/a/1190000039091691)

在 [Spring 单例 Bean 的创建](https://link.segmentfault.com/?url=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FqZ4xXlqpNzsdHkvFm02Yuw) 中介绍介绍了使用三级缓存。

> `singletonObjects`： 一级缓存，存储单例对象，Bean 已经实例化，初始化完成。
>
> `earlySingletonObjects`： 二级缓存，存储 `singletonObject`，这个 Bean 实例化了，还没有初始化。
>
> `singletonFactories`： 三级缓存，存储 `singletonFactory`。

Spring 使用三级缓存来解决循环依赖的问题，三级缓存分别是：

| 名称                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `singletonObjects`      | 一级缓存，存放完整的 Bean。                                  |
| `earlySingletonObjects` | 二级缓存，存放提前暴露的Bean，Bean 是不完整的，未完成属性注入和执行 init 方法。 |
| `singletonFactories`    | 三级缓存，存放的是 Bean 工厂，主要是生产 Bean，存放到二级缓存中。 |

大致流程：

![](https://images.yingwai.top/picgo/202108252126928.png)

**具体步骤**（这里只用 A，B 形成的循环依赖来举例）：

1. 实例化 A，此时 A 还未完成属性填充和初始化方法（@PostConstruct）的执行，A 只是一个半成品。
2. 为 A 创建一个 Bean 工厂，并放入到  singletonFactories 中。
3. 发现 A 需要注入 B 对象，但是一级、二级、三级缓存均未发现对象 B。
4. 实例化 B，此时 B 还未完成属性填充和初始化方法（@PostConstruct）的执行，B 只是一个半成品。
5. 为 B 创建一个 Bean 工厂，并放入到  singletonFactories 中。
6. 发现 B 需要注入 A 对象，此时在一级、二级未发现对象 A，但是在三级缓存中发现了对象 A，从三级缓存中得到对象 A，并将对象 A 放入二级缓存中，同时删除三级缓存中的对象 A。（注意，此时的 A 还是一个半成品，并没有完成属性填充和执行初始化方法）
7. 将对象 A 注入到对象 B 中。
8. 对象 B 完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除二级缓存中的对象 B。（此时对象 B 已经是一个成品）
9. 对象 A 得到对象 B，将对象 B 注入到对象 A 中。（对象 A 得到的是一个完整的对象 B）
10. 对象 A 完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除二级缓存中的对象 A。

#### [源码阅读](https://segmentfault.com/a/1190000023647227)

创建 Bean 的方法在`AbstractAutowireCapableBeanFactory::doCreateBean()`

1. **在构造`Bean`对象之后，将对象提前曝光到缓存中，这时候曝光的对象仅仅是构造完成，还没注入属性和初始化**

   ```java
   protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
       BeanWrapper instanceWrapper = null;
   	
       if (instanceWrapper == null) {
           // 1.实例化对象
           instanceWrapper = this.createBeanInstance(beanName, mbd, args);
       }
   
       final Object bean = instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null;
       Class<?> beanType = instanceWrapper != null ? instanceWrapper.getWrappedClass() : null;
      
       // 2.判断是否允许提前暴露对象，如果允许，则直接添加一个 ObjectFactory 到三级缓存
   	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
   				isSingletonCurrentlyInCreation(beanName));
       if (earlySingletonExposure) {
           // 添加三级缓存的方法详情在下方
           addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
       }
   
       // 3.填充属性
       this.populateBean(beanName, mbd, instanceWrapper);
       // 4.执行初始化方法，并创建代理
       exposedObject = initializeBean(beanName, exposedObject, mbd);
      
       return exposedObject;
   }
   ```

2. **提前曝光的对象被放入`Map<String, ObjectFactory<?>> singletonFactories`缓存中，这里并不是直接将`Bean`放入缓存，而是包装成`ObjectFactory`对象再放入**

   ```java
   public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
       protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
           Assert.notNull(singletonFactory, "Singleton factory must not be null");
           synchronized (this.singletonObjects) {
               // 判断一级缓存中不存在此对象
               if (!this.singletonObjects.containsKey(beanName)) {
                   // 添加至三级缓存
                   this.singletonFactories.put(beanName, singletonFactory);
                   // 确保二级缓存没有此对象
                   this.earlySingletonObjects.remove(beanName);
                   this.registeredSingletons.add(beanName);
               }
           }
       }
   }
   
   @FunctionalInterface
   public interface ObjectFactory<T> {
   	T getObject() throws BeansException;
   }
   ```

3. **为什么要包装一层`ObjectFactory`对象？**

   如果创建的`Bean`有对应的**代理**，那其他对象注入时，注入的应该是对应的**代理对象**；但是 Spring 无法提前知道这个对象是不是有**循环依赖**的情况，而**正常情况**下（没有循环依赖情况），Spring 都是在创建好**完成品 `Bean`** 之后才创建对应的**代理**。这时候 Spring 有两个选择：

   * 不管有没有**循环依赖**，都**提前**创建好**代理对象**，并将代理对象放入缓存，出现循环依赖时，其他对象直接就可以取到代理对象并注入。

   * 不提前创建好代理对象，在出现**循环依赖**被其他对象注入时，才实时生成**代理对象**。这样在没有循环依赖的情况下，`Bean`就可以按着 **Spring 设计原则**的步骤来创建。

   `Spring`选择了第二种方式，那怎么做到提前曝光对象而又不生成代理呢？
   Spring就是在对象外面包一层`ObjectFactory`，提前曝光的是`ObjectFactory`对象，在被注入时才在`ObjectFactory.getObject`方式内实时生成代理对象，并将生成好的代理对象放入到第二级缓存`Map<String, Object> earlySingletonObjects`。
   `addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));`：

   ```java
   public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
           implements AutowireCapableBeanFactory {
   
       protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
           Object exposedObject = bean;
           if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
               for (BeanPostProcessor bp : getBeanPostProcessors()) {
                   if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                       SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                       // 如果需要代理，这里会返回代理对象；否则返回原始对象
                       exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
                   }
               }
           }
           return exposedObject;
       }
   }
   ```

   为了防止对象在后面的**初始化（init）**时**重复代理**，在创建代理时，`earlyProxyReferences`缓存会记录已代理的对象。

   ```java
   public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
           implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
       private final Map<Object, Object> earlyProxyReferences = new ConcurrentHashMap<>(16);
               
       @Override
       public Object getEarlyBeanReference(Object bean, String beanName) {
           Object cacheKey = getCacheKey(bean.getClass(), beanName);
           // 记录已被代理的对象
           this.earlyProxyReferences.put(cacheKey, bean);
           return wrapIfNecessary(bean, beanName, cacheKey);
       }        
   }      
   ```

4. **注入属性和初始化**

   提前曝光之后：

   1. 通过`populateBean`方法注入属性，在注入其他`Bean`对象时，会先去缓存里取，如果缓存没有，就创建该对象并注入。
   2. 通过`initializeBean`方法初始化对象，包含创建代理。

   ```java
   public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
           implements AutowireCapableBeanFactory {
       protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
               throws BeanCreationException {
           ……
           // Initialize the bean instance.
           Object exposedObject = bean;
           try {
               populateBean(beanName, mbd, instanceWrapper);
               exposedObject = initializeBean(beanName, exposedObject, mbd);
           }
           catch (Throwable ex) {
               if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
                   throw (BeanCreationException) ex;
               }
               else {
                   throw new BeanCreationException(
                           mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
               }
           }
           ……
       }        
   }    
   // 获取要注入的对象
   public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
       protected Object getSingleton(String beanName, boolean allowEarlyReference) {
           // 一级缓存
           Object singletonObject = this.singletonObjects.get(beanName);
           if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
               synchronized (this.singletonObjects) {
                   // 二级缓存
                   singletonObject = this.earlySingletonObjects.get(beanName);
                   if (singletonObject == null && allowEarlyReference) {
                       // 三级缓存
                       ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                       if (singletonFactory != null) {
                           // Bean 工厂中获取 Bean
                           singletonObject = singletonFactory.getObject();
                           // 放入到二级缓存中 
                           this.earlySingletonObjects.put(beanName, singletonObject);
                           this.singletonFactories.remove(beanName);
                       }
                   }
               }
           }
           return singletonObject;
       }
   }
   ```

5. **放入已完成创建的单例缓存**

   在经历了以下步骤之后，最终通过`addSingleton`方法将最终生成的可用的`Bean`放入到`单例缓存`里。

   AbstractBeanFactory.doGetBean :point_right: DefaultSingletonBeanRegistry.getSingleton :point_right: AbstractAutowireCapableBeanFactory.createBean :point_right: AbstractAutowireCapableBeanFactory.doCreateBean :point_right: DefaultSingletonBeanRegistry.addSingleton

   ```java
   public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
   
       /** Cache of singleton objects: bean name to bean instance. */
       private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
   
       /** Cache of singleton factories: bean name to ObjectFactory. */
       private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
   
       /** Cache of early singleton objects: bean name to bean instance. */
       private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
   
       protected void addSingleton(String beanName, Object singletonObject) {
           synchronized (this.singletonObjects) {
               this.singletonObjects.put(beanName, singletonObject);
               this.singletonFactories.remove(beanName);
               this.earlySingletonObjects.remove(beanName);
               this.registeredSingletons.add(beanName);
           }
       }
   }
   ```



### [Spring 解决循环依赖必须要三级缓存吗？](https://juejin.cn/post/6882266649509298189)

**Spring 一开始提前暴露的并不是实例化的 Bean，而是将 Bean 包装起来的 ObjectFactory。为什么要这么做呢？**

这实际上涉及到 AOP，如果创建的 Bean 是有代理的，那么注入的就应该是代理 Bean，而不是原始的 Bean。但是 Spring 一开始并不知道 Bean 是否会有循环依赖，通常情况下（没有循环依赖的情况下），Spring 都会在完成填充属性，并且执行完初始化方法之后再为其创建代理。但是，如果出现了循环依赖的话，Spring 就不得不为其提前创建代理对象，否则注入的就是一个原始对象，而不是代理对象。因此，这里就涉及到应该在哪里提前创建代理对象？

通过上面的源码分析，可以知道**第三级缓存的目的是为了延迟代理对象的创建**，因为如果没有依赖循环的话，那么就不需要为其提前创建代理，可以将它延迟到初始化完成之后再创建。

而每次实例化完 Bean 之后就直接去创建代理对象，并添加到二级缓存中，直接忽略三级缓存。**测试结果是完全正常的**，**Spring 的初始化时间应该也是不会有太大的影响。**那为什么还要额外加一层缓存？

**如果要使用二级缓存解决循环依赖，意味着Bean在构造完后就创建代理对象，这样违背了Spring设计原则。Spring结合AOP跟Bean的生命周期，是在Bean创建完全之后通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来完成的，在这个后置处理的`postProcessAfterInitialization`方法中对初始化后的Bean完成AOP代理。如果出现了循环依赖，那没有办法，只有给Bean先创建代理，但是没有出现循环依赖的情况下，设计之初就是让Bean在生命周期的最后一步完成代理而不是在实例化后就立马完成代理。**



## Spring MVC

MVC 是一种设计模式，Spring MVC 是一款很优秀的 MVC 框架。Spring MVC 可以帮助我们进行更简洁的Web层的开发，并且它天生与 Spring 框架集成。Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。

### Spring MVC 工作原理

![http://images.yingwai.top/picgo/20210107190458.png](http://images.yingwai.top/picgo/20210107190458.png)

#### 流程说明

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler`来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

### `@RestController` vs `@Controller`

**`@Controller` 返回一个页面**

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。

![http://images.yingwai.top/picgo/20210107191319.png](http://images.yingwai.top/picgo/20210107191319.png)

**`@RestController` 返回JSON 或 XML 形式数据**

但`@RestController`只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。

![http://images.yingwai.top/picgo/20210107191335.png](http://images.yingwai.top/picgo/20210107191335.png)

**`@Controller + @ResponseBody` 返回JSON 或 XML 形式数据**

如果你需要在Spring4之前开发 RESTful Web服务的话，你需要使用`@Controller` 并结合`@ResponseBody`注解，也就是说`@Controller` +`@ResponseBody`= `@RestController`（Spring 4 之后新加的注解）。

> @ResponseBody 注解的作用是将 Controller 的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到HTTP 响应(Response)对象的 body 中，通常用来返回 JSON 或者 XML 数据，返回 JSON 数据的情况比较多。

![http://images.yingwai.top/picgo/20210107191355.png](http://images.yingwai.top/picgo/20210107191355.png)

**参考链接：**

* [SpringMVC源码之参数解析绑定原理](https://www.cnblogs.com/w-y-c-m/p/8443892.html)

