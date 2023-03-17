---
title: Prometheus+Grafana 搭建监控系统
copyright: true
date: 2022-04-26 22:03:19
tags: Prometheus
categories: 监控
---

# Prometheus+Grafana 搭建监控系统

今天将第一版监控系统上线，过程整个就是一路坎坷。不过踩坑，填坑，确实也是为自己积攒了一些小经验。

## Prometheus的服务发现

Prometheus的监控使用的是pull的模式，也就是每隔几秒钟去各个target采集一次metric。那么如果是多个target，如果是静态配置的话，那么就得在配置文件里面一个一个添加，尽管可以使用接口去更新配置文件，但如果服务太多，那工作量也很大。而且如果遇到微服务的情况并且容器化部署，那么可能ip地址都是随机改变的，那么就将更麻烦了。所以就有服务发现的模式出来了，有很多种实现的方式，consul，dns等等，针对我们现有的平台，我们选择了file_sd_config:

```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ['localhost:9090']
  - job_name:       'rancher_network_monitor'
    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 1m
    static_configs:
      - targets: ['192.168.7.3:8080','192.168.8.90:8080']
        labels:
          group: 'rancher_network_monitor'
    metrics_path: /metrics
  - job_name: 'filediscovery'
    scrape_interval: 5s
    file_sd_configs:
      - files: ['/home/config/*.json']
```

基于文件的方式，只需要在`/home/config`目录下增加json文件就可以了，这也是在网上找的一个方案，所以我也记录下来，万一也能帮助到别人了

<!--more-->

```json
[
    {
        "targets": [
            "10.10.10.1:65160",
            "10.10.10.2:65160"
        ],
        "labels": {
            "job":"Center",
            "service":"qtest"
        }
    },
    {
        "targets": [
            "10.10.10.3:65110",
            "10.10.10.4:65110"
        ],
        "labels": {
            "job":"Gateway",
            "service":"qtest"
        }
    }
]
```

## 监控报警规则 rule_files:

```yaml
rule_files:
   - './alert.yml'
```

规则可以在这里配置多个，这里其实也是支持通配符的。也就是：

```yaml
rule_files:
   - '/path/to/rule/*.yml'
```

我使用的是yml格式，如果你会的话，也可以是用`.rule`;

```yaml
groups:
- name: goroutines_monitor
  rules:
  - alert: go_goroutines_test
    expr: go_goroutines > 200
    for: 5s
    labels:
      name: goroutines_test
      severity: warning
    annotations:
      summary: "{{ $labels.instance }}，机器goroutines过高。" 
- name: instance_monitor
  rules:
  - alert: instance_down
    expr: up == 0
    for: 1m
    labels:
      name: instance
      severity: critical
    annotations:
      summary: "{{ $labels.instance }} 挂掉了,{{ $labels.job}}" 
```

上面的yaml就是定义了两个规则，`go_goroutines>200`以及`up==0`一个是go的协程数，一个是被监控目标是否可以正常采集。在则合理推荐使用labels，这样如果是使用alertmanager做预警，那么可以使用这些label。`for`的意思指接到一个触发报警rule的收将状态变成active，然后如果在for之内的评估期内如果没有报警就变成pending状态，如果有就变成firing，发送报警。

## alertmanager发送报警邮件

alertmanager相当于通知中心，它只会在Prometheus发送报警后，通过某些渠道（邮件，即时通讯等）发送通知。当然它在通知这方面做了相当多的事情。

### alertmanager 通知分组

如果是一个频繁的bug，引起Prometheus一直触发警报，如果不做限制，那么就会引起邮箱轰炸。alertmanager中的route就是做这个的。通过路由器将不同的邮件发送给不同的人。

