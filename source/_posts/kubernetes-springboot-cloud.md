---
title: SpringBoot&SpringCloud 组件容器化方案
copyright: true
date: 2022-08-08 21:02:42
tags: Kubernetes
categories: Kubernetes
---

# SpringBoot&SpringCloud 组件容器化方案

该方案为 Java 开发人员在使用 springboot 开发以及是用 springcloud 组件时，将 组件/服务 进行容器化部署运行的方案。
<!--more--> 
## 0. 前置约定

    对 Java 开发时使用到的相关开发基础组件版本约定：

1. Java Version: 1.8+
2. SpringBoot Version: 2.3.5.RELEASE
3. SpringCloud Version: Hoxton.SR9 （请注意，springcloud 版本有 springboot 版本要求）
4. Openshift Version: 3.11，4.6
5. Kubernetes Version: 1.11（对应 ocp3），1.19（对应 ocp4）

# 1. SpringBoot 服务容器化

    目前 Java 项目开发多基于 springboot ，开发完成后多以嵌入 web 服务器的方式构建成 jar 包方式运行服务。容器化过程中，我们需要将其进行容器化。下面将以一个用户服务`user-app`进行示例：

1. 项目结构；
    
    其中 src 为我们的源代码目录；Dockerfile 文件可以存放到项目根路径下，也可以自定义，自定义目录时，请注意在构建时需要指定文件路径。
    
    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6baf9b1-9002-41c1-b721-702ce15208ea/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f6baf9b1-9002-41c1-b721-702ce15208ea/Untitled.png)
    
2. 业务镜像构建；
    
    通过在项目中增加 `Dockerfile` 文件可以将业务服务制作成一个业务镜像，`Dockerfile`模板内容为：
    
    ```yaml
    # registry-c.chenzhijun.top 为金科提供的镜像仓库服务
    # openjdk:11.0.10-jdk-oracle 为原生 Java 基础镜像，可按需求定制
    FROM registry-c.chenzhijun.top/base/openjdk:11.0.10-jdk-oracle
    # 业务镜像构建指令
    COPY target/*.jar /home/
    # 容器运行时的默认启动命令，实际运行中可覆盖
    # user-app-0.0.1-SNAPSHOT.jar 请替换为实际maven、gradle 构建时的 jar 包名称
    CMD ["java","-jar","/home/user-app-0.0.1-SNAPSHOT.jar"]
    ```
    
    请注意：在构建镜像前需要先生成 jar 包，之后再使用 `docker build -t [YOUR_IMAGE_NAME] .` 来生成业务镜像。
    
