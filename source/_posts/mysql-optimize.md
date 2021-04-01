---
title: MySQL 优化
date: 2017-04-22 14:33:39
tags:
	- MySQL
categories: Database
---

## MySQL 优化基础

mysql 版本5.7

查询数据库版本: select @@version;

查询数据库的变量: 
	
```
	show variables;  -- 当前会话
	
	show session variables; -- 当前会话
	
	show global variables; -- 全局
```

当我设置`long_query_time`的时候用到了`set global long_query_time = 5`;但是查询的时候用到了`show global variables like '%long_query_time%'`这种情况下查到的值始终为10,中间的原因就是局部和全局的问题:
<!-- more -->
```
mysql> SHOW SESSION VARIABLES LIKE "long_query_time";
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+

mysql> SET @@GLOBAL.long_query_time = 1;

mysql> SHOW GLOBAL VARIABLES LIKE "long_query_time";
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+

mysql> SHOW VARIABLES LIKE "long_query_time";
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```
[](http://stackoverflow.com/questions/15541603/why-i-could-not-alter-the-variable-long-query-time-variable-at-runtime)

`long_query_time`记录的是时间的秒。如果设置为0秒，那么所有的sql都会被记录。

`show variables like 'log_queries_not_using_indexes';` 可以查看是否索引查询启用日志。所有没有索引的查询都会记录下来。

`show variables like 'slow_query_log_file';` 是否打开慢查询日志
`set global slow_query_log = on;` 打开慢查询日志

`show global variables like 'slow_query_log_file';` 查询慢查询日志文件路径
`set slow_query_log_file='/usr/local/var/mysql/logs/query_slow.log';` 设置慢查询日志文件路径


query_slow.log

```
# Time: 2017-04-22T07:02:51.980026Z
# User@Host: root[root] @ localhost [127.0.0.1]  Id:     3
# Query_time: 0.000179  Lock_time: 0.000074 Rows_sent: 2  Rows_examined: 2
SET timestamp=1492844571;
select * from store limit 10;
```
第一行是执行sql的时间；第二行是用户和执行的主机；第三行是sql总共执行时间，锁表，查出的row；第三行是执行时间的时间戳；第四行是执行的SQL语句。

`mysqldumpslow -t 3 logfile`:分析日志文件的前3条日志

`explain sql语句`:执行计划分析sql语句

