---
title: ElasticSearch 安装 (单机单节点/伪集群)
date: 2017-12-01 15:27:53
tags:
	- Elasticsearch
    - Java
categories: Elasticsearch
---

## ElasticSearch 安装 (单机单节点/单机多节点)

### ElasticSearch 简介

ElasticSearch(ES) 现在已经随着技术发展越来越火爆了。它基于Lucence搜索引擎，实现RestFul风格，开箱即用。广泛用于在网站上做站内搜索。

### 下载

这个忒简单了，会上网的人应该都会。

### 安装

ES 下载解压后，配置文件主要在`config`目录下，
包含文件：`elasticsearch.yml`,`jvm.options`,`log4j2.properties`。
这三个文件分别对应`ES配置`，`JVM配置`，`ES日志配置`。我们这里只讨论`elasticsearch.yml`的配置，其他的暂时不论。

### 单机单节点

单机单节点最爽了，为啥？因为简单啊。进入到解压后文件夹的`bin`目录，然后window平台双击`elasticsearch.bat`,*nix平台使用`sh elasticsearch`,之后再在控制台中看到如下，有个`started`：

![2017-12-01-15-42-20](/images/qiniu/2017-12-01-15-42-20.png)

因为我们什么配置都没改，所以ES使用默认配置，http端口为9200，TCP端口为9300。
这个时候我们访问下接口：`curl -XGET localhost:9200`,或者浏览器打开`localhsot:9200`,就会看到下面的输出：
![2017-12-01-15-48-31](/images/qiniu/2017-12-01-15-48-31.png)

单机很简单，真的很简单。
<!--more-->
### 单机多节点(伪集群)

部署完单机，下面就是集群了。集群，什么是集群了？一个服务在多台机器上部署，并且这些服务之间彼此之间内部高度紧密协作拥有某种联系，我们可以当作是这个服务的集群。在某种含义上，可以认为是一台服务器。

ES 伪集群：es服务在同一台机器上根据不同的端口启动服务，构成在本机上的一个集群模式。

![2017-12-03-14-57-03](/images/qiniu/2017-12-03-14-57-03.png)

以此为基础，我们来看看怎么配置。

主要**用到的配置属性**有这些，

![2017-12-01-16-09-24](/images/qiniu/2017-12-01-16-09-24.png)

我的本地ip地址为：`192.168.11.21`,

master 的 elasticsearch.yml:

```yml
cluster.name: notice-application
node.name: master
node.master: true
network.host: 192.168.11.21
# network.bind_host: 192.168.11.21
http.port: 9200
transport.tcp.port: 9300
discovery.zen.ping.unicast.hosts: ["192.168.11.21:9300","192.168.11.21:9310","192.168.11.21:9320"]
```

slave1 的 elasticsearch.yml:

```yml
cluster.name: notice-application
node.name: slave1
# network.publish_host: 192.168.11.21
# network.bind_host: 192.168.11.21
network.host: 192.168.11.21
http.port: 9210
transport.tcp.port: 9310
discovery.zen.ping.unicast.hosts: ["192.168.11.21:9300","192.168.11.21:9310","192.168.11.21:9320"]
```

slave2 的 elasticsearch.yml:
```yml
cluster.name: notice-application
node.name: slave2
# network.publish_host: 192.168.11.21
# network.bind_host: 192.168.11.21
network.host: 192.168.11.21
http.port: 9220
transport.tcp.port: 9320
discovery.zen.ping.unicast.hosts: ["192.168.11.21:9300","192.168.11.21:9310","192.168.11.21:9320"]
```

上面的配置，如果要你要体验下可以拷贝到你自己的ES中，将IP改成你的本地ip就可以看到了。

推荐一个图形化工具：[elasticsearh-head](https://github.com/mobz/elasticsearch-head),这货尽然还推出了[Chrome 插件](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm?utm_source=chrome-ntp-icon)。简直完美。

安装之后你就可以head插件看到集群配置了，下面是我的集群启动，电脑配置不太够，只启动了两台服务。
![2017-12-01-16-23-25](/images/qiniu/2017-12-01-16-23-25.png)

现在说正题，我们说下配置：

1. `cluster.name`: 它指代的是集群的名字，一个集群的名字必须唯一，节点根据集群名字加入到集群中

2. `node.name`: 节点名称，可以是自定义的方便分辨的名字，记住master也是一个节点。eg:master,slave

3. `node.master`: true/false 是否是集群中的主节点。

4. `network.host`: 设置`network.bind_host` 和 `publish_host`的默认值，这里设置成127.0.0.1和主机ip是有区别的，你可以使用curl -XGET "http://network.host/9200"看到结果

5. `network.bind_host`: 绑定服务器ip地址

6. `network.publish_host`: 绑定发布的地址

7. `http.port`: HttpRest 的接口，这个接口可以让你在浏览器访问

8. `transport.tcp.port`: 给Java或者其它节点的服务端口，代码里面用这个。

9. `discovery.zen.ping.unicast.hosts`: 这里是一组IP,我一般是使用`ip:port`这种书写方式，还有很多种方式，详情：[zen的介绍](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html)

### 安装中文分词插件

ElasticSearch 默认的[分词器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)对于中文的分词不是特别友好，英文的词使用空格隔开的，但是中文就不一样了。默认的分词器会将中文的字一个一个拆分，比如“中国”，默认的分词器就是“中”，“国”，然后去匹配。所以我们需要安装一个中文分词器，这里我选择的是[IK插件](https://github.com/medcl/elasticsearch-analysis-ik/)，它提供了一些友好的中文分词器，并且支持热更新分词热更新，注意根据自己的ES版本来选择IK的版本。github的readme上有两种安装方式，一种是用命令行模式：`./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.0.0/elasticsearch-analysis-ik-6.0.0.zip`。

另一种就是解压缩包安装方式，去[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)下载合适的release版本，然后解压到ES根目录下的plugins目录。

IK 提供了两种分词器：`ik_max_word`和`ik_smart_word`。

<!--
在`elasticsearch.yml`中配置默认的分词器：

```yml
index.analysis.analyzer.default.tokenizer : "ik_max_word"
index.analysis.analyzer.default.type: "ik_max_word"
```
-->
注意：如果要使用IK，你需要进行配置analyzer（字段文本的分词器），search_analyzer（搜索词的分词器）


### 参考文档：

https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html

分词器：https://www.elastic.co/guide/cn/elasticsearch/guide/cn/custom-analyzers.html ，
https://www.elastic.co/search?q=%E5%88%86%E8%AF%8D&section=Learn%2FDocs%2FElasticsearch%2FDefinitive+Guide，
https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis.html 

配置分词器：https://www.elastic.co/guide/cn/elasticsearch/guide/cn/configuring-analyzers.html