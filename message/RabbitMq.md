```
basicAck
```

# 简介

- rabbitmq 是一个跨平台的消息中间键

- 使用Erlang语言编写
- RabbitMq是基于AMQP协议的



## 什么是AMQP高级消息队列协议

- 一个二进制协议，面向消息中间件

- 基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制  

## AMQP核心概念

Server: 又称Brocker，接受客户端连接，实现AMQP实体服务

Connection：连接,应用程序与Broker的网络连接

Channel:网络信道，信息读写都是在Channel操作的，客户端可以建立多个Channel，类似于与数据库的会话

Message：消息，由Properties和Body组成

Virtual host ： 虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个vh里面有若干个Exchange和queue

Exchange:交换机接收消息，根据路由键转发消息到对应的queue

Binding:Exchange 和queue之间的虚拟连接，binding中可以包含routingkey

routing key:一个路由规则，虚拟机可以用它来确定如何路由一个特定消息

queue:消息队列，保存消息并将他们转发给消费者

![](../image/rabbitmq/Q0190724225531.png)

整体架构图

消息投递到对应exchange，消费者从queue消费，exchange和队列有个绑定关系，可以路由到指定的队列中

消息流转示意图

![](../image/rabbitmq/QQ20190724225830.png)

# 快速开始

## 环境搭建

1. 先下载三个安装包

安装erlang安装包（http://www.erlang.org/downloads ）

下载erlang的源码包进行安装

```shell
[root@localhost home]# tar -zxvf otp_src_23.3.tar.gz
# 编译
[root@localhost otp_src_23.3]# ./configure --prefix=/opt/erlang
# 如果报configure: error: No curses library functions found错
# 则还需要额外安装
[root@localhost otp_src_23.3]# yum install ncurses-devel
yum install gcc gcc-c++
[root@laoxiao otp_src_23.3]# yum install kernel-devel
## 重新编译后进行安装
[root@localhost otp_src_23.3]# ./configure --prefix=/opt/erlang
[root@localhost otp_src_23.3]# make && make install
```

修改/etc/profile配置文件配置如下内容

```shell
ERLANG_HOME=/opt/erlang
export PATH=$PATH:$ERLANG_HOME/bin
export ERLANG_HOME
```

验证是否安装成功

```shell
[root@localhost erlang]# erl 
1> io:format("hello") .
hellook
```



2. 安装RabbitMq

前往官网下载：https://www.rabbitmq.com/

```shell
## 解压文件夹
xz -d rabbitmq-server-generic-unix-3.8.15.tar.xz
tar -xvf rabbitmq-server-generic-unix-3.8.15.tar
## 建立软连接(rabbitmq_server-3.8.15后不能有/，否则会启动报错)
[root@localhost opt]# ln -s /home/rabbitmq_server-3.8.15 /opt/rabbitmq
```

配置环境变量

```shell
export PATH=$PATH:/opt/rabbitmq/sbin
export RABBITMQ_HOME=/opt/rabbitmq
```

后台启动

```shell
[root@laoxiao otp_src_24.0]# rabbitmq-server -detached
[root@laoxiao otp_src_24.0]# rabbitmqctl status
Status of node rabbit@laoxiao ...
```

## 创建一个root用户

- 添加新用户，用户名为“ root”，密码为“ root ” 

```shell
[root@laoxiao otp_src_24.0]# rabbitmqctl add_user root root
```

- 给root用户授所有权限 

```shell
[root@laoxiao otp_src_24.0]# rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
Setting permissions for user "root" in vhost "/" ...
```

- 设置管理员用户管理员权限

```shell
[root@laoxiao otp_src_24.0]# rabbitmqctl set_user_tags root administrator
Setting tags for user "root" to [administrator] ...
```

## DEMO

- 生产者

```java
public void provider() throws Exception {
    ConnectionFactory factory= new ConnectionFactory() ;
    factory.setHost(IP_ADDRESS) ;
    factory.setPort (PORT) ;
    factory.setUsername ("root" ) ;
    factory.setPassword ("root") ;
    //创建连接
    Connection connection= factory.newConnection();
    //创建信道
    Channel channel= connection.createChannel();
    //创建一个 type ＝ "direct", 持久化的、非自动删除的交换器
    channel.exchangeDeclare(EXCHANGE_NAME,  "direct", true , false , null );
    //创建一个持久化、非排他的、非自动删除的队列
    channel.queueDeclare(QUEUE_NAME , true , false , false , null);
    //将交换器与队列通过路由键绑定
    channel.queueBind (QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
    //发送一条持久化的消息 ： hello wor l d !
    String message = "Hello World!";
    channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
            MessageProperties.PERSISTENT_TEXT_PLAIN ,
            message.getBytes());
    //关闭资源
    channel.close ();
    connection.close() ;
}
```

