---
title: Prometheus 体验
copyright: true
date: 2022-04-16 13:03:19
tags: prometheus
categories: 监控
---

# Prometheus 监控

 Prometheus [普罗米修斯]主要的是从被监控项目中获取metrics。架构全景图：

![2018-04-16-14-32-33](/images/qiniu/2018-04-16-14-32-33.png)

## 安装与使用

Prometheus 的安装方式很多，我在本地是使用windows的包，Prometheus自打出生就支持docker所以，如果是*inx机器，建议安装docker然后体验。在本地我仅仅只需要执行目录下的`prometheus`就可以就可以进行体验了。Prometheus服务自身也会暴露出metrics，用来对自身进行指标收集和监控。在根目录最重要的一个配置文件是`prometheus.yml`，里面有三类大属性：global，rule_files，scrape_configs。具体的配置信息可以看这个：
[https://prometheus.io/docs/prometheus/latest/configuration/configuration/](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)，也可以看看中文文档：[https://songjiayang.gitbooks.io/prometheus/configuration/](https://songjiayang.gitbooks.io/prometheus/configuration/)

<!--more-->

在Prometheus的后台`localhost:9090`，选择一个 metric，然后点击 execute，之后就可以在下面的graph和console中看到输出的结果：

![2018-04-18-22-18-09](/images/qiniu/2018-04-18-22-18-09.png)

在`status`下选择`Configuration`然后可以看到`prometheus.yml`里面的定义。在`Target`下可以看到监控的目标源的ip地址的信息。

## metrics 类型

metrics 有四类，并且每一个类都有相应的客户端lib。在目标监控中需要暴露出相应的metrics给prometheus服务器进行收集，之后才能进行有效的信息分析，之后预警和监控。

Prometheus客户端lib提供4种主要的核心metric类型：Counter，Gauge，Histogram，Summary。

`Counter`：数值类型，只能增加，不能减少。用户计数请求服务，完成任务数，发生错误数。不要用在可能会减少的地方

`Gauge`：数值类型可以增加可以减少，变化的类型，类似于测量温度，当前内存使用量

`Histogram`：直方图对样本进行观察（比如请求的持续时间，响应大小）。并将其统计到一个可配置的bucket中。一个Histogram包括：

```yaml
<basename>_bucket{le="<upper inclusive bound>"}
<basename>_sum
<basename>_count：指的是相同的<basename>_bucket{le="+Inf"}
```

`Summary`：主要用于表示一段时间内数据采样结果，类似Histogram

```yaml
<basename>{quantile="<φ>"}   （0 ≤ φ ≤ 1）
<basename>_sum
<basename>_count
```

metrics定义如：`<metric name>{<label name>=<label value>, ...}`

metrics接口需要注意，每行要空行，最后以空行结束。

```
# HELP http_requests_total The total number of HTTP requests.
# TYPE http_requests_total counter
http_requests_total{method="post",code="200"} 1027 1395066363000
http_requests_total{method="post",code="400"} 3 1395066363000

# Escaping in label values:
msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:
metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:
something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="0.5"} 129389
http_request_duration_seconds_bucket{le="1"} 133988
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds{quantile="0.9"} 9001
rpc_duration_seconds{quantile="0.99"} 76656
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

## Exporter 

自定义Exporter其实就是暴露一个metrics接口让Prometheus服务器进行收集。所以只需要返回像上面的例子一样的文本数据就可以了。
官方推荐首先采用`Historam`。可以导入相应的客户端metric然后进行导入。

## 继续学习

1. Prometheus 的内置函数
2. Prometheus 自定发现监控目标
3. 报警规则，报警方式
4. 图表
5. 对接granafa