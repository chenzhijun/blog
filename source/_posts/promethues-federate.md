---
title: promethues 联邦集群
copyright: true
date: 2018-05-10 12:47:24
tags: prometheus
categories: 监控
---

# Prometheus 联邦集群

Prometheus的联邦集群我们使用它来作为Prometheus代理。因为我们是在监控rancher平台里面的docker容器里面的应用，那么拿到的就是容器的ip，而我们实际的Prometheus是部署在外部虚拟机上面的。这个时候外部的Prometheus就无法拿到rancher平台内部容器应用的metrics，所以部署一台prometheus到rancher组成联邦机，详细的官网有解释:[federate](https://prometheus.io/docs/prometheus/latest/federation/)，总体架构图如下
<!--more-->
![2018-05-10-12-59-05](/images/qiniu/2018-05-10-12-59-05.png)

外部连接方式：

```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  # - job_name: 'prometheus'
  #   # metrics_path defaults to '/metrics'
  #   # scheme defaults to 'http'.
  #   static_configs:
  #     - targets: ['localhost:9091']
  - job_name: 'federate'
    scrape_interval: 15s
    # honor_labels: true
    metrics_path: '/federate'
    params:
       'match[]':
        - '{job=~"prometheus"}'
        #- '{job="*"}'
    #     - '{job="prometheus"}'
    #     - '{__name__=~"job:.*"}'
        #  - '{job="rancher_network_monitor"}'
          - '{job="targets-server"}'
    static_configs:
     - targets:
       - 'localhost:9090'
       - '10.62.14.129:9090'
      #  - '10.62.12.3:9090'
      #  - '10.0.11.23:80'
```

额外注意就是这里的job_name下有一个match，这个好像必须要填写，嗯就是这样，只要在内部的Prometheus代理将job那么定义好，在外部再像上面的配置文件一个配置，就能在外部访问内部的prometheus的数据，而且可以保存这些数据在外部的Prometheus。
