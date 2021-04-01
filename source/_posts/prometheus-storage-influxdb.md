---
title: prometheus 使用influxdb 做永久存储
copyright: true
date: 2018-06-09 16:04:43
tags: prometheus
categories: 监控
---

# prometheus 使用远端存储

使用Prometheus的过程中，我们可以发现Prometheus默认是自己带有存储的，不过保存的时间为15天。但是对于公司而言，可能有时候会对数据进行统计分析，那么15天的数据将不会满足要求了。所以我们希望能够将数据永久存储起来，或者说能够让我们自己将数据进行处理。

这里我们要讲的就是Prometheus的 remote_storage 功能。Prometheus的remote_storage 其实是一个adapter，至于在adapter的另一端是什么类型的时序数据库它根本不关心，如果你愿意，你也可以编写自己的adpater。我这里采用官网提供的influxdb作为远端存储的实例。

存储的方式为：Prometheus ----发送数据---- > remote_storage_adapter ---- 存储数据 ----> influxdb。
<!--more-->
## 下载安装influxdb

我采用的docker安装influxdb，非常容易。下载influxdb的镜像，然后让它暴露出相应的端口：

`docker run -p 8086:8086 -v $PWD:/var/lib/influxdb --name influxdb influxdb` 这样，一个influxdb就准备好了。influxdb的一些介绍或者操作可以查看官网。
我们安装好influxdb之后需要在influxdb中创建一个prometheus的库：
`curl -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE prometheus"`。

## 准备remote_storage_adapter

在github上准备一个[remote_storage_adapter](https://github.com/prometheus/prometheus/blob/master/documentation/examples/remote_storage/remote_storage_adapter/README.md)的可执行文件，然后启动它，如果想获取相应的帮助可以使用:`./remote_storage_adapter -h`来获取相应帮助(修改绑定的端口，influxdb的设置等..)，现在我们启动一个remote_storage_adapter来对接influxdb和prometheus：
`./remote_storage_adapter -influxdb-url=http://localhost:8086/ -influxdb.database=prometheus -influxdb.retention-policy=autogen`，influxdb默认绑定的端口为`9201`

## 修改 prometheus.yml 配置对接adapter

前面的准备操作完了之后，就可以对prometheus进行配置了。修改`prometheus.yml`文件，在文件末尾增加：

```yaml
remote_write:
  - url: "http://localhost:9201/write"

remote_read:
  - url: "http://localhost:9201/read"
  ```

之后我们启动prometheus就可以看到influxdb中会有相应的数据了。如果验证我们采集的metrics数据被存储起来了呢？我们选取一个metric，过几分钟然后将prometheus停止，并且将data目录删除，重启prometheus，然后我们再查询这个metric，可以看到之前几分钟的数据还在那里。


### prometheus 高可用

最近在实践一个事情，就是Prometheus的高可用，我的想法是将所有数据都存储到influxdb，但是influxdb集群版本竟然是闭源的。我擦。。。不过这个事情倒不是最重要，实在不行我们自己弄集群版本。我的高可用选择是有多个prometheus进行的是采集，在yaml种只配置remote_write，然后让某几台Prometheus机器做查询，只配置remote_read，并且查询的prometheus不做scape-config，也就是没有采集任务，完全的查询客户端，当然还有规则报警。在实际中要注意`global.external_labels.monitor`这个配置必须是 **相同的值**。不然的话查询客户端不能查询到相应的metric。当然你也可以不设置这个值。
另外实践中我发现federate机器也就是prometheus的联邦机器，在另一个prometheus配置了一台federate之后，它会将从federate采集到的数据收集起来，并且存储到influxdb中，也就是相应的做了持久化存储。

目前还有两个问题没有解决：多个查询客户端prometheus如果都配置了报警rule的话，会不会产生单个报警重复报？另外还有就是influxdb的集群方案，该怎么操作？还要进一步研究才行啊。~~


### 附录，influxdb的一些操作：

influxdb常用操作：
    显示数据库：show databases
    创建一个库：create database database_name
    删除一个库：drop database database_name
    使用库：use database
    表操作：show measurements
    插入数据
    `insert <tbname>,<tags> <values> [timestamp]    `
    说明：
    tbname : 数据表名称
    tags : 表的tag域
    values : 表的value域
    timestamp ：当前数据的时间戳（可选，没有提供的话系统会自带添加）

    示例如下：
```
    > use testdb;
    Using database testdb
    > insert students,stuid=s123 score=89
    > show measurements;
    name: measurements
    name
    ----
    students
    ```