- 消费者

```java
public void consumer() throws Exception {
    Address[] addresses= new Address[] {new Address(IP_ADDRESS , PORT)};
    ConnectionFactory factory = new ConnectionFactory() ;
    factory.setUsername ("root") ;
    factory.setPassword ("root");
    //这里的连接方式与生产者的 demo 略有不同，注意辨别区别
    //创建连接
    Connection connection= factory.newConnection(addresses);
    //创建信道
    final Channel channel= connection.createChannel();
    //设置客户端最多接收未被 ack 的消息的个数
    channel.basicQos(64);
    Consumer consumer = new DefaultConsumer(channel) {
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            System.out.println("receive message:"+new String(body));
            channel.basicAck(envelope.getDeliveryTag(), false);
        }
    };
    channel.basicConsume(QUEUE_NAME, consumer);
    //保持控制台状态
    System.in.read();
    channel.close();
    connection.close();
}
```

## 网页登陆

- 启动web管理页面

```shell
## 查看对应的插件
[root@laoxiao sbin]# rabbitmq-plugins list
## 启动
[root@laoxiao sbin]# rabbitmq-plugins enable rabbitmq_management
```



<http://192.168.94.131:15672/#/>   guest/guest （root/root）

# Rabbit结构

- 整体结构

![](../image/rabbitmq/QQ20190724225653.png)

## 生产者与消费者

- Producer ： 生产者，就是投递消息的一方。 
- Consumer：消费者，就是接收消息的一方。 
- Broker：消息中间件的服务节点 。 
  - 对于 RabbitMQ 来说， 一个 RabbitMQ Broker 可以简单地看作一个 RabbitMQ 服务节点，
    或者 RabbitMQ 服务实例。大多数情况下也可以将一个 RabbitMQ Broker 看作一 台 RabbitMQ
    服务器 。 

![](../image/message/20210531155342.png)

## 队列 

- 队列是rabbitmq的内部对象，用于存储消息
- RabbitMQ 中消息都只能存储在队列中 
- 多个消费者可以订阅同一个队列，这时队列中的消息会被平均分摊（ Round-Robin ，即轮询） 
- Rabbi tMQ 不支持**队列层面**的广播消费 

## 交换器、路由键、绑定 

### Exchange

生产者将消息发送到 Exchange （交换器，通常也可以用大写的“X”来表示），由交换器将消息路由到一个或者多个队列中。如果路由不到，或许会返回给生产者，或许直接丢弃 

- 生产者将消息发送给交换器时， 需要一个 RoutingKey ， 当 BindingKey 和 RoutingKey 相匹
  配时，消息会被路由到对应的队列 中 

![](../image/message/20210531161612.jpg)

### RoutingKey 

路由键。生产者将消息发给交换器的时候， 一般会指定一个 RoutingKey ，用来指定这个消息的路由规则，而这个 Routing Key 需要与交换器类型和绑定键（ BindingKey）联合使用才能最终生效。 

```java
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
                MessageProperties.PERSISTENT_TEXT_PLAIN ,
                message.getBytes());
```

### Binding 

RabbitMQ 中通过绑定将交换器与队列关联起来，在绑定的时候一般会指定一个绑定键（ BindingKey ） ，这样 RabbitMQ 就知道如何正确地将消息路由到队列了 

```java
//将交换器与队列通过路由键绑定
channel.queueBind (QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY) ;
```

### 交换器类型

RabbitMQ 常用的交换器类型有 fanout 、 direct 、 topic 、 headers 这四种 

```java
//创建一个 type ＝ "direct", 持久化的、非自动删除的交换器
channel.exchangeDeclare(EXCHANGE_NAME,  "direct", true , false , null );
```

> fanout 

- 它会把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。 

> direct 

- 它会把消息路由到那些 BindingKey 和 RoutingKey**完全匹配**的队列中 

> topic 

topic 类型的交换器在匹配规则上进行了扩展，它与 direct 类型的交换器相似，也是将消息路由到 BindingKey 和 RoutingKey 相匹配的队列中，但这里的匹配规则有些不同，它约定 ：

- RoutingKey 为一个点号“ ． ”分隔的字符串（被点号“．”分隔开的每一段独立的字符
  串称为一个单词）
- 两种特殊字符串“＊”和“＃”，用于做模糊匹配 ,*于匹配一个单词，#用于匹配多规格单词（可以是零个）。 

## RabbitMQ 运转流程 

### 生产者

