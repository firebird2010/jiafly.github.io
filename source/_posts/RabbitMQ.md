
---
title: RabbitMQ实战
date: 2018-11-27 20:31:16
tags: [Rabbit]
categories: [学习笔记]
---
# 消息中间件
## 什么是消息中间
消息 (Message) 是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串、
JSON 等，也可以很复杂，比如内嵌对象。

消息队列中间件 (Message Queue Middleware，简称为 MQ) 是指利用高效可靠的消息传递 机制进行与平台无关的数据交流，并基于数据通信来进行分布式系统的集成。通过提供消息传 递和消息排队模型，它可以在分布式环境下扩展进程间的通信。

消息队列中间件，也可以称为消息队列或者消息中间件。它一般有两种传递模式:点对点
(P2P, Point-to-Point) 模式和发布/订阅 (Pub/Sub) 模式。点对点模式是基于队列的，消息生产 者 发送消息到队列，消息消费者从队列中接收消息，队列的存在使得消息的异步传输成为可能。 发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题 (topic)，主 题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而消息订阅者则从主题中 订阅消息。主题使得消息的订阅者与消息的发布者互相保持独立，不需要进行接触即可保证消 息的传递，发布/订阅模式在消息的一对多广播时采用 。

目前开源的消息中间件有很多，比较主流的有 RabbitMQ、 Kafka、 ActiveMQ、 RocketMQ 等。

## 消息中间件的作用
消息中间件凭借其独到的特性，在不同的应用场景下可以展现不同的作用 。总 的来说，消
息中间件的作用可以概括如下。
### 解耦
在项目启动之初来预测将来会碰到什么需求是极其困难的。消息中间件在处理过程 中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口，这允许你独 立地扩展或修改两边的处理过程，只要确保它 们遵守同样的接口约束即可。
### 冗余〈存储)
有些情况下，处理数据的过程会失败。消息中间件可以把数据进行持久化直 到它们已经被完全处理，通过这一方式规避了数据丢失风险。在把一个消息从消息中间件中删除之前，需要你的处理系统明确地指出该消息己经被处理完成，从而确保你的数据被安全地保 存直到你使用完毕。
### 扩展性
因为消息中间件解捐了应用的处理过程，所以提高消息入队和处理的效率是很容易的，只要另外增加处理过程即可，不需要改变代码，也不需要调节参数。
### 削峰
在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果以能处理这类峰值为标准而投入资源，无疑是巨大的浪费。使用消息中间件能够使关键组件支撑突发访问压力，不会因为突发的超负荷请求而完全崩惯。
### 可恢复性
当系统一部分组件失效时，不会影响到整个系统。消息中间件降低了进程间的稿合度，所以即使一个处理消息的进程挂掉，加入消息中间件中的消息仍然可以在系统恢复后 进行处理。
### 顺序保证
在大多数使用场景下，数据处理的顺序很重要，大部分消息中间件支持一定程度上的顺序性。
### 缓冲
在任何重要的系统中，都会存在需要不同处理时间的元素。消息中间件通过一个缓冲层来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速。该缓冲层有助于控制 和优化数据流经过系统的速度。
### 异步通信
在很多时候应用不想也不需要立即处理消息 。消息中间件提供了异步处理机制，允许应用把一些消息放入消息中间件中，但并不立即处理它，在之后需要的时候再慢慢处理。

# RabbitMQ入门
## RabbitMQ介绍
RabbitMQ 是采用 Erlang 语言实现 AMQP (Advanced Message Queuing Protocol，高级消息
队列协议)的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。并且支持多种客户端 如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。在易用性、扩展性、高可用性等方面表现不俗。

RabbitMQ主要是为了实现系统之间的双向解耦而实现的。当生产者大量产生数据时，消费者无法快速消费，那么需要一个中间层。保存这个数据。

AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

## RabbitMQ安装
- 使用docker安装(3-management版本自带管理后台)
```bash
$ docker pull rabbitmq:3-management 
```
- 启动rabbitMQ并且启动管理后台
```bash
$ docker run -d -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin --name rabbitmq --hostname=rabbitmqhostone  rabbitmq:3-management

$ docker start rabbitmq
```
- 查看管理后台
在浏览器打开 http://localhost:15672/ 输入用户名:admin 密码:admin 可进入管理后台


