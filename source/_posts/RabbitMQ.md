---
title: RabbitMQ
date: 2023.10.1
author: Aik
img: ../images/deepLearning.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 关于RabbitMQ的相关简介以及项目实战
categories: 中间件
tags:
  - MQ
  - Java
---



# 概述

RabbitMQ的结构和概念

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019193329731.png" alt="image-20231019193329731" style="zoom:50%;" />

RabbitMQ中的概念：

- channel：操作MQ的工具
- exchange：路由信息到队列中
- queue：缓存消息
- virtual host：虚拟主机，是对queue、exchange等资源的逻辑分组

# 常见消息模型

各种消息模型的官方用例：https://www.rabbitmq.com/getstarted.html

MQ的官方文档给出了5个MQ的Demo示例，对应了几种不同的用法：

- 基本消息队列（BasicQueue）
- 工作消息队列（WorkQueue）采用轮询分发消息
- 发布订阅（Publish、Subscribe）、根据交换机类型不同又分为三类
  - Fanout Exchange：广播
  - Direct Exchange：路由
  - Topic Exchange：主题

**HelloWorld案例**

基于最基础的消息队列模型来实现的，只包括三个角色：

- publisher：消息发布者，将消息发送到队列queue
- queue：消息队列，负责接收并缓存消息
- consumer：订阅队列，处理队列中的消息

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019194229110.png" alt="image-20231019194229110" style="zoom:50%;" />

基于消息队列的消息发送流程：

1. 简历connection
2. 创建channel
3. 利用channel声明队列
4. 利用channel向队列发送消息

基于消息队列的消息接收流程：

1. 简历connection
2. 创建channel
3. 利用channel声明队列
4. 定义consumer的消费行为handleDelivery()
5. 利用channel将消费者与队列绑定

# SpringAMQP

**AMQO（Advanced Message Qeueing Protocol）**

用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求。

**Spring AMQP**

基于AMQP协议定义的一套API规范，提供了模板来发送和接收消息。包含两部分，其中spring-amqp是基础抽象，spring-rabbit是底层的默认实现。

## 实际案例

### Basic Queue

**利用SpringAMQP实现HelloWorld中的基础消息队列功能**

1. 在父工程中引入spring-amqp的依赖
2. 在publisher服务中利用RabbitTemplate发送消息到simple.queue这个对垒
3. 在consumer服务中编写消费逻辑，绑定simple.queue这个队列

导入依赖

```xml
<!--AMQP依赖，包括RabbitMQ -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置

```yaml
spring:
    rabbitmq:
      host: localhost		# ip地址
      port: 5672			# 端口号
      virtual-host: /		# 虚拟主机
      username: xwy			# 用户名
      password: 123456		# 密码
```

单元测试

```java
@Autowired
private RabbitTemplate rabbitTemplate;

@Test
@RabbitListener(queuesToDeclare = @Queue("simple.queue"))
void testSendToSimpleQueue() {

    String queueName = "simple.queue";
    String message = "hello! spring amqp";
    rabbitTemplate.convertAndSend(queueName,message);
    log.info("发送到"+queueName+"的消息为"+message);
}
```

接收消息

```java
@RabbitListener(queues = "simple.queue")
void testReceiveFromSimpleQueue(String message) {
    log.info("接收到的消息为："+message);
}
```



### Work Queue

工作队类，提高消息处理速度，避免队列消息堆积。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019214604297.png" alt="image-20231019214604297" style="zoom:50%;" />

1. 在publisher服务中定义测试方法，每秒产生50条消息，发送到simple.queue

2. 在consumer服务中定义两个消息监听者，都监听simple.queue队列

3. 消费者1每秒处理50条消息，消费者2每秒处理10条消息

消费预取限制：

```yaml
spring:
	rabbitmq:
		listener:
			simple:
				prefetch:1	# 每次只能获取一条消息，处理完成才能获取下一个消息，默认是无限