3. 业务部署
    
    在通过步骤 2 生成业务基础镜像后，接下来需要将其进行业务部署。业务部署的deployment 模板内容：`user-app.yaml`
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: user-app
      name: user-app
      namespace: springcloud
    spec:
      selector:
        matchLabels:
          app: user-app
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: user-app
        spec:
          containers:
            - command:
                - java
                - '-jar'
                - /home/user-app-0.0.1-SNAPSHOT.jar
                - '--spring.config.additional-location=file:./config/'
              env:
                - name: PROFILE_ACTIVE
                  value: prod
              image: 'registry-c.chenzhijun.top/czj/user-app:latest'
              imagePullPolicy: Always
              name: user-app
              volumeMounts:
                - mountPath: /home/config
                  name: volume-config
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
          volumes:
            - configMap:
                defaultMode: 420
                name: userapp-config
              name: volume-config
    
    ---
    apiVersion: v1
    data:
      application-dev.yml: |-
        spring:
          profiles: dev
        config: uat-config
        eureka:
          client:
            serviceUrl:
              defaultZone: http://eureka-1:8761/eureka/,http://eureka-2:8762/eureka/,http://eureka-3:8763/eureka/
          instance:
            prefer-ip-address: true
      application-prod.yml: |-
        spring:
          profiles: prod
        config: uat-config
        eureka:
          client:
            serviceUrl:
              defaultZone: http://eureka-1:8761/eureka/,http://eureka-2:8762/eureka/,http://eureka-3:8763/eureka/
          instance:
            prefer-ip-address: true
      application-uat.yml: |-
        spring:
          profiles: uat
        config: uat-config
        eureka:
          client:
            serviceUrl:
              defaultZone: http://eureka-1:8761/eureka/,http://eureka-2:8762/eureka/,http://eureka-3:8763/eureka/
          instance:
            prefer-ip-address: true
    kind: ConfigMap
    metadata:
      name: userapp-config
      namespace: springcloud
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: user-app
      name: user-app
      namespace: springcloud
    spec:
      ports:
      - port: 8080
        protocol: TCP
        targetPort: 8080
        name: user-app
      selector:
        app: user-app
    status:
      loadBalancer: {}
    ```
    
    通过 `configmap` 来控制相关的配置时，我们需要制定以下 config 的位置。实际运行中的文件目录结构如下：
    
    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/328e4f73-992b-49a3-8e50-6a7c368f8438/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/328e4f73-992b-49a3-8e50-6a7c368f8438/Untitled.png)
    

 4.  检查部署结构

    单体的 springboot 服务结合 kubernetes 的 configmap 其实际效果不是特别大，因为明文的 configmap 可以直接在 springboot 项目中多放置配置文件使用。但在实际中，我们也可以使用 kubernetes 的 secret 来进行配置的简单加密，部署方式也是类似，只是将 configmap 改为 secret 即可。

    完整示例代码可以参考：[http://git.dev.cmrh.com/springcloud/user-app.git](http://git.dev.cmrh.com/springcloud/user-app.git)

## 2. SpringCloud 组件的容器化

    SpringCloud 全家桶中有非常多的组件，有些组件需要结合客户端使用，有些是属于独立中间件，现挑选其中常见组件进行容器化示例。示例组件包含：注册中心 Eureka，网关 Zuul，熔断监控 Hystrix、Turbine，微服务管理 SpringCloud Admin，客户端负载均衡器 Robbin。

     请注意，示例中 SpringCloud 版本为：Hoxton.SR9；对应的 Springboot 版本为：2.3.5.RELEASE。

### 2.1 Eureka 服务容器化

1. Eureka 高可用架构；
    
        Eureka 原生高可用方案为部署三个 Eureka 节点，节点直接互相注册来达成高可用；在容器化过程中，我们通过部署独立的三个 Eureka 实例：eureka-1，eureka-2，eureka-3；三实例互相注册来达成高可用。
    
2. Eureka 高可用部署；
    
        Eureka 作为通用的组件，可以使用通用的部署方式：
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: eureka-1
      name: eureka-1
      namespace: springcloud
    spec:
      replicas: 1
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          app: eureka-1
      template:
        metadata:
          labels:
            app: eureka-1
        spec:
          containers:
            - env:
                - name: PROFILE_ACTIVE
                  value: eureka-1
              image: 'registry-c.chenzhijun.top/czj/eureka-server:latest'
              imagePullPolicy: Always
              name: eureka-1
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: eureka-2
      name: eureka-2
      namespace: springcloud
    spec:
      replicas: 1
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          app: eureka-2
      template:
        metadata:
          labels:
            app: eureka-2
        spec:
          containers:
            - env:
                - name: PROFILE_ACTIVE
                  value: eureka-2
              image: 'registry-c.chenzhijun.top/czj/eureka-server:latest'
              imagePullPolicy: Always
              name: eureka-2
              resources: {}
              terminationMessagePath: /dev/termination-log
              terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: eureka-3
      name: eureka-3
      namespace: springcloud
    spec:
      replicas: 1
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          app: eureka-3
      template:
        metadata:
          labels:
            app: eureka-3
        spec:
          containers:
            - env:
                - name: PROFILE_ACTIVE
                  value: eureka-3
              image: 'registry-c.chenzhijun.top/czj/eureka-server:latest'
              imagePullPolicy: Always
              name: eureka-server
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: eureka-1
      name: eureka-1
      namespace: springcloud
    spec:
      ports:
      - port: 8761
        protocol: TCP
        targetPort: 8761
      selector:
        app: eureka-1
    status:
      loadBalancer: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: eureka-2
      name: eureka-2
      namespace: springcloud
    spec:
      ports:
      - port: 8762
        protocol: TCP
        targetPort: 8762
      selector:
        app: eureka-2
    status:
      loadBalancer: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: eureka-3
      name: eureka-3
      namespace: springcloud
    spec:
      ports:
      - port: 8763
        protocol: TCP
        targetPort: 8763
      selector:
        app: eureka-3
    status:
      loadBalancer: {}
    ```
    
3. Eureka 访问验证
    
        在 ocp 平台通过创建 route 来将 Eureka 服务暴露给业务使用。具体创建方式参考《OpenShift 基础服务操作指南-金科》[http://confluence.cmrh.com/pages/viewpage.action?pageId=81213827](http://confluence.cmrh.com/pages/viewpage.action?pageId=81213827)
    
4. 项目使用 Eureka 
    
        在与 Eureka 同 Namespace 下的服务中，在springboot 项目中增加如下配置：
    
    ```yaml
    eureka:
      client:
        serviceUrl:
          defaultZone: http://eureka-1:8761/eureka/,http://eureka-2:8762/eureka/,http://eureka-3:8763/eureka/
      instance:
        prefer-ip-address: true
    ```
    

### 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-app #需要改动
  namespace: cmft-user-app #与 project 保持一致
spec:
  selector:
    matchLabels:
      app: user-app #跟metadata.name保持一致
  replicas: 3
  template:
    metadata:
      labels:
        app: user-app
    spec:
      containers:
        - name: user-app #修改为项目名就行
          image: registry-c.chenzhijun.top/czj/user-app:latest #修改为实际的 image-name
          env:
          - name: PROFILE_ACTIVE
            value: prod
          - name: http_proxy
            value: proxy_url
          ports: #修改为实际的监听端口
            - containerPort: 8080 
          command: ["java","-xmx","-xms","-jar","/home/user-app-0.0.1-SNAPSHOT.jar"]
```