---
title: RabbitMQ-消息中间件（三）订阅与发布
date: 2017-09-22 10:53:40
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## 订阅与发布

之前的实例我们都是将任务精确的转发给某一个工作线程（消费者）。这节我们讲讲怎么将一个任务转发给多个消费者。这种模式就是闻名的“订阅/发布”模式。

我们的初步想法便是一个发布者发布了一个消息，多个消费者接收到消息，做不同的事情。

### 交换机(Exchanges)

前面已经说了了我们是通过队列来接受和发送消息的(p-q-c)[p:发布者，q:队列，r:消费者]。消息是由p 发布到q,q 再发给c。其实Rabbit又一个消息模型，在我们完成实例前，我们先来看看Rabbit的所有消息模型。

在Rabbit消息模型中最核心的点是生产者从不直接发送消息给队列。实际上，生产者都不知道生产的消息给了那个队列。

相反的，生产者只是给exchange(交换机)发送消息。交换机实际上是一个非常简单的东西，一边是从生产者接受消息，另一边就是将消息推送给队列。交换机必须明确的知道他们接到消息后该怎么处理消息：应该是将消息发送给一个明确的队列？还是应该给所有的队列？又或者是直接忽略不管？这些规则都是由交换机的类型去定义的。

![Rabbit消息模型](/images/qiniu/2017-09-22-11-12-44.png)

有一些可用的交换机类型是：`direc`,`topic`,`headers`,`fanout`。今天我们讲的类型主要是`fanout`。我们可以先声明一个类型：
<!--more-->
```java
channel.exchangeDeclare("exchange_name","fanout")
```

`fanout`交换机非常简单，就像它的名字一样（扇形交换机,广播），它收到消息后就广播给所有它知道的队列。

![扇形交换机](/images/qiniu/2017-09-22-11-52-29.png)

> listing exchanges 想知道服务器可以支持哪些交换机类型可以使用：`rabbitmqctl list_exchanges`

之前的例子中我们使用的是：`channel.basicPublic("","hello",null,message.getBytes());`,这种方式是没有声明`exchange_name`的。默认的话是使用一个无名交换机，消息经过特定的`routingKey`转发到队列。现在我们可以用我们自定的exchangeName来代替：

```java
channel.basicPublish("exchangeName","",null,message.getBytes());
```

### 临时队列

之前我们都是声明了特定的队列名称的,因为之前的消息模型中，我们需要将生产者和消费者指定到同一个队列，所以声明队列的名称对我们很重要。

但是在`fanout`本节中，我们需要的是的监听所有的消息，我们关注的是最新的消息，而不是旧数据。要解决这个，我们需要先做两件事：

1. 不管是什么时候我们连接到Rabbit 我们都需要一个空的，新队列。要做到这点，我们可以用一个随机名字来创建一个新队列，当然更好的是，让rabbitMQ帮我们选择一个随机名字。

2. 一旦我们的消费者消费完断开了和队列的连接，队列应该自己删除掉。

在Java中，当我们提供一个无参的方法：`queueDeclare()`,我们创建了一个不可持久化，独立的，可自动删除的带有随机生成名字的队列。

```java
String queueName = channel.queueDeclare().getQuere();
```

`queueName`就是我们生成的随机队列名，它可能像：`amq.gen-JzTY20BRgKO-HjmUJj0wLg`.

### 绑定

我们已经知道如何创建一个`fanout`类型的交换机和队列，现在我们需要告诉交换机发送消息给我们的队列。交换机和队列之间的这种关系就叫做绑定:

![交换机-队列关系图](/images/qiniu/2017-09-22-11-38-02.png)

> 列出所有已存在的绑定 `rabbitmqctl list_bindings`

全模型图为：

![发布-订阅关系图](/images/qiniu/2017-09-22-11-40-53.png)

### 实例编程

生产者程序和之前的代码产不多，最大的不同点就是我们现在想要发布消息给我们自己定义的`nameame`的exchange,而不再是之前无名的exchanage。现在当我们发送消息的时候，我们就必须提供一个`routingKey`,不过由于是`fanout`类型的exchange,它具有广播给所有的队列的作用。(routingKey的主要作用是在exchange和queue中做选择)。下面是生产者代码:

```java
package me.chenzhijun;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.Random;
import java.util.concurrent.TimeoutException;

/**
 * sudo rabbitmqctl list_exchanges
 *
 * @author chen
 * @version V1.0
 * @date 2017/9/22
 */
public class PublishProducer {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        String message = new Random().nextInt(100) + "";

        System.out.println("[sent] : " + message);

        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
        channel.close();
        connection.close();

    }
}
```

消费者代码：

```java
package me.chenzhijun;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/22
 */
public class SubcribeReceiver {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        Consumer consumer = new DefaultConsumer(channel) {

            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                String messsage = new String(body, "UTF-8");
                System.out.println("[x] Received : " + messsage+"...ooooooo");
            }
        };
        channel.basicConsume(queueName, true, consumer);

    }
}
```

特别注意，此例中的消费中要先启动。如果exchange没有绑定queue，那么交换机接收到消息会直接抛弃它。该例中，生产者只是声明了交换机，而不会创建队列。这种模型也跟我们之前说的也是一样的，生产者并不知道消息给了那个队列。

如果要多个消费者接收到不同的消息做不同的事情，那么就让消费者绑定到同一个exchange_name,然后监听就可以了。


#### 参考文档

[RabbitMQ part3](http://www.rabbitmq.com/tutorials/tutorial-three-java.html)