```



### 发布和订阅

发布（Publish）、订阅（Subscribe）

发布订阅模式与之前案例的区别就是允许将同一消息发送给多个消费者。实现方式是加入了exchange（交换机）

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020143330298.png" alt="image-20231020143330298" style="zoom:50%;" />

常见的交换机类型：

- Fanout：广播
- Direct：路由
- Topic：话题

exchange只负责消息路由，而不存储，路由失败则消息丢失。

#### Fanout

将接收到的消息路由到每一个与之绑定的queue

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020143542882.png" alt="image-20231020143542882" style="zoom:50%;" />

```java
@Configuration
public class FanoutConfig {

    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("xwy.fanout");
    }

    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    @Bean
    public Binding fanoutBinding1(FanoutExchange fanoutExchange,Queue fanoutQueue1){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    @Bean
    public Binding fanoutBinding2(FanoutExchange fanoutExchange,Queue fanoutQueue2){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
    
}
```

交换机的作用：

- 接收publisher发送的消息
- 将消息按照规则路由到与之绑定的队列
- 不能缓存消息，路由失败，消息丢失
- FanoutExchange会将消息路由到每一个与之绑定的queue

声明队列、交换机、绑定关系的Bean是什么？

- Queue
- FanoutExchange
- Binding



#### DirectExchange

会将接收到的消息根据规则路由到指定的Queue，因此成为路由模式（routes）

- 每一个Queue都与Exchange设置一个BindingKey
- 发布者发送消息时，指定消息的RoutingKey
- Exchange将消息路由到BindingKey与消息RoutingKey一致的队列

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020151537207.png" alt="image-20231020151537207" style="zoom:50%;" />

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020151744152.png" alt="image-20231020151744152" style="zoom:50%;" />

Direct可以模拟Fanout，但需要指定好bindingKey

可以利用@RabbitListener声明Exchange、Queue、RoutingKey

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue2"),
        exchange = @Exchange(name = "xwy.direct",type = ExchangeTypes.DIRECT),
        key = {"red","blue"}
))
```

发送消息时指定发送的交换机及routingKey

```java
rabbitTemplate.convertAndSend(exchange,"red",message);
```



#### TopicExchange

与DirectExchange类似，都能够通过规则指定发送的交换机及队列，区别在于topicExchange必须是多个单词的列表，并且以 **.** 分割。

在指定BindingKey的时候可以使用通配符

- #：代指0个或多个单词
- *：代指一个单词

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020155756969.png" alt="image-20231020155756969" style="zoom:50%;" />



### 消息转换器

Spring的对消息对象的处理是由org.springframework.amqp.support.converter.MessageConverter来处理的。

而默认实现是SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化。如果要修改只需要定义

一个MessageConverter类型的Bean即可。推荐用JSON方式序列化，步骤如下：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

引入依赖

```java
@Bean
public MessageConverter messageConverter(){
    return new Jackson2JsonMessageConverter();
}
```

配置类中声明Bean，覆盖默认Spring的默认Bean

SpirngAMQP中消息的序列化和反序列化：

- 利用MessageConverter实现的，默认时JDK的序列化
- 发送方和接收方必须使用相同的MessageConverter

## 发送者的可靠性

### 生产者重连

由于网络波动，可能会出现客户端连接MQ失败的情况，通过配置开启连接失败后的重连机制：

```yaml
spring:
	rabbitmq:
		connection-timeout: 200s # 设置MQ的连接超时时间
        template:
          retry:
            enabled: true     # 开启超时重试机制
            initial-interval: 1000ms  # 失败后的初始等待时间
            multiplier: 1     # 失败后下次的等待时长倍数
            max-attempts: 2   # 最大重试次数
```

SpringAMQP提供的重试机制是阻塞式的重试，由于线程阻塞会影响性能。

一般不建议使用，如果必须要使用，尽量减短超时以及等待时间



### 生产者确认

有Publisher Confirm和Publisher Return两种确认机制。开启确认机制后，在MQ成功收到消息后会返回确认消息给生产者。返回的结果有以下几种情况：

- 消息投递到了MQ，但是路由失败。此时会通过Publisher Return返回路由异常原因，然后返回ACK，告知投递成功（一般这种情况只可能是routingKey填写错误导致）

- 临时消息投递到了MQ，并且入队成功，返回**ACK**，告知投递成功

- 持久消息投递到了MQ，并且入队完成持久化，返回**ACK**，告知投递成功

- 其它情况都会返回**NACK**，告知投递失败

  <img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231020193952874.png" alt="image-20231020193952874" style="zoom:50%;" />



### **如何保证生产者发送的可靠性？**

首先，可以在RabbitMQ中配置生产者的重连机制，在连接MQ的时候产生网络波动的情况下进行有限次数的重连，避免由于网络波动产生的发送失败。其次，如果是由于其他的原因导致的失败，MQ还支持生产者确认机制，只要开启了生产者确认机制，当生产者发送了消息到MQ时，MQ会给一个ACK回执，发送失败时会返回NACK回执，根据返回的消息，如果是NACK就可以进行重发。通过以上手段就可以基本保证生产者消息的可靠性，但是通过这种方式会增加系统的负担，因此大多数情况下并不需要开启此机制，除非对于消息可靠性有着较高的要求。



## MQ的可靠性

在默认情况下，RabbitMO会将接收到的信息保存在内存中以降低消息收发的延迟。这样会导致两个问题：

- 一旦MQ宕机，内存中的消息会丢失

- 内存空间有限，当消费者故障或处理过慢时，会导致消息积压，引发MQ阻塞



### 数据持久化

RabbitMQ实现数据持久化包括3个方面：

- 交换机持久化
- 队列持久化
- 消息持久化

Spring默认开启持久化

### Lazy Queue

从RabbitMQ的3.6.0版本开始，就增加了Lazy Queue的概念，也就是惰性队列。

惰性队列的特征如下：

- 接收到消息后直接存入磁盘而非内存（内存中只保留最近的消息，默认2048条)

- 消费者要消费消息时才会从磁盘中读取并加载到内存