1. 生产者连接到 RabbitMQ Broker ， 建立一个连接（ Connection ），开启一个信道（ Channel ) 
2. 生产者声明一个交换器 ，并设置相关属性，比如交换机类型、是否持久化等 
3. 生产者声明 一个队列井设置相关属性，比如是否排他、是否持久化、是否自动删除等 
4. 生产者通过路由键将交换器和队列绑定起来 
5. 生产者发送消息至 RabbitMQ Broker，其中包含路由键、交换器等信息 
6. 相应的交换器根据接收到的路由键查找相匹配的队列 。 
7. 如果找到，则将从生产者发送过来的消息存入相应的队列中。 
8. 如果没有找到 ，则根据生产者配置的属性选择丢弃还是回退给生产者 
9. 关闭信道 
10. 关闭连接

### 消费者

1. 消费者连接到 RabbitMQ Broker，建立一个连接（ Connection ），开启一个信道（ Channel) 
2. 消费者向 RabbitMQ Broker 请求消费相应队列中的消息，可能会设置相应的回调函数，以及做一些准备工作 
3. 等待 RabbitMQ Broker 回应并投递相应队列中的消息，消费者接收消息。 
4. 消费者确认（ ack ）接收到的消息 
5. RabbitMQ 从队列中删除相应己经被确认的消息 
6. 关闭信道。 
7. 关闭连接。 

# RabbitMq 代码开发

## 连接 RabbitMQ 

- 地址方式

```java
ConnectionFactory factory= new ConnectionFactory() ;
factory.setHost(IP_ADDRESS) ;
factory.setPort (PORT) ;
factory.setUsername ("root" ) ;
factory.setPassword ("root") ;
//创建连接
Connection connection= factory.newConnection();
//创建信道
Channel channel= connection.createChannel();
```

- URI模式

```java
factory.setUri("amqp://root:root@"+IP_ADDRESS+":"+PORT);
Connection connection= factory.newConnection();
//创建信道
Channel channel= connection.createChannel();
```

- Channel 实例是非线程安全的 

## 声明exchange

- exchange：交换机名称
- type：交换器的类型，常见的如 fanout、 direct 、 topic 
- durable：设置是否持久化 
- autoDelete ：设置是否自动删除。 自动删除的前提是至少有一个队列或者交换器与这个交换器绑定 ， 之后所有与这个交换器绑定的队列或者交换器都与此解绑 
- internal：设置是否是内置的 ，如果设置为 true，则表示是内置的交换器，客户端程
  序无法直接发送消息到这个交换器中，只能通过交换器路由到交换器这种方式。 

```java
Exchange.DeclareOk exchangeDeclare(String exchange,
                                              String type,
                                              boolean durable,
                                              boolean autoDelete,
                                              boolean internal,
                                              Map<String, Object> arguments) throws IOException;
```

## 声明队列

- exclusive:设置是否排他 

```java
Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                             Map<String, Object> arguments) throws IOException;
```

## 消费消息 

RabbitMq的消费模式分两种 ： 推（ Push ）模式和拉（ Pull ）模式 。 推模式采用 channel.basicConsume
进行消费，而拉模式则是调用 Basic.Get 进行消费。 

## push模式

在推模式中，可以通过持续订阅的方式来消费消息 

```java
String basicConsume(String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback) throws IOException;
```

## Get模式

通过 channel.basicGet 方法可以单条地获取消息 

如果设置 autoAck 为 false ， 那么同样需要调用channel . basicAck 来确认消息己被成功接收。 

```java
GetResponse response = channel.basicGet(QUEUE_NAME, false);
System.out.println(new String(response.getBody()));
channel.basicAck(response.getEnvelope().getDeliveryTag(), false);
```

```tex
Basic . Consume 将信道（ Channel ）直为接收模式，直到取消队列的订阅为止。在接收模式期间， RabbitMQ 会不断地推送消息给消费者，当然推送消息的个数还是会受到 Basic.Qos的限制．如果只想从队列获得单条消息而不是持续订阅，建议还是使用 Basic.Get 进行消费．但是不能将 Basic.Get 放在一个循环里来代替 Basic.Consume ，这样做会严重影响 RabbitMQ的性能．如果要实现高吞吐量，消费者理应使用 Basic.Consume 方法。
```



# Rabbitmq高级特性

- 消息如何保障100%的投递成功
- 在海量订单产生的业务高峰期，如何避免消息的重复消费问题

## 消息的100%投递成功

什么是生产端的可靠性投递

- 保障消息的成功发送
- 保障mq节点的成功接收
- 发送端收到mq的确认应答
- 完善的消息补偿机制（消息投递失败我该怎么处理）

常用方案：

- 消息落库，对消息状态进行打标
  - 消息投递时，将消息持久化到库中，消息到达消费端，进行持久库的状态修改，对没有响应的消息，轮询发送，做最大努力的次数，如5次
  - 1、持久化消息记录 status=0 2 发送消息，3消费端回送响应给生产端，4生产端修改持久库状态 status=1,5如果超过时间status=0,则定时任务再抽取消息再次投递,如果次数超过一定次数，status=2,
