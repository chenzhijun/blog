---
title: vuejs 从拷项目到开发上线
copyright: true
date: 2019-04-08 21:56:22
tags: vuejs
categories: vuejs
---

# vuejs 从拷项目到开发上线

最近组里缺人手，我们都是一堆后端，却没有一个前端。所以你看到了一堆的后台接口，但就是没有一个页面。囧~
本着求人不如求己，活人不能被尿憋死不是~所以准备自己开始干。

## 前端选型

首先想了一下，我们应该怎么开发前端了？使用thymeleaf这种貌似是最合适的，毕竟类似于以前用jsp，对不对~当然有点不一样。
但后来想想，都前后端分离了，以后我们要真的做牛逼了。要是给我们分配个前端，那我们不又要改死啊。
然后觉得前后端不分离，逼格不够，也不好维护。那好咯，那就定好方向前端分离。

前后端分离的方向定好，那就先想想是自己完全自己弄一个，还是直接用模板套一个？一致觉得，自己从头写，费时费力，一群后端去弄不太现实，毕竟还是有很多要调的，适配啊，排版啊。所以投票决定，选模板。选了模板，那就好弄了，github上找了前十个优秀的模板。有一些是基于bootstrap，然后每个页面都是html，所以这种就相当于用jQuery或者javascript用ajax这种吧。想想就被以前写dom支配的恐惧。后来了解到有个vue，人家都说好用，其实公司还有很多都是vue，我们想了想，要是用bootstrap那种估计就是靠我们自己了。要是vue，要是我们实际搞不定，在公司也好请外援。想了想，那就找vue的模板吧。找来找去，找了个最简单实用的。毕竟我们需求很简单，列表显示，表格添加，搜索框就好了，后管管理页面是真的好搞啊。找了个最简单的功能又全的，感谢github，感谢可爱的爱分享的工程师们。

## vue 使用

模板选好了，这下来到重头戏了。大家都不会啊，以前都是听说，我们前端用vue，特么到底怎么用啊。不过既然选好了方案，选好了模板，其实就很明确了，不会那就学啊。vue官网一翻，一遍浏览。大概的使用方式也是了解了。

### vue 数据绑定

vue的模式感觉跟以前的不一样，看整个项目发现其实就一个html页面，也就是index.html，然后就是在里面有个APP.vue，通过不同的xx.vue来选择相应的页面，其实就是他们口中的单页式应用。就是一个html页面，然后根据不同的path来显示不同的内容。另外需要转换一个观点，vue不像传统的面向页面来编程，因为它的数据绑定感觉太好用了。如果是在html标签中只需要用`{{ key }}`就可以使用在js中定义的key的值。有几个好用的标签：

感觉吧，说的不太全，还是要实际用起来才有那种感觉。[模板地址](https://github.com/chenzhijun/vue-manage-system)


## 使用中遇到的问题

在实际使用中遇到一些问题，一开始有点懵逼，不过到后来都解决了。

### 本地开发-跨域问题

这个一开始的时候有点蒙圈，本地是使用`npm run dev`启动的，启动后发现，跨域了。这个该怎么搞哦。后来在vue.config.js中发现一个文件：

```js
module.exports = {
    baseUrl: './',
    productionSourceMap: false,
    devServer: {
        proxy: {
            '/m-paas':{
                target: 'http://localhost:8080/m-paas',
                changeOrigin: true,
                pathRewrite:{
                    '/m-paas': ''
                }
            }
        }
    }
}
```

其实可以看到只要打开changeOrigin 就可以了。其它的都不需要改动。

### 部署

开发完了，代码也写好了。重点来了，怎么部署到服务器？来跟我三步走

1. step1，构建应用

使用`npm run build`先进行构建，完成后会在根路径下生成一个dist文件夹。

2. step2，准备nginx服务器

下载一个nginx，现在都是使用nginx来做前端的服务器。将刚刚的dist文件夹的内容copy到服务器某个路径。
比如`/app/html`，或者就是nginx目录下的html。随自己开心就好

3. step3，修改nginx配置

目录确定后，我们就需要修改nginx的配置了。如下：
```conf

 server {
        listen       127.0.0.1:6000;
        #server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /m-web/ {
           root   /app/html/dist;
           index  index.html index.htm;
        }
        location /m-api {
            proxy_pass http://localhost:8080/m-api;
            #proxy_pass http://api-m.chenzhijun.me/m-api;
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

然后启动 nginx 就好了。
