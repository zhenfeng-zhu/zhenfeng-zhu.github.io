---
date: "2018-12-01 15:54:53"
title: mysql
---

## MySQL基本架构

客户端

server层

- 连接器：管理连接，权限验证

- 查询缓存：命中规则，直接返回结果  8.0之后全部删除了这个模块

- 分析器：词法分析，语法分析

- 优化器：执行计划生成，索引选择

- 执行器：操作引擎，返回结果

存储引擎：存储数据，提供读写接口



数据库中的长连接指连接成功之后，如果客户端持续有请求，则一直使用同一个连接。短连接是指每次执行完很少的几次查询之后就断开连接，下次再重新建立。

如果全部使用长连接，会导致mysql内存涨的很快，可能出现OOM，因此要定期断开长连接，或者在执行一个比较大的操作之后，执行mysql_reset_connection重置一下。

## 日志系统

### redo log重做日志

redo log是innodb引擎特有的。物理日志，记录的是某个数据页上做了什么修改。循环写入。

WAL技术：Write-Ahead Logging：关键点就是先写日志，再写磁盘。当一条记录更新时，先把记录写到redolog中，更新到内存，这时这个更新操作就成功了。然后innodb引擎就会在适当的时候，将这个操作记录更新到磁盘中。因此在数据库异常重启的时候，之前的提交的记录不会丢失。

### binlog归档日志

binlog是server层实现的，所有的引擎都可以使用。binlog是逻辑日志，记录的是这个语句的原始逻辑。binlog是写到一定大小后，切换下一个，不会覆盖以前的日志。

因此一个update操作就是：

找到该行

判断数据页是否在内存中，如果是返回行数据，否则从磁盘读入到内存中。

将值进行更新，写入新行

新行更新到内存

写入redolog，处于prepare阶段

写入binlog

提交事务，处于commit阶段。

这个就是两阶段提交。

## 事务隔离

Isolation：隔离性

脏读，幻读，不可重复读

隔离的越严实，效率越低。

SQL的标准隔离级别：

读未提交：一个事务没提交的时候，它做的变更就能被别的事务看到

读提交：一个事务提交之后，做的变更才能被其他事务看到

可重复读：一个事务执行时看到的数据，总是跟这个事务启动时看到的数据时一致的。

串行化：顾名思义，对于同一行记录，写会加写锁，读也会加读锁。当读写锁冲突时，后访问的事务，必须等前一个事务完成。

在实现的时候，数据库会创建一个视图，访问的时候以视图的逻辑结果为准。

- 可重复读，这个视图是在事务启动时创建，整个事务存在期间都用这个视图。
- 读提交，这个视图在每个sql语句开始执行的时候创建
- 读未提交直接返回记录的最新值，没有视图的概念。
- 串行化是用加锁的方式。

mysql在每条记录更新的时候，都会记录一条回滚操作，记录上的最新值都可以通过回滚操作，得到前一个状态的值。当没有事务需要用到回滚日志时，就会被删除。所以不建议使用长事务，这样会占用存储空间和锁。

mysql启动事务的方式

- 显式启动：begin或者start transaction。配套的提交语句是commit，回滚语句是rollback。
- set autocommit=0这个命令会将这个线程的自动提交关闭。意味着如果只执行一个select语句，事务就启动了，而且不会自动关闭，除非主动执行commit或者rollback，或者断开连接。

因此一般set autocommit=1，打开显示启动的模式。