- 消息的延迟投递，做二次确认

## 幂等性

在海量订单产生的业务高峰，如果避免重复消费

消费端实现幂等性，就意味着，我们的消息永远不会被消费多次

实现方式

- 唯一id+指纹码机制，利用数据库主键去重
  - 唯一id_指纹码（唯一规则）
  - select count(1) from order where id=唯一id+指纹码
  - 如果=0则进行insert，否则不进行处理
- 利用redis原子性实现
  - 如果数据入库，那么数据库与redis如何做到原子性

## 消费端的确认与拒绝 

### Ack 机制

- 消费者在订阅队列时，可以指定 autoAck 参数，当 autoAck 等于 false时，RabbitMQ 会等待消费者显式地回复确认信号后才从内存 （或者磁盘）中移去消息 
- 当 autoAck 等于 true 时， RabbitMQ 会自动把发送出去的消息置为确认，然后从内存（或者磁盘）中删除 
- 如果 RabbitMQ 一直没有收到消费者的确认信号，并且消费此消息的消费者己经断开连接，则 RabbitMQ 会安排该消息重新进入队列，等待投递给下一个消费者 

## 消息未投递到对应队列

```java
void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body)
            throws IOException;
```

- 当 mandatory 参数设为 true 时，交换器无法根据自身的类型和路由键找到一个符合条件的队列，那么 RabbitMQ 会调用 Basic.Return 命令将消息返回给生产者 ，为true则直接丢弃
- channel . addReturnListener 可以监听是否正确路由到合适队列 

```java
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY+1, true,
        MessageProperties.PERSISTENT_TEXT_PLAIN ,
        message.getBytes());

channel.addReturnListener(new ReturnListener() {
    @Override
    public void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消息未投递：" + new String(body));
    }
});
```

## 备份交换机



## 生产者

### 事务

channel.txSelect()、
channel.txCommit() 和 channel.txRollback() 。 channel.txSelect()用于将当前的信道设置成事务模式， channel.txCommit() 用于提交事务 ，  channel.txRollback() 用于事务回滚 

```java
try {
    channel.txSelect();
    channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            "commit.....".getBytes());
    //int i = 1 / 0;
    channel.txCommit();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        channel.txRollback();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 确认机制

消息的确认，是指消息投递后(到达队列)，**broker收到消息，则会 给生产者一个应答**。生产者进行应答，确认这条消息是否正常发送broker

- 每发送一批消息后，调用 channel.waitForConfirms() 方法，等待服务器的确认返回 。 

```java
try {
    channel.confirmSelect();
    channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            "confirm.....".getBytes());
    if(!channel.waitForConfirms()) {
        //可以操作消息未投递成功的相关业务
        System.out.println("消息未投递成功.....");
    }
} catch (Exception e) {
    e.printStackTrace();
}
```

- 异步监听方式
  - handleAck 和 handleNack ，分别用来处理RabbitMQ 回传的 Basic.Ack 和 Basic .Nack 
  - 此处的basicPublish发送消息可以循环批量发送，监听中可以收到批量的ack消息
  - handleNack：内部发生异常才会调用
  - handleAck：每一条都会调用（除非调用了Nack）

```java
channel.confirmSelect();
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
        MessageProperties.PERSISTENT_TEXT_PLAIN,
        "confirm.....".getBytes());
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {    
        //deliveryTag:消息的唯一的标签
        //multiple:是否批量
    } 
    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
    }
});
```

**事务和同步确认的方式在QPS上都不如异步确认，所以生产上建议使用异步确认的方式,然后在对应的ack中操作相应的业务**

## Return 消息机制

用于处理一些不可路由的消息

要求:

先添加监听

```java
channel.addReturnListener(new ReturnListener() {
    //replyCode:响应码
    public void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) throws IOException {

    }
});
```

```java
//mandatory设置为true，设置为true，就不会把消息记录删除
void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body)
        throws IOException;
```



## 消费端数据限流

单个的客户端无法同时处理大量的消息

RabbitMQ提供了qos（服务质量保证）功能，在非自动确认消息（no_ack设置为false）的前提下，如果一定数目的消息未被确认前，不进行消费新的消息

**Basic.Qos 的使用对于拉模式的消费方式无效． **

### 应用场景

如果用N个消费者，有的消费者快，有的消费者慢，那么消费者慢的积累了大量的任务

### 代码示例

- 相关api：

```java
/**
*  prefetchCount qos的上限

   global:
       fales: channel上新的消费者需要遵从 prefetchCount 的限定值
       true:  channel上所有的消费者都需要遵从 prefetchCount 的限定值

*/
void basicQos(int prefetchCount, boolean global)
```

- 具体代码

```java
try {
    channel.basicQos(3, false);
} catch (IOException e) {
    e.printStackTrace();
}
Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("receive message:"+new String(body));
        channel.basicAck(envelope.getDeliveryTag(), false);
        try {
            Thread.sleep(10000l);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
};
```



## 消费端的ACK机制与重回队列

- ack : 成功处理消息
- nack:处理消息失败，让生产者重新发

> 在Consumer的handleDelivery方法中

```java