## 相关概念介绍
### 生产者和消费者
- Producer:生产者，就是投递消息的一方。
生产者创建消息，然后发布到 RabbitMQ 中。消息一般可以包含2个部分:消息体和标签(Label)。消息体也可以称之为payload，在实际应用中，消息体一般是一个带有业务逻辑结构的数据，比如一个JSON字符串。当然可以进一步对这个消息体进行序列化操作。消息的标签用来表述这条消息，比如一个交换器的名称和一个路由键。生产者把消息交由RabbitMQ，RabbitMQ 之后会根据标签把消息发送给感兴趣的消费者(Consumer)。

- Consumer:消费者，就是接收消息的一方。
消费者连接到RabbitMQ服务器，并订阅到队列上。当消费者消费一条消息时，只是消费消息的消息体(payload).在消息路由的过程中，消息的标签会丢弃，存入到队列中的消息只有消息体，消费者也只会消费到消息体，也就不知道消息的生产者是谁，当然消费者也不需要知道。

- Broker:消息中间件的服务节点。
对于RabbitMQ来说，一个RabbitMQBroker可以简单地看作一个RabbitMQ服务节点，或者RabbitMQ服务实例。大多数情况下也可以将一个RabbitMQ Broker看作一台RabbitMQ服务器。

RabbitMQ运转流程
![RabbitMQ运转流程](/image/rabbit/mq-yunzhuanliucheng.jpg)

### 交换器Exchange，路由RoutingKey，绑定Binding
- Exchange:交换器
![交换器](/image/rabbit/mq-exchange.jpg)
MQ中我们暂时可以理解成生产者将消息投递到队列中，但是实际上这个在RabbitMQ中不会发生。真实情况是，生产者将消息发送到Exchange(交换器)，由交换器将消息路由到一个或者多个队列中。如果路由不到，或许会返回给生产者，或许直接丢弃。
RabbitMQ中的交换器有四种类型，下面会一一介绍，并且会使用代码详细说明。

- RoutingKey:路由键
生产者将消息发给交换器的时候，一般会指定一个RoutingKey，用来指定这个消息的路由规则，而这个RoutingKey需要与交换器类型和绑定键(BindingKey)联合使用才能最终生效。在交换器类型和绑定键(BindingKey)固定的情况下，生产者可以在发送消息给交换器时，通过指定RoutingKey来决定消息流向哪里。

- Binding:绑定
RabbitMQ中通过绑定将交换器与队列关联起来，在绑定的时候 一般会指定一个绑定键(BindingKey)，这样RabbitMQ就知道如何正确地将消息路由到队列了，如下图所示:
![绑定](/image/rabbit/mq-banding.jpg)

### 交换器类型
RabbitMQ 常用的交换器类型有fanout、direct、topic、headers这四种。AMQP协议里还提
到另外两种类型:System和自定义，这里就不详细介绍了。
#### fanout
就是我们熟悉的广播模式或者订阅模式，它会把所有发送到该ExChange的消息全部路由到所有与该交换器绑定的队列中。如下图：
![fanout](/image/rabbit/mq-fanout.jpg)
#### direct(默认的交换器类型)
direct类型的交换器路由规则也很简单，它会把消息路由到那些 BindingKey和 RoutingKey 完全匹配的队列中。如下图：
![direct](/image/rabbit/mq-direct.jpg)
#### topic
上面讲到direct类型的交换器路由规则是完全匹配BindingKey和RoutingKey，但是这种严格的匹配方式在很多情况下不能满足实际业务的需求。topic类型的交换器在匹配规则上进行了扩展，它与direct类型的交换器相似，也是将消息路由到BindingKey和RoutingKey相匹配的队列中，但这里的匹配规则有些不同，它约定:
- RoutingKey为一个点号"."分隔的字符串(被点号"."分隔开的每一段独立的字符串称为一个单词)，如"com.rabbitmq.client"，"java.util.concurrent","com.hidden.client"等
- BindingKey也是点号"."分隔
- BindingKey中可以存在两种特殊字符串"星号"和"#"，用于做模糊匹配，其中"星号"用于匹配一个单词，”#"用于匹配多规格单词(可以是零个)。
如下图：
![topic](/image/rabbit/mq-topic.jpg)
思考: 
1.路由建 "com.rabbitmq.client"会路由到哪一个队列？
2.路由建 "com.hidden.client"会路由到哪一个队列？
3.路由建 "com.hidden.demo"会路由到哪一个队列？
4.路由建 "java.util.concurrent"会路由到哪一个队列？
5.路由建 "java.rabbitmq.demo"会路由到哪一个队列？

