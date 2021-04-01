---
title: nginx-http-1.1
copyright: true
date: 2019-12-18 21:57:00
tags:
categories:
---

# Nginx Http 1.1 Host规范

最近发现一个异常问题。一个web服务挂载在nginx后端，然后client通过socket连接后，构建成http-post请求，发现最后nginx会直接返回499，发现其实是客户端断链接，其实应该不对。http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_client_abort


之后的是http1.1的规范。https://tools.ietf.org/html/rfc2616#section-14.23 http1.1中，如果Host为空会直接返回400的状态码