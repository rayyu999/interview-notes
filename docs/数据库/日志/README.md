# 日志

* [为了让你彻底弄懂 MySQL 事务日志，我通宵肝出了这份图解！-五分钟学算法](https://www.cxyxiaowu.com/10740.html)

* [面试官的灵魂一击：你懂 MySQL 事务日志吗？](https://www.jianshu.com/p/cf827567012f)

MySQL日志系统是数据库的重要组件，用于记录数据库的更新和修改。若数据库发生故障，可通过不同日志记录恢复数据库的原来数据。因此实际上日志系统直接决定着MySQL运行的鲁棒性和稳健性。

MySQL的日志有很多种，如二进制日志（binlog）、错误日志、查询日志、慢查询日志等，此外InnoDB存储引擎还提供了两种日志：redo log（重做日志）和undo log（回滚日志）。这里将重点针对InnoDB引擎，对重做日志、回滚日志和二进制日志这三种进行分析。

## Binlog（二进制日志）

### 概念

Binlog 是`逻辑日记`，用于记录数据库执行的写入操作（查询不记录）信息，`Server` 层记录和引擎层无关，并且是以追加方式进行写入，可以通过参数 `max_binlog_size` 设置每个 Binlog 文件的大小，文件大小达到设定值时会生成新的文件来保存日记。

逻辑日记可以简单理解为记录的就是sql语句。

### 作用

在实际应用中，主要用在两个场景：主从复制和数据恢复

- 主从复制场景：在 Master 主端开启 Binlog，将 Binlog 发生到各个 Slave 从端，Slave 从端重放 Binlog 从而达到主从数据一致

  [MySQL 主从复制及配置实现](https://segmentfault.com/a/1190000008942618)

- 数据恢复场景：通过使用 mysqlbinlog 工具来恢复数据



### 刷盘时机

Binlog 大致记录过程是先写 Binlog Buffer，然后通过刷盘时机，控制刷入 OS Buffer，控制 fsync() 进行写入 Binlog File 日记磁盘的过程。

对于 Binlog，MySQL 是通过参数 sync_binlog 参数来控制刷盘时机，取值是 0、1 和 N 三种值。0 表示由系统自行判断何时调用 sync() 写入磁盘；1 表示每次事务 commit 都要调用 fsync() 写入磁盘；N 表示每 N 个事务，才会调用 fsync() 写入磁盘。

![](https://images.yingwai.top/picgo/20210901112708.jpg)

### 记录格式

有三种格式：statement，row和mixed。

- **statement** 模式下，每一条会修改数据的sql都会记录在binlog中。不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。由于sql的执行是有上下文的，因此在保存的时候需要保存相关的信息，同时还有一些使用了函数之类的语句无法被记录复制。
- **row** 级别下，不记录sql语句上下文相关信息，仅保存哪条记录被修改。记录单元为每一行的改动，基本是可以全部记下来但是由于很多操作，会导致大量行的改动(比如alter table)，因此这种模式的文件保存的信息太多，日志量太大。
- **mixed**，一种折中的方案，普通操作使用statement记录，当无法使用statement的时候使用row。

此外，新版的MySQL中对row级别也做了一些优化，当表结构发生变化的时候，会记录语句而不是逐行记录。

MySQL 5.7.7 版本之前默认格式是 STATEMENT，版本之后默认是 ROW，可以通过参数 binlog-format 指定。

------

## 事务日志

### Redo log（重做日志）

#### 概念

重做日志（redo log）是InnoDB引擎层的日志，用来记录事务操作引起数据的变化，记录的是数据页的物理修改，这个页 “做了什么改动”。如：add xx记录 to Page1，向数据页Page1增加一个记录。

#### 作用

- 前滚操作：具备 `crash-safe` 能力，提供断电重启时解决事务丢失数据问题。

  [为什么 redo log 具有 crash-safe 的能力，是 binlog 无法替代的？](https://cloud.tencent.com/developer/article/1757612?from=article.detail.1699992)

- 提高性能：先写Redo log记录更新。当等到有空闲线程、内存不足、Redo log满了时`刷脏`。写 Redo log 是顺序写入，刷脏是随机写，节省的是随机写磁盘的 IO 消耗（转成顺序写），所以性能得到提升。此技术称为WAL技术：Write-Ahead Logging，它的关键点就是先写日记磁盘，再写数据磁盘。

#### 两阶段提交

更新内存后引擎层写 Redo log 将状态改成 prepare 为预提交第一阶段，Server 层写 Binlog，将状态改成 commit为提交第二阶段。两阶段提交可以确保 Binlog 和 Redo log 数据一致性。

* [什么是redo日志的"两阶段提交"](https://zhuanlan.zhihu.com/p/267377055)

##### 容灾恢复过程

MySQL的处理过程如下：

- 判断 redo log 是否完整，如果判断是完整（commit）的，直接用 Redo log 恢复
- 如果 redo log 只是预提交 prepare 但不是 commit 状态，这个时候就会去判断 binlog 是否完整，如果完整就提交 Redo log，用 Redo log 恢复，不完整就回滚事务，丢弃数据。

只有在 redo log 状态为 prepare 时，才会去检查 binlog 是否存在，否则只校验 redo log 是否是 commit 就可以啦。怎么检查 binlog：一个完整事务 binlog 结尾有固定的格式。

#### 刷盘时机

Undo log 的刷盘时机和 Redo log 差不多，Redo log 每次先写入 Redo Log Buffer 中，然后通过刷盘时机控制刷入 OS Buffer 时间和刷入日记磁盘的时间。



### Undo log（回滚日志）

#### 概念

Undo log是 `逻辑日记` 、回滚日记。保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读，这样发生错误时，根据执行 Undo log 就可以回滚到事务之前的数据状态。

#### 作用

- 回滚数据：当程序发生异常错误时等，根据执行 Undo log 就可以回滚到事务之前的数据状态，保证原子性，要么成功要么失败。
- MVCC 一致性视图：通过 Undo log 找到对应的数据版本号，是保证 MVCC 视图的一致性的必要条件。

