---
title: ElasticSearch 简介与使用
date: 2017-12-03 14:32:47
tags:
	- Elasticsearch
    - Java
categories: Elasticsearch
---

## ElasticSearch 简介与使用

Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。Elastic 的底层用的是 Lucene。如果你想用 Lucene的话必须自己去写接口，而Elastic将这些进行了一层封装，并且提供restful接口，让使用者达到开箱即用。

### 基本概念

#### Node 与 Cluster

Elastic 实际上是一个分布式数据库，它可以存储数据，能够让多台服务器协同工作，每个服务器可以运行多个Elastic服务。每一个Elastic服务实例都可以称作一个节点（Node），一组节点就构成了集群（Cluster）。

#### Index，Type，Document

Elastic中通过索引(Index)，类型（Type），文档（Document）三个值来定义了Elastic中存储的数据结构。索引相当于我们在数据库中的库名字，类型相当于表，文档就是实际存储的数据内容。比如一堆书，这个就是“书”索引（Index），按照“武侠”，“技术”等进行分类（Type),每一本书就是实际上存储的数据(Document)。在Elastic中多个文档组成了一个索引，而文档可以通过分类来方便查询。这里的Type实际上是逻辑上的分组。

### ES的增删改

ES的增删改查遵循restful的风格，所以在使用在非常方便：

```java

// 增加
POST localhost:9200/accounts/person/1
{
    "name":"Jack",
    "lastname":"Mic",
    "description":"A very handsome man"
}


//删除数据

DELETE localhost:9200/accounts/person/1

POST accounts/persion/1/_update
{
    "doc":{
        "description":"this is change description"
    }
}
```

### ES的查询

为什么要单独讲讲查询？ElasticSearch，从名字中就可以直接看出，search占据了elastic的很大一部分。很多时候我们使用ES也是主要因为它方便的查询功能，在ES中查询的方式有以下几种：

1. 不带条件返回所有索引下的所有文档：

```java
    GET localhost:9200/_search?pretty // pretty是将返回的json进行格式化
    GET /_search?size=5 // size 是记录条数
    GET /_search?size=5&from=5 // from是只从哪页开始，类似SQL分页查询
```

2. 根据索引，类型，文档id获取到唯一值：

```java
    GET localhost:9200/accounts/person/1
```

3. 不带body的查询，[轻量搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html)

```java
    // 返回所有文档中有‘jack’的数据
    GET localhost:9200/accounts/person/_search?q=Jack

    // 返回所有类型中为tweet的tweet字段带有‘elasticsearch’的数据
    GET /_all/tweet/_search?q=tweet:elasticsearch
```

4. 带body的条件查询：[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

```java

GET localhost:9200/accounts/person/_search
{
    "query":{
        "term":{
            "name":{
                "value":"Jack"
            }
        }
    }
}

```

官方文档提供了一些实例：[查询表达式](https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-dsl-intro.html)