#### headers(不常用)
headers类型的交换器不依赖于路由键的匹配规则来路由消息，而是根据发送的消息内容中
的headers属性进行匹配。在绑定队列和交换器时制定一组键值对，当发送消息到交换器时，RabbitMQ会获取到该消息的headers(也是一个键值对的形式)，对比其中的键值对是否完全匹配队列和交换器绑定时指定的键值对，如果完全匹配则消息会路由到该队列，否则不会路由到该队列。headers类型的交换器性能会很差，而且也不实用，基本上不会看到它的存在。


## 交换器类型详解
### 
新建rabbit-demo工程，在其中新建两个mudle 一个为rabbit-consumer 另一个为rabbit-producer
pom.xml
```POM
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
```
rabbit-producer/application.properties
```
server.port=1180
# 端口 1181 消费者  1180 生产者

spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
spring.rabbitmq.port=5672
```
rabbit-producer/application.properties
```
server.port=1181
# 端口 1181 消费者  1180 生产者

spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
spring.rabbitmq.port=5672
```
项目创建完成

### 声明一个队列各个参数的意义
name: 队列的名称 字符串
durable: 是否持久化, 队列的声明默认是存放到内存中的，如果rabbitmq重启会丢失，如果想重启之后还存在就要使队列持久化，保存到Erlang自带的Mnesia数据库中，当rabbitmq重启之后会读取该数据库
exclusive: 是否排外的，有两个作用，一：当连接关闭时connection.close()该队列是否会自动删除；二：该队列是否是私有的private，如果不是排外的，可以使用两个消费者都访问同一个队列，没有任何问题，如果是排外的，会对当前队列加锁，其他通道channel是不能访问的，如果强制访问会报异常，一般等于true的话用于一个队列只能有一个消费者来消费的场景。
autoDelete: 是否自动删除，当最后一个消费者断开连接之后队列是否自动被删除，可以通过RabbitMQ Management，查看某个队列的消费者数量，当consumers = 0时队列就会自动删除
arguments: 队列中的消息什么时候会自动被删除？ 是一个Map<String, Object>，他有如下参数
   "x-message-ttl": 1000  设置队列中的所有消息的生存周期
   "x-expires": 1000  当队列在指定的时间没有被访问就会被删除
   "x-max-length": 10  限定队列的消息的最大值长度，超过指定长度将会把最早的几条删除掉
   "x-max-length-bytes":  限定队列最大占用的空间大小， 一般受限于内存、磁盘的大小
   "x-dead-letter-exchange": "" 当队列消息长度大于最大长度、或者过期的等，将从队列中删除的消息推送到指定的交换机中去而不是丢弃掉
   "x-dead-letter-routing-key": ""  将删除的消息推送到指定交换机的指定路由键的队列中去
   "x-max-priority":  优先级队列，声明队列时先定义最大优先级值(定义最大值一般不要太大)，在发布消息的时候指定该消息的优先级， 优先级更高（数值更大的）的消息先被消费
   "x-queue-mode": "lazy" 先将消息保存到磁盘上，不放在内存中，当消费者开始消费的时候才加载到内存中
   "x-queue-master-locator"

