---
title: SpringCloud 集成 Consul
copyright: true
date: 2019-05-31 21:10:57
tags: consul
categories: SpringCloud
---

# SpringCloud 集成 Consul

SpringBoot ，SpringCloud 可以说是在 Spring 里面最流行的，确实它的上手要比以前开发真的方便很多，约定优于配置。

springcloud可以理解成基于springboot的各种解决方案。

## 集成consul

我们没有使用eureka作为注册中心，而是使用consul，为什么了？因为eureka停止更新了。嗯，是的，如果没有人继续维护了，那我就觉得使用起来就会有局限性，这个不是在前期，而是在后期。而且官方都不更新维护了，以后我发现bug，都没有提PR的地方了~~~~嘿嘿。其实个人觉得注册中心以后可能会下沉，比如现在的k8s，就已经在底层平台解决了服务发现与注册的问题。当然那是扯远了，我们暂时还是先用consul做我们的注册中心，毕竟一套k8s也并不是那么好玩的。

在官网下载consul的安装包，然后使用`consul agent -dev`我们就可以在通过`http://IP:8500`端口来访问了。
<!--more-->
## Springcloud 集成consul

### 服务提供方

我们的代码结构如下：

![2019-05-31-21-09-48](/images/qiniu/2019-05-31-21-09-48.png)

我们使用springcloud提供的`spring-cloud-starter-consul-discovery`,这是集成了consul的starter，完整的`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.chenzhijun</groupId>
    <artifactId>spring-consul-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-consul-server</name>
    <description>consul project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.22</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

然后看下我们的`application.yml`:

```yaml

server:
  port: 8080
logging:
  level:
    org.springframework.cloud.consul: DEBUG
management: # 需要一个health端口来让consul回调
  endpoints:
    web:
      base-path: /admin
  server:
    servlet:
      context-path: /admin
    port: 18080
spring:
  application:
    name: user-app
  cloud:
    consul:
      host: localhost # consul 地址
      port: 8500
      discovery:
        instance-id: user-app # 注册到consul的名字
        management-port: 18080 # consul会来访问这个端口+health-check-path 来判断应用是否正常
        health-check-path: ${management.server.servlet.context-path}${management.endpoints.web.base-path}/health
```

在服务提供方我们的代码如下：

```java
package me.chenzhijun.consul;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
@EnableDiscoveryClient
public class ConsulApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsulApplication.class, args);
    }

    @RequestMapping("/hello")
    public String home() {
        return "Hello World123";
    }

}

```

我们这里需要使用`@EnableDiscoveryClient` 这样就能让项目注册到consul了。

### 服务调用方

既然是注册中心，我们有了服务提供方，当然需要服务调用方啦。调用方的结构如下：

![2019-05-31-21-31-41](/images/qiniu/2019-05-31-21-31-41.png)

pom文件的内容类似`pom.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>me.chenzhijun</groupId>
    <artifactId>spring-consul-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-consul-client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

`application.yml`:

```yaml
spring:
  application:
    name: user-client
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        instance-id: user-client
        health-check-path: ${management.endpoints.web.base-path}/health
server:
  port: 8088

logging:
    level:
      org.springframework.cloud.consul: DEBUG
management:
  server:
    port: 18088
  endpoints:
    web:
      base-path: /admin
```

我们再看`Application.java`的内容：

```java
package me.chenzhijun.calltest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import java.net.URI;
import java.util.List;

@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class CalltestApplication {

    public static void main(String[] args) {
        SpringApplication.run(CalltestApplication.class, args);
    }

    @Autowired
    private RestTemplate restTemplate;

    //restTemplate 必须使用@LoadBalanced创建
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/abc")
    public String test() {
        List<ServiceInstance> list = discoveryClient.getInstances("user-app");
//        if (list != null && !list.isEmpty()) {
//            String serviceId = list.get(0).getServiceId();
//            URI uri = list.get(0).getUri();
//            System.out.println(uri.toString());
//            String url = "http://" + list.get(0).getHost() + ":" + list.get(0).getPort() + "/hello";
//            RestTemplate restTemplate = new RestTemplate();
//            String result = restTemplate.getForEntity(url, String.class).getBody();
//            ResponseEntity<String> forEntity = restTemplate.getForEntity(url, String.class);
//            System.out.println(forEntity.getBody());
//        }

        // 这里我们使用的是服务名user-app直接调用，注意这里的resttemplate一定要用@Bean @LoadBalanced 不然会报错
        ResponseEntity<String> forEntity = restTemplate.getForEntity("http://user-app/hello", String.class);
        return forEntity.getBody();
    }
}
```

之后在浏览器里面访问`http://localhost:8088/abc`就能看到返回`Hello World123`了。

ps:
1：为什么RestTemplate必须要使用@LoadBalanced ?
2：springboot actuator starter 能否去掉，去掉的话应该怎么实现？
3：你觉得独立的注册中心未来的路会是怎样？

今天5月31日，改日回答，[记住来看答案](http://chenzhijun.top/2019/05/31/springcloud-consul-integration/)。