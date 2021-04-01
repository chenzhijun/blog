---
title: MySQL 用户与权限设置
copyright: true
date: 2018-12-17 17:00:27
tags: MySQL
categories: MySQL
---

# MySQL 用户与权限设置

这几天开发完发现dba对于权限控制的比较严，通常是没有root权限的，在正式上线前，还是希望能再通过一个普通用户的权限来做一次预发布。这样可以看看到底会有哪些坑，提前踩一下可能比较好。后来发现，其实你只要准备需求提给dba就ok了，人家会帮你处理的非常好~~~。总之在这过程中遇到的问题，先记录一下吧。
<!--more-->
## MySQL创建用户

第一个就是创建一个普通用户。使用root登陆后，创建一个用户：`CREATE USER 'username'@'localhost' IDENTIFIED BY 'paasword';`其中`username`,`paasword`按需修改。一般来说这里需要注意的就是`localhost`，为什么？因为这个字段稍不注意就会才坑，这里的`localhost`和`%`，其实指代的是只允许本地登录和允许所有地方登陆。如果你的应用部署在同一台机器上，那么你用localhost没问题，但是如果应用和数据库是分开机器部署的，那么这里要写成`%`，不然就会出现远程无法连接。当然还有很多其他的设置，比如什么过期时间啊，证书登录啊，具体的可以看下官方文档:[Create User](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)。

## 给MySQL用户赋权限

如果做开发的话，你就会发现，你的jdbc或者其他语言连接数据库，都需要选择选择一个库。也就是你必须先在数据库里面建立一个database，但是如果你用root建立一个database，比如：`create database DB_USER;`这个时候上一步创建的用户是无法访问这个库的。如果你切换到刚刚的用户，那么你也是没有权限建立数据库的。但我们通常开发都会写上库名，那这个时候怎么操作了？嗯，就是先用root建库，然后将权限库的权限赋值给新用户。具体操作如下：

```sql
CREATE DATABASE db_user;
GRANT ALL ON db_user.* TO 'username'@'%';
```
这里一定要注意`%`,`localhost`，如果这里不同，那就是两个用户。

当然，如果你觉得只给查的权限就足够了，那么只需要`GRANT SELECT ON db_user.* TO 'test'@'%';`。那么有没有更详细的了了？当然有，文档始终是最详细的，我只是记录我使用的过程。文档地址：[grant](https://dev.mysql.com/doc/refman/8.0/en/grant.html#grant-overview)


一些比较使用的使用方式：

1. 查看当前用户：`select CURRENT_USER();`

2. 查看当前用户权限：`show grants;`

