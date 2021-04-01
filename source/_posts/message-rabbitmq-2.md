---
title: RabbitMQ-消息中间件（二）任务队列
date: 2017-09-18 15:02:28
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## 消息中间件 RabbitMQ (二) 任务队列

>第一篇中简单介绍了mq的使用，那么第二节中来了解下mq的其它内容.

### 工作队列

这节我们将建立一个工作队列来讲任务分发到不同的消费工作者中去。工作队列的主要思想是避免在做资源密集型任务的同时又不得不等待一个个完成。这种时候我们像消息一样封装所要完成的任务然后将它发送到队列中，在后台运行的工作线程会将任务拿出，然后执行这个任务。当运行多个工作线程的时候，任务对他们都是可见的。

网络应用中如果一个任务无法快速执行完，那么工作队列就非常有用。
<!--more-->
### 准备

像上节一样，我们准备两个类，一个生产(Tasker.java)一个消费(Worker.java).

`Task.java` ：

```java

package me.chenzhijun;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * 发送者
 *
 * @author chen
 * @version V1.0
 * @date 2017/9/18
 */
public class Tasker {
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        //String message = getMessage(new String[]{"this is my message"});
        //channel.basicPublish("", QUEUE_NAME, null, message.getBytes());

        for (int i = 0; i < 10; i++) {
            String message = getMessage(new String[]{"this is my message"});
            message = message + i;
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        }

        channel.close();
        connection.close();

    }

    private static String getMessage(String[] strings) {
        if (strings.length < 1)
            return "Hello World!";
        return joinStrings(strings, " ");
    }

    private static String joinStrings(String[] strings, String delimiter) {
        int length = strings.length;
        if (length == 0) return "";
        StringBuilder words = new StringBuilder(strings[0]);
        for (int i = 1; i < length; i++) {
            words.append(delimiter).append(strings[i]);
        }
        return words.toString();
    }

}

```

`Worker.java` :

```java

package me.chenzhijun;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * 接受者
 *
 * @author chen
 * @version V1.0
 * @date 2017/9/18
 */
public class Worker {
    private final static String QUEUE_NAME = "task_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        Consumer consumer = new DefaultConsumer(channel) {

            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                super.handleDelivery(consumerTag, envelope, properties, body);
                String messageReceiv = new String(body, "UTF-8");
                System.out.println("[x] received: " + messageReceiv);

                try {
                    doWork(messageReceiv);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println(" [x] Done");
                }
            }
        };

        channel.basicConsume(QUEUE_NAME, false, consumer);


    }


    private static void doWork(String task) throws InterruptedException {
        for (char ch : task.toCharArray()) {
            if (ch == '.') Thread.sleep(1000);
        }
    }
}

```

在这次的模拟中，我们需要同时启动多个`Worker`，这样我们才能在控制台中看到不同的Worker会打印不同的值。


### 循环调度

使用任务队列的一个优点是易于并行工作，当一个worker在工作的时候，我们可以将任务给另一个worker。默认情况下，RabbitMQ会发送任务给下一个消费者，着这样每一个消费者能得到一个比较平均的任务数量。这种任务转发方式叫做循环调度。可以启动两个Worker进行尝试一下。

![2017-09-18-17-13-33](/images/qiniu/2017-09-18-17-13-33.png)

![2017-09-18-17-13-59](/images/qiniu/2017-09-18-17-13-59.png)


### 消息确认

一个任务执行的时间可能需要花费数秒。我们可能想要知道一个消费者在执行了一个长时间任务的时候，如果只是完成了一部分，然后挂掉了怎么办？用我们现在的代码，一旦RabbitMQ传递了一个消息给消费者，它就会立即将它做删除标记。所以在这个问题上，如果你在消费者执行期间kill掉了它，我们就永久的失去了这个消息。我们也会丢失掉所有MQ中间件给它的但是实际上并没有被正确处理的所有消息，以及这些消息的所有详细内容。

但是实际开发中，我们并不想失去任何一个队列中的任务。如果一个worker挂掉了，我们可能想要将这个任务转发给其他的worker。

为了保证消息绝不丢失，RabbitMQ支持`[消息完成确认](http://www.rabbitmq.com/confirms.html)`,就像网络http请求的`ACK`(确认)。消费者在接收到详细完整的消息，并且完成了消息的任务，然后会发送一个ack给RabbitMQ，RabbitMQ才会删除它。

如果消费者线程死了(channel关闭，connection关闭，或者TCP 连接丢失)并且挂掉之前没有发送`ACK`。RabbitMQ会知道消息没有圆满完成，然后会让它重新排到队列中去，如果在这个时候有其它的消费者在线，它就会马上将消息推送给其他消费者。这个方式就能确保尽管有worker不稳定挂掉了，消息也不会丢失。

