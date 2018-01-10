# RabbitMQ

## 1. 概念

- `AMQP`：Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计，主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
- `RabbitMQ`： 一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等。
- `ConnectionFactory、Connection`：Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。ConnectionFactory为Connection的制造工厂。
- `Channel`：大部分的业务操作是在Channel这个接口中完成的，包括定义`Queue`、定义`Exchange`、`绑定`Queue与Exchange、`发布消息`等。 
- `Queue`：RabbitMQ的内部对象，用于存储消息。 生产者（P）生产消息并最终投递到Queue中，消费者（C）可以从Queue中获取消息并消费。
- `Exchange`： 生产者将消息发送到Exchange（交换器，X），由Exchange将消息路由到一个或多个Queue中（或者丢弃）。
    - `routing key`：消费者将消息发送给Exchange时，一般会指定一个routing key。
    - `Binding key`：在把Exchange和Queue做绑定（Binding）的同时，一般会指定一个binding key。
    - `Exchange Types`：不同type不同的路由规则。
        - `fanout`类型： `所有`发送到该Exchange的消息，会被路由到所有与它绑定的Queue中。
        - `direct`类型： 消息会被路由到那些binding key与routing key`完全匹配`的Queue中。
        - `topic`类型： 和严格的direct类型相似，将消息路由到binding key与routing key相`模糊匹配`的Queue中。
            - binding key与routing key一样，都是句点号“. ”分隔的字符串
            - binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）
        - `headers`类型： 不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性（一个键值对的形式）进行完全匹配。

其他特性：

- `Message acknowledgment`：消费者在消费完消息后发送一个`消息回执（Message acknowledgment）`给RabbitMQ，RabbitMQ收到后才能将该消息从Queue中移除。记得发回执哦，否则消息就不停地重复消费了！
- `Message durability`：如果我们希望即使在RabbitMQ服务重启的情况下，也不会丢失消息，我们可以将Queue与Message都设置为可持久化的（durable）。
    - 但依然解决不了小概率丢失事件的发生（比如RabbitMQ服务器已经接收到生产者的消息，但还没来得及持久化该消息时RabbitMQ服务器就断电了），此时我们要用到`事务`来保障。
- `Prefetch count`：同一个Queue中的消息会被平摊给多个同时订阅消费者。如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置prefetchCount来限制Queue`每次发送给每个消费者的消息数`，比如我们设置prefetchCount=1，则Queue每次给每个消费者发送一条消息；消费者处理完这条消息后Queue会再给该消费者发送一条消息。
- `RPC`：MQ本身是基于异步的消息处理，生产者将消息发送到RabbitMQ后不会知道消费者（C）处理成功或者失败（甚至连有没有消费者来处理这条消息都不知道）。如果我们需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于RPC（Remote Procedure Call，远程过程调用）。

## 2. 安装（ubuntu）

http://www.rabbitmq.com/install-debian.html

## 3. 教程

http://www.rabbitmq.com/tutorials/tutorial-one-javascript.html