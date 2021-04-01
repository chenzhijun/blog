---
title: RabbitMQ-消息中间件（六）RPC
date: 2017-09-26 14:44:24
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## RabbitMQ-消息中间件（六）RPC

### Remote Procedure Call (RPC)

第二节讲过可以将耗时的任务通过工作队列给多个工作线程，如果我们想要调用一个在远程其它服务器上的一个功能并且等待它执行完后的结果，我们应该怎么做？是的，这是一个不同的场景，这种模式被叫做`Remote Procedure Call`或者说是`RPC`。

在本节中，我们准备使用RabbitMQ来构建一个RPC系统。一个客户端和一个可扩展的RPC服务端。如果我们没有耗时的任务，那么就值得转发，我们准备建造一个仿RPC服务，并且它返回一个斐波那契数。

### 客户端接口

为了演示一个RPC服务是怎样被调用，我们准备创建一个简单的客户端。它暴露出一个叫`call`的方法接口，通过这个接口发送RPC请求，并且等待返回结果。
<!--more-->
```java
FibonacciRpcClien fibonacciRpc = new FibonacciRpcClient();
String result = fibonacciRpc.call("4);
System.out.println("result:"+result);
```

> RPC tips:
> 尽管RPC是一个在计算中非常通用的模式，它还是经常被喷。最明显的问题就是程序员不知道一个被调用的方法到底是本地方法或者是一个很慢的RPC服务。在一个复杂的系统中造成的结果非常让人迷惑，并且非增加调试时候不必要的复杂性。滥用RPC可能会导致代码异常复杂混乱。
> 如果一定要使用的话，参考下面的建议：
> 1 明确知道被调用的方法是本地的还是远程的
> 2 一定要为系统编写文档，确保确保组件之间的依赖非常清晰
> 3 处理异常情况，当RPC 服务长时间宕机了，客户端应该怎样操作？
> 如果可以的话，尽量避免使用RPC。你应该使用异步的方式来替换像RPC这种同步的。在下一个阶段异步推回结果

### 回调队列

总的来说在RabbitMQ上做RPC是非常简单的。一个客户端发送一个请求的消息，然后服务端返回响应的消息。为了接收到响应，我们需要在发送请求消息的时候附带一个回调队列地址。我们可以使用默认的队列：

```java
callbackQueueName=channel.queueDeclared().getQueue();

BasicProperties props = new BasicProperties.Builder().replyTo(callbackQueueName);

channel.basicPublish("","rpc_queue",props,message.getBytes());
```

> Message properties . AMQP 0-9-1 协议预先定义了14个关于消息的属性。大多数属性使用的很少，一些特别的比如：
> deliveryMode: 标记一个消息作为persistent(持久化默认值为2),或者transient(非2的值)。
> contentType： 用来描叙mime-type 编码格式。例如经常使用的JSON编码：application/json。
> replyTo: 通常用来命名一个回调队列。
> correlationId: 用来关联RPC响应和请求。

我们使用的时候需要导入：`import com.rabbitmq.client.AMQP.BasiProperties;`.

### Correlation Id （关联 ID）

在上面提出的方法中，我们建议未每一个RPC请求创建一个回调队列。它是非常低效的。但是幸运的是我们有一个更好的方式，我们可以为每一个客户端创建一个回调队列。

尽管我们用为客户端的形式替换了每一个rpc请求，但是它还是带来了新的问题，队列收到响应后无法知道这个响应应该属于哪个请求。现在我们就可以使用`correlationId`属性了。我们为每一个请求设置一个不同的correlationId。然后，当我们在回调队列里面收到请求的时候我们会关注这个属性，在这个属性值得基础上 我们可以匹配到哪个响应属于哪个请求。如果我们收到一个correlationId不存在值，我们可以安全的丢掉这个消息，因为这个响应不属于我们的请求。
你可能会问，为什么我们应该忽略在回调队列里面的不明correlationId的消息，而不是用一个error来报错？这是因为这种情况在服务端是存在的，比如RPC服务可能在发送给我们结果的时候就刚好挂掉了，但是还没有给请求发送一个确认的消息，尽管这种情况很少，但是还是存在；如果这种情况发生了，重启的RPC服务会再次处理这个请求，这就是为什么在客户端我们应该对重复响应进行友好的处理，理论上，RPC应该是幂等的。

### 工作流程

![2017-09-26-15-58-35](/images/qiniu/2017-09-26-15-58-35.png)

