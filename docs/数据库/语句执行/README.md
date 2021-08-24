# 语句执行



## MySQL 基础架构

### 概览

下图是 MySQL 的一个简要架构图，从下图你可以很清晰的看到用户的 SQL 语句在 MySQL 内部是如何执行的。

![](https://images.yingwai.top/picgo/20210824101954.png)

上图涉及的一些组件：

- **连接器：**身份认证和权限相关（登录 MySQL 的时候）。
- **查询缓存：**执行查询语句的时候，会先查询缓存（MySQL 8.0 版本后移除，因为这个功能不太实用）。
- **分析器：** 没有命中缓存的话，SQL 语句就会经过分析器，分析器说白了就是要先看你的 SQL 语句要干嘛，再检查你的 SQL 语句语法是否正确。
- **优化器：**按照 MySQL 认为最优的方案去执行。
- **执行器：**执行语句，然后从存储引擎返回数据。

简单来说 MySQL 主要分为 Server 层和存储引擎层：

- **Server 层**：主要包括连接器、查询缓存、分析器、优化器、执行器等，所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图，函数等，还有一个通用的日志模块 binglog 日志模块。
- **存储引擎**： 主要负责数据的存储和读取，采用可以替换的插件式架构，支持 InnoDB、MyISAM、Memory 等多个存储引擎，其中 InnoDB 引擎有自有的日志模块 redolog 模块。**现在最常用的存储引擎是 InnoDB，它从 MySQL 5.5.5 版本开始就被当做默认存储引擎了。**



## SQL 关键字执行顺序

`from` -> `on` -> `join` -> `where` -> `group by`（开始使用select中的别名，后面的语句中都可以使用别名）-> `sum`、`count`、`max`、`avg` -> `having` -> `select` -> `distinct` -> `order by` -> `limit`

### 关键词含义

- **from**：需要从哪个数据表检索数据

- **on**：关联条件

- **join**：对需要关联查询的表进行关联

   关联查询时，数据库会选择一个驱动表，然后用此表的记录去关联其他表
  ​ left join一般以左表为驱动表（right join一般为右表）,inner join一般以结果集少的表为驱动表,left join某些情况下会被查询优化器优化为inner join

  - 驱动表选择原则：在对最终结果集没影响的前提下，优先选择结果集最少的那张表作为驱动表
  - 在**使用索引关联**的情况下，有`Index Nested-Loop join`和`Batched Key Access join`两种算法
  - 在**未使用索引关联**的情况下，有`Simple Nested-Loop join`和`Block Nested-Loop join`两种算法
  - `Simple Nested-Loop join`（SNLJ，简单嵌套循环连接）算法：根据on条件，从驱动表取一条数据，然后全表扫面被驱动表，将符合条件的记录放入最终结果集中。这样驱动表的每条记录都伴随着被驱动表的一次全表扫描
    - 匹配次数：外层表行数*内层表行数
  - `Index Nested-Loop Join`（INLJ，索引嵌套循环连接）算法：索引嵌套循环连接是基于索引进行连接的算法，索引是基于内层表的，通过外层表匹配条件直接与内层表索引进行匹配，避免和内层表的每条记录进行比较， 从而利用索引的查询减少了对内层表的匹配次数
    - 匹配次数：外层表行数*内层表索引高度
  - `Block Nested-Loop Join`（BNLJ，缓存块嵌套循环连接）算法：缓存块嵌套循环连接通过一次性缓存多条数据，把参与查询的列缓存到Join Buffer 里，然后拿join buffer里的数据批量与内层表的数据进行匹配，从而减少了内层循环的次数（遍历一次内层表就可以批量匹配一次Join Buffer里面的外层表数据）。
    当不使用`Index Nested-Loop Join`的时候，默认使用`Block Nested-Loop Join`
  - `Batched Key Access join`（BKAJ）算法：和SNLJ算法类似，但用于被join表上有索引可以利用，那么在行提交给被join的表之前，对这些行按照索引字段进行排序，因此减少了随机IO，排序这才是两者最大的区别，但是如果被join的表没用索引呢？那就使用BNLJ了
  - 什么是`Join Buffer`?
    - `Join Buffer`会缓存所有参与查询的列而不是只有Join的列。
    - 可以通过调整`join_buffer_size`缓存大小
    - `join_buffer_size`的默认值是256K，`join_buffer_size`的最大值在`MySQL 5.1.22`版本前是`4G`，而之后的版本才能在64位操作系统下申请大于`4G`的`Join Buffer`空间。
    - 使用`Block Nested-Loop Join`算法需要开启优化器管理配置的`optimizer_switch`的设置`block_nested_loop`为`on`，默认为开启。
  - 在选择Join算法时，会有优先级，理论上会优先判断能否使用INLJ、BNLJ：
    Index Nested-LoopJoin > Block Nested-Loop Join > Simple Nested-Loop Join
  - 注：可以使用explain查找驱动表，结果的第一张表即为驱动表，但执行计划在真正执行时可能发生改变

- **where**：过滤表中数据的条件

  - 执行顺序：自下而上、从右到左
  - 注：对数据库记录生效，无法对聚合结果生效，可以过滤掉最大数量记录的条件必须写在where子句末尾，不能使用聚合函数（sum、count、max、avg）

- **group by**：如何将上面过滤出的数据分组

  - 执行顺序：从左往右
  - 注：尽量在group by之前使用where过滤，避免之后使用having过滤

- **avg**：求平均值

- **having**：对上面已经分组的数据进行过滤的条件

  - 注：对聚合结果过滤，因此很耗资源，可以使用聚合函数

  - 例：筛选统计人口数量大于100W的地区

     select region, sum(population), sum(area) from bbc group by region having sum(population)>1000000，不能用where筛选超过100W的地区，因为不存在这样的一条记录

- **select**：查看结果集中的哪个列或列的计算结果

- **distinct**：对结果集重复值去重

- **order by**：按照什么样的顺序来查看返回的数据

  - 执行顺序：从左到右
  - 注：很耗资源

- **limit**：截取出目标页数据
