---
title: RabbitMQ-消息中间件（五）Topics
date: 2017-09-25 16:29:45
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## RabbitMQ-消息中间件（五）Topics

### Topics 前言

上一章我们写了个路由日志系统，用`direct`的放交换机，比较了`fanout`和它之间的区别，一个是广播，一个是绑定。
尽管我们使用`direct`交换机做了改进，但是它还是有一些限制，它没法根据不同的几个规则了区分。我们之前的日志是分了四种:info、debug、warn、error。但是我们可能实际中了，对error又分为系统错误或者业务错误。有时候我们针对业务开发，就不想去管理系统错误，那怎么办？
为了再一次提高我们的日志系统，我们需要学习另一种交换机：`topic`交换机。

### Topic exchange

发送给`top`交换机的消息不能随意定义`routing_key`,它必须是一串单词，并且用点`.`分开。这些单词可以任意定义，通常他们是描叙这些消息的共同特点。`routing key`示范例如：`stock.usd.nyse`,`nyse.vmw`,`quick.orange.rabbit`。你可以定义你喜欢的任何单词，不过记得总共不能超过`255 bytes`。
<!--more-->

`routing_key`定义好了，那么`binding_key`也必须和它保持同样的格式。`topic`得原理其实和`direct`是类似的：一个拥有特殊routingKey的消息，经过交换机转发给一个匹配它的bindingKey的队列。然而对于`topic`来说，有两个关于bingdingKey的重要场景：

1. \* (星号) 用来表示一个精确的单词；
2. # 用来表示0个或者多个单词；

可以看下图加深印象：

![关联图](/images/qiniu/2017-09-25-16-47-25.png)

在这个例子中，我们准备发送所有描叙动物的消息，这些消息的routingKey由三个单词组成(其中两个.)。第一个词在routeKey里面描叙速度，第二个表示颜色，第三个表示物种：`<speed>.<colour>.<species>`。

我们先创建三个`bingdingKey`,队列1用bindingKey：`*.orange.*`;队列2用bindingKey：`*.*.rabbit`和`lazy.#`。

这些队列可以被总结为：

* Q1 只对橙色动物感兴趣；
* Q2 对所有兔子和所有懒的动物感兴趣；

一个消息如果带有`quick.orange.rabbit`,会被转发给两个队列，如果是`lazy.orange.elephant`也会转发给两个队列，如果是`quick.orange.fox`会只转发给第一个队列Q1，如果是`lazy.brown.fox`会只转发给第二个队列Q2.`lazy.pink.rabbit`匹配了Q2的两个bingdingKey，但是它只会接收一次消息。`quick.brown.fox`不会匹配任何绑定，所以它会被丢弃。

如果我们打破下规则，用一个单词或者四个词的routKey，比如`orange`或者`quick.orange.male.rabbit`?消息会匹配不上任何的bingdingKey,然后会被丢失掉。

在另一方面，如果用的是`lazy.orange.male.rabbit`,就算它是四个单词，他也会匹配到bingdingKey为`lazy.#`,然后消息转发给Q2队列。

> Topic exchange是非常强大的交换机，并且可以像其他交换机一样工作。当一个队列只用`#`来作为bingdingKey，它会接受所有的消息，就相当于一个`fanout`交换机。当`*`和`#`没有在bingdingKey中使用，那么就相当于一个`direct`交换机。

### 代码实战

生产者代码：

```java
package me.chenzhijun.topic;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/25
 */
public class EmitLogTopic {
    private static final String EXCHANGE_NAME = "log_topic";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");//可以自定义
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        String routeKey = "errorMessage.server";//system.error; errorMessage.server
        String message = "key is " + routeKey + ",this is message";

        channel.basicPublish(EXCHANGE_NAME, routeKey, null, message.getBytes());
        System.out.println("message is send:" + message);
        channel.close();
        connection.close();

    }
}

```

消费者代码：
```java
package me.chenzhijun.topic;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/25
 */
public class EmitLogReceiver {

    private static final String EXCHANGE_NAME = "log_topic";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");


        String queueName = channel.queueDeclare().getQueue();
        String[] messageType = new String[]{"#", "system.*", "*.server"};
        for (String serverType : messageType) {
            channel.queueBind(queueName, EXCHANGE_NAME, serverType);
        }

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                System.out.println("[X] received,routeKey:" + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
            }
        };

        channel.basicConsume(queueName, true, consumer);

    }
}

```

控制台输出结果：

![2017-09-25-17-30-02](/images/qiniu/2017-09-25-17-30-02.png)

在管理后台查看channel，可以看到生成的三个队列图:

![生成的队列图](/images/qiniu/2017-09-25-17-28-06.png)