- 支持数百万条的消息存储

在3.12版本后，所有队列都是Lazy Queue模式，无法更改。

```java
@RabbitListener(queuesToDeclare = @Queue(
		...
		arguments = @Argument(name = "x-queue-mode",value = "lazy")
))
```

### 如何保证消息的可靠性

- 首先通过配置可以让交换机、队列、以及发送的消息都持久化。这样队列中的消息会持久化到磁盘，MQ重启消息依然存在。

- RabbitMQ在3.6版本引入了LazyQueue，并且在3.12版本后会称为队列的默认模式。LazyQueue会将所有消息都持久化。

- 开启持久化和生产者确认时，RabbitMQ只有在消息持久化完成后才会给生产者返回ACK回执。



## 消费者的可靠性

### 消费者确认机制

为了确认消费者是否成功处理消息，RabbitMQ提供了消费者确认机制（Consumer Acknowledgement)。当消费

者处理消息结束后，应该向RabbitMQ发送一个回执，告知RabbitMQ自己消息处理状态。回执有三种可选值：

- ack：成功处理消息，RabbitMQ从队列中删除该消息

- nack：消息处理失败，RabbitMQ需要再次投递消息

- reject：消息处理失败并拒绝该消息，RabbitMQ从队列中删除该消息



SpringAMQP已经实现了消息确认功能。并允许我们通过配置文件选择ACK处理方式，有三种方式：

- none：不处理。即消息投递给消费者后立刻ack，消息会立刻从MQ删除。非常不安全，不建议使用

- manual：手动模式。需要自己在业务代码中调用api，发送ack或reject，存在业务入侵，但更灵活

- auto：自动模式。SpringAMQP利用AOP对我们的消息处理逻辑做了环绕增强，当业务正常执行时则自动返回ack。当业务出现异常时，根据异常判断返回不同结果：

  - 如果是业务异常，会自动返回nack

  - 如果是消息处理或校验异常（消息转换类型异常），自动返回reject

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto
```

### 消费失败处理

当消费者出现异常后，消息会不断requeue（重新入队)到队列，再重新发送给消费者，然后再次异常，再次

requeue,无限循环，导致mq的消息处理飙升，带来不必要的压力。

我们可以利用Spring的retry机制，在消费者出现异常时利用本地重试，而不是无限制的requeue到mq队列：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true		# 开启重试机制
          initial-interval: 1000ms	# 重试等待时间
          multiplier: 1		# 下次失败后的等待时间倍数
          max-attempts: 3	# 最大重试次数
          stateless: true	# 业务中包含事务就设置为false
```

在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要有MessageRecoverer接口来处理，它包含三种不同的实现：

- RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认就是这种方式

- ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队

- RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定的交换机

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231021121402110.png" alt="image-20231021121402110" style="zoom:50%;" />

反复处理不了的消息，通过投递到error.queue交给人工处理。

**消费者如何保证消息一定被消费？**

- 开启消费者确认机制为auto，由spring确认消息处理成功后返回ack，异常时返回nack

- 开启消费者失败重试机制，并设置MessageRecoverer，多次重试失败后将消息投递到异常交换机，交由人工处理



### 保证业务幂等性

幂等是一个数学概念，用函数表达式来说就是$$f(x)=f(f(fx))$$。在程序开发中，指的是同一个业务，执行一次或多次对业务状态的影响是一致的。

幂等：

- 查询业务，例如根据id查询商品
- 删除业务，例如根据id删除商品

非幂等：

- 用户下单业务，需要扣减库存
- 用户退款业务，需要恢复余额

#### 唯一消息id

方案一，是给每个消息都设置一个唯一id，利用id区分是否是重复消息：

- 每一条消息都生成一个唯一的id，与消息一起投递给消费者

- 消费者接收到消息后处理自己的业务，业务处理成功后将消息id保存到数据库

- 如果下次又收到相同消息，先去数据库查询判断该id是否存在，存在则为重复消息放弃处理。

#### 消息后置处理器

可以自行更改messageID，默认的是UUID，可以换成自己设置的ID，或者雪花算法生成的ID。

```java
@Component
public class RabbitPostProcessor implements MessagePostProcessor {

    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        MessageProperties messageProperties = message.getMessageProperties();
        messageProperties.setMessageId(RandomUtil.randomNumbers(6));
        return message;
    }
}
```

```java
    @Resource
    RabbitPostProcessor rabbitPostProcessor;

    @Test
    void testSendToDirectExchange() {
        String exchange = "xwy.direct";
        String message = "hello! rabbit__";
        rabbitTemplate.convertAndSend(exchange,"yellow",message,rabbitPostProcessor);
	}
```

**存在的问题：**

存在业务侵入，需要额外处理传入的ID，写入数据库，判断数据是否存在

 

#### 业务判断

基于业务本身做判断。例如直接在支付后修改订单状态为已支付，修改订单的同时做个where判断。这样就是会修改未支付的订单，从而保证业务的幂等性。

