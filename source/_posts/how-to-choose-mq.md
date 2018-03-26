---
title: 如何选择消息队列?各大MQ对比
date: 2018-03-26 11:29:09
categories: 
	- 编程
tags: 
	- 消息队列
	- Kafka
---


分布式环境下,系统之间通信,通常有HTTP,MQ,RPC这么三种;

## MQ的常见用途

1. 异步处理
1. 应用解耦
1. 流量削锋
1. 消息通讯

## MQ用途举例

1. 用户注册后,要发注册邮件和注册短信
1. 用户下单后,订单系统需要通知库存系统(这个例子不是很好,可能有人说下单扣库存必须状态一致) 
1. 秒杀,团抢活动,日志处理
1. 消息队列一般都内置了高效的通信机制,因此也可以用在纯的消息通讯,点对点或发布订阅模式

## JMS消息服务

JMS-API是一个消息服务的标准/规范,允许应用程序基于JavaEE平台创建,发送,接收和读取消息

在JMS标准中,有两种消息模型P2P(Point to Point),Publish/Subscribe(Pub/Sub)

### P2P模式

包含三个角色: 消息队列(Queue),发送者(Sender),接收者(Receiver);

每个消息都被发送到一个特定的队列,接收者从队列中获取消息,队列保留着消息,直到他们被消费或超时

### Pub/Sub模式

包含三个角色: 主题(Topic),发布者(Publisher),订阅者(Subscriber); 

多个发布者将消息发送到Topic,系统将这些消息传递给多个订阅者


---

一般商用的容器,比如WebLogic,JBoss,都支持JMS标准,开发上很方便,但免费的比如Tomcat,Jetty等则需要使用第三方的消息中间件;

**本文探讨MQ如下**

## ActiveMQ

1. 符合JavaEE中JMS规范的MQ实现,同时也不局限JMS规范,也支持其他一些MQ协议如stomp协议、AMQP协议等
1. 从诞生起就是为了企业应用而设计的,JMS规范本身也是企业应用系统的规范,并不适合互联网应用
1. 互联网应用通常面对的是海量的数据,并且通常对事务一致性的要求相对较弱,而企业应用对事务一致性的要求就相对很高
1. 互联网应用为了处理大量请求,通常采用集群处理的方式,而JMS规范并不重视分布式应用,这个集群不仅仅是服务端broker的集群,还包括生产者和消费者都可能是一个又一个集群,而传统的JMS规范是没有明确处理这些情况的
1. 互联网应用还有一个问题是异构系统特别多,而JMS规范只是JavaEE这个平台上的规范,对异构系统的接入也是一个比较麻烦的地方,不同的实现有很大的差异

