---
title: 解决 logstash 输出到 Elasticsearch 时指定 document_id
copyright: true
date: 2018-04-02 17:05:32
tags: elk
categories: elk
---

今天使用elk搜集日志的时候想到一个事情，是否可以指定es索引中的document_id。查了资料之后发现还真有这个：

```yaml

elasticsearch {
        hosts => [ "localhost:9200" ]
        index => "%{[fields][service_name]}-%{+YYYY.MM.dd}"
        document_id => "%{@timestamp}"
  }

```

就是在`logstash.conf`中的output中，设置elasticsearch里面document_id就可以了。就是这么简单。当然我这里是用timestamp做的id，其实可以自己换成一个其它的。
es 对一个 id 添加两次是支持的，里面的字段 version 会加 1。