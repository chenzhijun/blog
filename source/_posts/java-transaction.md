---
title: Java 事务处理（包括spring事务管理）
date: 2018-03-14 22:11:25
tags: Java
categories: Java
---

# Java 事务处理（包括spring事务管理）

## JDBC 的事务处理

Java 中的事务处理有三部分：

1. 自动提交模式
2. 事务隔离级别
3. 保护点

其实实际上java中的事务处理最终依赖的是各数据库的事务处理实现。如果使用的数据库不支持事务，或者提供的数据库驱动程序没有支持事务，那么也是巧妇难为无米之炊。

事务自动提交的方式有：DML（DML），DDL（create），select 查询后结果集关闭，储存过程执行后。

事务隔离级别有：脏读;不可重复读;幻读。

保护点：部分事务回滚;选择性释放。

DriverManager--> Connection --> Statement --> ResultSet --> ResultSetMetaData。

```java
        Connection connection = null;
        try {
            connection = DriverManager.getConnection("");
            //将事务的自动提交关系
            connection.setAutoCommit(false);
            
            /*
            业务逻辑（操作数据库）处理
            doService();
            */

            //提交事务 
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                //还原自动提交事务
                connection.setAutoCommit(true);
                //关闭connection
                connection.close();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }
```

保护点，就是我们在处理一段逻辑中，不需要全部回滚回滚当前事务，只需要回滚到当前的保护点就好了：

```java

        Connection connection = null;
        Savepoint savepoint = null;
        try {
            connection = DriverManager.getConnection("");
            savepoint = connection.setSavepoint();
            connection.commit();
        } catch (SQLException e) {

            if(null!=connection){
                try {
                    connection.rollback(savepoint);
                } catch (SQLException e1) {
                    e1.printStackTrace();
                }
            }
            e.printStackTrace();

        } finally {
            try {
                connection.close();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }

```

## Spring 的事务实现与原理

手受了伤，一个手打字不便。稍等，别忘记了。记录到
gtd里面去

今日为2018年3月26日

spring 事务管理有以下一些有点<摘自官网文档16节 Transaction Management>：

1. 接口一致性编程模型，适用于Java Transaction API(JTA)，JDBC，Hibernate，Java Persistence API(JPA),Java Data Objects(JDO)
2. 支持声明式事务管理
3. 编程式事务api接口简单
4. 与Spring的数据访问抽象的完美集成

第一节还有全局事务管理和本地事务管理。还看到了EJB CMT（Container Managed Transaction）.Spring 事务抽象接口为PlatformTransactionManager接口，这个是用到了策略模式，可以看到传递transactionStatus为参数。
[2018-03-26-20-18-37](/images/qiniu/2018-03-26-20-18-37.png)

很多英文没看懂，明天继续啃。