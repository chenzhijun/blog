---
title: RabbitMQ-消息中间件（一）
date: 2017-09-15 10:13:47
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## 消息中间件 RabbitMQ (一)

### 简述

RabbitMQ 是一个消息中间件，类似于传统的邮局，不过RabbitMQ充当了邮箱，邮局，邮递员的角色。和邮局不同的是，它使用消息接受，存储，发送二进制数据。

### RabbitMQ 的一些术语

`Producing`: 生产者，发送消息方，只发送消息。

`queue`: 消息队列，允许消息从RabbitMQ传送到应用程序，消息只能被存储在队列中。队列仅仅被宿主机的内存和磁盘空间限制，本质上它是一个大的缓存块。生产者可以往队列发消息，消费者也可以从队列中接受消息。

`Consuming`:消费者，类似于收信人，消费者程序通常是在等待接收消息。

> ps:生产者，消费者和中间件不应该在同一台宿主机上面，实际中大多数应用都不会这样做。

### "Hello World"
<!--more-->
Send.java,发布者发送消息

```java
package me.chenzhijun;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/15
 */
public class Sender {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost"); //连接本地的消息中间件，如果是其它机器换成ip就行了,也可以对连接进行授权，协议版本等控制
//        connectionFactory.setPassword();
        Connection connection = connectionFactory.newConnection();//socket连接, 大多数任务的完成都是调用connection的api
        Channel channel = connection.createChannel();

        //1 发送消息之前，必须先声明一个发送消息的队列，然后我们往队列里面发送消息
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);// 队列声明是幂等的，它只会在不存在的时候创建
        String message = "Hello,World";

        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());// 消息内容是一个字节数组，可以用使用任何编码

        //2 关闭
        channel.close();
        connection.close();

        System.out.println("[x] sent:" + message);

    }
}

```

Receiver.java,先要声明确定建立连接，因为需要等待接收消息

```java
package me.chenzhijun;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/16
 */
public class Receiver {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        //1 前面的操作都是类似，都是需要链接的一些配置
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        System.out.println("[X] receiving message");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                String message = new String(body, "UTF-8");
                System.out.println("[x] Received : " + message);
            }
        };
        channel.basicConsume(QUEUE_NAME,true,consumer);
    }
}
```

maven仓库包：

```
    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>3.5.7</version>
        </dependency>
    </dependencies>
```

使用Demo的时候需要先下载安装RabbitMQ，可能还需要安装erlang，安装完成后打开网站`localhost:15672`,就可以看到rabbitMQ的管理后台了。


参考文档：

[RabbitMQ Java](http://www.rabbitmq.com/tutorials/tutorial-one-java.html)





































































