这里是不会有任何消息超时的，RabbitMQ只会在Worker挂掉之后才会重新转发消息，就算一个处理消息的过程会花费很长很长的时间。

默认情况下消息确认是打开的，[Manual message acknowledgments](http://www.rabbitmq.com/confirms.html)。在之前的例子中我们明确的关闭了它`autoAck=true`;如果你在声明中用到了`autoAck=false`;

```java

boolean autoAck=false;
channel.basicConsume(QUEUE_NAME, autoAck, consumer);

```

这个时候把线程时间拉长一点，然后强制kill掉，这个worker挂掉之后也不会重新被MQ转发消息了。

> ps: 忘记消息确认会引发一个非常严重的后果，如果它不能释放未确认的消息的内容,RabbitMQ会不停的吃掉内存空间，如果要调试的话可以在Linux下使用`sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged` 或者windows下使用`rabbitmqctl.bat list_queues name messages_ready messages_unacknowledged`.

### 消息持久化

我们知道了就算消费者挂了，也能保证任务不会丢失。但是如果我们的RabbitMQ服务器宕机了呢？队列中的任务还是会丢失。

当RabbitMQ退出或者宕机的时候，它会丢失掉所有的队列和消息，除非你对它进行设置。要保证消息不会丢失，必须先做两件事情，我们需要标记队列和消息可以持久化。

首先我们需要保证RabbitMQ绝不会丢失掉我们的队列。为了做到这个，我们需要声明它为`durable`持久化。

```java

boolean durable = true;
channel.queueDeclare(QUEUE_NAME,durable,false,false,null);

```

上面的命令本身没有错误，但是如果队列中已经有了`QUEUE_NAME`的队列，并且前面定义的队列不是`durable`的，RabbitMQ不会允许你重新用不同的参数定义一个已经存在的队列，如果尝试这样做的话，它会返回一个错误。变通的方法就是取另一个唯一的名字。如果队列名字改了，消费者和生产者要对应到一个队列上。

声明了队列为持续化之后，我们也需要将消息标记为持久化，通过设置`MessageProperties`(继承至`BasicProperties`) 的值为`PERSISTENT_TEXT_PLAIN`.

```java

import com.rabbitmq.client.MessageProperties;

channel.basicPublish("",QUEUE_NAME,MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());

```

> 消息持久化注意：标记消息持久化不能完全的保证消息不会被丢失，尽管它告诉了RabbitMQ保存消息到磁盘，但是在RabbitMQ接收到消息，这里仍然有一个很短的时间，可能让RabbitMQ没有保存它到磁盘。RabbitMQ不允许对每一个消息进行`fsync`，它仅仅是存到缓存当中而不是真的写入到磁盘当中。这种持久化的保证不是特别稳定的，但是它也足够我们在简单的任务队列中使用。如果你需要稳定的强保证你可以使用`[publish confirms](https://www.rabbitmq.com/confirms.html)`


### 公平转发

你可能注意到了任务分发有时候并不是精确的像我们期待的那样，例如有一种场景：两个worker，当所有的消息不管是简单的还是难的，可能出现一个worker非常忙，而另一个却没有什么任务。RabbitMQ是不会知道这回事的，然后还是照样的循环分配任务。

这种事情的发生是因为RabbitMQ只是在一个消息进入到队列中转发一个消息，它不会去为消费者考虑未确认的消息，它只是盲目的一个接一个的转发消息给一个消费者。

为了防止这种事情，我们可以使用`basicQos`方法来设置`prefetchCount=1`属性值。它会告诉RabbitMQ不要在同一个时间段给一个worker太多的任务，或者换种说话，不要转发一个新的消息给一个worker直到它完成前一个队列并且回复`ACK`。另外，MQ会将这个任务转发给下一个不忙的worker。

```java

int prefetchCount=1;
channel.basicQos(prefetchCount);

```

> 注意队列大小，如果所有的worker都很忙，队列就有可能会堵塞，你可能就需要持续关注它，或者增加worker，或者采用其它的策略来代替。

使用消息的`消息确认`(acknowledgments)和`prefetchCount`,你可以建立一个工作队列，持久化的设置可以让RabbitMQ就算重启，任务也不会消失。

如果想要了解更多的关于`Channel`的方法和`MessageProperties`的属性，你可以在线看[文档](http://www.rabbitmq.com/releases/rabbitmq-java-client/current-javadoc/overview-summary.html)







参考文档：
[RabbitMQ Java](http://www.rabbitmq.com/tutorials/tutorial-two-java.html)