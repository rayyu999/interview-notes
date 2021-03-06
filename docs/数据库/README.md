# 数据库

以下大部分内容针对MySQL。

## 三大范式

* **第一范式：**每个列都不可以再拆分（字段不可分）

  解释：原子性，字段不可再分，否则就不是关系数据库；

* **第二范式：**在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分（有主键，非主键字段依赖主键）

  解释：唯一性，一个表只说明一个事物；

* **第三范式：**在第二范式的基础上，非主键列只依赖于主键，不依赖于其他主键（非主键字段不能相互依赖）

  解释：每列都与主键有直接关系，不存在传递依赖。



## 存储引擎

查看存储引擎命令：

```sql
SHOW ENGINES
```

在MySQL中，不需要在整个服务器中使用同一种存储引擎，针对具体的要求，可以对每一个表使用不同的存储引擎。Support列的值表示某种引擎是否能使用：YES表示可以使用、NO表示不能使用、DEFAULT表示该引擎为当前默认的存储引擎。下面来看一下其中几种常用的引擎。

### [各主要存储引擎对比](https://www.cnblogs.com/pengpengdeyuan/p/15001739.html)

|         功能         | InnoDB | MyISAM |           Memory            |
| :------------------: | :----: | :----: | :-------------------------: |
|       索引结构       |  B+树  |  B+树  | 默认Hash，也可以使用B树索引 |
|   备份/时间点恢复    |   是   |   是   |             是              |
|       聚集索引       |   是   |   否   |             否              |
|       压缩数据       |   是   |   是   |             否              |
|       加密数据       |   是   |   是   |             是              |
|       外键支持       |   是   |   否   |             否              |
|     全文搜索索引     |   是   |   是   |             否              |
| 地理空间数据类型支持 |   是   |   是   |             否              |
|       索引缓存       |   是   |   是   |             否              |
|       锁定粒度       |   行   |   表   |             表              |
|         MVCC         |   是   |   是   |             是              |
|       复制支持       |   是   |   是   |            限量             |
|       储存限制       |  64TB  | 256TB  |            内存             |

### 关于 MyISAM 和 InnoDB 的选择问题

大多数时候我们使用的都是 InnoDB 存储引擎，在某些读密集的情况下，使用 MyISAM 也是合适的。不过前提是你的项目不介意 MyISAM 不支持事务、崩溃恢复等缺点。



## 优化

### 如果一条sql语句特别慢，如何分析？

**偶尔很慢：**

1. 数据库在刷新脏页，例如 redo log 写满了需要同步到磁盘
2. 执行的时候，遇到锁，如表锁、行锁（可以用 `show processlist` 命令查看当前状态）

**一直很慢：**

1. 没有用上索引：
   * 该字段没有索引
   * 由于对字段进行运算、函数操作导致无法用索引
2. [数据库选错了索引](https://www.cnblogs.com/kubidemanong/p/10734045.html)（可以用 `explain` 命令查看执行计划，也可以在查询时用 `force index(a)` 来强制使用索引 a）



## SQL 语言

SQL语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

### 数据查询语言 DQL

数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE子句组成的查询块：

SELECT <字段名表>
FROM <表或视图名>
WHERE <查询条件>

### 数据操纵语言 DML

数据操纵语言DML主要有三种形式：

1. 插入：INSERT
2. 更新：UPDATE
3. 删除：DELETE

### 数据定义语言 DDL

用于定义和管理 SQL 数据库中的所有对象的语言：

1. 创建表：CREATE
2. 修改表：ALTER
3. 删除表：DROP