### fanout代码实现
#### 在消费者项目中添加一个配置类
```JAVA
package com.jiafly.rabbit.consumer.fanout;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author liuyi
 * @date 2018/12/1 4:35 PM
 */
@Configuration
public class FanoutConfig {

    // 声明一个队列，后面有很多属性
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanout_exchange");
    }

    @Bean
    public Binding fanoutBinding1(){
        return BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
    }

    @Bean
    public Binding fanoutBinding2(){
        return BindingBuilder.bind(fanoutQueue2()).to(fanoutExchange());
    }

}
```
#### 在消费者项目中添加一个消息监听类
```JAVA
package com.jiafly.rabbit.consumer.fanout;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author liuyi
 * @date 2018/12/1 4:40 PM
 */
@Component
@Slf4j
public class FanoutConsumer {

    @RabbitListener(queues = "fanout.queue1")
    @RabbitHandler
    public void fanoutConsumer1(String msg){
        log.info("1fanoutConsumer 接收消息msg: {}", msg);
    }


    @RabbitListener(queues = "fanout.queue2")
    @RabbitHandler
    public void fanoutConsumer2(String msg){
        log.info("2fanoutConsumer 接收消息msg: {}", msg);
    }

}
```
#### 在生产者项目中添加一个消息发送controller
```JAVA
package com.jiafly.rabbit.producer;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author liuyi
 * @date 2018/11/28 11:50 AM
 * 生产者
 */
@RestController
@RequestMapping()
@Slf4j
public class FanoutProducer {

    @Autowired
    private AmqpTemplate template;

    /**
     * fanout类型
     * @param msg 消息内容
     * @return 消息内容
     * 虽然fanout类型下不需要routingKey 但是在调用convertAndSend方法时还是需要配置routingKey
     * 只是routingKey可以任意指定
     */
    @RequestMapping("/fanout/{msg}")
    public String fanoutProducer(@PathVariable("msg") String msg){
        log.info("fanout生产消息 msg:{}", msg);
        // 第一个参数是交换器名称 第二个参数是routingKey名称，fanout模式写任何key都会被无视 第三个是要发送的消息
        template.convertAndSend("fanout_exchange","", msg);
        return msg;
    }
}
```
#### 测试
分别启动两个项目，在浏览器的路径上输入 http://localhost:1180/fanout/测试消息 ，就可在消费者项目控制台中看到绑定这个fanout模式交换器的队列接收到的消息在控制台打印出来了。

### direct代码实现
#### 在消费者项目中添加一个配置类
```JAVA
package com.jiafly.rabbit.consumer.direct;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * @author liuyi
 * @date 2018/11/29 1:40 AM
 */
@Configuration
public class DirectConfig {

    /**
     * 定义两个队列
     * @return
     */
    @Bean
    public Queue directQueue1() {
        return new Queue("direct.queue1",true);
    }

    @Bean
    public Queue directQueue2() {
        return new Queue("direct.queue2", true);
    }

    @Bean
    public Queue directQueue3() {
        Map<String, Object> map = new HashMap<>(16);
        return new Queue("direct.queue3",true,true, true, map);
    }


    /**
     * 定义 exchange
     * @return
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct_exchange",true,true);
    }

    /**
     * 队列1 绑定 exchange 并且设置routingKey为direct.1
     * @return
     */
    @Bean
    public Binding directBinding1(){
        return BindingBuilder.bind(directQueue1()).to(directExchange()).with("direct.routing.key1");
    }

    /**
     * 队列2 绑定 exchange 并且设置routingKey为direct.2
     * @return
     */
    @Bean
    public Binding directBinding2(){
        return BindingBuilder.bind(directQueue2()).to(directExchange()).with("direct.routing.key2");
    }

    @Bean
    public Binding directBinding3(){
        return BindingBuilder.bind(directQueue3()).to(directExchange()).with("direct.routing.key1");
    }
}
```
#### 在消费者项目中添加一个消息监听类
```JAVA
和fanout相同，只是监听的队列不同而已
```
#### 在生产者项目中添加一个消息发送controller
```JAVA
package com.jiafly.rabbit.producer;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author liuyi
 * @date 2018/11/28 11:50 AM
 * 生产者
 */
@RestController
@RequestMapping()
@Slf4j
public class DirectProducer {

    @Autowired
    private AmqpTemplate template;

    /**
     * direct类型
     * @param msg 消息内容
     * @return 消息内容
     */
    @RequestMapping("/direct/queue1/{msg}")
    public String directProducerOne(@PathVariable("msg") String msg) {
        log.info("生产者生产消息:" + msg);
        // 第一个参数是交换器名称 第二个参数是routingKey名称 第三个是要发送的消息
        template.convertAndSend("direct_exchange", "direct.routing.key1", msg);
        return msg;
    }

    @RequestMapping("/direct/queue2/{msg}")
    public String directProducerTwo(@PathVariable("msg") String msg) {
        log.info("生产者生产消息:" + msg);
        // 第一个参数是交换器名称 第二个参数是routingKey名称 第三个是要发送的消息
        template.convertAndSend("mq-direct_exchange", "direct.routing.key2", msg);
        return msg;
    }
}
```
#### 测试
分别启动两个项目，在浏览器的路径上输入 http://localhost:1180/direct/queue1/测试消息1 ，就可在消费者项目中看到打印的信息。
如果需要两个队列接受相同的信息，只需要将两个队列绑定的routingKey设置为一样即可

