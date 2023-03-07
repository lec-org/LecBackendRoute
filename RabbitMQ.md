# RabbitMQ 消息队列

## 1. 基本概念

### 1.1 MQ概述
是消息传输过程中保存消息的同期.多用于分布式系统.
![](http://oss.re1ife.top/media/20221220160820.png)

### 1.2 优势

- 1. 应用解耦
耦合性越高,容错性越低.
![](http://oss.re1ife.top/media/20221221204148.png)
右边三个系统其中一个一个失效不影响整个系统.
也可以有效增加系统减少系统.

- 2. 异步提速
没有MQ:![](http://oss.re1ife.top/media/20221221204317.png)
	- 总共耗时: 全部时间相加
有MQ:  ![](http://oss.re1ife.top/media/20221221204328.png)
	- 总耗时: 20 + 5 = 25ms

- 3. 消蜂填谷
峰值状态下系统宕机: ![](http://oss.re1ife.top/media/20221221204448.png)
消蜂: ![](http://oss.re1ife.top/media/20221221204518.png)

直观图: 
![](http://oss.re1ife.top/media/20221221204538.png)

填谷: 高峰被消, 但是会消息积压, 消费完成积压消息就是填谷.

### 1.3 缺点

- 系统可用性降低
系统引入外部依赖越多,系统稳定性越差, 一旦mq宕机 会造成影响.

- 系统复杂度提高
MQ异步调用如何保证没有被重复消费, 如何处理消息丢失情况?

- 一致性问题

使用条件:
1. 生产者不用从消费者获得反馈. 引入MQ之后直接调用. 
2. 允许短暂不一致性
3. 有效果 : 解耦 提速 消蜂收益大于成本

### 1.4 常见MQ产品

 ![](http://oss.re1ife.top/media/20221221205256.png)
也有用redis的...

### 1.5 简介

AMQP (高级消息队列协议): 一个网络协议, 应用层. 为面向消息的中间件设计.  比如HTTP协议

![](http://oss.re1ife.top/media/20221221205928.png)

RabbitMQ接触架构:
![](http://oss.re1ife.top/media/20221221210029.png)

- Broker:接收和分发消息的应用,RabbitMQ Server就是 Message Broker
-  Virtual host:出于多租户和安全因素设计的,把 AMQP 的基本组件划分到一个虚拟的分组中,类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时,可以划分出多个vhost,每个用户在自己的 vhost 创建 exchange/queue 等
-  Connection:publisher/consumer 和 broker 之间的 TCP 连接
-  Channel:每一次访问 RabbitMQ 都建立一个 Connection,在消息量大的时候建立 TCP Connection的开销将是巨大的,效率也较低。Channel 是在 connection 内部建立的逻辑连接.
- Exchange:message 到达 broker 的第一站,根据分发规则,匹配查询表中的 routing key,分发消息到queue 中去。常用的类型有:direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
-  Queue:消息最终被送到这里等待 consumer 取走
-   Binding:exchange 和 queue 之间的虚拟连接,binding 中可以包含 routing key。Binding 信息被保存到 exchange 中的查询表中,用于 message 的分发依据

六种工作模式: 
 - 简单模式
 - worke queues
 - Publish/Subscribe 发布与订阅
 - Routing 路由模式
 - Topics 主题模式
 - RPC (远程调用 不太算RabbitMQ里面的)

### 1.6 JMS

JAVA 消息服务应用接口, Java平台中关于消息中间件的API

例如: JDBC

### 1.7 安装和配置

官网: http://www.rabbitmq.com/

启动服务
`service rabbitmq-server start`
开启管理界面
`sudo rabbitmq-plugins enable rabbitmq_management`

修改默认配置
![](http://oss.re1ife.top/media/20221221215825.png)

```bash
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.9.13/plugins/rabbit-3.9.13/ebin/rabbit.app

#logback 修改[guest]
```

**创建允许远程连接的用户**
```bash
rabbitmqctl add_user username password
rabbitmqctl set_permissions -p  / username ".*" ".*" ".*"
rabbitmqctl set_user_tags username administrator
```
### 管理控制台

默认端口: 15672

## 快速入门

### 入门程序

1. 创建生产者 消费者
2. 添加以来
3. 编写生产者发生消息
4. 编写消费者接受消息

producer 程序：
```java
  
public class ProducerHelloWorld {  
    public static void main(String[] args) throws IOException, TimeoutException {  
        //1.创建连接工厂  
        ConnectionFactory factory = new ConnectionFactory();  
        //2.设置参数  
        factory.setHost("oj.re1ife.top");  
        factory.setPort(5672);  
        factory.setVirtualHost("/itcast");  
        factory.setUsername("re1ife");  
        factory.setPassword("qq11qq22qq33");  
        //3.创建 connection        
        Connection connection = factory.newConnection();  
        //4. 才创建 channel        
        Channel channel = connection.createChannel();  
        //5. 创建队列queue  
        /*queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)         * 1,queue: 队列名称  
         * 2,durable 是否持久化，mq重启后还在  
         * 3,exclusive：  
         *   * 是否独占 只有一个消费者监听  
         *   * connection关闭时是否删除队列  
         * 4.autoDelete 是否自动删除 没有Consumer时候自动删除  
         * 5.arguments： 参数  
         *         */  
        channel.queueDeclare("hello_word",true,false,false,null);  
        //6。 发送消息  
        /*  
         *String exchange, String routingKey, AMQP.BasicProperties props, byte[] body         * 1. exchange: 交换机名称。 简单模式使用默认的“”  
         * 2.routingKey： 路由名称  
         * 3.props： 配置信息  
         * 4.body:发送的消息数据  
         */        String body="hello rabbitmq";  
        channel.basicPublish("","hello_word",null,body.getBytes());  
        //7.释放资源  
        channel.close();  
        connection.close();  
    }  
}
```


consumer代码
```java
package com.example.consumer;  
  
import com.rabbitmq.client.*;  
  
import java.io.IOException;  
import java.util.concurrent.TimeoutException;  
  
public class ConsumerHelloWorld{  
    public static void main(String[] args) throws IOException, TimeoutException {  
        ConnectionFactory factory = new ConnectionFactory();  
        factory.setHost("oj.re1ife.top");  
        factory.setUsername("re1ife");  
        factory.setPassword("qq11qq22qq33");  
        factory.setPort(5672);  
        factory.setVirtualHost("/itcast");  
        Connection connection = factory.newConnection();  
        Channel channel = connection.createChannel();  
        channel.queueDeclare("hello_word",true,false,false,null);  
  
        //接收消息  
        /*  
        *String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback        
        * 1. queue 队列名称  
        * 2. autoAck 是否自动确认  
        * 3. callback 回调对象  
        */        Consumer consumer = new DefaultConsumer(channel){  
  
  
            /*  
            回调方法 收到消息之后自动执行该方法  
  
             1. consumerTag: 表示  
             2. envelope: 一些信息: 交换机 路由器key  
             3. properties: 配置信息  
             4. body数据  
             */            @Override  
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
                System.out.println("consumerTag: "+consumerTag);  
                System.out.println("Exchange: " + envelope.getExchange());  
                System.out.println("RoutingKey: "+envelope.getRoutingKey());  
                System.out.println("properties: "+properties);  
                System.out.println("body:"+new String(body));  
  
            }  
        };  
  
  
        //不需要关闭资源  
        channel.basicConsume("hello_word",true,consumer);  
    }  
  
}
```


## RabbitMQ 工作模式

### WorkQueues 工作队列模式
![](http://oss.re1ife.top/media/20221228153731.png)

- 与入门简单模式相比多了一个或多个消费端, 多个消费端消费同一个队列中的消息
- 应用场景: 任务过重的情况

```java
//consumer代码 与consumer1相同
  
public class ConsumerWorkQueue {  
    public static void main(String[] args) throws IOException, TimeoutException {  
        ConnectionFactory factory = new ConnectionFactory();  
        factory.setHost("oj.re1ife.top");  
        factory.setUsername("re1ife");  
        factory.setPassword("qq11qq22qq33");  
        factory.setPort(5672);  
        factory.setVirtualHost("/itcast");  
        Connection connection = factory.newConnection();  
        Channel channel = connection.createChannel();  
        channel.queueDeclare("work_queue",true,false,false,null);  
  
        //接收消息  
        /*  
         *String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback         * 1. queue 队列名称  
         * 2. autoAck 是否自动确认  
         * 3. callback 回调对象  
         */        Consumer consumer = new DefaultConsumer(channel){  
  
  
            /*  
            回调方法 收到消息之后自动执行该方法  
  
             1. consumerTag: 表示  
             2. envelope: 一些信息: 交换机 路由器key  
             3. properties: 配置信息  
             4. body数据  
             */            @Override  
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {  
  
                System.out.println("body:"+new String(body));  
  
            }  
        };  
  
  
        //不需要关闭资源  
        channel.basicConsume("work_queue",true,consumer);  
    }  
}

```

例子: consumer和consumer1消费:
![](http://oss.re1ife.top/media/20221228154508.png)
分担均匀 

- 消费者之间的关系为竞争关系.
- 使用案例:短信服务.
工作队列消费者获取消息的时候会阻塞队列。

### Pub/Sub 订阅模式

![](http://oss.re1ife.top/media/20221228154710.png)

- P: 生产者, 发送消息的程序,不在队列中, 直接发给X 交换机
- C: 消费者
- Queue: 消息队列, 缓存消息, 接受消息
- X: 交换机,一方面接受生产者发送消息, 另一方面, 知道如何处理消息(将消息交给哪个队列)

- **交换机常用类型:** 
	- Fanout: 广播, 消息交给所有绑定的交换机队列
	- DIrect: 定向, 吧消息交给符合指定的Routing Key
	- Topic: 通配符, 把消息交给符合routing pattern(路由模式)的队列
	- **交换机**: 只负责转发消息, 没有储存消息的能力. 若没有符合路由规则的队列或者没有任何队列与交换机绑定. **消息会丢失**.


#### Fanout

- 生产者
```java
  
public class ProducerPubSub {  
    public static void main(String[] args) throws IOException, TimeoutException {  
        ConnectionFactory factory = new ConnectionFactory();  
        factory.setPort(5672);  
        factory.setUsername("re1ife");  
        factory.setHost("oj.re1ife.top");  
        factory.setPassword("qq11qq22qq33");  
        factory.setVirtualHost("/itcast");  
        Connection connection = factory.newConnection();  
        Channel channel = connection.createChannel();  
  
        //创建交换机  
        /*  
        * String exchange, BuiltinExchangeType type, boolean durable, boolean autoDelete, Map<String, Object> arguments        * 1. exchange 交换机名称  
        * 2. type 交换机类型  
        *   DIRECT("direct") : 定向  
            FANOUT("fanout") : 扇形(广播 )            TOPIC("topic")   : 通配符方式  
            HEADERS("headers") : 参数匹配  
        * 3. durable : 是否持久化  
        * 4. autoDelete : 自动删除 * 5. internal : 内部使用  
        * 6. argument : 参数  
        */        String exchangeName = "test_fanout";  
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT,true,false,false,null);  
  
  
        //创建队列  
        String queue1Name = "test_fanout_queue1";  
        String queue2Name = "test_fanout_queue2";  
        channel.queueDeclare(queue1Name,true,false,false,null);  
        channel.queueDeclare(queue2Name,true,false,false,null);  
  
        //绑定队列和交换机  
        /*  
         * String queue: 队列名称  
         * String exchange ： 交换机名称  
         * String routing key： 若交换机为fanout 则，routingkey为“”  
         */        channel.queueBind(queue1Name,exchangeName,"");  
        channel.queueBind(queue2Name, exchangeName, "");  
        //发送消息  
        String body = "hello puub/sub";  
        channel.basicPublish(exchangeName, "", null,body.getBytes());  
        //释放资源  
        channel.close();  
        connection.close();  
  
    }
}
```

点击运行之后，会生成一个`test_fanout`的交换机，绑定了两个队列。每个队列中都有一个`hello puub/sub`的消息

consumer代码：
```java
  
public class ConsumerPubSub{  
    public static void main(String[] args) throws IOException, TimeoutException {  
        ConnectionFactory factory = new ConnectionFactory();  
        factory.setHost("oj.re1ife.top");  
        factory.setUsername("re1ife");  
        factory.setPassword("qq11qq22qq33");  
        factory.setPort(5672);  
        factory.setVirtualHost("/itcast");  
        Connection connection = factory.newConnection();  
        Channel channel = connection.createChannel();  
        String queue1Name = "test_fanout_queue1";  
        String queue2Name = "test_fanout_queue2";  
          
        Consumer consumer = new DefaultConsumer(channel){  
            @Override  
            public void handleDelivery(String consumerTag, com.rabbitmq.client.Envelope envelope, com.rabbitmq.client.AMQP.BasicProperties properties, byte[] body) throws java.io.IOException {  
                System.out.println("body: " + body.toString());  
                System.out.println("将日志信息打印到控制台。。。");  
            };  
  
        };  
        channel.basicConsume(queue1Name, true,consumer);  
  
    }  
}
```
![](http://oss.re1ife.top/media/20230108171218.png)


conusmer1 只是与conusmer2的队列名称不同。

#### Routing 路由模式

- 模式说明：
	- 队列和交换机绑定，需要指定一个routing key
	- 消息发送方向交换机发送消息时，需要指定消息的routing key
	- 交换机不把消息给每个绑定队列， 根据routing key进行判断。 一致的才会收到消息。

需求前提： 日志信息分为`info`级别和`error` 级别， info直接打印， error存入数据库。以此来考虑。

![](http://oss.re1ife.top/media/20230108171917.png)


consumer 样例代码:
```java
public class ConsumerRouting2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory  factory = new ConnectionFactory();
        factory.setHost("oj.re1ife.top");
        factory.setPort(5672);
        factory.setUsername("re1ife");
        factory.setPassword("qq11qq22qq33");
        factory.setVirtualHost("/itcast");

        Connection connection =  factory.newConnection();
        Channel channel = connection.createChannel();
        String queueName = "test_direct_queue2";

        Consumer consumer = new DefaultConsumer(channel){

            @Override
            public void handleDelivery(String consumerTag, com.rabbitmq.client.Envelope envelope, com.rabbitmq.client.AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("队列2收到消息:" + new String(body) +" 消息级别:" + envelope.getRoutingKey());

            };
            
        };

        channel.basicConsume(queueName,true , consumer);
        
    }
}
```

producer 代码:
```java

public class ProducerRouting {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("oj.re1ife.top");
        factory.setPort(5672);
        factory.setUsername("re1ife");
        factory.setPassword("qq11qq22qq33");
        factory.setVirtualHost("/itcast");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        //创建交换机
        String exchangeName = "test_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT,true,false,false,null);

        //创建队列
        String queue1Name = "test_direct_queue1";
        String queue2Name = "test_direct_queue2";
        channel.queueDeclare(queue1Name, false, false, false, null);
        channel.queueDeclare(queue2Name, false, false, false, null);

        //绑定交换机和队列
        channel.queueBind(queue1Name , exchangeName, "error");
        channel.queueBind(queue2Name , exchangeName, "info");
        channel.queueBind(queue2Name , exchangeName, "warning");
        channel.queueBind(queue2Name , exchangeName, "error");


        //发送消息
        String infoMsg = "小明登陆";
        String warnMsg = "小明越界";
        String errorMsg = "系统崩坏";
        channel.basicPublish(exchangeName,"info", null,infoMsg.getBytes());
        channel.basicPublish(exchangeName,"warning", null,warnMsg.getBytes());
        channel.basicPublish(exchangeName,"error", null,errorMsg.getBytes());

        channel.close();
        connection.close();
    }
}
```

consumer1 的代码只需要改动`queueName`即可.

运行示意图:
![](http://oss.re1ife.top/media/20230108174240.png)
![](http://oss.re1ife.top/media/20230108174439.png)


#### Topics通配符模式

![](http://oss.re1ife.top/media/20230108174702.png)

右面的图很好的说明了通配符模式, 例如`usa.news` 可以匹配第一个队列`usa.*`和第二个队列 `*.news`.
- 星号(\*）代表一个英文单词（例如cn）
- 井号（#）代表零个、一个或多个英文单词.

需求: 所有error存入数据库, 所有order系统日志存入数据库

producer代码:
```java
public class ProducerTopics {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setPort(5672);
        factory.setUsername("re1ife");
        factory.setHost("oj.re1ife.top");
        factory.setPassword("qq11qq22qq33");
        factory.setVirtualHost("/itcast");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //创建交换机
        String exchangeName = "test_topics";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC,true,false,false,null);


        //创建队列
        String queue1Name = "test_topics_queue1";
        String queue2Name = "test_topics_queue2";

        //routing key 系统的名称.日志级别
        //需求: 所有error存入数据库, 所有order系统日志存入数据库

        channel.queueDeclare(queue1Name,true,false,false,null);
        channel.queueDeclare(queue2Name,true,false,false,null);

        //绑定队列和交换机
        /*
         * String queue: 队列名称
         * String exchange ： 交换机名称
         * String routing key： 若交换机为fanout 则，routingkey为“”
         */
        channel.queueBind(queue1Name, exchangeName, "#.error");
        channel.queueBind(queue1Name, exchangeName, "order.*");
        channel.queueBind(queue2Name, exchangeName, "*.*");


        //发送消息
        String errorMsg = "系统崩坏";
        String infoMsg = "游客登陆";
        String orderMsg  ="小刘下单了";
        //routing key 多级
        channel.basicPublish(exchangeName,"system.info", null,infoMsg.getBytes());
        channel.basicPublish(exchangeName,"order.info", null,orderMsg.getBytes());
        channel.basicPublish(exchangeName,"system.error", null,errorMsg.getBytes());
        //释放资源
        channel.close();
        connection.close();

    }
}


```

consumer:
```java
public class ConsumerTopics2 {

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory  factory = new ConnectionFactory();
        factory.setHost("oj.re1ife.top");
        factory.setPort(5672);
        factory.setUsername("re1ife");
        factory.setPassword("qq11qq22qq33");
        factory.setVirtualHost("/itcast");

        Connection connection =  factory.newConnection();
        Channel channel = connection.createChannel();
        String queueName = "test_topics_queue2";

        Consumer consumer = new DefaultConsumer(channel){

            @Override
            public void handleDelivery(String consumerTag, com.rabbitmq.client.Envelope envelope, com.rabbitmq.client.AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("队列2收到消息:" + new String(body) +" 消息级别:" + envelope.getRoutingKey());

            };
            
        };

        channel.basicConsume(queueName,true , consumer);
        
    }
}


```


topics 使用通配符可以更加灵活.

## SpringBoot 整合 RabbitMQ

### 生产者整合
1. 创建工程
2. 引入依赖
```xml

<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
3. 编写yml配置文件
```yml
spring:
  rabbitmq:
    host: 
    password: 
    port: 5672
    username: 
```
3. 定义交换机\\队列以及绑定关系的配置类
```java


@Configuration
public class RabbitMQConfig {

    public static final String EXCHANGE_NAME = "boot_topic_exchange";
    public static final String QUEUE_NAME = "boot_queue";
    
    //1 交换机
    @Bean(name = "bootExchange")
    public org.springframework.amqp.core.Exchange bootExchange(){

        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
    }
    //2 队列

    @Bean(name = "bootQueue") 
    public Queue bootQueue(){
        return QueueBuilder.durable(QUEUE_NAME).build(); 
    }

    // 3 交换机和队列绑定
    /*
     * 1. 知道哪个队列
     * 2. 知道哪个交换机
     * 3.routing KEY
     */

    @Bean
    public Binding bindQueuExchange(@Qualifier("bootExchange") org.springframework.amqp.core.Exchange exchange, @Qualifier("bootQueue") Queue queue){
        return BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();

    }
}

```

**注意**: Rabbit有几个不同的Exchange, 有点难以将区分. 要注意使用的是哪一种.

5. 注入RabbitTemplate , 调用方法 完成消息发送

### 消费者整合
1. 工程创建
2. 坐标引入
3. 编写yml
4. 定义监听类 使用`@RabbitListener`完成队列监听
```java
@Component
public class RabbitMQListener {
    @RabbitListener(queues = "boot_queue")
    public void ListenerQueue(Message message){
        System.out.println(message);

    }    
}
```

## 进阶概述

- 高级特性
	- 消息可靠性投递
	- consumer ACK
	- 消费端限流
	- TTL
	- 死信队列
	- 延迟队列
	- 消息可靠性分析和追踪
	- 管理
- 应用问题
	- 可靠性保障
	- 消息幂等性处理
	- 
- 集群搭建


### 消息应答
消费者完成一个任务可能需要时间, 在这段时间中若突然中断. 消息被消费而标记删除是不合理的. 因此需要**在接受并且处理掉消息之后告诉rabbitmq已经处理完成.**

- 自动应答: 消息发送后立即被确认已经传送成功.
	- 高吞吐量和数据传输安全性方面权衡
	- 缺点: 消费者那边可以传递过载的消息,没有对传递的消息数量进行限制, 由于接收太多还来不及处理的消息,导致这些消息的**积压**. 占用过高被**操作系统杀了**.
	- **应用**: 消费者可以高效并以某种速率能够处理这些消息
	- 应答方法: 
		1. Channle.basicACK(用于肯定确认) rabbitmq知道该消息并且成功处理
			1. Mulitple: 
				1. true 含义: 表示批量应答
					- 比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是 8 那么此时5-8 的这些还未应答的消息都会被确认收到消息应答.
				2. false : 只会应答tag的消息
		2. Channle.basicNack (否定)
		3. Channle.basicReject (否定) 不处理消息 直接丢弃
![](http://oss.re1ife.top/media/20230110153434.png)

**默认消息采用自动应答**, 我们要想消息不丢失,应该改为手动应答.
```java
public static void main(String[] args){
	Channel channle = RabbitUtils.getChannel(); //将获取channle的代码封装到了util中
	channel.queueDeclare(TASK_QUEUE, false, false,false,null);
	sout("等待接受消息");
	DeliverCallback deliverCallBack = (consumerTag, delivery) -> {
		String message  = new String(delivery.getBody(),"UTF-8");
		sout(message);
		channel.basickAck(delivery.getEnvelope().getDeliveryTag(),false); //手动应答
		
	};
	boolean autoACK  =  false;
	channel.basicConsume(TASK_QUEUE, autoACK, deliveryCallback, consumerTag ->{});
}
```


### 不公平分发
RabbitMQ 消息分发采用轮询分发. 在有一个消费者处理快一个慢的时候不好.

为了避免可以设置参数`channel.basicQos(1);`
![](http://oss.re1ife.top/media/20230110154303.png)

## 高级特性

### 消息的可靠投递

两种方式:
 - confirm 确认模式
	 - ![](http://oss.re1ife.top/media/20230110155456.png)
 - return 退回模式

消息投递路径: 
producer --> rabbitmq broker --> exchange --> queue --> consumer

 - producer 到 exchange 会返回一个confirmCallback的回调函数(返回 true/fasle)
	 - 无论成功与否都会发生
 -   exchange 到 queue 投递失败就会返回一个 returnCallback


#### 确认策略

默认情况下发布确认没有开启, 若要开启需要调用confirmSelect, 每次使用时都要调用.

- 单个确认发布

最简单的确认方式, 同步确认. 发布一个消息之后只有他被确认才可以继续发布.

`waitForConfirmsOrDie(long)` 这个方法被确认之后才返回.
**缺点**: 发布速度特别慢.

```java
public static void publishMessageIndividually() throws Exception{
	Channel channel = RabbitUtil.getChannel();
	// 队列声明 
	String queueName =UUID.randomUUID().toString();
	channel.queueDeclare(queueName, true, false, false,null);
	//开启确认
	channel.confirmSelect();
	//开始时间
	Long  begin = System.currentTimeMillis();
	for(int i = 0; i < 1000;i++){
		String message = i+"";
		 channel.basicPublish("", queueName, null, message.getBytes());
		 boolean flag = channel.waitForConfirms();
		 if(flag){
			System.out.println("i: "+i+" 消息发送成功!");
		 }
	}

	long end = System.currentTimeMillis();
	System.out.println("用时: "+(end - begin)+ "ms");



}
```


- 批量确认发布
	同步式消息确认, 也阻塞消息发布.
	缺点: 发生故障时, 不知道是由于那一个产生的.

```java
    //批量确认
    public static void publishMessageBatch() throws Exception{
        Channel channel = RabbitUtil.getChannel();
        // 队列声明 
        String queueName =UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false,null);
        //开启确认
        channel.confirmSelect();
        //开始时间
        Long  begin = System.currentTimeMillis();


        //批量确认消息大小
        int batchSize = 100;
        //批量发布确认    
        for(int i = 0; i < 1000;i++){
            String message = i+"";
            channel.basicPublish("", queueName, null, message.getBytes());
            if(i % batchSize == 0) channel.waitForConfirms();
        }

        long end = System.currentTimeMillis();
        System.out.println("用时: "+(end - begin)+ "ms");



    }
```
- 异步确认发布

缺点: 编程逻辑复杂
优点: 性价比高

使劲发送, 成没成功让别人告诉你.
![](http://oss.re1ife.top/media/20230110164212.png)



```java
    //异步发布确认
    public static void publishMessageAsync() throws Exception{
        Channel channel = RabbitUtil.getChannel();
        // 队列声明 
        String queueName =UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false,null);
        //开启确认
        channel.confirmSelect();
        //开始时间
        Long  begin = System.currentTimeMillis();

        //消息监听器 那些成功了哪些失败
            // 成功消息监听
        ConfirmCallback ackCallback = (deliveryTag, Multiple ) -> {
            System.out.println("确认消息: "  + deliveryTag);
        };
            //失败消息监听
        ConfirmCallback nackCallback = (deliveryTag, Multiple ) -> {
            System.out.println("未确认消息: " + deliveryTag);
        };
        channel.addConfirmListener(ackCallback, nackCallback);


        for (int i = 0; i < 1000; i++) {
            String message = i+"";
            channel.basicPublish("", queueName, null, message.getBytes());
        }
        

        long end = System.currentTimeMillis();
        System.out.println("用时: "+(end - begin)+ "ms");

    }
```

既然有异步未确认的消息, 那该怎么处理呢?
	-  把未确认的消息放到一个基于内存的被发布线程访问的队列, `ConcurrentLinkedQueue` 在 `Confrim Callbacks` 与发布线程之间进行消息的传递.
 ```java
  public static void publishMessageAsync() throws Exception{
        Channel channel = RabbitUtil.getChannel();
        // 队列声明 
        String queueName =UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false,null);
        //开启确认
        channel.confirmSelect();
        //线程安全的哈希表
        // 关联消息和序号
        // 轻松批量删除题目
        // 线程安全
        ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

        

        //消息监听器 那些成功了哪些失败
            // 成功消息监听
        ConfirmCallback ackCallback = (deliveryTag, Multiple ) -> {
            // 删除已经确认消息 Multiple 注意是不是批量确认的
            if(Multiple){
                ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(deliveryTag);
                confirmed.clear();
            }else{
                outstandingConfirms.remove(deliveryTag);
            }
            
            System.out.println("确认消息: "  + deliveryTag);
        };
            //失败消息监听
        ConfirmCallback nackCallback = (deliveryTag, Multiple ) -> {
            String message = outstandingConfirms.get(deliveryTag);
            System.out.println("未确认消息:" + message +" tag: " + deliveryTag);

        };
        Long  begin = System.currentTimeMillis();

        channel.addConfirmListener(ackCallback, nackCallback);


        for (int i = 0; i < 1000; i++) {
            String message = i+"";
            channel.basicPublish("", queueName, null, message.getBytes());
            //记录所有要发送消息
            outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
        }
        

        long end = System.currentTimeMillis();
        System.out.println("用时: "+(end - begin)+ "ms");

    }
```

#### 发布确认高级

rabbitmq的发布确认方案: ![](http://oss.re1ife.top/media/20230113224910.png)

从图中可知, 当生产者投递消息失败则存入`缓存`中. 当交换机受到对应消息则清除缓存, 交换机和队列是BROKER部分, 则BROKER 部分无法接受消息则无法使用,

- 代码架构图
![](http://oss.re1ife.top/media/20230113225251.png)

- 添加配置类
需要修改配置文件:
`spring.rabbitmq.publisher-confirm-type = correlated`
可使用值: 
	- None: 禁用发布确认模式
	- CORRELATED: 发布消息成功后出发回调方法
	- SIMPLE: 两种效果
		- 1. 和CORRELATED 一样出发回调方法
		- 2. 发布成功后使用rabbitTemplate.waitForConfirms或rabbitTemplate.waitForConfirmsOrDie 方法等待Broker节点返回发送结果. waitForConfirmsOrDie收到False会关闭channel
- 
```java

//发布确认高级
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE = "confirm.exchange";
    public static final String CONFIRM_QUEUE = "confirm.queue";
    public static final String CONFIRM_ROUTING_KEY = "key1";

    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        return new DirectExchange(CONFIRM_EXCHANGE);
    }
    
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE).build();
    }

    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue queue, @Qualifier("confirmExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(CONFIRM_ROUTING_KEY);
    }

}

```
- 生产者
```java
//开始发消息 测试确认
@Slf4j
@RestController
@RequestMapping("/confrim")
public class ProducerController{
    @Autowired
    private RabbitTemplate rabbitTemplate;

    //发消息
    @GetMapping("/sendMessage/{message}")
    public void sendMessage(@PathVariable String message){
        CorrelationData correlationData = new CorrelationData("1");
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE, ConfirmConfig.CONFIRM_ROUTING_KEY,message,correlationData);

        log.info("发送消息内容:{} ",message);
    }
}
```
- 消费者
```java

@Component
@Slf4j
public class ConfirmConsumer {
    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE)
    public void receiveConfirmMsg(Message message){
        
        log.info("接受到的confirm.queue消息: {}",new String(message.getBody()));
    }
}

```
- 回调接口
```java

@Component
@Slf4j
//回调消息
public class ProducerCallBack implements RabbitTemplate.ConfirmCallback{
    //将实现注入到接口
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
    }

    /*
     * 交换机确认回调方法
     * 1. 发消息 交换机受到了 回调
     *  1.1 coorelationData 保存回调消息的ID和相关信息
     *  1.2 交换机ack true
     *  1.3 cause null
     * 2. 发送消息 交换机接受失败 回调
     *  2.1 coorelationData 保存回调消息的ID和相关信息
     *  2.2 交换机ack false
     *  2.3 cause 失败原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if(ack){
            log.info("交换机受到id为:{} 的消息", correlationData.getId());
        }else{
            log.info("交换机还未受到id为:{}的消息, 原因:{}",correlationData.getId(), cause);
        }
        
    }

    
}
```

- 正常使用实例: 
![](http://oss.re1ife.top/media/20230114140527.png)
- 模拟交换机名称错误:
![](http://oss.re1ife.top/media/20230114140641.png)
- 队列名称错误: 
![](http://oss.re1ife.top/media/20230114141104.png)
routingkey正常的消息一切正常 , 而没有受到routingKey不正常的
因此我们还要设置队列的消息回退. 

#### 消息回退
- Mandatory参数
在仅开启生产者确认及之下, 交换机 受到后给生产者返回确认, 但是交换机找不到routingKey时候会直接将消息丢弃. 生产者不知道.


实现`RabbitTemplate.ReturnConfirmCallback`
```java
package com.example.bootrabbit.CallBack;

@Component
@Slf4j
@SuppressWarnings("all")

//回调消息
public class ProducerCallBack implements RabbitTemplate.ConfirmCallback,ReturnCallback{
    //将实现注入到接口
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnCallback(this);
    }

    /*
     * 交换机确认回调方法
     * 1. 发消息 交换机受到了 回调
     *  1.1 coorelationData 保存回调消息的ID和相关信息
     *  1.2 交换机ack true
     *  1.3 cause null
     * 2. 发送消息 交换机接受失败 回调
     *  2.1 coorelationData 保存回调消息的ID和相关信息
     *  2.2 交换机ack false
     *  2.3 cause 失败原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if(ack){
            log.info("交换机受到id为:{} 的消息", correlationData.getId());
        }else{
            log.info("交换机还未受到id为:{}的消息, 原因:{}",correlationData.getId(), cause);
        }
        
    }


    //消息不可到达时候回退
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey){
        log.error("消息{},被交换机{}退回,退回原因{}, 路由key:{}", new String(message.getBody()),exchange,replyText,routingKey);
        
        
    }

}

```
![](http://oss.re1ife.top/media/20230114205423.png)

### 交换机备份

代码架构图: 
![](http://oss.re1ife.top/media/20230114214401.png)

- 修改配置类
```java
//发布确认高级
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE = "confirm.exchange";
    public static final String CONFIRM_QUEUE = "confirm.queue";
    public static final String CONFIRM_ROUTING_KEY = "key1";
    public static final String BACUP_EXCHANGE = "backup.exchange";
    public static final String BACK_QUEUE = "backup.queue";
    public static final String WARN_QUEUE = "warning.queue";

    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
    
        return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE).durable(true).alternate(BACUP_EXCHANGE).build(); 
    }
    
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE).build();
    }

    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACUP_EXCHANGE);
    }

    @Bean("backupQueue")
    public Queue backupQueue(){
        return QueueBuilder.durable(BACK_QUEUE).build();
    }
    
    @Bean("warnQueue")
    public Queue warningQueue(){
        return QueueBuilder.durable(WARN_QUEUE).build();
    }



    @Bean
    public Binding backupQueueBindingToBackupExchange(@Qualifier("backupExchange") FanoutExchange exchange, @Qualifier("backupQueue") Queue queue){
        return BindingBuilder.bind(queue).to(exchange); 
    }

    @Bean
    public Binding warnQueueBindingToBackupExchange(@Qualifier("warnExchange") FanoutExchange exchange, @Qualifier("backupQueue") Queue queue){
        return BindingBuilder.bind(queue).to(exchange);
    }

    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue queue, @Qualifier("confirmExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with(CONFIRM_ROUTING_KEY);
    }

}

```

- 消费者
```java
@Slf4j
@Component
public class WarningConsumer {
    @RabbitListener(queues = ConfirmConfig.WARN_QUEUE)
    public void receiveWarningMsg(Message message){
        log.error("报警发现不可路由消息: {}", new String(message.getBody()));
    }
}
```

测试结果: ![](http://oss.re1ife.top/media/20230114220812.png)
同时可以得出 **备份交换机** 优先级高于MANDATORY
### 死信队列

无法被消费的消息

应用场景: 保证订单业务消息数据不丢失, 可以使用RabbitMQ的死信队列机制. 消息消费发生异常, 将消息投入死信队列中. 订单被取消


#### 死信的来源
- 消息TTL过期
- 队列到达最大长度 无法再添加数据到mq
- 消息被拒绝 (basic.reject 或 basic.nack) 并且requeue = false
![](http://oss.re1ife.top/media/20230110193106.png)


- TTL实现:
normal-consumer 代码:
```java
public class NormalConsumer {
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_EXCHANGE = "dead_exchange";
    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitUtil.getChannel();
        //死信交换机和普通交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        
        Map<String, Object> arguments = new HashMap<>();
        //过期时间
        // arguments.put("x-message-ttl", 10000);
        //正常队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //死信Routing Key
        arguments.put("x-dead-letter-routing-key", "lisi");
        //普通队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        //死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false,false,null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
             System.out.println("normal接受的消息是:" + new String(message.getBody()));
        };
        
        channel.basicConsume(NORMAL_QUEUE, deliverCallback, consumerTagt -> {});

    }
}

```


producer 代码:
```java
//生产者队列
public class Producer {

    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_EXCHANGE = "dead_exchange";
    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel  channel = RabbitUtil.getChannel();

        //死信队列 TTL
        AMQP.BasicProperties properties = new AMQP.BasicProperties()
            .builder().expiration("10000").build();


        for (int i = 0; i < 10; i++) {
            String messsage = "info: " +i;
            channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", properties, messsage.getBytes());

        }

        
        
    }
}


```

死信队列的消费者代码:
```java
public class DeadConsumer {
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_EXCHANGE = "dead_exchange";
    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String DEAD_QUEUE = "dead_queue";
    
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitUtil.getChannel();
        DeliverCallback deliverCallback = (consumerTage, message) ->{
            System.out.println("死信队列受到消息:" + new String( message.getBody()));
        };
        channel.basicConsume(DEAD_QUEUE, deliverCallback, consumerTag ->{}); 
    }
}

```

- 队列最大长度实现

只需要在设置参数的时候添加:` arguments.put("x-max-length",6);`
```java
public class NormalConsumer {
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_EXCHANGE = "dead_exchange";
    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitUtil.getChannel();
        //死信交换机和普通交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        
        Map<String, Object> arguments = new HashMap<>();
        //过期时间
        // arguments.put("x-message-ttl", 10000);
        //正常队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //死信Routing Key
        arguments.put("x-dead-letter-routing-key", "lisi");
        //max-length:
        arguments.put("x-max-length",6);
        //普通队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        //死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false,false,null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
             System.out.println("normal接受的消息是:" + new String(message.getBody()));
        };
        
        channel.basicConsume(NORMAL_QUEUE, deliverCallback, consumerTagt -> {});

    }
}

```

- 被拒绝实现
```java
//normal-consumer代码
public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitUtil.getChannel();
        //死信交换机和普通交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        
        Map<String, Object> arguments = new HashMap<>();
        //过期时间
        // arguments.put("x-message-ttl", 10000);
        //正常队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //死信Routing Key
        arguments.put("x-dead-letter-routing-key", "lisi");
        //普通队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        //死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false,false,null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            String msg = new String(message.getBody());

            //模拟被拒绝的条件
            if(msg.equals("info5")){
                System.out.println("拒绝: " + msg);
                channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
            }else{
                System.out.println("normal接受的消息是:" + new String(message.getBody()));
            }
             
        };
        
        channel.basicConsume(NORMAL_QUEUE, deliverCallback, consumerTagt -> {});

    }
```





### 延迟队列

相当于死信队列的TTL删去NormalConsumer 

延迟队列是有序的, 重要特性体现在延时属性. 延时队列中的元素希望在指定时间后取出或处理.

使用场景: 
1. 十分钟内没有支付的订单
2. 新创建的店铺, 十天内没有上传过商品.
3. 用户创建后, 三天内没有登陆
4. ....


实战架构图:
![](http://oss.re1ife.top/media/20230110230652.png)

延迟队列配置类:
```java
@Configuration
public class TtkQueueConfig {
    // 死信和普通交换机 和 队列
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_EXCHANGE = "dead_exchange";
    public static final String DEAD_QUEUE  = "dead_queue";
    public static final String NORMAL_QUEUE1  = "normal_queue1";
    public static final String NORMAL_QUEUE2  = "normal_queue2";
    

    //交换机
    @Bean("normalExchange")
    public DirectExchange normalExchange(){
        return new DirectExchange(NORMAL_EXCHANGE);
    }

    @Bean("deadExchange")
    public DirectExchange deadExchange(){
    return new DirectExchange(DEAD_EXCHANGE);
    }

    //队列
    @Bean("normalQueue1")
    public Queue normalQueue1(){
        Map<String, Object> argument = new HashMap<>();
        //设置死信交换机
        argument.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        //设置死信Routing key
        argument.put("x-dead-letter-routing-key","YD");
        //TTl
        argument.put("x-message-ttl", 10000);
        return QueueBuilder.durable(NORMAL_QUEUE1).withArguments(argument).build();
    }

    @Bean("normalQueue2")
    public Queue normalQueue2(){
        Map<String, Object> argument = new HashMap<>();
        //设置死信交换机
        argument.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        //设置死信Routing key
        argument.put("x-dead-letter-routing-key","YD");
        //TTl
        argument.put("x-message-ttl", 40000);
        return QueueBuilder.durable(NORMAL_QUEUE2).withArguments(argument).build();

    }


    //死信队列
    @Bean("deadQueue")
    public Queue deadQueue(){
        return QueueBuilder.durable(DEAD_EXCHANGE).build();
    }

    @Bean
    public Binding normalQueue1BindingNExchange(@Qualifier("normalQueue1") Queue queue1, @Qualifier("normalExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue1).to(exchange).with("XA");
    }

    @Bean
    public Binding normalQueue2BindingNExchange(@Qualifier("normalQueue2") Queue queue2, @Qualifier("normalExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue2).to(exchange).with("XB");
    }

    @Bean
    public Binding deadQueueBindingDExchange(@Qualifier("deadQueue") Queue dQueue, @Qualifier("deadExchange") DirectExchange exchange){
        return BindingBuilder.bind(dQueue).to(exchange).with("YD");
    }



}

```


生产者我们使用了RestFul风格来接受我们要生产的信息:
```java
@Controller
@RequestMapping("/ttl")
@Slf4j
public class SendMsgController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable String message){
      log.info("当前时间: {},发送消息给来ttl:{}",new Date().toString(),message);  
      rabbitTemplate.convertAndSend(TtkQueueConfig.NORMAL_EXCHANGE, "XA", "消息来自TTL为10s:" + message);
      rabbitTemplate.convertAndSend(TtkQueueConfig.NORMAL_EXCHANGE, "XB", "消息来自TTL为40s:" + message);
    }

    @GetMapping("/test")
    @ResponseBody
    public String ooo(){
        return "ggggg";
    }
}
```
消费者
```java
@Component
@Slf4j
public class DeadLetterQueueConsumer {
    //接受消息
    @RabbitListener(queues = TtkQueueConfig.DEAD_QUEUE)
    public void receiveD(Message message, Channel channel) throws Exception{
        String msg = new String(message.getBody());
        log.info("当前时间: {}, 受到死信队列消息: {}",new Date().toString() ,msg);
    }
}

```

#### 延迟队列优化方案:

按照以上要求每增加一个, 要新开一个队列,麻烦.

我们可以新增一个队列`QC` , 不设置TTL时间.
![](http://oss.re1ife.top/media/20230113202634.png)

TTL的时间由生产者决定:

根据示意图我们要先创建一个新的队列然后绑定, 因此需要在配置类中修改:
```java
	@Bean("producerTtlQueue")
    public Queue producerTtlQueue(){
        Map<String, Object> arg = new HashMap<>();
        arg.put("x-dead-letter-exchange", TtkQueueConfig.DEAD_QUEUE);
        arg.put("x-dead-letter-routing-key","YD");
        return QueueBuilder.durable(PRODUCER_TTL_QUEUE).withArguments(arg).build();
    }
	
	@Bean
    public Binding bindingProducerTtlQueue(@Qualifier("producerTtlQueue") Queue queue,@Qualifier("normalExchange") DirectExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with("XC");
    }


```

增加发送信息的接口
```java
// 消息和 ttl
    @GetMapping("/sendExpirationMsg/{message}/{ttl}")
    public void sendMsg(@PathVariable String message,@PathVariable String ttl){
      log.info("当前时间{}, 发送一条{}ms的ttl信息给队列QC: {}", new Date().toString(),ttl,message);
      rabbitTemplate.convertAndSend(TtkQueueConfig.NORMAL_EXCHANGE,"XC",message,msg -> {
        //发送消息的 延迟时间设置
        msg.getMessageProperties().setExpiration(ttl);
        return msg;
      });

      
    }
```

![](http://oss.re1ife.top/media/20230113215026.png)

**问题**: 按照预想情况读取消息的顺序应该是 `你好2` -> `你好1`.
		但是由于队列是一种先进先出的数据结构,因此`你好1` 堵塞了`你好2`



#### RabbitMQ插件实现延迟队列

插件下载官网: https://www.rabbitmq.com/community-plugins.html 
插件: **rabbitmq_delayed_message_exchange**
方法: 解压放到mq的插件目录 重启mq 执行`rabbitmq-plugins enable rabbitmq_delayed_message_exchange`


基于死信实现的延迟队列:![](http://oss.re1ife.top/media/20230113221600.png)

基于插件实现的延迟队列:![](http://oss.re1ife.top/media/20230113221853.png)

死信队列延迟位置在: 队列
插件延迟位置在: 交换机


- 基于插件实现的延迟队列实现

代码架构图: ![](http://oss.re1ife.top/media/20230113221944.png)


配置类实现, 主要在与交换机的实现不同了

```JAVA

@Configuration
public class DelayQueueConfig {
    public static final String DELAY_QUEUE = "delay.queue";
    public static final String DELAY_EXCHANGE = "delay.exchange";
    public static final String DELAY_ROUTING_KEY = "delay.routingKey";

    //基于插件的交换机 不一样!!!
    @Bean("delayExchange")
    public CustomExchange delayExchange(){
        /*
         * 1. 交换机名称
         * 2. 交换机类型
         * 3. 持久化
         * 4. 自动删除
         * 5. 其他参数
         */


        Map<String ,Object> arg = new HashMap<>();
        arg.put("x-delayed-type", "direct");
        
         return new CustomExchange(DELAY_EXCHANGE, "x-delayed-message", true, false, arg);
    }

    @Bean("delayQueue")
    public org.springframework.amqp.core.Queue queue(){
        return QueueBuilder.durable(DELAY_QUEUE).withArguments(null).build();
    }

    @Bean
    public Binding delayedQueueBinding(@Qualifier("delayExchange") CustomExchange exchange,
                                        @Qualifier("delayQueue") org.springframework.amqp.core.Queue queue){
                                  
        return BindingBuilder.bind(queue).to(exchange).with(DELAY_ROUTING_KEY).noargs();

    }

}

``` 

生产者变化不大:
```java
//插件延迟队列 消息和ttl
    @GetMapping("/sendDelayMsg/{message}/{delayTime}")
    public void sendMsg(@PathVariable String message,@PathVariable Integer delayTime){

      rabbitTemplate.convertAndSend(DelayQueueConfig.DELAY_EXCHANGE,DelayQueueConfig.DELAY_ROUTING_KEY, message,msg -> {
        //发送消息的 延迟时间设置
        msg.getMessageProperties().setDelay(delayTime);
        return msg;
      });
      log.info("当前时间{}, 发送一条{}ms的ttl信息给队列QC: {}", new Date().toString(),delayTime,message);

    }
```

消费者:
```java
//基于插件的延迟消息
@Component
@Slf4j
public class DelayQueueConsumer {
    @RabbitListener(queues = DelayQueueConfig.DELAY_QUEUE)
    public void receiveMsg(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间: {}, 基于插件的受到延时队列消息: {} 延迟时间: {}",new Date().toString(),msg,message.getMessageProperties().getDelay());
    }
}
```

![](http://oss.re1ife.top/media/20230113224324.png)
实验结果满足我们的预期了.

**延时队列建议一般用插件.**

### 其他知识
#### 幂等性

**概念**: 用户对于同一操作发起的一次请求或者多次请求的结果是一致的,不会因为多次点击而产生了副作用。
**举例**: 用户购买商品后支付,支付扣款成功,但是返回结果的时候网络异常,此时钱已经扣了,用户再次点击按钮,此时会进行第二次扣款,返回结果成功.

**消息重复消费**: 消费者在消费 MQ 中的消息时,MQ 已把消息发送给消费者,消费者在给 MQ 返回 ack 时网络中断,故 MQ 未收到确认信息,该条消息会重新发给其他的消费者,或者在网络重连后再次发送给该消费者,但实际上该消费者已成功消费了该条消息,造成消费者消费了重复的消息。

**解决思路**: 使用全局ID或者写一个唯一标识UUID或时间戳....

**消费段幂等性保障**: 在海量订单生成的业务高峰期,生产端有可能就会重复发生了消息,这时候消费端就要实现幂等性,这就意味着我们的消息永远不会被消费多次,即使我们收到了一样的消息。业界主流的幂等性有两种操作:
1. 唯一 ID+指纹码机制,利用数据库主键去重
	1. 指纹码: 我们的一些规则或者时间戳 + 别的服务给到的唯一信息码. ,基本都是由我们的业务规则拼接而来,但是一定要保证唯一性,然后就利用查询语句进行判断这个 id 是否存在数据库中,优势就是实现简单就一个拼接,然后查询判断是否重复;劣势就是在高并发时,如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能,但也不是我们最推荐的方式。
2. 利用 redis 的原子性去实现
	1. redis执行setnx,天人幂等性


#### 优先级队列

**场景:** 
> 在我们系统中有一个订单催付的场景,我们的客户在天猫下的订单,淘宝会及时将订单推送给我们,如果在用户设定的时间内未付款那么就会给用户推送一条短信提醒,很简单的一个功能对吧,但是,tmall商家对我们来说,肯定是要分大客户和小客户的对吧,比如像苹果,小米这样大商家一年起码能给我们创造很大的利润,所以理应当然,他们的订单必须得到优先处理,而曾经我们的后端系统是使用 redis 来存放的定时轮询,大家都知道 redis 只能用 List 做一个简简单单的消息队列,并不能实现一个优先级的场景,所以订单量大了后采用 RabbitMQ 进行改造和优化,如果发现是大客户的订单给一个相对比较高的优先级, 否则就是默认优先级。

示意图: 序号/优先级
![](http://oss.re1ife.top/media/20230114224817.png)

- 实现
- ![](http://oss.re1ife.top/media/20230114225002.png)
队列代码添加:
```java
Map<String, Object> params = new HashMap();
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);
```

消费者代码添加优先级:
```java
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
```


#### 惰性队列

**惰性队列:** 消息保存在磁盘中.

**应用场景**: 当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时,惰性队列就很有必要了。

>持久化的消息,在被写入磁盘的同时也会在内存中驻留一份备份。


可以使用`channel.queueDeclare`参数设置, `Policy`设置(优先级队列),.
```java
Map<String, Object> args = new HashMap<>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false,false,false,args);
```
- 两种模式:
	- defalut 默认
	- lazy


### 集群搭建

![](http://oss.re1ife.top/media/20230114230423.png)
节点之间互通.

1. 修改 3 台机器的主机名称
`vim /etc/hostname`
2.配置各个节点的 hosts 文件,让各个节点都能互相识别对方
```
vim /etc/hosts
10.211.55.74 node1
10.211.55.75 node2
10.211.55.76 node3
```
3.以确保各个节点的 cookie 文件使用的是同一个值
在 node1 上执行远程操作命令
```bash
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie
```
4.启动 RabbitMQ 服务,顺带启动 Erlang 虚拟机和 RbbitMQ 应用服务(在三台节点上分别执行以
下命令)
`rabbitmq-server -detached`
5.在节点 2 执行
```
rabbitmqctl stop_app
(rabbitmqctl stop 会将 Erlang 虚拟机关闭,rabbitmqctl stop_app 只关闭 RabbitMQ 服务)
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node1
rabbitmqctl start_app(只启动应用服务)
```
6.在节点 3 执行
```
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@node2
rabbitmqctl start_app
```

7.集群状态
`rabbitmqctl cluster_status`
8.需要重新设置用户
```
创建账号
rabbitmqctl add_user admin 123
设置用户角色
rabbitmqctl set_user_tags admin administrator
设置用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

9.解除集群节点(node2 和 node3 机器分别执行)
```
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl cluster_status
rabbitmqctl forget_cluster_node rabbit@node2(node1 机器上执行)
```

### 镜像队列


搭建好的集群, 在一个节点上创建一个队列, 其他节点不同步.

**镜像队列:**  可以将队列镜像到集群中的其他 Broker 节点之上,如果集群中
的一个节点失效了,队列能自动地切换到镜像中的另一个节点上以保证服务的可用性。

**搭建步骤: 
>1.启动三台集群节点
  2.随便找一个节点添加 policy
  ![](http://oss.re1ife.top/media/20230114231954.png)

  3.在 node1 上创建一个队列发送一条消息,队列存在镜像队列
  ![](http://oss.re1ife.top/media/20230114232008.png)

  4.停掉 node1 之后发现 node2 成为镜像队列
  ![](http://oss.re1ife.top/media/20230114232021.png)

  5.就算整个集群只剩下一台机器了 依然能消费队列里面的消息

### Haproxy+Keepalive 负载均衡

结构图:
![](http://oss.re1ife.top/media/20230114232416.png)

我刚看到时候就有个问题 能不能直接用nginx负载均衡, 区别是什么: http://www.ha97.com/5646.html

### FederationExchange

**应用场景**: 成都有一个Broker , 美国有一个Broker. 就近选择 减少延时.


- 搭建步骤
	1. 开启插件 `rabbitmq-plugins enable rabbitmq_federation` `rabbitmq-plugins enable rabbitmq_federation_manage`
	2. 原理图(在 node2 上创建fed_exchange)
		![](http://oss.re1ife.top/media/20230114233026.png)
	3. 在 downstream(node2)配置 upstream(node1)
	![](http://oss.re1ife.top/media/20230114233118.png)
	4. policy
	 ![](http://oss.re1ife.top/media/20230114233143.png)