if((Integer)properties.getHeaders().get("num") == 0) {
     //是否为批量的，是否重回队列
    channel.basicNack(envelope.getDeliveryTag(), false, true);
} else {
    channel.basicAck(envelope.getDeliveryTag(), false);
}
```

## 过期时间（ TTL) 

队列/消息 的生存时间，超时则进行清除

### 过期消息

- 有两种方式设置

1. 通过队列属性设置，队列中所有消息都有相同的过期时间。

```java
Map<String, Object> map = Collections.singletonMap("x-message-ttl", 6000);
channel.exchangeDeclare(EXCHANGE_NAME,  "direct", true , false , null);
//创建一个持久化、非排他的、非自动删除的队列
channel.queueDeclare(QUEUE_NAME + "ttl", true , false , false , map);
```

2. 对消息本身进行单独设置，每条消息的 TTL 可以不同 
   1. 这种方式，判断消息过时是消息投递时判断的

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
//设置持久化
builder.deliveryMode(2);
//设置过期时间
builder.expiration("6000");
AMQP.BasicProperties build = builder.build();
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, true, build,
        message.getBytes());
```

- 特性
  - 如果不设置 TTL，则表示此消息不会过期 ;
  - 如果将 TTL 设置为 0， 则表示除非此时可以直接将消息投递到消费者，否则该消息会被立即丢弃 

### 队列TTL

- 队列超过限定的时间则删除队列
- 删除队列会连带消息一起删除

```java
Map<String, Object> map = Collections.singletonMap("x expires", 6000);
//将交换器与队列通过路由键绑定
channel.queueBind (QUEUE_NAME+ "ttl1", EXCHANGE_NAME, ROUTING_KEY+"ttl1") ;
```

### 死信队列（DLX）

当消息在队列中没有消费者消费时，它能够重现publish到另一个Exchange中

死信队列情况

- 消息被拒绝（basic.reject/basic.nack）并且requeue=false
- 消息TTL过期
- 队列达到最大长度 

```java
// 这就是一个普通的交换机 和 队列 以及路由
		String exchangeName = "test_dlx_exchange";
		String routingKey = "dlx.#";
		String queueName = "test_dlx_queue";
		
		channel.exchangeDeclare(exchangeName, "topic", true, false, null);
		
		Map<String, Object> agruments = new HashMap<String, Object>();
		agruments.put("x-dead-letter-exchange", "dlx.exchange");
		//这个agruments属性，要设置到声明队列上
		channel.queueDeclare(queueName, true, false, false, agruments);
		channel.queueBind(queueName, exchangeName, routingKey);
		
		//要进行死信队列的声明:
		channel.exchangeDeclare("dlx.exchange", "topic", true, false, null);
		channel.queueDeclare("dlx.queue", true, false, false, null);
		channel.queueBind("dlx.queue", "dlx.exchange", "#");
		
		channel.basicConsume(queueName, true, new MyConsumer(channel));
```

- 可以为队列设置死信队列的路由键：args.put (”x-dead-letter-routing-key”,”dlx-routing-key”); 

### web界面解析

由 Web 管理页面可以看出，两个队列都被标记了“D”这个是 durable 的缩写，即设置了队列持久化。queue.normal 这个队列还配置了 TTL、 DLX 和 DLK，其中 DLX 指的是x-dead-letter-routing-key这个属性。 

## 延迟队列 

- 应用场景
  - 在订单系统中， 一个用户下单之后通常有30分钟的时间进行支付，如果30分钟之内
    没有支付成功，那么这个订单将进行异常处理 
- RabbitMQ 本身没有直接支持延迟队列的功能，但是可以通过 DLX 和 TTL 模拟出延迟队列的功能。 
  - 如:设置队列的过期时间，然后进入对应的死信队列

## 持久化

- 持久化分为三部分
  - 交换机持久化，队列持久化，消息持久化
- 交换机未持久化，重启后交换机消失，不会影响消息的存在
- 队列未持久化，队列消失，则对应的消息也会消失
- 想要消息持久化，消息和队列都要设置持久

# RabbitMQ 管理 

## 多vhost 

