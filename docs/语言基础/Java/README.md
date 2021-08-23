# Java 基础



## 基本数据类型

| 基本类型 | 大小   |        最小值         |           最大值           | 默认值  | 包装器类型 | 备注                                             |
| -------- | ------ | :-------------------: | :------------------------: | :-----: | :--------: | ------------------------------------------------ |
| boolean  | -      |           -           |             -              |  false  |  Boolean   | 表示一位的信息                                   |
| char     | 16-bit | Unicode 0（`\u0000`） | Unicode 2^16-1（`\uffff`） | 'u0000' | Character  |                                                  |
| byte     | 8-bit  |        `-128`         |           `127`            |    0    |    Byte    | 数字类型都用的补码表示，因此取值范围会少一位     |
| short    | 16-bit |        `-2^15`        |          `2^15-1`          |    0    |   Short    |                                                  |
| int      | 32-bit |        `-2^31`        |          `2^31-1`          |    0    |  Integer   |                                                  |
| long     | 64-bit |        `-2^63`        |          `2^63-1`          |   0L    |    Long    | L不分大小写，但小写容易与数字1混淆，最后使用大写 |
| float    | 32-bit |       `1.4E-45`       |       `3.4028235E38`       |   0f    |   Float    | 浮点数不能用来表示精确的值，如货币               |
| double   | 64-bit |      `4.9E-324`       |  `1.7976931348623157E308`  |   0d    |   Double   | double类型同样不能表示精确的值，如货币           |
| void     | -      |           -           |             -              |    -    |    Void    |                                                  |



## 泛型

Java 泛型（generics）是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

### 泛型使用

泛型一般有三种使用方式：泛型类、泛型接口、泛型方法。

#### 泛型类

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

如何实例化泛型类：

```java
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

#### 泛型接口

```java
public interface Generator<T> {
    public T method();
}
```

实现泛型接口，不指定类型：

```java
class GeneratorImpl<T> implements Generator<T>{
    @Override
    public T method() {
        return null;
    }
}
```

实现泛型接口，指定类型：

```java
class GeneratorImpl<T> implements Generator<String>{
    @Override
    public String method() {
        return "hello";
    }
}
```

#### 泛型方法

```java
public static < E > void printArray( E[] inputArray )
   {
         for ( E element : inputArray ){
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
```

使用：

```java
// 创建不同类型数组： Integer, Double 和 Character
Integer[] intArray = { 1, 2, 3 };
String[] stringArray = { "Hello", "World" };
printArray( intArray  );
printArray( stringArray  );
```

### [泛型通配符](https://juejin.cn/post/6844903917835419661)

**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 java 类型
- T (type) 表示具体的一个 java 类型
- K V (key value) 分别代表 java 键值中的 Key Value
- E (element) 代表 Element

> **T 和 ? 的区别：** `?` 和 `T` 都表示不确定的类型，区别在于我们可以对 `T` 进行操作，但是对 `?` 不行； `T` 是一个 确定的 类型，通常用于泛型类和泛型方法的定义，`?` 是一个 不确定 的类型，通常用于泛型方法的调用代码和形参，不能用于定义类和泛型方法。

### [泛型擦除](https://www.cnblogs.com/wuqinglong/p/9456193.html)

**Java 的泛型是伪泛型，这是因为 Java 在编译期间，所有的泛型信息都会被擦掉，这也就是通常所说类型擦除 。**

```java
List<Integer> list = new ArrayList<>();

list.add(12);
//这里直接添加会报错
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
//但是通过反射添加，是可以的
add.invoke(list, "kl");

System.out.println(list)
```

如在代码中定义`List<Object>`和`List<String>`等类型，在编译后都会变成`List`，JVM看到的只是`List`，而由泛型附加的类型信息对JVM是看不到的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法在运行时刻出现的类型转换异常的情况，类型擦除也是 Java 的泛型与 C++ 模板机制实现方式之间的重要区别。

**参考链接：**

* [Java 泛型，你了解类型擦除吗？](https://blog.csdn.net/briblue/article/details/76736356)



## 编译期与运行期

### 编译期

编译期是指把源码交给编译器编译成计算机可以执行的文件的过程，在Java中也就是把Java代码编成class文件的过程。编译期只是做了一些翻译功能，并没有把代码放在内存中运行起来，而只是把代码当成文本进行操作，比如检查错误。

在编译期，将java代码翻译为字节码文件的过程经过了四个步骤，**词法分析**，**语法分析**，**语义分析**，**代码生成**四个步骤。

![](https://images.yingwai.top/picgo/202108232311415.png)

**词法分析**

- 词法分析是编译的第一阶段。词法分析器的主要任务是读入源程序的输入字符，将它们组成词素，生成并输出一个词法单元序列，这个词法单元序列被输出到语法分析器进行语法分析。

**语法分析**

- 语法分析程序从扫描程序中获取记号形式的源代码，并完成定义程序结构的语法分析 （syntax analysis ），这与自然语言中句子的语法分析类似。语法分析定义了程序的结构元素及其关系。通常将语法分析的结果表示为语法树。

**语义分析**

- 程序的语义就是它的“意思”，它与语法或结构不同。程序的语义确定程序的运行，但是大多数的程序设计语言都具有在执行之前被确定而不易由语法表示和由分析程序分析的特征。这些特征被称作静态语义（static semantic），而语义分析程序的任务就是分析这样的语义，语义具有只有在程序执行时才能确定的特性，由于编译器不能执行程序，所以它不能由编译器来确定）。一般的程序设计语言的典型静态语义包括声明和类型检查。由语义分析程序计算的额外信息，它们通常是作为注释或“装 饰”增加到树中（还可将属性添加到符号表中）。

**代码生成**

- 代码生成器得到中间代码，并生成目标代码。



### 运行期

运行期是把编译后的文件交给计算机执行，直到程序运行结束。所谓运行期就把在磁盘中的代码放到内存中执行起来，在Java中把磁盘中的代码放到内存中就是类加载过程，类加载是运行期的开始部分。

