# kafka

## 基本概念

![image-20211204142532043](D:\JavaDevloper\学习\poto\kafaka基本概率.png)

### Topic与partition

![image-20211204142630850](D:\JavaDevloper\学习\poto\TOPIC-Partition.png)

分区的目的：分散磁盘IO

### 副本概念Replica

![image-20211204142728946](D:\JavaDevloper\学习\poto\Replica.png)

### ISR详解（In Sync Replicas）

主副节点同步问题

![image-20211204142757926](D:\JavaDevloper\学习\poto\ISR详解.png)

![image-20211204142905320](D:\JavaDevloper\学习\poto\ISR2.png)

挑选leader只有ISR集合副本才有资格

HW: High Watermark,高水位线，消费者只能最多拉取到高水位线的消息

LEO：Log  End Offset, 日志文件的最后一条记录的offset(偏移量)

ISR集合与HW和LEO直接存在着密不可分的关系

![image-20211204143137983](D:\JavaDevloper\学习\poto\HW-LEO.png)

![image-20211204143318238](D:\JavaDevloper\学习\poto\HW-LEO2.png)

![image-20211204143443546](D:\JavaDevloper\学习\poto\HW-LEO3.png)

HW：当新增消息写入到replica后，改变

## kafka生产者核心知识梳理

重要参数：

acks: 指定发送消息后，Broker端至少有多少个副本接受到消息；默认为acks=1;

acks=0: 生产者发送消息之后，不需要等待任务服务器响应；

acks=-1,acks=all: 生产者在消息发送后，需要等待ISR中所有的副本都成功写入消息之后才能够收到来着服务器的成功响应；

max.request.siaze: 限制生产者客户端能发送的消息最大值

retries和retry.backoff.msretries:重试次数和重试间隔，默认100

compression.type: 这个参数用来指定消息的压缩方式，默认值为none,可选配置gzip,snappy,lz4

connection.max.idle.ms: 多久之后关闭限制的连接，默认值是540000ms,即9分钟

linger.ms： 生产者发送ProducerBatch之前等待更多消息（ProducerRecord）加入ProducerBatch的时间，默认0

batch.size:累计多少条消息，进行一次批量发送

buffer.memory:缓存提升性能参数，默认32M

receive.buffer.bytes: 设置socket接受消息缓冲区（SO_RECBUF）的大小，默认值32768B，即32KB

send.buffer.bytes:设置发送socket消息缓冲区（SO_SNDBUF）的大小，默认值131072B,即128KB

request.timeout.ms:producer等待请求响应的最长时间，默认值30000（ms）

\1. acks：acks 是生产者客户端中一个非常重要的参数，它涉及消息的可靠性和吞吐量之间的权衡。acks有三个取值 默认为1 ， 0 ， -1 or all ，面试题：

A1: 说一下acks 3个取值代表什么含义，分别适用于什么样的应用场景？

Q1:

A2: acks= -1 or acks=all 是不是一定能够保障消息的可靠性呢？

Q2: min.insync.replicas = 2

### kafka分区器使用与最佳实践

![image-20211204145352000](D:\JavaDevloper\学习\poto\kafka分区.png)

![image-20211204145418448](D:\JavaDevloper\学习\poto\kafak分区2.png)



## Kafka之消费者

### 消费者与消费者组

![image-20211204145517834](D:\JavaDevloper\学习\poto\kafka消费者.png)



消息中间件模型：

点对点（P2P,point-to-point）模式和发布/订阅（PUB/SUB）模式

点对点模式是基于队列的，消息生产者发送消息到队列，消费者从队列消费消息

发布订阅模式定义如何向一个内容节点发部和订阅消息，这个内容节点称为主题（topic），主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题 ，而消息订阅者从主题中订阅消息。

kafka同时支持两种模式，消费者都属于一个消费者组，相当于点对点，属于不同的消费者组，相当于发布/订阅模式

### kafka消费者核心使用

参数使用：

bootstrap.servers :用来指定连接kafka集群所需的broker地址清单

key.deserializer和values.deserializer: 反序列化参数

group.id:消费者所属的消费者组

subscribe:消息主题订阅

assign; 只订阅主题的某个分区

![image-20211204152024384](D:\JavaDevloper\学习\poto\kafka提交偏移.png)

自动提交：

enable.auto.commit,默认值为true

提交周期间隔：auto.commit.interval.ms,默认值为5秒

 手工提交：

enable.auto.commit = false

提交方式：commitSync& commitAsync

同步提交：整体提交和分区提交

## kafka重要参数

fetch.min.bytes :一次拉取最小数据量，默认为1B

fetch.max.bytes:一次拉取最大数据量，默认为50M

max.partition.fetch.bytes: 一次fetch请求，从一个partition中取得records最大大小，默认1M

fetch.max.wait.ms:fetch请求发给broker后，在broker中可能会被阻塞的时长，默认为500、

max.poll.records；consumer每次poll()时取到的最大条数，默认为500