每一个 vhost 本质上是一个独立的小型 RabbitMQ 服务器，拥有自己独立的队列、交换器及绑定关系等，井且它拥有自己独立的权限。 

- 客户端在连接的时候必须制定一个 vhost 。 RabbitMQ 默认创建的 vhost 为“/” 

### vhost 相关命令

- 基础命令

```shell
### 查询vhost列表
[root@laoxiao sbin]# rabbitmqctl list_vhosts
Listing vhosts ...
name
/
### 添加vhost
[root@laoxiao sbin]# rabbitmqctl add_vhost vhost1
Adding vhost "vhost1" ...

## 删除vhost
[root@laoxiao sbin]# rabbitmqctl delete_vhost vhost1
Deleting vhost "vhost1" ...

```

- 权限命令

```shell
rabbitmqctl set permissions [-p vhost] {user} {conf} {write} {read｝
```

conf ： 一个用于匹配用户在哪些资源上拥有可配置权限的正则表达式。
write ：一个用于匹配用户在哪些资源上拥有可写权限的正则表达式 。
read ： 一个用于匹配用户在哪些资源上拥有可读权限的正则表达式。 

```shell
### 授权
[root@laoxiao sbin]# rabbitmqctl set_permissions --vhost vhost1 root ".*" ".*" ".*"
Setting permissions for user "root" in vhost "vhost1" ...

### 清除权限
[root@laoxiao sbin]# rabbitmqctl clear_permissions --vhost vhost1 root
Clearing permissions for user "root" in vhost "vhost1" ...

## 查看某个vhost的授权列表
[root@laoxiao sbin]# rabbitmqctl list_permissions --vhost vhost1
Listing permissions for vhost "vhost1" ...
user	configure	write	read
root	.*	.*	.*

```

## 用户管理 

- 添加用户