我们的RPC工作的方式就像上图这样：
 * 当一个客户端启动，它创建了一个匿名的独立的回调队列。
 * 对于一个RPC请求，客户端发送一个消息，消息有两个属性：`replyTo`,设置回调队列；`correlationId`,为每一个请求设置一个唯一值。
 * 请求发送到`rpc_queue`队列。
 * RPC工作线程(服务端)等待队列中的请求的消息。当请求出现，它就开始工作，工作完后，使用`replyTo`中的属性来返回一个结果消息给客户端。
 * 客户端等待回调队列中的消息数据，当一个消息出现，它会先检查`correlationId`属性，检查响应中的值匹配上了请求中的值。

### 实践

斐波那契算法：

```java
private static int fib(int n){
    if(n == 0) return 0;
    if(n == 1) return 1;
    return fib(n-1)+fib(n-2);
}
```

非常简单并且没有太多边界判断，只支持正整数。我们只是用来演示而已，如果要其他的牛逼的算法，自己可以试着写。

RPC 服务端代码：RPCServer.java

```java
package me.chenzhijun.rpc;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/26
 */
public class RpcServer {

    private static final String RPC_QUEUE_NAME = "rpc_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");

        Connection connection = connectionFactory.newConnection();

        final Channel channel = connection.createChannel();
        channel.basicQos(1);

        System.out.println("[x] awaiting rpc request");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                AMQP.BasicProperties replyProperties = new AMQP.BasicProperties.Builder().correlationId(properties.getCorrelationId()).build();
                String response = "";
                String message = new String(body, "UTF-8");
                int n = Integer.parseInt(message);
                System.out.println("[x] fib(" + message + ")");
                response += fib(n);
                channel.basicPublish("", properties.getReplyTo(), replyProperties, response.getBytes("UTF-8"));
                channel.basicAck(envelope.getDeliveryTag(), false);
                synchronized (this) {
                    this.notify();
                }
            }
        };
        channel.basicConsume(RPC_QUEUE_NAME,false,consumer);

//        while (true){
            synchronized (consumer){
                try {
                    consumer.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
//        }

    }

    public static int fib(int n) {
        if (n == 1) {
            return 1;
        }
        if (n == 0) {
            return 0;
        }
        return fib(n - 1) + fib(n - 2);
    }
}

```

和前面的章节一样，我们建立了connection,channel,queue。我们可能想要运行一个或者多个线程，为了负载我们需要设置`channel.basicQos()`中的`prefetchCount`设置。
我们可以使用`basicConsumer`来访问我们设置的回调queue，我们提供了DefaultConsumer来做一些工作并且发送回来response。

RPC客户端代码：RpcClient.java

```java
package me.chenzhijun.rpc;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.UUID;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/26
 */
public class RpcClient {
    private Connection connection;
    private Channel channel;
    private String requestQueueName = "rpc_queue";
    private String replyQueueName;

    public RpcClient() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        connection = factory.newConnection();
        Channel channel = connection.createChannel();
        replyQueueName = channel.queueDeclare().getQueue();

    }

    public String call(String message) throws Exception {
        final String corraltionId = UUID.randomUUID().toString();
        AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder().correlationId(corraltionId).replyTo(replyQueueName).build();

        channel.basicPublish("", requestQueueName, properties, message.getBytes("UTF-8"));
        final BlockingQueue<String> response = new ArrayBlockingQueue<String>(1);
        channel.basicConsume(replyQueueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                if (properties.getCorrelationId().equals(corraltionId)) {
                    response.offer(new String(body, "UTF-8"));
                }
            }
        });
        return response.take();
    }

    public void close() throws Exception {
        connection.close();
    }
}

```

客户端代码理解也不难：
 * 先创建connection，channel，然后声明一个回调的队列来为了(replies)。
 * 我们订阅了回调队列，所以我们可以接收到RPC的response。
 * 我们的call方法是实际上的RPC请求。
 * 我们先生成一个唯一的correlationId，然偶后保存它，它的作用是匹配正确的响应response。
 * 接下来，我们发布了请求request的消息，消息带有replyTo和correlationId.

 接下来我们我们就等待何时的响应返回。因为我们的消费者转发处理在一个分开的线程，在响应到达前，我们需要准备一些东西来挂起我们的主线程main。使用`BlockingQueue`就是一种解决办法，我们在此列中创建了一个`ArrayBlockingQueue`设置了capacity为1,因为我们仅仅需要它等待一个响应。

 `handleDelivery`方法只做了一个非常简单的工作，对于每一个消费者的响应信息，它会检查是否correlationId是否是我们需要的，如果是的话，它会将它放进`BlockingQueue`.

 同时`main`线程是一直在等待从`BlockingQueue`中拿到响应。

最后我们返回响应结果给用户。



参考资料：

[rabbitMQ RPC](http://www.rabbitmq.com/tutorials/tutorial-six-java.html)