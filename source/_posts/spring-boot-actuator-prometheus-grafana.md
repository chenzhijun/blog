---
title: Spring Boot 应用可视化监控
date: 2017-11-02 16:09:10
tags:
	- Java
categories: Java
---

# Spring Boot 应用可视化监控

> 使用spring-actuator 并且使用prometheus, grafana 做可视化视图展示

总体过程图：

![2017-11-02-17-21-51](/images/qiniu/2017-11-02-17-21-51.png)

## 监控

### SpringBoot 应用监控
SpringBoot 其实也整合了 ops 的功能，也就是运维的部分能力。通过引入包`spring-boot-starter-actuator`来监控相关的指标信息,详情文档：[Actuator 介绍](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready)。另外在新版本的`actuator`中已经有了加密信息，所以对于一些信息的获取可能需要授权，因此我们还需要引入`spring-security`,pom 文件如下：
<!--more-->
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
```

当然既然引入了`spring-security`,我们就需要对其做一些配置，我的完整配置是这样的：

```propeties

# 应用的api端口
server.port=8818
# 启用基础认证
security.basic.enabled = true

# 安全路径列表，逗号分隔，此处只针对/admin路径进行认证
security.basic.path = /admin

# 认证使用的用户名
security.user.name = admin

# 认证使用的密码。 默认情况下，启动时会记录随机密码。
security.user.password = 123456

# 可以访问管理端点的用户角色列表，逗号分隔
management.security.roles = SUPERUSER

# actuator暴露接口使用的端口，为了和api接口使用的端口进行分离
management.port = 8099

# actuator暴露接口的前缀
management.context-path = /admin

# actuator是否需要安全保证
management.security.enabled = true

# actuator的metrics接口是否需要安全保证
endpoints.metrics.sensitive = false

# actuator的metrics接口是否开启
endpoints.metrics.enabled=true

# actuator的health接口是否需要安全保证
endpoints.health.sensitive=false

# actuator的health接口是否开启
endpoints.health.enabled=true

```

### 指标采集

采集应用的指标信息，我们使用的是`prometheus`,相应的我们引入包:

```xml
    <dependency>
        <groupId>io.prometheus</groupId>
        <artifactId>simpleclient_spring_boot</artifactId>
        <version>0.0.26</version>
    </dependency>
```

之后在程序中开启相应的配置：

```java
@SpringBootApplication
@EnablePrometheusEndpoint
@EnableSpringBootMetricsCollector
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

这个时候我们可以开始启动我们的应用程序，并且访问相关接口:[http://localhost:8099/admin/prometheus](http://localhost:8099/admin/prometheus)

输入 properties 文件中的账号密码，就能看到下图：
![2017-11-02-16-26-20](/images/qiniu/2017-11-02-16-26-20.png)

## 数据收集

我们采集了指标信息之后就可以开始数据收集了，这个时候我们需要用到`Prometheus`工具，注意这里是工具，不再是 jar 包了。我使用的是 prometheus 的 docker 镜像，当然你也可以根据需要自己选择,先准备一份 Promethus 的配置文件,更多的配置文档请查看：[配置文档](https://prometheus.io/docs/operating/configuration/)：
```yaml
global:
  scrape_interval: 10s
  scrape_timeout: 10s
  evaluation_interval: 10m
scrape_configs:
  - job_name: spring-boot
    scrape_interval: 5s
    scrape_timeout: 5s
    metrics_path: /admin/prometheus
    scheme: http
    basic_auth:
      username: admin
      password: 123456
    static_configs:
      - targets:
        - 192.168.11.54:8099
```

之后我们准备服务：

```shell
docker run -d --name prometheus -p 9090:9090 -v D:\chenzhijun\test\actuator\prometheus\prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

```

请注意，`D:\chenzhijun\test\actuator\prometheus\prometheus.yml` ，是我的配置文件存放地址，我们需要将它放到容器里面去，所以用了`-v`来做文件映射。`/etc/prometheus/prometheus.yml`这个是容器启动的时候去取的默认配置，这里我是直接覆盖掉了它。`prom/prometheus`这是镜像，如果本地没有，就回去你设置好的镜像仓库去取。

启动完成后用`docker ps`看下是否已经启动成功，之后打开浏览器输入：
`http://localhost:9090/targets`,如果看到下图就是成功了：
![2017-11-02-16-40-26](/images/qiniu/2017-11-02-16-40-26.png)

> ps: 这里需要注意一点，我们在`prometheums.yml`中使用的IP地址一定要准确，因为我是docker访问的，所以我使用的是宿主机的地址

## 数据可视化展示

同样的我也是使用 docker ：

```shell
    docker run --name grafana -d -p 3000:3000 grafana/grafana
```

成功之后访问：[http:localhost:3000](http:localhost:3000)，输入账号密码：`admin/admin`。
之后就开始配置 grafna。

1. 先配置数据源,这里稍微注意下 ip 地址
![2017-11-02-16-49-26](/images/qiniu/2017-11-02-16-49-26.png)

2. 新建 dashboard
![2017-11-02-16-51-04](/images/qiniu/2017-11-02-16-51-04.png)

3. 配置图形数据
![2017-11-02-16-51-28](/images/qiniu/2017-11-02-16-51-28.png)

4. 选择指标,这里的指标数据只能是`promethues`采集到了的数据[http://localhost:9090/graph](http://localhost:9090/graph):

![2017-11-02-16-52-58](/images/qiniu/2017-11-02-16-52-58.png)

4.1. `prometh`采集的数据[http://localhost:9090/graph](http://localhost:9090/graph)

![2017-11-02-16-54-12](/images/qiniu/2017-11-02-16-54-12.png)

5. 最终结果

![2017-11-02-17-03-01](/images/qiniu/2017-11-02-17-03-01.png)

## 源码

[actuator 源码](https://gitee.com/chenzhijun/actuator)

## 参考资料

[SpringBoot 应用监控踩坑集锦](http://www.jianshu.com/p/ebee9a0dc15c)

[Spring Boot 应用可视化监控](http://www.jianshu.com/p/7ecb57a3f326)

[prometheus_started](https://prometheus.io/docs/introduction/getting_started/)

[Grafana](http://docs.grafana.org/installation/docker/)