### topic代码实现
#### 在消费者项目中添加一个配置类
```JAVA
package com.jiafly.rabbit.consumer.topic;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author liuyi
 * @date 2018/11/29 8:42 AM
 */
@Configuration
public class TopicConfig {

    /**
     * 创建队列
     *
     * @return
     */
    @Bean
    public Queue topicQueue1() {
        return new Queue("topic.queue1", true);
    }

    @Bean
    public Queue topicQueue2() {
        return new Queue("topic.queue2", true);
    }

    /**
     * 创建交换器
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("topic_exchange");
    }

    /**
     * 绑定
     */
    @Bean
    public Binding topicBinding1() {
        return BindingBuilder.bind(topicQueue1()).to(topicExchange()).with("*.jiafly.*");
    }

    @Bean
    public Binding topicBinding2() {
        return BindingBuilder.bind(topicQueue2()).to(topicExchange()).with("com.jiafly.*");
    }
}
```
#### 在消费者项目中添加一个消息监听类
```JAVA
和fanout相同，只是监听的队列不同而已
```
#### 在生产者项目中添加一个消息发送controller
```JAVA
package com.jiafly.rabbit.producer;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author liuyi
 * @date 2018/11/28 11:50 AM
 * 生产者
 */
@RestController
@RequestMapping()
@Slf4j
public class TopicProducer {

    @Autowired
    private AmqpTemplate template;

    /**
     * topic类型
     * @param msg 消息内容
     * @return 消息内容
     */
    @RequestMapping("/topic/{msg}")
    public String topicProducerOne(@PathVariable("msg") String msg) {
        log.info("topic:生产消息:" + msg);
        template.convertAndSend("topic_exchange", "com.jiafly.test", msg);
        return msg;
    }
}
```
#### 测试
分别启动两个项目，在浏览器的路径上输入 http://localhost:1180/topic/测试消息 ，就可在消费者项目中看到打印的信息。
如果需要两个队列接受相同的信息，只需要将两个队列绑定的routingKey使用*或者#表示


## 延时队列
### 延时队列能做什么
- 订单业务：在电商/点餐中，都有下单后 30 分钟内没有付款，就自动取消订单。
- 短信通知：下单成功后 60s 之后给用户发送短信通知。
- 失败重试：业务操作失败后，间隔一定的时间进行失败重试

这类业务的特点就是：非实时的，需要延迟处理，需要进行失败重试。一种比较笨的方式是采用定时任务，轮训数据库，方法简单好用，但性能底下，在高并发情况下容易弄死数据库，间隔时间不好设置，时间过大，影响精度，过小影响性能，而且做不到按超时的时间顺序处理。另一种就是用Java中的DelayQueue 位于java.util.concurrent包下，本质是由PriorityQueue和BlockingQueue实现的阻塞优先级队列。，这玩意最大的问题就是不支持分布式与持久化。

在 AMQP 协议中，或者 RabbitMQ 本身没有直接支持延迟队列的功能，但是可以通过前面 所介绍的 DLX 和 TTL 模拟出延迟队列的功能。所以在介绍延时队列之前，首先介绍下DLX(Dead-Letter-Exchange)和TTL(Time to Live)。
![延时队列](/image/rabbit/delay-mq.jpg)
### 死信交换器DLX(Dead-Letter-Exchange)
DLX：死信队列，用来存储有超时时间信息的消息， 并且可以设置当消息超时时，转发到另一个指定队列(此处设置转发到router), 无消费者，当接收到客户端消息之后，等待消息超时，将消息转发到指定的Router队列。

Router: 转发队列，用来接收死信队列超时消息， 如上示例消息，在接收到之后，消费者将消息解析，获取queueName，body,再向所获取的queueName队列发送一条消息，内容为body.

