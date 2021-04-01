---
title: MySQL 存储过程
date: 2017-11-21 19:26:35
tags: 
	- MySQL
categories: Database
---

## MySQL 存储过程

MySQL存储过程是一个存储在MySQL数据库中的一段SQL代码，类似于我们平常在程序语言中的一个自定义函数。其实说到底，SQL也是一种语言，它也可以定义函数，定时器等等。只不过它是直接操作的数据库中的数据。

平常能用到存储过程的机会不多，以前在支付公司的时候更别说直接操作存储过程了，这根本不可能。任何需要做的处理逻辑都需要在代码中处理，所以这次接到写存储过程的任务，我还是有点忐忑的，一个是自己根本不会。O(∩_∩)O哈！完成的时间还比较紧。不过有挑战就有机遇，这个是没有错的。
<!--more-->
整个一周下来，也算是对存储过程的时候有了一点小小的心得，所以做次记录，如果下次遇到，能提醒自己。
<!--more-->
### 定义存储过程

定义一个存储过程是比较简单的，类似与我们写一个Java方法，定义的语法为：

```sql
    DELIMITER // 
    CREATE PROCEDURE proc4(IN param1 INT,OUT param2 varchar(50),INOUT param3 varchar(50))
    BEGIN
    SELECT max(order_type) INTO param1 FROM cncsen.order_info;
    select @param1; 
    select * from order_info orderInfo where orderInfo.ORDER_TYPE=param1; 
    END //
    DELIMITER ;
```

现在我们逐句来解析下：

`DELIMITER` : 这个是声明一个结束符，可以这样认为，解析器找到之前默认是找到`;`就认为这句话结束，而我们用`DELIMITER`将原来的`;`改为`//`,这样解析器只有遇到`//`才会认为当前句子结束；

`CREATE` : 这个和创建表，创建数据库是一个意思；

`PROCEDURE` : 这个是指创建存储过程，类似于`TABLE`,`DATABASE`；

`proc4` ：这个的意思是存储过程名字；

`OUT / IN / INOUT` : 相当于权限定义，OUT 是指该参数可以当做出参，不能作为入参；IN 是指该参数为可以作为入参，不能作为出参；INOUT 是指该参数既可以作为出参，也可以作为入参；（入参指作为参数传进来；出参是指作为返回值，传给其他人用)；

`INT / VARCHAR ` : 这个是基本类型，相当于Java里面的预置基本类型。

`BEGIN ** END` : begin和end定义的是一个语句块，从哪里开始，到哪里结束。可以看到后面还有`//`，而它刚好就是我们之前定义的结束符号，到这里我们的定义也就完成了。最后当然还要将结束符号切换回`;`。

这里我们就定义好了一个存储过程，其实我们主要关注的是begin和end之间的事情，我们可以在这里写各种SQL语句。当然我们也可以在这里建立临时表，然后将表中的数据插入到其它表中。


### 存储过程实例

首先创建两个表，一个是user信息表，一个是user日志表。下面是两个表的建表语句：

`TBL_USER`

```sql
CREATE TABLE `tbl_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  `age` varchar(45) DEFAULT NULL,
  `address` varchar(45) DEFAULT NULL,
  `stu_id` int(11) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `last_update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8
```

`TBL_USER_LOG`

```sql
CREATE TABLE `tbl_user_log` (
  `user_id` int(11) DEFAULT NULL,
  `user_name` varchar(45) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(52) DEFAULT NULL,
  `stu_id` int(11) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `last_update_time` datetime DEFAULT NULL,
  `now_time` datetime DEFAULT NULL,
  `param_address` varchar(25) CHARACTER SET big5 DEFAULT NULL COMMENT '	',
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`),
  UNIQUE KEY `id_UNIQUE` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8

```

下面我们开始写一个存储过程，该存储过程很简单就是将`tbl_user`表的数据备份一份到`tbl_user_log`中比如：

```sql
DELIMITER //
CREATE PROCEDURE proc(IN start_date varchar(32),IN end_date varchar(32),
						IN name varchar(50),INOUT address varchar(50))
BEGIN
	INSERT INTO tbl_user_log(user_id,user_name,age,address,stu_id,create_time,last_update_time,now_time)
		SELECT id,name,age,address,stu_id,create_time,last_update_time,current_time 
        FROM tbl_user
        WHERE create_time>=@start_date
        AND create_time<@end_date
        AND name=@name;
END //
DELIMITER