[MetaQ作者庄晓丹专访](http://www.iteye.com/magazines/107)

## RabbitMQ

1. RabbitMQ是使用Erlang编写的一个开源的消息队列,本身支持很多的协议: AMQP,XMPP,SMTP,STOMP,也正因如此,它非常重量级,更适合于企业级的开发
2. 同时实现了Broker构架,这意味着消息在发送给客户端时先在中心队列排队,对路由,负载均衡或者数据持久化都有很好的支持
3. 它的实现语言是天生具备高并发高可用的erlang语言,AMQP协议更多用在企业系统内,对数据一致性,稳定性和可靠性要求很高的场景,对性能和吞吐量还在其次

## ZeroMQ

1. ZeroMQ号称最快的消息队列系统,尤其针对大吞吐量的需求场景;仅提供非持久性的队列,也就是说如果宕机,数据将会丢失
2. 其中,Twitter的Storm 0.9.0以前的版本中默认使用ZeroMQ作为数据流的传输(Storm从0.9版本开始同时支持ZeroMQ和Netty作为传输模块)
3. ZeroMQ只是一个网络编程的Pattern库,将常见的网络请求形式(分组管理,链接管理,发布订阅等)模式化,组件化
4. 简而言之socket之上,MQ之下;对于MQ来说,网络传输只是它的一部分,更多需要处理的是消息存储,路由,Broker服务发现和查找,事务,消费模式(ack,重投等),集群服务等

## Kafka

### 简介

Kafka是Linkedin开源的MQ系统,主要特点是基于Pull模式来处理消息,追求高吞吐量; 一开始的目的就是用于日志收集和传输,0.8开始支持复制(High Available),不支持事务,适合产生大量数据的互联网服务的数据收集业务;Kafka是可靠的分布式日志存储服务,用简单的话来说,你可以把Kafka当作可顺序写入的一大卷磁带,可以随时倒带,快进到某个时间点重放;

### 特点

1. 快速持久化,可以在O(1)的系统开销下进行消息持久化
1. 高吞吐,在一台普通服务器上可以达到10W/s的吞吐速率
1. 完全的分布式系统,Broker,Producer,Consumer都原生自动支持分布式,自动实现负载均衡
1. 同时支持离线和实时处理(搞两个Consumer Group即可)

### Kafka和RabbitMQ比较

[最权威的是kafka的提交者写一篇文章](http://www.quora.com/What-are-the-differences-between-Apache-Kafka-and-RabbitMQ)

1. RabbitMQ比Kafka成熟,在可用性,稳定性,可靠性,RabbitMQ超过Kafka
1. Kafka设计的初衷就是处理日志的,可以看做是一个日志系统,针对性很强,所以它并没有具备一个成熟MQ应该具备的特性
1. Kafka的性能(吞吐量,TPS)比RabbitMQ要强,这篇文章的作者认为,两者在这方面没有可比性

### 抉择

1. Kafka在开源许可证、产品活跃度、性能、安全性、可扩展性等方面优于RabbitMQ
1. Kafka采用的许可证更宽松，活跃度更高，性能远高于RabbitMQ，在安全性和可扩展性方面能够提供更好的保障
1. Kafka仅在功能上略少于RabbitMQ，但是已经具备了主要的功能

## MetaQ

1. 现在metaQ其实有两个大分支了,一个是庄晓丹维护的已开源的,另外一个是淘宝内部的,本质结构原理没太大区别,只不过开源的已经去掉了对淘系相关的依赖;
2. 然后淘系的metaQ已经到3.*版本了,但是文档比较乱,深入到细节时,发现好乱,一个点有好几种说法,火大,干脆自己看metaq的源码,有点意思,做个笔记记录下
3. 怕我以后忘记了,有少量的章节和图片从内网拿来的,大部分是自己写的 [樵夫后院-MetaQ架构原理](http://jameswxx.iteye.com/blog/2034111)
4. Kafka用Scala写的,MetaQ用Java实现;
5. 参考Kafka实现的,具体可以看看[作者的采访](http://www.iteye.com/magazines/107) 

## RocketMQ

1. RocketMQ的前身是MetaQ,当Metaq3.0发布时,产品名称改为RocketMQ
2. RocketMQ(一)介绍: https://blog.csdn.net/zshake/article/details/64586142
3. 说说MQ之RocketMQ: http://valleylord.github.io/post/201607-mq-rocketmq
4. RocketMq(metaQ)与Kafka: http://www.liaoqiqi.com/post/232

## Notify

1. 淘宝的notify是一个非常有特色的消息中间件;它用创新地方式解决了分布式事务的问题,用相对较低的成本,实现了跨micro service的最终一致性;这种把最终一致性用application queue而不是database replication queue的方式来实现,把IT技术层面的跨业务的事务变成一个业务层面的单据传递的概念,非常值得推广
2. Notify是淘宝自主研发的一套消息服务引擎,是支撑双11最为核心的系统之一,在淘宝和支付宝的核心交易场景中都有大量使用;消息系统的核心作用就是三点:解耦,异步和并行
3. 带事务解决方案
4. 不开源

## Redis

1. 支持MQ的发布与订阅功能,可以看成是一个Utility,当MQ来用就算了;

---

## 结论

1. 	企业级应用 优先RabbitMQ ActiveMQ次之
2. 	互联网应用 Kafka > RocketMQ > MetaQ

---

## 参考

1. 	 https://blog.csdn.net/konglongaa/article/details/52208273
2. 	 http://www.iteye.com/magazines/107

---