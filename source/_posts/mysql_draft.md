---
title: 数据库常用函数汇总
date: 2017-04-21 00:00:08
tags: 
	- MySQL
categories: Database
---

## MySQL 常用函数和语句

### DDL语句

修改表的某个字段定义:`alter table tableName modify clounmnName column_definition [first|After col_name]`

增加表字段:`alter table tablename add column column_definition [first|after col_name]`

eg:

```
alter table emp add column age int(3);

alter table emp add birth date after ename;
```
删除表字段:`alter table tablename drop column col_name`
<!--more-->
修改字段名:`alter table tablename change [column] old_col_name column_definition [First|after col_name]`

eg:

```alter table emp change age age1 int(4)```

修改表名称`alter table tablename rename [to new tablename]`

按照sal排序后查询从第二条记录开始的3条记录`limit`:`select * from emp order by sal limit 1,3.`

```
select concat('drop table test1',table_name,';') and table_name like 'tmp%'
```
`bit`类型的数据列用`bin(),hex()`来显示

mysql 只给表中的第一个`timestamp`类型设置默认值为系统日期，如果有第二个timestamp类型，则默认为0值。规定`timestamp`类型字段只能有一列的默认值为`current_timestamp`;_timestamp_ 和**时区**有关，插入的时候先转换成本地时区后存放，从数据库取出来时也同样需要将日期转换成本地时区后显示。

查看时区: `show variables like 'time_zone'`

表中的第一个timestamp自动设置为系统时间。如果插入时为null，该列自动设置为当前的日期和时间


### 函数

#### 基础函数

`LPAD(str,n,pad); RPAD(str,n,pad)`:用字符串pad对str最左边和最右边进行填充，知道长度为n个长度为止。


`LTRIM(str); RTRIM(str)`:去掉字符串str左侧和右侧空格

`REPEAT(str,n)`:返回str重复n次

`REPLACE（str,a,b)`: 用字符串b替换字符串str中所有出现的字符串

`STRCMP(s1,s2)`:比较s1，s2的ASCII码值得大小

`TRIM(str)`: 去掉str左右两端的空格

`SUBSTRING(str,x,y)`:返回从字符串str中的第x位置起y个字符长度的子串

`COUNT(*),COUNT(id)`: 如果为\*，就是返回总行数；如果只是某个列，特别注意如果该列有值为null，那么count不计数

#### 数值函数

`ABS(x)`函数:返回x的绝对值

`CEIL(x)`函数:返回大于x的最小整数

`FLOOR(x)`函数:返回小于x的最大整数，和CEIL的用法刚好相反

`MOD(x,y)`:返回x/y的模

`RAND()`函数:返回0~1内的随机值

`ROUND(x,y)`函数:返回参数x的四舍五入的有y位小数的值

`TRUNCATE(x,y)`函数:返回数字x截断为y位小数的结果

#### 日期与时间函数

`curdate()`:返回当前日期，只包含年月日

`curtime()`:返回当前时间，只包含时分秒

`now()`:返回当前的日期和时间，年月日时分秒全都包含

`UNIX_timestamp(date)`:返回日期date的UNIX时间戳，Unix_timestamp(now())

`FROM_unixtime(unixtime)`:返回unixtime时间戳的日期值，和unix_timestamp(date)互为逆操作

`week(date),year(date)`:前者返回所给的日期是一年的第几周，后者返回所给的日期是哪一年

`hour(time)和minute(time)`:前者返回所给时间的小时，后者返回所给时间的分钟
比如当前时间为23:34,selec hour(curtime()),minute(curtime())==>23  34

`monthname(date)`:返回date的英文名称,四月份的话,select monthname(curdate()) ==> April

`Date_format(date,fmt)` : 将date格式化成fmt格式，select date_format(now(),'%M,%D,%Y')


`date_add(date,INTERVAL expr type)`: 返回与所给日期date相差INTERVAL时间段的日期


`datediff(date1,date2)`:用来计算两个日期之间相差的天数

#### 流程函数

`IF(value,t,f)`: select if(salary>2000,'high','low') from salary; 如果value成立，则显示为t，否则为f

`ifnull（value1，value2)` 如果value1为null则显示为value2

`case when [value] then [result].. else[default] END` : 与if-else差不多 select case when id<5 then 'err1'  else 'err>5' end  from city_tbl

#### 其它函数

`database()`:返回当前数据库名

`version()`:返回版本

`user()`:返回当前登录用户

`inet_aton(IP)`:返回IP地址的网络字节序表示：select init_aton('192.168.1.1')==》3232235777

`inet_ntoa(num)`:返回网络字节序的IP地址标识：select inet_ntoa(3232235777)==>192.168.1.1

`password(str)`:返回字符串str的加密版本，一个41位长的字符串

`MD5(str)`:返回字符串的str的MD5值，常用来对应用中的数据进行加密


`show engines`:返回存储引擎列表

`show variables like 'have%'`:返回have为前缀的所有系统变量

`alter table *** auto_increment=n;`  设置强制自动增长列的初始值，该强制默认值存储在内存中，如果该值在使用之前数据库重启，那么需要在数据库启动后重新设置

对于innoDB表，自动增长列必须是索引

对于text,blob字段的表，如果进场做删除和修改记录，要定时执行optimize table对表进行碎片整理

**`help ‘show engines’`**:如果忘记了某个方法，某个关键字，都可以用help来帮助






















