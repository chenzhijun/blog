---
title: 编写属于自己的springboot-starter
date: 2018-01-31 10:52:56
tags:
    - Spring
Categories: Spring
---

# 编写属于自己的springboot-starter

最近手痒，想实现一下自己的starter，感概于springboot的starter导入方便，要是能有一款自己的starter，那样多么好，比起之前的maven导入方式，好像更有意思。
主要可以加重对springboot的自动配置装载的理解。

## 实现过程

starter的简易目录如下，"autoconfiguration","domain","service","META-INF"：
<!--more-->
![2018-01-31-11-03-47](/images/qiniu/2018-01-31-11-03-47.png)

这几个包的作用如下：
`domain`一般放置配置的参数，比如我们通常在`application.properties`中用到的`xxx.yyy=value`,其中`xxx.yyy`就是在这里配置的,前面的`xxx`我们一般指前缀，用以标识某个类级别，后面的`yyy`
一般会是指代属性，如果是驼峰标识的属性在properties文件中，默认为`-`连接。其中前缀的定义方式使用注解`@ConfigurationProperties`

![2018-01-31-11-12-21](/images/qiniu/2018-01-31-11-12-21.png)

`service`服务的实际配置使用，我们获取到参数之后，总要将参数用起来，那么参数配置才有价值，而这些都是在service中做处理，也就是业务处理。所以可以想象的到，在service类里面，我们肯定需要一个
domain的类来接收配置的参数。

![2018-01-31-11-16-39](/images/qiniu/2018-01-31-11-16-39.png)

`autoconfiguration`包其实就是我们的自动装载配置了，它需要告诉系统我们要让哪个配置类自动装载，spring的ioc要管理bean，前提就是这个bean要被加载到了容器中，所以我们需要注入配置类(domain)
使用@Configuration指明当前类是配置类，使用@EnableConfigurationProperties指明快速注册到spring中的类。

![2018-01-31-11-20-27](/images/qiniu/2018-01-31-11-20-27.png)

另外就是resource下的META-INF包下的`spring.factories`文件，这里说的是元信息，也是指明我们的配置启动类是在哪个目录，在其它的spring项目里面也可以看到这个文件。

![2018-01-31-19-33-18](/images/qiniu/2018-01-31-19-33-18.png)

现在我们看下pom文件我们导入的包：

![2018-01-31-19-35-17](/images/qiniu/2018-01-31-19-35-17.png)

我在这里加入了configuration-processor包，这种情况下能够方便我们在配置文件中使用我们自定义的前缀，然后进行提示。

![2018-01-31-19-37-34](/images/qiniu/2018-01-31-19-37-34.png)

注意到我这里有些jar包引入的时候使用optional，这种方式声明的包不会让jar包顺延，也就是在使用方我们仍需导入这个包才能使用。