# :fire:Spring整合Rocketmq



不同于 RabbitMQ、ActiveMQ、Kafka 等消息中间件，Spring 社区已经通过多种方式提供了对这些中间件产品集成，例如通过 spring-jms 整合 ActiveMQ、通过 Spring AMQP 项目下的 spring-rabbit 整合 RabbitMQ、通过 spring-kafka 整合 kafka ，通过他们可以在 Spring 项目中更方便使用其 API 。目前在 Spring 框架中集成 RocketMQ 有三种方式，**一**是将消息生产者和消费者定义成 bean 对象交由 Spring 容器管理，**二**是使用 RocketMQ 社区的外部项目 rocketmq-jms（https://github.com/apache/rocketmq-externals/tree/master/rocketmq-connect-jms）然后通过 spring-jms 方式集成使用，https://github.com/apache/rocketmq-spring是如果你的应用是基于 spring-boot 的，可以使用 RocketMQ 的外部项目 rocketmq-spring-boot-starter（https://github.com/apache/rocketmq-spring）比较方便的收发消息。

总的来讲 rocketmq-jms 项目实现了 JMS 1.1 规范的部分内容，目前支持 JMS 中的发布/订阅模型收发消息。

这种方式不推荐使用，不够灵活。

所以

如果整合spring，推荐使用第一种方式，相对灵活，自由度高。

如果是sprinboot，就更方便了。官方提供了Rocket-Spring项目用于将Rocketmq整合进Springboot

支持以下功能特性：

---



-  同步发送
-  异步发送
-  one-way发送
-  发送顺序消息
-  批量发送
-  发送事务消息
-  发送延迟消息
-  并发消费（广播/集群）
-  顺序消费
-  支持消息过滤（使用tag/sql）
-  支持消息轨迹
-  认证和授权
-  request-reply模式

---

官方地址：https://github.com/apache/rocketmq-spring/wiki

官方手册非常清晰。

---

