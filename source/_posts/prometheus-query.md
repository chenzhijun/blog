---
title: Prometheus 常用的查询语句
copyright: true
date: 2022-07-16 17:03:19
tags: Prometheus
categories: 监控
---

## Prometheus 常用的查询语句

通常我们会使用Grafana作为图表展示，然后选择Prometheus作为数据源的方式来进行我们想要的图表展示。当然我们也可以在Grafana的官网上找到相应的dashboard来直接导入，这样省去了自己手工配置的麻烦。不过知道一些必要的Prometheus查询语句能帮我们更好的选择Grafana的dashboard，然后我们可以自定义做些配置。

### Prometheus一些术语

Prometheus的metrics分为四类（counter,gauge,histogram,summary）详情([metrics-type](https://prometheus.io/docs/concepts/metric_types/))，metrics_name的命令也应该符合一定的规范：[metrics-name](https://prometheus.io/docs/concepts/data_model/)，[METRIC AND LABEL NAMING](https://prometheus.io/docs/practices/naming/)

`metrics_name` ： 也就是指标名，通常我们都会用如http_request_total等进行查询;
`metrics_label` ：指标的标签，也就是metrics_name{label-1="a",label-2="b"}这种;
`metrics_value` : 通常用指标名+标签查出来一个值，该值根据metrics的类型可能为浮点数，也可能为整数。

<!-- more -->

### 0,基础查询

也就是使用metrics_name+metrics_label的组合进行查询，这种查询的效率的基础是你要知道明确知道相应的name和label，如果有错误拼写，则可能数据无法展示，Prometheus的查询工具能帮我们模糊匹配出所有的metrics_name。可以使用label进行筛选。
如果只记得部分metrics_name,那么可以使用内置的label：`{__name__=~"metrics_name_you_remember:.*"}`这样去匹配出来你想要的标签。

### 1,正则匹配查询

Prometheus里面用的最多的查询可能就是正则匹配了。经常配合Grafana的变量(variable)一起使用。比如：
`sum(irate(node_disk_reads_completed{device!~"dm-.*"}[5m]))`,这里面的`device!~"dm-.*"`，后面
引号内的`dm-.*`就是不匹配`dm-`前缀的所有metrics_name。如果要匹配的话使用`=~`。这样要注意，如果是或
的话要用`(regrex_A|regrex_B)`,用小括号加`|`。


### 2,常用的操作符和函数

Prometheus支持常用的操作符：`+,-,*,/,>=,<,>,....`等

count是对查询的结果数量进行总和+; 而sum是对查询出来的value进行总和+;

它也支持比如topk,bottomk,min,max等等。

### 更多的查询

在官网还有更多的查询，通常我们是结合Grafana来做。官网的查询:[proetheus-query](https://prometheus.io/docs/Prometheus/latest/querying/functions/)。