具体代码实现:
#### 在消费者项目中添加一个配置类
```JAVA
package com.jiafly.rabbit.consumer.delay;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;

/**
 * @author liuyi
 * @date 2018/12/2 12:44 AM
 */
@Configuration
public class DelayConfig {

    /**
     * 定义一个交换机
     *
     * @return
     */
    @Bean
    public DirectExchange delayExchange() {
        return new DirectExchange("delay_exchange", true, false);
    }

    /**
     * 转发队列
     *
     * @return
     */
    @Bean
    public Queue routerQueue() {
        return new Queue("router.queue", true, false, false);
    }

    /**
     * 转发队列绑定交换机
     *
     * @return
     */
    @Bean
    public Binding routerBinding() {
        return BindingBuilder.bind(routerQueue()).to(delayExchange()).with("router.routing.key");
    }


    /**
     * 死信队列
     *
     * @return
     */
    @Bean
    public Queue deadLetterQueue() {
        HashMap<String, Object> arguments = new HashMap(16);
        arguments.put("x-dead-letter-exchange", "delay_exchange");
        arguments.put("x-dead-letter-routing-key", "router.routing.key");
        return new Queue("dead.letter.queue", true, false, false, arguments);
    }


    /**
     * 死信队列绑定交换机
     *
     * @return
     */
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder.bind(deadLetterQueue()).to(delayExchange()).with("dead.letter.routing.key");
    }
}
```
#### 在消费者项目中添加一个消息监听类
```JAVA
package com.jiafly.rabbit.consumer.delay;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author liuyi
 * @date 2018/12/3 6:13 PM
 */
@Component
@Slf4j
public class DelayConsumer {

    @RabbitListener(queues = "router.queue")
    @RabbitHandler
    public void delayConsumer(String msg) {
        log.info("delay.queue1接收消息:{}", msg);
    }
}
```
#### 在生产者项目中添加一个消息发送controller
```JAVA
package com.jiafly.rabbit.producer;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.support.converter.AbstractJavaTypeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author liuyi
 * @date 2018/12/3 7:17 PM
 */
@RestController
@RequestMapping()
@Slf4j
public class DelayProducer {

    @Autowired
    private AmqpTemplate template;

    /**
     * 延迟队列
     * @param msg 消息内容
     * @return 消息内容
     */
    @RequestMapping("/delay/{msg}")
    public String delayProducerOne(@PathVariable("msg") String msg) {
        log.info("delay:生产消息:" + msg);
        template.convertAndSend("delay_exchange", "dead.letter.routing.key", msg, message -> {
            message.getMessageProperties().setExpiration(30 * 1000 + "");
            return message;
        });
        return msg;
    }
}
```
#### 测试
分别启动两个项目，在浏览器的路径上输入 http://localhost:1180/delay/测试消息 ，就可在消费者项目中看到打印的信息。 30秒后可以看到消费者管理后台有刚刚发送的消息被打印出来了。


## 消息的持久化
为了保证RabbitMQ在重启、奔溃等异常情况下数据没有丢失，除了对消息本身持久化为，还需要将消息传输经过的队列(queue)，交互机进行持久化(exchange)，持久化以上元素后，消息才算真正RabbitMQ重启不会丢失。
创建时候的参数:
- durable 
是否持久化，如果true，则此种队列叫持久化队列（Durable queues）。此队列会被存储在磁盘上，当消息代理（broker）重启的时候，它依旧存在。没有被持久化的队列称作暂存队列（Transient queues）。 
- execulusive 
表示此对应只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable 
- autoDelete 
当没有生成者/消费者使用此队列时，此队列会被自动删除。 
(即当最后一个消费者退订后即被删除)


设置消息持久化必须先设置队列持久化，要不然队列不持久化，消息持久化，队列都不存在了，消息存在还有什么意义。消息持久化需要将交换机持久化、队列持久化、消息持久化，才能最终达到持久化的目的。其实在前面就已经使用持久化了。我们在管理后台去看一下。


## 消息的确认与拒绝
消费者在处理消息的时候偶尔会失败或者有时会直接崩溃掉。而且网络原因也有可能引起各种问题，对于此AMQP有两种处理方式：
- 自动确认模式:
当RabbbitMQ将消息发送给应用后，消费者端自动回送一个确认消息，此时RabbitMQ删除此消息。

- 显式确认模式:
消费者收到消息后，可以在执行一些逻辑后，消费者自己决定什么时候发送确认回执（acknowledgement），RabbitMQ收到回执后才删除消息，这样就保证消费端不会丢失消息

如果一个消费者在尚未发送确认回执的情况下挂掉了，那么消息会被重新放入队列，并且在还有其他消费者存在于此队列的前提下，立即投递给另外一个消费者。如果当时没有可用的消费者了，消息代理会死等下一个注册到此队列的消费者，然后再次尝试投递。 
RabbitMQ里的消息是不会过期。当消费者挂掉后，RabbitMQ会不断尝试重推。所有单个消息的推送可能花费很长的时间.