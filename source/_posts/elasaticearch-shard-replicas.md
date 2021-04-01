---
title: elasticsearch 集群主分片和副本分片
date: 2017-12-18 16:20:45
tags:
---

集群的健康状态查询：

```shell
curl -XGET 'localhost:9200/_cluster/health?pretty'
```

```json
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

`status`字段是最为重要的，有三个值：green，yellow，red。三者的含义：

1. green：所有主分片和副分片都能正常运行；
2. yellow: 所有主分片都能正常运行，但不是所有副分片都能正常运行；
3. red：有主分片没能正常运行。

<!--more-->
在es中存数据需要先建立索引，而索引实际上是一个或多个物理分片的逻辑命令空间。

一个分片是一个底层的工作单元，存储全部数据中的一部分。它是一个Lucene的实例，本身它就是完整的搜索引擎。

文档被存储和索引在分片中，应用程序直接与索引交互不是与分片交互。

> 理论尚分片可以存储Integer.max_value-128个文档。实际最大值还需要参考你的使用场景：包括你使用的硬件， 文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。

副本分片是主分片的拷贝，也就是指一个备份。索引在创建的时候就已经确定了确定的分片数量(默认为5个)，一旦索引创建成功，主分片数就不能再改变，但是副分片数可以被随时改变。
创建一个确定主分片的数的索引：
使用3个主分片和6个主分片分别创建`blogs`和`accounts`索引:

3个blogs主分片，1个备份分片
```shell

curl -XPUT 'localhost:9200/blogs?pretty' -H 'Content-Type: application/json' -d'
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
'

```

6个accounts主分片，1个备份分片：

```shell

curl -XPUT 'localhost:9200/accounts?pretty' -H 'Content-Type: application/json' -d'
{
   "settings" : {
      "number_of_shards" : 6,
      "number_of_replicas" : 1
   }
}
'

```

之后结果如图：
![2017-12-18-16-48-44](/images/qiniu/2017-12-18-16-48-44.png)


如果我们觉得备份分片不够，比如我想将accounts调整为2个备份分片：

```shell

curl -XPUT 'localhost:9200/accounts/_setting' -H 'Content-Type: application/json' -d'
{
      "number_of_replicas" : 2
}
'

```

之后结果如图:

![2017-12-18-16-52-35](/images/qiniu/2017-12-18-16-52-35.png)


如果我们的备份分片没有分配到节点上，集群的健康值就会变成yellow：

![2017-12-18-16-56-13](/images/qiniu/2017-12-18-16-56-13.png)

我们可以为每一个分片(包括主分片，副本分片)分配一个节点。这样会提高我们的搜索效率，节点数/主分片数。

上图的副本分片都没有分配到节点上，全部副本分片都是unassigned，在一个原始数据节点存储副本分片，想想也是知道没有任何意义。该节点挂掉之后，副本数据肯定也会挂掉。


还有一种就是集群健康状态为红：red。这种情况下指的是主分片中有一个或多个主分片为不可用状态。

![2017-12-18-16-22-15](/images/qiniu/2017-12-18-16-22-15.png)