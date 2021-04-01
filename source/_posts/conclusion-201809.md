---
title: conclusion-201809
copyright: true
date: 2018-09-08 14:53:15
tags:
categories:
---

#

1: linux命令行替换文本中的字符串。`sed -i "s/100.69.224.18:10099/100.69.224.27:9999/g" /file.xml`

2: rancher平台的相关监控。

```
mysql:

获取mysql用户是否可以远程登陆
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'exporter';
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'exporter'@'localhost';
GRANT SELECT ON cattle.* TO 'exporter'@'%' with MAX_USER_CONNECTIONS 3;
help flush;
show grants for 'exporter'@'localhost';
flush PRIVILEGES;

docker run -d -p 9104:9104 -e DATA_SOURCE_NAME="exporter:exporter@(127.0.0.1:3306)/" prom/mysqld-exporter:latest

haproxy:

docker run -d -p 9101:9101 prom/haproxy-exporter:latest --haproxy.scrape-uri="http://www.haproxy.com/haproxy?stats;csv"


```

3: sort 根据低三列排序

docker stats -a --no-stream |awk '{print $1,$8,$3,$4}'|sort -k 2,4n

sort -k a,bn
a为第几列，b为几个字符，n为数字比较，1234，1222，从第一个字符开始，到第四个字符都会比较

4: 给基础镜像安装常用工具：ping,curl,wget,netstat。一般源里面就已经包含了，ping的源为：`apt install inetutils-ping或者
apt install net-tools`

5：某次给rancher平台扩容，直接将主机加上之后，将原来的应用直接*2启动，平台爆掉了。server不到。