```yaml
route: 
  # How long to initially wait to send a notification for a group
  # of alerts. Allows to wait for an inhibiting alert to arrive or collect
  # more initial alerts for the same group. (Usually ~0s to few minutes.)
  # group_wait: 30s
  # How long to wait before sending a notification about new alerts that
  # are added to a group of alerts for which an initial notification has
  # already been sent. (Usually ~5m or more.)
  group_interval: 30s
  # If an alert has successfully been sent, wait 'repeat_interval' to resend them.
  repeat_interval: 10s    
  #  A default receiver
  receiver: monitor-admin  
  group_by: [name, alertname]
  routes:
  # All alerts with service=mysql or service=cassandra
  # are dispatched to the database pager.
  - receiver: 'go-admin'
    group_wait: 30s
    group_by: [name, alertname]
    match:
      name: goroutines_test
      alertname: go_goroutines_test
  # All alerts with the team=frontend label match this sub-route.
  # They are grouped by product and environment rather than cluster
  # and alertname.
  - receiver: 'instance-admin'
    group_interval: 1m
    group_by: [name, alertname]
    match:
      name: instance
```

这里的`group_by: [label,...]`将通知分组，`group_interval: 30s`指的是如果在30s内收到同一组的邮件告警，那么将合并他们为一条邮件然后发送。`repeat_interval: 10s`当第一次发送邮件后下一次发送邮件的间隔。这个值一定要改，尤其是上线之后。routes主要是将邮件进行区分发送，这里我们之前在rule里面配置的label就可以起作用了。使用match来匹配那些label然后做不同的路由。

### alertmanager 邮件抑制

邮件抑制的意思，我觉得更应该是邮件优先原则。指的是如果有两组告警，一组告警非常紧急，一组不重要，如果两组一起来告警，那么可能会引起，邮箱里面重要的告警被不重要的淹没了，从而导致我们忽略了某些重要告警。所以我们可以进行配置：

```yaml
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  # Apply inhibition if the alertname is the same.
  # equal: ['alertname', 'cluster', 'service']
  equal: ['monitor','job']
```

上面的规则就是，如果有serverity为critical的标签，那么serverity为warning的告警先不发，只发critical的告警。在Prometheus的rule里面如下配置：
![2018-04-26-23-54-39](/images/qiniu/2018-04-26-23-54-39.png)

所以，多配置点label还是有点用的，哈哈。

ps：抑制的是某一种高级别的邮件发送，而这个邮件会按照定好的时间间隔一直发送，不会说只发一条就不发了。


### alertmanager 邮件静默

邮件静默其实就是指在某一段时间内，将某一类型的告警暂时忽略不让它发送告警。（又用到label了。。）创建静默的方式有两种，一个是直接在告警信息上创建；另一个是直接new silence：
![2018-04-26-23-58-20](/images/qiniu/2018-04-26-23-58-20.png)

需要配置一个label。之后一定一定要配置时间，第一次使用的时候，我以为静默是一直忽略，然后我就下班了。之后回到家中，2h默认时间过后，我的邮箱爆炸了。。。所以静默是有时间限制的，一定要，一定要设置。
![2018-04-26-23-59-00](/images/qiniu/2018-04-26-23-59-00.png)

## alertmanager的邮件模板

```yaml
templates:
  - './templates/*.tmpl'


receivers:
  - name: 'monitor-admin'
    email_configs:
    - to: 'noreply_czj@163.com'
      headers: { Subject: "[WARN] {{ .CommonLabels.alertname}} 报警邮件" }
      html: '{{ template "email.czj.html" . }}'
```

```tmpl
{{ define "email.czj.html" }}
<table>
    <tr><td>报警名</td><td>xxx</td><td>xx</td><td>xxx</td><td>xxx<font color='red'>[xxx]</font></td></tr>
    {{ range $i, $alert := .Alerts }}
        <tr><td>{{ index $alert.Labels "alertname" }}</td><td>{{ $alert.StartsAt }}</td><td>{{ $alert.Labels.from }}</td><td>{{ $alert.Labels.to }}</td><td>{{ $alert.Labels.value }}</td></tr>
    {{ end }}
   

</table>

{{ end }}
```