rabbitmqctl add user {username} {password ｝ 

- 修改用户密码

rabbitmqctl change_password {username} {newpassword ｝ 

## WEB管理

- 创建用户

![](../image/message/20210604172255.png)

# 整合spring

创建配置项

```java
@Configuration
@ComponentScan(value = "com.xiao")
public class BaseConfig {
    @Bean
    public ConnectionFactory connectionFactory(){
        /*com.rabbitmq.client.ConnectionFactory connectionFactory = new com.rabbitmq.client.ConnectionFactory();
        connectionFactory.setHost("192.168.94.131");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setConnectionTimeout(60000);
        //设置是否自动重连
        connectionFactory.setAutomaticRecoveryEnabled(true);
        connectionFactory.setNetworkRecoveryInterval(3000);

        return new CachingConnectionFactory(connectionFactory);*/
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses("192.168.94.131:5672");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setVirtualHost("/");
        return connectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        //如果设置为false，则spring不会加载
        rabbitAdmin.setAutoStartup(true);
        return rabbitAdmin;
    }
}
```

测试交换机创建与队列创建

```java
@Test
public void test1(){
    //创建Exchange
    rabbitAdmin.declareExchange(new DirectExchange("test.Direct",false,false));
    rabbitAdmin.declareExchange(new TopicExchange("test.Topic", false, false));
    rabbitAdmin.declareExchange(new FanoutExchange("test.Fanout", false, false));
    //创建队列
    rabbitAdmin.declareQueue(new Queue("test.direct.queue",false));
    //绑定Exchange和队列(先创建队列和Exchange)
    rabbitAdmin.declareBinding(new Binding("test.direct.queue",//绑定的队列
            Binding.DestinationType.QUEUE,
            "test.Direct", //绑定的Exchange
            "direct", //routingkey
            new HashMap<>()));
    //topic方式
    rabbitAdmin.declareBinding(BindingBuilder
            .bind(new Queue("test.direct.queue",false))//队列
            .to(new TopicExchange("test.Topic",false,false))//交换机
            .with("bind.#"));//routingkey
    //直连方式
    rabbitAdmin.declareBinding(
            BindingBuilder
                    .bind(new Queue("test.fanout.queue", false))
                    .to(new FanoutExchange("test.fanout", false, false)));
    //清空队列数据
    rabbitAdmin.purgeQueue("test.direct.queue", false);
}
```

## RabbitAdmin源码

它实现了InitializingBean接口，InitializingBean的afterPropertiesSet是在bean加载之后执行，在这个方法中的initialize()方法声明了三个空的集合,然后从spring容器主公获取对应的bean，放入集合中

```java
Collection<Exchange> contextExchanges = new LinkedList<Exchange>(
      this.applicationContext.getBeansOfType(Exchange.class).values());
Collection<Queue> contextQueues = new LinkedList<Queue>(
      this.applicationContext.getBeansOfType(Queue.class).values());
Collection<Binding> contextBindings = new LinkedList<Binding>(
      this.applicationContext.getBeansOfType(Binding.class).values());
```

最终由rabbitTemplate.execute来执行

## SpringAMQP声明

```java
/**
 * 初始化启动时，创建交换机和队列，并进行绑定
 * @return
 */
@Bean
public TopicExchange exchange001() {
    return new TopicExchange("topic001", true, false);
}

@Bean
public Queue queue001() {
    return new Queue("queue001", true); //队列持久
}

@Bean
public Binding binding001() {
    return BindingBuilder.bind(queue001()).to(exchange001()).with("spring.*");
}

@Bean
public TopicExchange exchange002() {
    return new TopicExchange("topic002", true, false);
}

@Bean
public Queue queue002() {
    return new Queue("queue002", true); //队列持久
}

@Bean
public Binding binding002() {
    return BindingBuilder.bind(queue002()).to(exchange002()).with("rabbit.*");
}

@Bean
public Queue queue003() {
    return new Queue("queue003", true); //队列持久
}

@Bean
public Binding binding003() {
    return BindingBuilder.bind(queue003()).to(exchange001()).with("mq.*");
}
```

## 消息消费者监听

```java
MessageListenerAdapter@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory){
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    //绑定监听队列
    container.addQueueNames("queue001", "queue002", "queue003");
    //设置当前消费者数量
    container.setConcurrentConsumers(1);
    //最大的消费者数量
    container.setMaxConcurrentConsumers(3);
    //是否重回队列
    container.setDefaultRequeueRejected(false);
    //自动签收
    container.setAcknowledgeMode(AcknowledgeMode.AUTO); //签收模式
    //消费端的标签策略
    container.setConsumerTagStrategy((queue)-> queue + "_" + UUID.randomUUID().toString());
    //消息监听
    ChannelAwareMessageListener messageListener=(message, channel)->{
        String msg = new String(message.getBody());
        System.out.println("----------消费者: " + msg);
    };
    container.setMessageListener(messageListener);
    return  container;
}
```

## MessageListenerAdapter消息监听

1、第一种方式,MessageDelegate类默认的调用handleMessage方法

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer1(ConnectionFactory connectionFactory){
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    //绑定监听队列
    container.addQueueNames("queue001", "queue002", "queue003");
    //设置当前消费者数量
    container.setConcurrentConsumers(1);
    //最大的消费者数量
    container.setMaxConcurrentConsumers(3);
    //是否重回队列
    container.setDefaultRequeueRejected(false);
    //自动签收
    container.setAcknowledgeMode(AcknowledgeMode.AUTO); //签收模式
    //消费端的标签策略
    container.setConsumerTagStrategy((queue)-> queue + "_" + UUID.randomUUID().toString());
    //消息监听
    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    container.setMessageListener(adapter);
    return  container;
}
```

```java
public class MessageDelegate {

   public void handleMessage(byte[] messageBody) {
      System.err.println("默认方法, 消息内容:" + new String(messageBody));
   }
   
```

2、自定义调用方法

```java
MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
adapter.setDefaultListenerMethod("consumeMessage");
container.setMessageListener(adapter);
```

```java
public void consumeMessage(byte[] messageBody) {
   System.err.println("字节数组方法, 消息内容:" + new String(messageBody));
}
```

3、自定义调用方法和传入参数

```java
MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
adapter.setDefaultListenerMethod("consumeMessage");
//转换器: 从字节数组转换为String
adapter.setMessageConverter(new TextMessageConverter());
container.setMessageListener(adapter);
```

```java
public void consumeMessage(String messageBody) {
   System.err.println("字符串方法, 消息内容:" + messageBody);
}
```

```java
public class TextMessageConverter implements MessageConverter {

	@Override
	public Message toMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
		return new Message(object.toString().getBytes(), messageProperties);
	}

	@Override
	public Object fromMessage(Message message) throws MessageConversionException {
		String contentType = message.getMessageProperties().getContentType();
		if(null != contentType && contentType.contains("text")) {
			return new String(message.getBody());
		}
		return message.getBody();
	}

}
```

4、将队列与方法一一对应

```java
MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
Map<String, String> queueOrTagToMethodName = new HashMap<>();
queueOrTagToMethodName.put("queue001", "method1");
queueOrTagToMethodName.put("queue002", "method2");
adapter.setQueueOrTagToMethodName(queueOrTagToMethodName);
adapter.setMessageConverter(new TextMessageConverter());
container.setMessageListener(adapter);
```

```java
public void method1(String messageBody) {
   System.err.println("method1 收到消息内容:" + new String(messageBody));
}

public void method2(String messageBody) {
   System.err.println("method2 收到消息内容:" + new String(messageBody));
}
```

## MessageConverter 消息转换器

一般来说，用于交互的消息是以byte[]方式传输的，如果我们想以 其他方式发送或者接收，那么就要用到消息装换器

# 整合spring boot

## 生产端整合

1 配置文件配置

```yaml
spring:
  rabbitmq:
    addresses: 192.168.94.131:5672
    username: guest
    password: guest
    virtual-host: /
    connection-timeout: 15000
    ##是否消息确认
    publisher-confirms: true
    publisher-returns: true
    template:
      mandatory: true
```

2 生产端整合

```java
//自动注入RabbitTemplate模板类
@Autowired
private RabbitTemplate rabbitTemplate;

//回调函数: confirm确认
final RabbitTemplate.ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
   @Override
   public void confirm(CorrelationData correlationData, boolean ack, String cause) {
      System.err.println("correlationData: " + correlationData);
      System.err.println("ack: " + ack);
      if(!ack){
         System.err.println("异常处理....");
      }
   }
};
//回调函数: return返回
final RabbitTemplate.ReturnCallback returnCallback = new RabbitTemplate.ReturnCallback() {
   @Override
   public void returnedMessage(org.springframework.amqp.core.Message message, int replyCode, String replyText,
                        String exchange, String routingKey) {
      System.err.println("return exchange: " + exchange + ", routingKey: "
            + routingKey + ", replyCode: " + replyCode + ", replyText: " + replyText);
   }
};


//发送消息方法调用: 构建Message消息
public void send(Object message, Map<String, Object> properties) throws Exception {
   MessageHeaders mhs = new MessageHeaders(properties);
   Message msg = MessageBuilder.createMessage(message, mhs);
   rabbitTemplate.setConfirmCallback(confirmCallback);
   rabbitTemplate.setReturnCallback(returnCallback);
   //id + 时间戳 全局唯一（在做confirm的时候，可以根据correlationData来找到唯一的消息）
   CorrelationData correlationData = new CorrelationData("1234567890");
   rabbitTemplate.convertAndSend("exchange-1", "springboot.abc", msg, correlationData);
}
```

## 消费端整合

配置文件

```yaml
spring:
  rabbitmq:
    addresses: 192.168.94.131:5672
    username: guest
    password: guest
    virtual-host: /
    connection-timeout: 15000

    listener:
      simple:
        ## 消费者消息确认 （Ack）:手动确认
        acknowledge-mode: manual
        #并发消费者的个数
        concurrency: 5
        #最大并发消费者的个数
        max-concurrency: 10
```

消费端配置

```java
@Component
public class RabbitReceiver {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "queue-1", //绑定队列
                    durable="true"),
            exchange = @Exchange(value = "exchange-1",
                    durable="true",
                    type= "topic",
                    ignoreDeclarationExceptions = "true"),
            key = "springboot.*" //rowkey
    )
    )
    @RabbitHandler
    public void onMessage(Message message, Channel channel) throws Exception {
        System.err.println("--------------------------------------");
        System.err.println("消费端Payload: " + message.getPayload());
        Long deliveryTag = (Long)message.getHeaders().get(AmqpHeaders.DELIVERY_TAG);
        //手工ACK
        channel.basicAck(deliveryTag, false);
    }
}
```

对象序列化的方式接收消息

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "${spring.rabbitmq.listener.order.queue.name}",
                durable="${spring.rabbitmq.listener.order.queue.durable}"),
        exchange = @Exchange(value = "${spring.rabbitmq.listener.order.exchange.name}",
                durable="${spring.rabbitmq.listener.order.exchange.durable}",
                type= "${spring.rabbitmq.listener.order.exchange.type}",
                ignoreDeclarationExceptions = "${spring.rabbitmq.listener.order.exchange.ignoreDeclarationExceptions}"),
        key = "${spring.rabbitmq.listener.order.key}"
)
)
@RabbitHandler
public void onOrderMessage(@Payload com.xiao.entity.Order order, //对象序列化接收消息
                           Channel channel,
                           @Headers Map<String, Object> headers //获取头信息
                            ) throws Exception {
    System.err.println("--------------------------------------");
    Long deliveryTag = (Long)headers.get(AmqpHeaders.DELIVERY_TAG);
    //手工ACK
    channel.basicAck(deliveryTag, false);
}
```

# RabbitMq 集群模式

## 镜像模式

把需要的队列做成镜像队列，存在于多个节点，属于RabbitMQ的HA方案,如图，三台服务器的节点数据都会实现实时的同步，保证数据100%不丢失

![](../image/RabbitMq/2019年9月16日232703.png)



## 多活模式

如图,federation是建立在node上的，也就是说各个node可以实现数据的同步

![](../image/rabbitmq/11464886-327320bb11d0123a.webp)

