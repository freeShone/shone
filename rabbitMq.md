# 1.初识RabbitMQ核心概念

## AMQP

AMQP：Advanced Message Queuing Protocol  高级消息队列协议

AMQP：

![image-20211204114117050](poto\image-20211204114117050.png)

AMQP核心概念：

Server：又称Broker，接受客户端的连接，实现AMPQ实体服务

Connection：连接，应用程序与Broker的网络连接

Channel：信道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道。客户端可建立多个Channel，每个Channel代表一个会话任务

Message：消息，由Properties和Body组成，Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性；Body则就是消息体内容。

Virtual host：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个Virtual Host里面可以有若干个Exchange和Queue，同一个Virtual Host里面不能有相同名称的Exchange或Queue。

Exchange：交换机，接受消息，根据路由键转发消息到绑定的队列

Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者

Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key

Routing Key：一个路由规则，虚拟机可用它确定如何路由一个特定消息

## Exchange种类

### Exchange之Direct

直达，转发消息到队列中，routing key必须与bingding key 完全匹配

### Exchange之Topic

主题，转发消息到队列中，routing key与bingding key模糊匹配

符号“#”匹配一个或者多个词

符号“*”匹配一个词

例如：log.#   可匹配log.info.oa

​			log.*  匹配log.error

### Exchange之Fanout

广播，不处理路由键，将消息发送到所有绑定的队列中



## Rabbitmq高级特性

### 生产端可靠性投递与消费端幂等性

生产者可靠性投递：

1.消息落库，对消息状态进行达标

![image-20211204115922276](poto\image-20211204115922276.png)

2.确认机制和返回机制

Confirm确认消息：

消息确认，生产者投递消息，如果broker接受到消息，给生产者一个应答，生产者接受到应答，确认消息是否正常发送到Broker

![image-20211204120156871](poto\image-20211204120156871.png)

Confirm确认消息实现：

第一步：在channel上开启确认模式：channel.confirmSelect();

第二部：在channel增加监听，addConfirmListener，监听成功和失败的返回结果，根据具体结果对消息进行重新发送，记录日志等后续处理

return消息机制：

return listener 用于处理一些不可路由消息，消息生产者给指定的exchange和routing key发送消息，如果该exchange不存在或者路由key找不到，这个时候就需要监听这种不可达的消息，使用return listener.

配置项：

Mandatory:如果为true，则监听器会接受路由不可达消息，然后进行处理，如果为false，broker端自动删除该消息。

消费端幂等性：

消费端可能后重复消息一条消息，需要保证幂等性



### 消费端特性讲解_流控服务和ACK重回队列

#### 消费端限流：

假设rabbitmq服务器上堆积了上万条未处理的消息，打开一个消费端，巨量的消息会全部推送过去，但是单个客户端无法处理这么多数据。Rabbitmq提供了一种qos(服务质量保证)，即在非自动确认消息的前提下，如果一定数据的消息（通过consumer或者channel设置Qos的值）未被确认前，不进行新消息的消费。

API : void BasiQos(int prefetchSize, short prefetchCount, bool global )

prefetchCount :告诉RabbitMq不要同时给消费者推送对于N个数据，即一旦有N个数据还没有ack，则该consumer将block掉，直到有消息ack

global : 设置是channel级别还是consumer级别；

### TTL消息与死信队列详解

TTL：Time To Live ,生存时间

RabbitMq支持消息的过期时间，在消息发送时可以进行设置，还支持队列的过期时间，从消息入队列开始计算，只要超过了队列的超时时间配置，那么消息会自动的清除。

死信队列：DLX，Dead-letter-exchange

利用DLX，当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个exchange,这个exchange就是DLX。

消息变成死信的几种情况：

1.消息被拒（basic.reject/basic.nack）并且消息requeue=false

2.消息TTL过期

3.队列达到最大长度

DLX也是一个正常的exchange，它能在任何队列上被指定，实际上就是设置某个队列的属性

当这个队列中有死信的时候，RabbitMq就会自动的将这个消息重新发布到设置的exchange上，进而被路由到另外一个队列

死信队列设置：

首先设置死信队列的exchange和queue然后进行绑定：

Exchange:dlx.exchange

queue: dlx.queue

Routing key :#

然后我们进行正常交换机，队列，绑定，只不过我们需要在队列上加一个参数即可：arguments.put("x-dead-letter-exchange",dlx.exchange);

## Rabbitmq集群搭建

镜像模式：

集群模式非常经典的就是Mirror镜像模式，保证100%数据不丢失，在实际工作中也是用的最多。

Mirror镜像队列，目的是为了保证rabbitmq数据 的高可靠性解决方案，主要就是现实数据同步，一般来讲2-3个节点实现数据同步(对于100%数据可靠性解决方案一般是3个节点）集群架构如下：

![image-20211204125319797](poto\image-20211204125319797.png)

## RabbitMq整合SpringBoot

### 生产端核心配置：

spring.rabbitmq.publisher-confirms = true

spring.rabbitmq.publisher-returns = true

spring.rabbitmq.template.mandatory = true

### 消费端核心配置：

spring.rabbitmq.listener.simple.acknoeledge-mode = MANUAL

spring.rabbitmq.listener.simple.concurrency = 1

spring.rabbitmq.listener.simple.max-concurrency = 5

### @RabbitListener注解

![image-20211204130010054](poto\image-20211204130010054.png)
