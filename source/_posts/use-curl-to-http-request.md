---
title: 使用CURL发送http请求
date: 2018-01-09 10:41:16
tags:
    - CURL
categories: CURL
---

# 使用CURL发送http请求

之前都是postman发送数据，但是在服务器上有时候没法用postman连接，其实挂VPN是可以的，但是为了显示b格(好吧，其实是被逼而已)使用curl发送http请求。

其实curl可以做的事情非常多，我们这里仅仅只是用到了一小一小小部分。

发送get请求：`curl localhost:8888/path/to/api`

发送post请求带json数据：

```shell
curl "http://localhost:8888/path/to/api" -H "Content-Type: application/json" --data '{"pageNumber":1,"pageSize":10}'
```

当然还有一招，你可以使用火狐的控制台，查看网络，选中接口，复制为CURL。嘿嘿~~~