# MyBatis



## MyBatis 与传统的 JDBC 的区别

> * JDBC 是 Java 提供的操作数据库的 API；
>   * 传统 JDBC 操作数据：
>     1. 需要连接数据库，注册驱动和数据库信息
>     2. 操作Connection，打开 Statement 对象
>     3. 通过Statement执行SQL， 返回结果到ResultSet对象
>     4. 使用ResultSet读取数据，然后通过代码转化为具体的POJO对象
>     5. 关闭数据库的相关资源
>   * 弊端：工作量相对较大，每次都要创建、关闭、获取
> * MyBatis 是一个支持普通 SQL 查询，存储过程和高级映射的持久层框架。
>   * MyBatis是对JDBC的封装，它可以基于xml或者注解的方式进行配置和原始映射，消除了几乎所有的JDBC代码和参数的手动设置和对结果集的封装。
>   * MyBatis可以使用简单的XML配置，将接口和Java的POJO（普通的Java对象）映射成数据库中的记录。

对比 JDBC，MyBatis 的优点如下：

1. **优化获取和释放：**MyBatis通过dataSource实现数据库连接池的配置，实现了隔离解耦，统一从dataSource中获取数据库连接，具体实现通过让用户配置应对变化。
2. **SQL统一管理，对数据库进行存取操作：**MyBatis将所有的SQL统一放在配置文件（XXXMapper.xml）中统一管理，不需要再次编译，而传统的JDBC的SQL语句分布在代码中，修改之后需要再次编译。可读性差，不便于维护。
3. **生成动态SQL语句：**MyBatis还可以通过标签动态的生成SQL语句。
4. **能够对结果集进行映射：**MyBatis可以直接将结果映射为自己需要的类型，如：Javabean，map，list等。而在使用JDBC时，我们要从返回的结果集ResultSet中获取结果封装为我们需要的类型。

