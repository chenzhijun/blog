---
title: million-data-insert-mysql-with-java-springboot-mybatis
tags: mybatis
---

# mybatis 插入百万条数据

最近从一个地方获取到了一个文件，内容不大不小，8个属性，一张表，记录大概110万条，这些数据都存储在excel表格当中，想着导入到数据库中。尝试了集中方式。优化的方式肯定有很多，不涉及到mysql的优化，就是原始MySQL5.7没有做任何优化，读入excel方式也是一样。

## 原始方式 

很简单，就是将数据读入到list中，然后遍历list，一条一条插入。这种方式是很原始的方式，如果没事干，那就放在晚上凌晨跑定时任务也没大影响。大概花费了30分钟吧。这种方式如果在平常是有点受不了的。

## 批量插入

使用mysql的批插入，就是利用了mybatis的foreach，其实是MySQL的values(),()。这种方式的话，大该一次性能插入2000条数据，冲过的会抛出异常。花费时间，比原始方式插入还是快一点，但还是有点慢，平常测试啥的不行。

## 多线程方式

采用多了50个线程，用线程池管理，

总长度：1086712

ok complete work...
start:Sat Sep 07 20:15:29 CST 2019
end:Sat Sep 07 20:29:53 CST 2019
startTime:Sat Sep 07 20:15:29 CST 2019,endTime:Sat Sep 07 20:29:53 CST 2019