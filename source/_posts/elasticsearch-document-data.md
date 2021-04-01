---
title: elasticsearch-document-data
date: 2017-12-19 09:29:04
tags:
	  - Elasticsearch
categories: Elasticsearch
---

## 什么是文档

通常情况下，文档类似与对象。在es的术语中文档指的是根对象，或者称为最顶层对象。它被序列化成json存储在es中，并且有一个唯一ID。
> 字段可以是任何字符，但不能包含时间段

### 文档的元数据

#### 索引:_index

ES中文档有三个特定的属性：index,type,id;(type可能会在以后的版本中去掉，现在的版本6.0中依然存在)。在es分片中我们提到，数据是存储在索引中的，而索引指的是单个，或者多个分片的逻辑命名空间。索引广义上来说，有点类似我们的数据库database名称。一个库名就对应着一个索引。

> 索引名字必须小写，不能以下划线开头，不能有逗号

#### 类型:_type

type的含义是将具有相同属性或相似属性的文档集合在一起。它是索引中对数据的逻辑分区。有点类似于我们在数据库中的表。

> 类型名可以是大小写，不能以下划线开头，不能包含逗号，长度限制在256字符。

#### 唯一ID:_id

如果说空间中的一个点由xyz三轴表示，那么es中的文档位置就由索引，类型，id来确定。Id是一个字符串，你可以自己生成，也可以让ES帮你生成。id类似与表中的主键。

文档还有其它的元数据，但是总的来说，最最重要的就是这三个。毕竟你最重要的还是数据(文档)。

>自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。

你可以这样理解，图书馆有各种书，管理员将书分成了很多类，每一个类下面有个唯一的书编号，知道这些，你就能找到那本特定的书。其中图书馆就是ElasticSearch，各种书就是index，分类就是type，唯一id就是书编号。实际的那本书就是一个文档。

### 创建索引文档
<!--more-->
1. 带id创建索引文档,id是你自己生成的唯一id，使用put方法：

```java
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

2. 如果是自动生成的，请使用POST方法：

```java
POST /{index}/{type}
{
  "field": "value",
  ...
}
```

### 获取文档

前面我们说过，如果确定index,type,documentId，我们就能获取一个文档值。对于文档的获取方式我们可以获取到文档的全部值，或者是文档的部分值。

1. 根据三元素获取文档全部值：

```shell
   GET /website/blog/123?pretty
```

返回结果：

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
  }
}
```

其中需要注意字段`found:true`,如果找到了值，这里就是true，如果没有找到，这里回事false，并且响应的HTTP状态码会是404。

2. 返回文档的一部分：

```shell
GET /website/blog/123?_source=title,text
```
这里就是只需要:title,text。返回结果：

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```

3. 如果只需要`_source`字段不需要任何元数据：

```java
GET /website/blog/123/_source
```

返回结果：

```json
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```

### 检查文档是否存在

如果不需要数据，只需要检查在某一个时刻某个文档是否存在，因为有可能这一秒钟结果返回不存在，下一秒钟就创建了，所以用检查的时候主要注意这个时效性。
不需要数据的时候我们可以使用HEAD方法来进行请求，格式为：`curl -i -XHEAD http://ip:port/index/type/id`;
比如:

```shel<tab>l
curl -i -XHEAD http://localhost:9200/website/blog/123
```

### 更新文档

在ES中文档是不可修改的。如果需要更新现有的文档，需要重建索引或者进行替换。也就是说，你可以再调用一次创建索引的接口。
只不过在返回的响应中会看到元数据域有部分改变：

```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false 
}
```

可以看到，version已经变成了2，created是false。created为false，是因为相同的索引，type，id已经存在。在内部ElasticSearch会标记旧文档为已删除，并增加一个全新的文档。虽然旧文档不能再访问，但是es不会立即删除它。当索引的数据越来越多的时候，es会在后台删除？(什么时候删除，学完记得看下文档)

### 确保只能创建文档，非更新

上面更新的方式我们知道，创建的时候有可能也是更新文档。那么如果保证这次的创建请求,是创建了一个全新的文档而非更新一个文档了？我们可以采取一些手段来保证。

1. 使用post方式进行增加，让es自动生成文档id

2. 如果采用自己的id，在末尾可以加参数：

a) 使用param的方式创建：

```s
    PUT /website/blog/123?op_type=create
```
    
b) 使用restful风格：
    
```s
    PUT /website/blog/123/_create
```

如果创建过程中有错误，会返回409错误状态码：

```json
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```

### 删除文档

删除的规则也遵循restful风格，所以类推如下：

```java
    DELETE /website/blog/123
```

如果有值，那么返回200，并且返回json：

```json
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```

如果没有找到，将会返回404:

```json
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```

删除跟更新一眼哥也是做一个标记操作，不会立即将文档删除，随着索引的数据越来越多，es会在后台进行删除。