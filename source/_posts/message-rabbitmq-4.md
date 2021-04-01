---
title: RabbitMQ-消息中间件（四）路由
date: 2017-09-25 11:10:48
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## RabbitMQ-消息中间件（四）路由

### 路由

上面一节我们用交换机`fanout`的方式进行了广播，这种方式如果在不需要对消息进行区分是没关系的，可以使用。但是有些场景下，比如日志打印，对于一些error的日志，我可能需要的是要保存到磁盘中进行持久化，而对于一些warn，info的日志，我可能只是想着输出下就可以了。那么这种情况下我们还用fanout的方式可能就不行了，我可能想要的是某一个队列接受特定的某一类消息做特别处理。比如我想要一个error队列专门来监听error的消息做打印，其它的队列就不管error的消息了。

### 绑定(Bingdings)

在前面，其实我们已经创建过“绑定”了。比如我们使用到了：`channel.queueBind(queueName, EXCHANGE_NAME, "");`,“绑定”：中文意思就是你和我之间有些关联，我们之间有关系；那么MQ也是，它的关联是`队列`和`交换机`，它绑定的是队列和交换机的关系，也就是说，这个队列对这个交换机感兴趣，会接受这个交换机的信息。但是接受什么消息了？这个可以在绑定的时候用另外一个参数来设置，也就是`路由`：俗成绑定的key。`channel.queueBind(queueName, EXCHANGE_NAME, "binding key,set by yourself");`。这里我们可以看到，`绑定key`是否生效其实是跟`交换机`类型相关的，如果交换机是`fanout`，那就跟`key`没关系了，因为它是广播，只要有队列绑定到我交换机，产生了关系，我就懒的管你要不要，直接给你。这个有点类似中国爸妈，你是我孩子，只要我想给，你不要也得要，哈哈。

<!--more-->

### Direct 交换机

前面章节，我们采用的`fanout`交换机，它是给所有队列发送消息。很明显在我们的日志系统需求里面不太适合，我们希望对消息进行一次过滤，过滤之后再发送给不同的队列做处理，比如有的队列接受消息去打印，有的去写磁盘等等。所以请注意了*如果你不想管事，那就用fanout，把消息给你，你自己去做处理，直观无脑给消息就行，其他的就不用关心了。*。用不了`fanout`，那我们用什么了？rabbitMQ有四种交换机，我们可以选择`direct`。`direct`交换机背后的算法规则很简单，队列的`bingding key`和消息的`routing key`那个能精确匹配就给队列发消息。

![direct 关联图](/images/qiniu/2017-09-25-11-37-27.png)

图中非常简单，两个队列同时绑定了`direct`交换机，Q1的范围是根据`bingding key`为orange的值得消息。它表明，如果消息中的`routing key` 为orange，那么消息就发送给Q1。Q2绑定了两个：black、green，如果消息中有`routing key `为black和green的都发送个Q2。那么如果消息没有routing key,或者队列没有bingding key，那么这种消息，交换机会直接丢弃。


### 多绑定

![多绑定图](/images/qiniu/2017-09-25-11-43-22.png)

多个队列绑定同一个`bingding key`,在rabbitMQ中是合法的。如上图中，如果是这种形式，那么`direct`交换机的作用就是类似`fanout`，不过只有当消息的`routing key`和`bingding key`一直，上图中是消息如果路由key是black，那么交换机就会将消息分发给Q1和Q2两个队列。


### 实践: 发送日志(生产者)

现在回到我们之前的问题，如果我们想要不同的队列，比如Q1将日志写入磁盘，Q2打印warn日志，Q3接收info日志。我们可以使用`drect`交换机，然后给不同的队列绑定`bingding key`。所以对于我们的生产者端：

1. 声明交换机还是跟之前的类似，我们必须先声明一个交换机：

```java
channel.exchangeDeclare(EXCHANGE_NAME,"direct");
```

2. 传递消息：

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

### 订阅(Subscribing)

消费者对消息的接收和之前的有一个不同，我们用bingding key来绑定消费者只对感兴趣的消息进行接收：

```java
String queueName = channel.queueDeclare().getQueue();
for(String serverType : msgType){ //serverType,每一个queue绑定一个消息类型
    channel.queueBind(queueName,EXCHANGE_NAME,serverType);
}
```

![实际绑定图](/images/qiniu/2017-09-25-13-54-52.png) 

### 代码

生产者，在发送消息的时候特别注意`routing key`,在此例子中也就是我们for循环里面的key：

```java
package me.chenzhijun.route;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/25
 */
public class EmitLogDirect {
    private static final String EXCHANGE_NAME = "log_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");

        Map<String, String> map = new HashMap<String, String>();
        map.put("warn", "this is message of warn .....");
        map.put("debug", "this is message of debug .....");
        map.put("info", "this is message of info .....");
        map.put("error", "this is message of error .....");

        for (String key : map.keySet()) {
            channel.basicPublish(EXCHANGE_NAME, key, null, map.get(key).getBytes());
            System.out.println(EXCHANGE_NAME + ",serverType:" + key + ",message:" + map.get(key));
        }

        channel.close();
        connection.close();

    }
}

```

消费者,这里我们使用的是两种方式，第一种是全部绑定到一个队列，另外注释的是开启4个队列，对不动的消息做不同的处理：

```java
package me.chenzhijun.route;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/25
 */
public class ReceiveLogDirect {

    private static final String EXCHANGE_NAME = "log_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");

//        String[] messageType = new String[]{"warn", "info", "debug", "error"};
//        for (String serverType : messageType) {
//            String queueName = channel.queueDeclare().getQueue();
//            channel.queueBind(queueName, EXCHANGE_NAME, serverType);
//            Consumer consumer = createConsumer(serverType, channel);
//            boolean isAck = true;
//            channel.basicConsume(queueName, isAck, consumer);
//        }
        String queueName = channel.queueDeclare().getQueue();
        String[] messageType = new String[]{"warn", "info", "debug", "error"};
        for (String serverType : messageType) {
            channel.queueBind(queueName, EXCHANGE_NAME, serverType);
        }

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("[x] received,routeKey: " + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
            }
        };
        boolean isAck = true;
        channel.basicConsume(queueName, isAck, consumer);



    }

    private static Consumer createConsumer(String serverType, Channel channel) {
        if (serverType.equals("warn")) {
            return new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("[x] received,routeKey: " + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
                    System.out.println("warn消息我们只是打印.....:" + new String(body, "UTF-8"));
                }
            };
        } else if (serverType.equals("info")) {
            return new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("[x] received,routeKey: " + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
                    System.out.println("info消息我们可以忽略.....:" + new String(body, "UTF-8"));
                }
            };
        } else if (serverType.equals("debug")) {
            return new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("[x] received,routeKey: " + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
                    System.out.println("debug消息我们只做调试.....:" + new String(body, "UTF-8"));
                }
            };
        } else if (serverType.equals("error")) {
            return new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    System.out.println("[x] received,routeKey: " + envelope.getRoutingKey() + ",message:" + new String(body, "UTF-8"));
                    System.out.println("error消息很重要，存磁盘.....:" + new String(body, "UTF-8"));
                }
            };
        }

        System.out.println("其它消息不管了。。。");
        return null;
    }
}

```

参考文档：

[RabbitMQ Routing](http://www.rabbitmq.com/tutorials/tutorial-four-java.html)