```

如果你要执行这个存储过程：

```sql
set @start_date='2017-11-16 12:12:12';
set @end_date='2017-11-17 12:12:12';
set @name='';
set @address='';
CALL proc(@start_date,@end_date,@name,@address);
```

然后在`tbl_user_log`表中你就可以看到符合你的需求的数据了。


### Event 事件

我们经常需要做一些定时任务，在数据库中我们如果需要做一些定时任务，这个时候就需要用到Event了。

创建Event的语法为：

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
    EVENT
    [IF NOT EXISTS]
    event_name
    ON SCHEDULE schedule
    [ON COMPLETION [NOT] PRESERVE]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT 'string']
    DO event_body;

schedule:
    AT timestamp [+ INTERVAL interval] ...
  | EVERY interval
    [STARTS timestamp [+ INTERVAL interval] ...]
    [ENDS timestamp [+ INTERVAL interval] ...]

interval:
    quantity {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |
              WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |
              DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}
```

具体的字段详情就不一一解释了，如果需要详细了解可以[查看Event官网信息](https://dev.mysql.com/doc/refman/5.7/en/create-event.html)。

事件的执行需要打开数据库的配置：

```sql
SET GLOBAL event_scheduler = ON;
SET @@global.event_scheduler = ON;
SET GLOBAL event_scheduler = 1;
SET @@global.event_scheduler = 1;
```
具体详情可以[查看Event官方文档](https://dev.mysql.com/doc/refman/5.7/en/events-configuration.html)


### 游标

通常我们需要将查出的数据做一些处理，比如先对一个表进行select，然后再通过某个字段汇总，或者进行一些汇总后的总处理，这个时候如果我们需要遍历记录做处理，那么就需要用到游标了。说白了，游标就是用来遍历select之后的数据的。
游标的使用非常简单，我们看一个实例：

```sql
CREATE PROCEDURE curdemo()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE a CHAR(16);
  DECLARE b, c INT;
  DECLARE cur1 CURSOR FOR SELECT id,data FROM test.t1;
  DECLARE cur2 CURSOR FOR SELECT i FROM test.t2;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  OPEN cur1;
  OPEN cur2;

  read_loop: LOOP
    FETCH cur1 INTO a, b;
    FETCH cur2 INTO c;
    IF done THEN
      LEAVE read_loop;
    END IF;
    IF b < c THEN
      INSERT INTO test.t3 VALUES (a,b);
    ELSE
      INSERT INTO test.t3 VALUES (a,c);
    END IF;
  END LOOP;

  CLOSE cur1;
  CLOSE cur2;
END;
```

主要注意的无非就是`DECLARE cur1`,先声明;然后open，定义一个LOOP循环，read_loop 这里是循环名称。当然这里也定义了一个退出状态条件-done，最后CLOSE。
确实很简单。

定义个test表

```sql
CREATE TABLE `test` (
  `name` varchar(50) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

之后用游标先查出`tbl_user`的数据，然后插入到test表中：

```sql
DELIMITER |
CREATE EVENT event_sale_data
    ON SCHEDULE
      EVERY 1 HOUR
    DO
      BEGIN
      DECLARE done INT DEFAULT FALSE;
      DECLARE CITY_NAME VARCHAR(50);
      DECLARE cursor_user CURSOR FOR SELECT NAME FROM tbl_user;
      DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
      OPEN cursor_user;
      read_loop: LOOP
		FETCH cursor_user INTO user_name;
			IF done THEN
				LEAVE read_loop;
			END IF;
            INSERT INTO demo.test VALUE(user_name);
	  END LOOP;
    END |
DELIMITER ;
```

存储过程，游标，事件的基本使用就是这些了，非常简单，一开始做的时候很痛苦，也不知道该怎么下手。冷静下来，每一次不会，都是一个机会。

另外，我在event里面如果将select的数据作为变量传递到存储过程当中的时候，存储过程总是获取值失败，也就是参数没法传递到存储过程中，这个问题很奇怪，还在解决中。

还有一个特别有意思的，如果定义了查询-插入的存储过程，也就是将一个表的数据查询后插入到另一个表中，如果你在mysqlworkbench中直接调用call proc()，那么总是会将上一次查询的数据重复的插入的新表中，也就是重复插入，但是第一次的时候没有，只有手动调用两次以上的时候才会出现重复数据。如果是事件触发，却又没有重复数据插入。这个问题，我怀疑是缓存的问题。嗯，踩过的坑就这两个了。作为备忘，时刻提醒自己。