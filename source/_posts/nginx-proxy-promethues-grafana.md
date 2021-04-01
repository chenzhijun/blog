---
title: nginx 设置prometheus和grafana的反向代理
copyright: true
date: 2018-05-02 22:57:50
tags: ["nginx","prometheus","grafana"]
categories: monitor
---

# nginx 设置promethues和grafana的反向代理

在配置完promethues，和grafana之后，可能需要上生产环境，这个时候如果有下面两种情况，那么就可能需要用到代理；
1. 端口只开发80,或者8080等特别的几个端口，端口数量有限；
2. 不希望暴露给外部端口号，使用子路经来区分；eg: http://{ip}/prometheus,http://{ip}/grafana

nginx 配置，监听server的端口为80，然后通过子路径在内部反向代理出去：

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
        location /grafana/ {
            proxy_pass http://127.0.0.1:3000/;
        }
        location /promethues/ {
            proxy_pass http://127.0.0.1:9090/prometheus/;
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

<!--more-->

alertmanager很大的情况等同与prometheus，可以等同配置。

## 设置promethues的代理，子路径
接下来需要的是将prometheus和grafana在启动或者配置文件中做一些更改，prometheus的相对来说比较简单，主要实在启动的时候根据命令行的参数来进行子路径设置。

在启动的时候设置`web.external-url`使用下面的命令：

```shell
./prometheus --web.external-url=promethues
```

结果如图：

![2018-05-03-13-04-03](/images/qiniu/2018-05-03-13-04-03.png)

还可以使用`./promethues --help`获取更多的命令行参数，alertmanager同样也适用。

## 设置grafana的代理，子路径

grafana的代理需要在`default.ini`中配置root_url:`root_url = %(protocol)s://%(domain)s:/grafana`

之后再重启就可以了。记住，再nginx中，proxy_pass 不要带上后缀。添加反向代理后，如果访问使用`http://localhost:3000/grafana`或者`http://localhost:3000`页面会显示不全。但是使用nginx代理后的路径：`http://localhost/grafana`就可以看到全页面了。

![2018-05-03-13-54-16](/images/qiniu/2018-05-03-13-54-16.png)

## 参考资料

[Running Grafana behind a reverse proxy](http://docs.grafana.org/installation/behind_proxy/)
