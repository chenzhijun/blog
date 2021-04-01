---
title: nginx从docker容器的环境变量中获取值
copyright: true
date: 2019-04-08 22:56:33
tags:
categories:
---

# nginx从docker容器的环境变量中获取值

这篇接上一篇[vuejs 从拷项目到开发上线](http://chenzhijun.me/2019/04/08/vuejs-starter/)

## Docker 部署

另外一个问题就是如何制作成镜像了？可以看到我们在nginx.conf中有api服务的地址，这个地址可能在不同的环境(di,sit,prd)都不一样，那能否通过容器的env来改变nginx.conf中的值呢？

不查不知道，一查发现，我擦，还真有，nginx镜像本身就已经具备了。不过是李template的方式，来生成conf文件话不多说，直接上代码，先创建一个`nginx.conf.template`的文件：

```conf

server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
    root         /app/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    location /m-web/ {
        root /app/html/dist;
    }
    location /m-web/m-api/ {
        proxy_pass http://${M_API_SITE}/m-api/;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}

```

之后准备Dockerfile：

```dockerfile

FROM nginx:latest
RUN rm -rf /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/nginx.conf
ENV M_API_SITE 192.168.1.19:8089
ADD nginx.conf.template /etc/nginx/conf.d/nginx.conf.template
ADD dist /app/html/dist
CMD ["/bin/bash", "-c", "envsubst < /etc/nginx/conf.d/nginx.conf.template > /etc/nginx/conf.d/nginx.conf && exec nginx -g 'daemon off;'"]

```

之后`docker build -t xxx:latest .`，之后我们就可以通过`-e M_API_SITE=192.168.123.1:8080`设置容器的环境变量来设置不同的api地址了。