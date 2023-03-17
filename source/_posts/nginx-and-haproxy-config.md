---
title: nginx-and-haproxy-config
copyright: true
date: 2021-10-06 22:24:08
tags:
categories:
---

# Nginx 和 Haproxy 配置文件

目标：通过 Nginx 和 Haproxy 的常用配置实现服务的反向代理。
<!--more-->
## NGINX 配置

nginx 常用的配置文件一般处于 `/etc/nginx/conf/nginx.conf`。只有改这个文件之后执行 `nginx -s reload` 才可以动态加载，如果是 include 下面的，不会生效。

```conf
#daemon off;
user root root;
worker_processes auto;
worker_cpu_affinity auto;
error_log /var/log/ingress-gateway/error.log;
events {
  use epoll;
  worker_connections 20000;
  multi_accept on;
}

#TCP 方式
stream {
  upstream env1-lb-tcp {
    server 10.244.13.227:19080 weight=1 max_fails=1 fail_timeout=10s
  }
  server {
    listen 19080;
    proxy_pass env1-lb-tcp;
    proxy_connect_timeout 2s;
  }
}

# HTTP 方式
http {
  server_tokens off;
  sendfile on;
  tcp_nodelay on;
  tcp_nopush on;
  port_in_redirect off;
  keepalive_timeout 0;
  underscores_in_headers on;
  charset utf-8;

  include mime.types;
  lua_use_default_type off;

  log_format main '[$time_local]`$http_x_up_calling_line_id`"$request"`"$http_user_agent"`$staTus`[$remote_addr]`$http_x_log_uid`"$http_referer"`$request_time`$body_bytes_sent`$http_x_forwarded_proto`$http_x_forwarded_for`$host`$http_cookie`$upstream_response_time`xd';
  client_header_buffer_size 4k;
  large_client_header_buffers 8 16k;
  server_names_hash_bucket_size 128;
  client_max_body_size 0m;

  client_header_timeout 30s;
  client_body_timeout 180s;
  send_timeout 180s;
  lingering_close off;

  upstream smp-env-http-di1.19089.gts-seata-metrics.up {
    server 10.244.13.227:19089 weight=1 max_fails=1 fail_timeout=10s;
  }


  server {
    listen 19089;
    server_name smp-env-http-di1;

    access_log /var/log/ingress-gateway/vhost_access.log main;
    error_log /var/log/ingress-gateway/vhost_error.log;
    # for support ssl        # for support data zip        # for support backend server.
    # 后端default_backend的Web服务器可以通过X-Forwarded-For获取用户真实IP
    #proxy_redirect                  off;
    #proxy_next_upstream             error timeout invalid_header http_500 http_502 http_503 http_504;
    proxy_headers_hash_bucket_size 6400;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
      root html;
    }
    location / {

      proxy_ignore_client_abort on;
      proxy_pass http://smp-env-http-di1.19089.gts-seata-metrics.up/;
      proxy_connect_timeout 2s;
      proxy_set_header Host $http_host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_buffer_size 128k;
      proxy_buffers 4 256k;
      proxy_busy_buffers_size 256k;
    }
  }
}

```

## Haproxy 配置

```conf
global
  daemon
  log  127.0.0.1 local0 info
  tune.ssl.default-dh-param 2048
  maxconn  20000
  pidfile  /app/haproxy/run/haproxy.pid
  stats  socket /app/haproxy/lib/haproxy/stats
  tune.bufsize  131072
  user nginx
  group nginx
  tune.ssl.default-dh-param 2048

defaults
  log  global
  maxconn  10000
  mode  http
  option  dontlog-normal
  option  http-server-close
  retries  3
  stats  enable
  timeout  http-request 10s
  timeout  queue 1m
  timeout  connect 10s
  timeout  client 1m
  timeout  server 30m
  timeout  check 10s

listen Stats
  bind 0.0.0.0:10000
  mode http
  stats enable
  stats uri /
  stats refresh 5s
  stats show-node
  stats show-legends
  stats hide-version

frontend https_frontend
  #bind *:80
  mode http
  bind *:443 ssl crt  /app/haproxy/cert/chenzhijun.top.pem
  acl secure dst_port eq 443
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  #redirect scheme https if !{ ssl_fc }
  rspadd Strict-Transport-Security:\ max-age=31536000;\ includeSubDomains;\ preload
  rsprep ^Set-Cookie:\ (.*) Set-Cookie:\ \1;\ Secure if secure
  option httpclose
  default_backend web_server

backend web_server
  mode http
  balance roundrobin
  http-request add-header X-Forwarded-Proto https if { ssl_fc }
  cookie SERVERID insert indirect nocache
  server lb1 10.0.0.1:80 check port 80 inter 10s fastinter 2s downinter 3s rise 3 fall 3
  server lb2 10.0.0.2:80 check port 80 inter 10s fastinter 2s downinter 3s rise 3 fall 3

listen harbor-ceph
  bind 0.0.0.0:80
  mode tcp
  maxconn 4086
  server s1 oss.chenzhijun.top:80
```