邮件模板这里我是在网上找了一个非常详细的帖子。我觉得写的非常详细，主要注意就是要在模板的一行的define，要和receivers的html中tempate相对应。之后就是在模板中引用go语言的变量用`点+大写变量名`。详细可以看这边博客[alertmanager 邮件模板](https://segmentfault.com/a/1190000008695463)。从源码解释，很详细。

<!-->
Data:.Alerts：[{firing map[instance:10.62.14.80:8080 job:example-random monitor:codelab-monitor to:10.62.12.3 alertname:node_connect from:10.62.14.80 group:production] map[summary:机器 10.62.14.80:8080 ，10.62.14.80，10.62.12.3] 2018-04-19 15:44:51.3912519 +0800 CST 0001-01-01 00:00:00 +0000 UTC http://FTSZ-NB0078:9090/graph?g0.expr=host_to_host_http_connect+%3C%3D+0&g0.tab=1} {firing map[monitor:codelab-monitor to:10.62.14.80 alertname:node_connect from:10.62.14.80 group:production instance:10.62.14.80:8080 job:example-random] map[summary:机器 10.62.14.80:8080 ，10.62.14.80，10.62.14.80] 2018-04-19 15:44:51.3912519 +0800 CST 0001-01-01 00:00:00 +0000 UTC http://FTSZ-NB0078:9090/graph?g0.expr=host_to_host_http_connect+%3C%3D+0&g0.tab=1}]
status:.status: firing
Receiver:.Receiver: team-X-mails
GroupLabels:.GroupLabels: map[alertname:node_connect]
CommonLabels:.CommonLabels: map[from:10.62.14.80 group:production instance:10.62.14.80:8080 job:example-random monitor:codelab-monitor alertname:node_connect]
CommonAnnotations:.CommonAnnotations: map[]
ExternalURL:.ExternalURL: http://FTSZ-NB0078:9093 

<-->

## alertmanager 邮件配置

注意一个，是否开启ssl。我们用的是http，所以把ssl关闭。我用163配置的：

```yaml
global:
  smtp_smarthost: 'smtp.163.com:25'
  smtp_from: '********@163.com'
  smtp_auth_username: '*******@163.com'
  smtp_auth_password: '*******'
  smtp_require_tls: false
```

一般本地测试的时候：`smtp_require_tls`这个可以是true，但是一般服务器不通外网可能不行。

## prometheus 与 alertmanager 降低日志级别

日志调到debug级别，两者都可以用`.alertmanager --help`或者`./promtheus --help`来查看所有可选项。

alertmanager:`./alertmanager --config.file='config163.yml' --log.level=debug`;

prometheus: `./prometheus --web.enable-lifecycle --config.file='prometheus.yml' --log.level=debug --web.external-url=http://localhost:9090/pro --web.route-prefix=pro`

后面Prometheus的`--web.external-url=http://localhost:9090/pro`，`--web.route-prefix=pro`。这两个参数就类似加个服务号，如果有子路径做反向代理，Prometheus最好启动的时候就加上这两个。

比如在nginx用子路径做反向代理：
```conf

    server {
        listen       127.0.0.1:80;
        #server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;   
        #    proxy_pass http://127.0.0.1:9090/;
        }

        location /alertmanager/ {
            proxy_pass http://127.0.0.1:9093/;
        }

        location /pro/ {
            proxy_pass http://127.0.0.1:9090/pro/;
        }
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

```

暂时就这些吧。


## 参考链接
[GETTING STARTED prometheus.io](https://prometheus.io/docs/prometheus/latest/getting_started/)
[alertmanager邮件模版](https://segmentfault.com/a/1190000008695463)
[alertmanager报警规则详解](https://www.kancloud.cn/huyipow/prometheus/527563)
[Prometheus智能化报警流程避免邮件轰炸](http://blog.51cto.com/xujpxm/2055970)
[Custom Alertmanager Templates](https://prometheus.io/blog/2016/03/03/custom-alertmanager-templates/)