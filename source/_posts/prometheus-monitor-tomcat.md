---
title: Prometheus 监控 Java 应用
copyright: true
date: 2018-05-09 21:16:35
tags: prometheus
categories: 监控
---

# Prometheus 监控 Java 应用

Prometheus 监控 Java 应用有两种方式：一种是使用官方提供的jar包，然后嵌入到应用中就可以了。这种方式一般都是新项目。我认为也是最合适的一种。不过这种情况一般是理想而已。而除了这种方式，第二种是prometheus的[jmx_exporter](https://github.com/prometheus/jmx_exporter)。

今天我们讨论的就是第二种。使用jmx_exporter的方式来监控我们的java应用程序。我们的java应用基本上是使用tomcat作为服务器的。这种情况下有两种方式，一种是基于springboot的jar包启动方式，一种是直接下载tomcat软件之后，将应用打成war包部署的方式。
<!--more-->
jmx_exporter的使用非常简单，但是如果不了解就会非常懵逼。jmx_exporter实际也是基于java的jmx通过暴露Mbean来做为代理，使用http的方式来给Prometheus进行指标采集。

## jar 包启动应用
如果是jar包启动的方式，那么github上面就已经有示例了。可以参照：`java -javaagent:./jmx_prometheus_javaagent-0.3.0.jar=9151:config.yaml -jar yourJar.jar`，这种方式启动。这种属于在应用启动的时候就给它加上代理。

这种方式是没有加认证的，如果需要加认证，嗯，有点麻烦，实验过一次，后来发现，还是算了。可能在虚拟机上直接运行程序还好，但是打成docker镜像就真的就有点多余了。

这种方式是监控的内嵌tomcat的启动的应用，在访问`http://ip:9151/metrics`,`/metrics`可有可无，这个时候可以看到很多tomcat指标，当然如果你的config.yaml没有改动，那么可能并不会看到，因为官网的[config.yaml](https://github.com/prometheus/jmx_exporter/blob/master/example_configs/tomcat.yml)中rules下的pattern:Catalina*****，这里是不适用与内嵌tomcat的。内嵌的tomcat需要修改为`Tomcat`。

```yaml
---
startDelaySeconds: 0
#hostPort: 192.168.226.128:8999
#jmxUrl: service:jmx:rmi:///jndi/rmi://127.0.0.1:8999/jmxrmi
ssl: false
wercaseOutputName: true
lowercaseOutputLabelNames: true
rules:
- pattern: 'Tomcat<type=GlobalRequestProcessor, name=\"(\w+-\w+)-(\d+)\"><>(\w+):'
  name: tomcat_$3_total
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat global $3
  type: COUNTER
- pattern: 'Tomcat<j2eeType=Servlet, WebModule=//([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), name=([-a-zA-Z0-9+/$%~_-|!.]*), J2EEApplication=none, J2EEServer=none><>(requestCount|maxTime|processingTime|errorCount):'
  name: tomcat_servlet_$3_total
  labels:
    module: "$1"
    servlet: "$2"
  help: Tomcat servlet $3 total
  type: COUNTER
- pattern: 'Tomcat<type=ThreadPool, name="(\w+-\w+)-(\d+)"><>(currentThreadCount|currentThreadsBusy|keepAliveCount|pollerThreadCount|connectionCount):'
  name: tomcat_threadpool_$3
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat threadpool $3
  type: GAUGE
- pattern: 'Tomcat<type=Manager, host=([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), context=([-a-zA-Z0-9+/$%~_-|!.]*)><>(processingTime|sessionCounter|rejectedSessions|expiredSessions):'
  name: tomcat_session_$3_total
  labels:
    context: "$2"
    host: "$1"
  help: Tomcat session $3 total
  type: COUNTER
- pattern: ".*"

```

当然，你也看到了我的pattern这里有个：`".*"`为啥这样？因为这样可以让所有的jmx metrics全部暴露出来，这方式有点暴力但是很好，很有用。

## Tomcat war包应用

war包应用部署就不说了，在bin目录启动`./startup.sh`就好了。但是如果要加上jmx_exporter。那么我们需要加上要给东西，进入bin目录（`$TOMCAT_HOME/bin`），将jmx_exporter.jar包文件和config.yaml文件复制到这里。然后修改里面的一个catalina.sh的脚本，找到`JAVA_OPTS`,加上代理：

```shell
JAVA_OPTS="$JAVA_OPTS $JSSE_OPTS"

JAVA_OPTS="$JAVA_OPTS -javaagent:$PWD/jmx_prometheus_javaagent-0.3.0.jar=9151:$PWD/config.yaml"
```

这种启动tomcat之后，你就可以通过访问9151端口来访问metrics了。这里要记得，修改上面的config.yaml中pattern的部分，改为和github上一样就可以了，也就是pattern为Catalina****。

## 监控远程的 tomcat

如果你不修改catalina.sh，可以采用在bin目录新建一个setenv.sh文件，`修改为可执行文件`。然后加入下面的设置：
```shell
#CATALINA_OPTS='-javaagent:/home/chen/jmx_prometheus_javaagent-0.3.0.jar=9151:/home/chen/config.yaml'
CATALINA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false'
#JAVA_OPTS='-javaagent:/home/chen/jmx_prometheus_javaagent-0.10.jar=9151:/home/chen/config.yaml'
#CATALINA_OPTS='-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.password.file=/home/chen/jmxremote.password'
```

这样在启动的时候tomcat就会暴露出8999端口并且打开metrics。上面是jmx不带认证的。

然后在另一端下载jmx_exporter，这里需要去下载`0.10`版本，在github 的release中找一下就好了。如果是远程的tomcat jmx，最新版本的代理反正是用不了，我的测试中`0.3.0`是用不了的，`0.10`是可以用的。

```java
java -javaagent:./jmx_prometheus_javaagent-0.10.jar=9150:config.yaml
```

相应的config.yaml需要做一些修改：

```yaml
---
startDelaySeconds: 0
hostPort: 192.168.226.128:8999
jmxUrl: service:jmx:rmi:///jndi/rmi://192.168.226.128:8999/jmxrmi
ssl: false
wercaseOutputName: true
lowercaseOutputLabelNames: true
rules:
- pattern: 'Tomcat<type=GlobalRequestProcessor, name=\"(\w+-\w+)-(\d+)\"><>(\w+):'
  name: tomcat_$3_total
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat global $3
  type: COUNTER
- pattern: ".*"

```

当然如果是springboot，现在有`spring-boot-starter-actuator`的包，是可以直接暴露metrics的，当然也可以引入prometheus提供的client jar。

实际中，还是的多动手，实践出真知。