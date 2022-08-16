# MQ概述

MQ全称 Message Queue（**消息队列**），是在消息的传输过程中**保存消息**的**容器**。多用于分布式系统之间进行通信



分布式系统之间的通信有两种：

​	1、直接RPC远程调用

---

![image-20220601205424336](http://fgcy-pic.zhamao.ml/image-20220601205424336.png)

---



2、借助第三方 完成间接通信



---

![image-20220601205557504](http://fgcy-pic.zhamao.ml/image-20220601205557504.png)

---



小结

- MQ，消息 队列，存储消息的中间件
- 分布式系统通信两种方式：直接远程调用 和 借助第三方 完成间接通信
-  发送方称为生产者，接收方称为消费者







# MQ 的优势和劣势



## 优势：

### 应用解耦

---

![image-20220601210109976](http://fgcy-pic.zhamao.ml/image-20220601210109976.png)

---

问题：

当库存系统出故障时，有可能导致订单系统也出故障，从而导致整个系统故障 【容错性就越低】

当增加新的系统或移除某个系统时，需要更改大量代码 【可维护性就越低】



注意：

系统的耦合性越高，容错性就越低，可维护性就越低



引入MQ

---

![image-20220601210857112](http://fgcy-pic.zhamao.ml/image-20220601210857112.png)

---

使用MQ好处：

容错性：订单系统把消息发送到MQ中就默认成功了；即使此时库存系统发生故障；【因为故障只是一时的，而消息存放在MQ中是安全的，数据最终是一致的】

可维护性：订单系统只管往MQ中发送消息，无论新增了多少个系统都与订单系统无关；因为只要新增的系统从MQ中取数据；

使用 MQ 使得应用间解耦，提升容错性和可维护性





### 异步提速

---

![image-20220601212017595](http://fgcy-pic.zhamao.ml/image-20220601212017595.png)

---

问题：

一个下单操作耗时：20 + 300 + 300 + 300 = 920ms【该下单过程是一个同步过程】
用户点击完下单按钮后，需要等待920ms才能得到下单响应，太慢！



引入MQ：

---

![image-20220601212201794](http://fgcy-pic.zhamao.ml/image-20220601212201794.png)

---

用户点击完下单按钮后，只需等待25ms就能得到下单响应 (20 + 5 = 25ms)。
提升用户体验和系统吞吐量（单位时间内处理请求的数目）。



### 削峰填谷

---

![image-20220601212538913](http://fgcy-pic.zhamao.ml/image-20220601212538913.png)

---

A系统每秒只能处理1000个请求，当每秒请求5000次，A系统就会挂掉





引入MQ

---

![image-20220601213049196](http://fgcy-pic.zhamao.ml/image-20220601213049196.png)

---

让MQ接收这每秒5000个请求，然后由A系统每秒从MQ中拉取1000个请求 【这样就可以做到削峰，填谷】



---

![image-20220601213340602](http://fgcy-pic.zhamao.ml/image-20220601213340602.png)

---



使用了 MQ 之后，限制消费消息的速度为1000，这样一来，高峰期产生的数据势必会被积压在 MQ 中，高峰
就被“削”掉了，但是因为消息积压，在高峰期过后的一段时间内，消费消息的速度还是会维持在1000，直
到消费完积压的消息，这就叫做“填谷”。



使用MQ后，可以提高系统稳定性





## MQ 的劣势



---

![image-20220601215030222](http://fgcy-pic.zhamao.ml/image-20220601215030222.png)

---





### 系统可用性降低

系统引入的外部依赖越多，系统稳定性越差。一旦 MQ 宕机，就会对业务造成影响。如何保证MQ的高可用？





### 系统复杂度提高

MQ 的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过 MQ 进行异步调用。

如何保证消息没有被重复消费？

怎么处理消息丢失情况？

那么保证消息传递的顺序性？



### 一致性问题

A 系统处理完业务，通过 MQ 给B、C、D三个系统发消息数据，如果 B 系统、C 系统处理成功，D 系统处理
失败。

如何保证消息数据处理的一致性？





# 使用 MQ 需要满足什么条件

- 生产者不需要从消费者处获得反馈。引入消息队列之前的直接调用，其接口的返回值应该为空，

​		这才让明明下层的动作还没做，上层却当成动作做完了继续往后走，即所谓异步成为了可能。

- 容许短暂的不一致性
- 确实是用了有效果。即解耦、提速、削峰这些方面的收益，超过加入MQ，管理MQ这些成本。







# 常见的 MQ 产品

目前业界有很多的 MQ 产品，例如 RabbitMQ、RocketMQ、ActiveMQ、Kafka、ZeroMQ、MetaMq等

也有直接使用 Redis 充当消息队列的案例，而这些消息队列产品，各有侧重

在实际选型时，需要结合自身需求及 MQ 产品特征，综合考虑



---

![image-20220601220103030](http://fgcy-pic.zhamao.ml/image-20220601220103030.png)

---



注意：由于 RabbitMQ 综合能力强劲，所以接下来的课程中，我们将主要学习 RabbitMQ。



 

# RabbitMQ 简介

AMQP，即 Advanced Message Queuing Protocol（高级消息队列协议），是一个网络协议，是应用层协议的一个开放标准

为面向消息的中间件设计。基于此协议的**客户端**与**消息中间件**可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

2006年，AMQP 规范发布。类比HTTP。



---

![image-20220601221126734](http://fgcy-pic.zhamao.ml/image-20220601221126734.png)

---





## RabbitMQ 基础架构如下图：



---

![image-20220601221251700](http://fgcy-pic.zhamao.ml/image-20220601221251700.png)

---

porducer、consumer是客户端；需要与RabbitMQ Server建立TCP连接进行通信【一个TCP连接中有多个channel管道，通过这些管道进行通信】

Broker【RababitMQ server】中有许多虚拟主机【virtual host】，有一种逻辑分区（隔离）的概念

virtual host 虚拟主机中有许多的exchange交换机和队列queue；交换机可以绑定到不同的队列中



- Broker：接收和分发消息的应用，RabbitMQ Server就是 Message Broker



-  Virtual host：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念

​		当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个vhost，每个用户在自己的 vhost 创建 exchange／queue 等



- Connection：publisher／consumer 和 broker 之间的 TCP 连接



- Channel：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection的开销将是巨大的，效率也较低。

​		Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的 channel 进行通讯，

​		AMQP method 包含了channel id 帮助客户端和message broker 识别 channel，所以 channel 之间是完全隔离的。

​		Channel 作为轻量级的 Connection 极大减少了操作系统建立 TCP connection 的开销



- Exchange：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到queue 中去

​		常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)



- Queue：消息最终被送到这里等待 consumer 取走



- Binding：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key。Binding 信息被保存到 exchange 中的查询表中

​		用于 message 的分发依据



注意：rabbitmq是基于AMQP



## RabbitMQ 提供了 6 种工作模式

简单模式、work queues、Publish/Subscribe 发布与订阅模式、Routing路由模式、Topics 主题模式

RPC 远程调用模式（远程调用，不太算 MQ；暂不作介绍）。


> 官网对应模式介绍：https://www.rabbitmq.com/getstarted.html



---

![image-20220601221737765](http://fgcy-pic.zhamao.ml/image-20220601221737765.png)

---



## JMS

-  JMS 即 Java 消息服务（JavaMessage Service）应用程序接口，是一个 Java 平台中关于面向消息中间件的API
- JavaEE中13项规范中的一项
- JMS 是 JavaEE 规范中的一种，类比JDBC
-  很多消息中间件都实现了JMS规范，例如：ActiveMQ。RabbitMQ 官方没有提供 JMS 的实现包，但是开源社区有







小结

1. RabbitMQ 是基于 AMQP 协议使用 Erlang 语言开发的一款消息队列产品。

2.  RabbitMQ提供了6种工作模式，我们学习5种。这是今天的重点。
3.  AMQP 是协议，类比HTTP。
4.  JMS 是 API 规范接口，类比 JDBC







# RabbitMQ的安装

> RabbitMQ 官方地址：http://www.rabbitmq.com/



## 1. 安装依赖环境

在线安装依赖环境：

```shell
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
```



## 2. 安装Erlang

上传

erlang-18.3-1.el7.centos.x86_64.rpm
socat-1.7.3.2-5.el7.lux.x86_64.rpm
rabbitmq-server-3.6.5-1.noarch.rpm

```sh
# 安装
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
```

如果出现如下错误

---

![1565526174751](http://fgcy-pic.zhamao.ml/1565526174751.png)

---



说明gblic 版本太低。我们可以查看当前机器的gblic 版本

```shell
strings /lib64/libc.so.6 | grep GLIBC
```

---

![1565526264426](http://fgcy-pic.zhamao.ml/1565526264426.png)

---



当前最高版本2.12，需要2.15.所以需要升级glibc

- 使用yum更新安装依赖

  ```shell
  sudo yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make -y
  ```

- 下载rpm包

  ```shell
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-utils-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-static-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-common-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-devel-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-headers-2.17-55.el6.x86_64.rpm &
  wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/nscd-2.17-55.el6.x86_64.rpm &
  ```

- 安装rpm包

  ```shell
  sudo rpm -Uvh *-2.17-55.el6.x86_64.rpm --force --nodeps
  ```

- 安装完毕后再查看glibc版本,发现glibc版本已经到2.17了

  ```shell
  strings /lib64/libc.so.6 | grep GLIBC
  ```

  

  ---

  ![1565528746057](http://fgcy-pic.zhamao.ml/1565528746057.png)

  ---

  





## 3. 安装RabbitMQ

```sh
# 安装
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm

# 安装
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```



## 4. 开启管理界面及配置

```sh
# 开启管理界面
rabbitmq-plugins enable rabbitmq_management
# 修改默认配置信息
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app 
# 比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest
```






## 5. 启动

```sh
service rabbitmq-server start # 启动服务
service rabbitmq-server stop # 停止服务
service rabbitmq-server restart # 重启服务
```



- 设置配置文件

```shell
cd /usr/share/doc/rabbitmq-server-3.6.5/

cp rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
```





## 6. 配置虚拟主机及用户

### 6.1. 用户角色

RabbitMQ在安装好后，可以访问`http://ip地址:15672` ；其自带了guest/guest的用户名和密码；如果需要创建自定义用户；那么也可以登录管理界面后，如下操作：



- 默认用户

---

![1565098043833](http://fgcy-pic.zhamao.ml/1565098043833.png)

---



- 添加用户

---

![1565098315375](http://fgcy-pic.zhamao.ml/1565098315375.png)

---



**角色说明**：

1、 超级管理员(administrator)

可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

2、 监控者(monitoring)

可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

3、 策略制定者(policymaker)

可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

4、 普通管理者(management)

仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

5、 其他

无法登陆管理控制台，通常就是普通的生产者和消费者





### 6.2. Virtual Hosts配置

像mysql拥有数据库的概念并且可以指定用户对库和表等操作的权限。RabbitMQ也有类似的权限管理；在RabbitMQ中可以虚拟消息服务器Virtual Host，每个Virtual Hosts相当于一个相对独立的RabbitMQ服务器，每个VirtualHost之间是相互隔离的。exchange、queue、message不能互通。 相当于mysql的db。Virtual Name一般以/开头。



#### 6.2.1. 创建Virtual Hosts



---

![1565098496482](http://fgcy-pic.zhamao.ml/1565098496482.png)

---





#### 6.2.2. 设置Virtual Hosts权限



---

![1565098585317](http://fgcy-pic.zhamao.ml/1565098585317.png)

---





---

![1565098719054](http://fgcy-pic.zhamao.ml/1565098719054.png)

---





## 与RabbitMQ相关的端口

----

![image-20220602095533350](http://fgcy-pic.zhamao.ml/image-20220602095533350.png)

---





# RabbitMQ 快速入门

- 需求：使用简单模式完成消息传递



使用简单模式：

---

![image-20220602125151365](http://fgcy-pic.zhamao.ml/image-20220602125151365.png)

---



在上图的模型中，有以下概念：

- P：生产者，也就是要发送消息的程序
-  C：消费者：消息的接收者，会一直等待消息到来
-  queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息



## 生产者

- pom依赖

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.fgcy</groupId>
    <artifactId>rabbitmq-producer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.6.0</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
~~~





- 一个任意的main方法

~~~java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/*
 *
 * @since: 1.8
 * @description：发送消息
 * @author: fgcy
 * @date: 2022/6/2
 */
public class Helloword {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //5.创建队列
        /*
        public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare
        (String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) 
        参数：
        1.queue：队列名称
        2.duiable：是否持节化；mq重启后还在
        3.exclusive：
            - 是否独占；即：只能有一个消费者监听此队列
            - 当connection关闭时，是否删除队列
        4.autoDelete：是否自动删除；当没有consumer时，自动删除
        5.arguments：参数；配置怎么删除队列的一些参数

        * */
        //若果有hello_world的队列则不会创建；否则创建
        channel.queueDeclare("hello_world", true, false, false, null);


        //6.发送消息

        /*
          public void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException {
           参数：
           1.exchange：交换机名称 简单模式下交换机会使用默认 ”“
           2. routingKey:路由名称
           3.props:配置信息
           4.body:发送那个消息
        */
        String body = "hello rabbitmq!~~~~";
        channel.basicPublish("", "hello_world", null, body.getBytes(StandardCharsets.UTF_8));
        channel.close();
        connection.close();
    }
}
~~~



## 消费者



~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class HelloWorld {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //5.创建队列
        /*
        public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare
        (String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
        参数：
        1.queue：队列名称
        2.duiable：是否持节化；mq重启后还在
        3.exclusive：
            - 是否独占；即：只能有一个消费者监听此队列
            - 当connection关闭时，是否删除队列
        4.autoDelete：是否自动删除；当没有consumer时，自动删除
        5.arguments：参数；配置怎么删除队列的一些参数

        * */
        //若果有hello_world的队列则不会创建；否则创建
        channel.queueDeclare("hello_world", true, false, false, null);


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("consumerTag:" + consumerTag);
                System.out.println("Exchange:" + envelope.getExchange());
                System.out.println("RoutingKey:" + envelope.getRoutingKey());
                System.out.println("properties:" + properties);
                System.out.println("body:" + new String(body));

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        channel.basicConsume("hello_world", true, consumer);

    //    不要关闭资源
    }
}

~~~





# RabbitMQ 的工作模式

## 简单模式 HelloWorld
一个生产者、一个消费者，不需要设置交换机（使用默认的交换机）。



## 工作队列模式 Work Queue
一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认的交换机）



## 发布订阅模式 Publish/subscribe
需要设置类型为 fanout 的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消
息发送到绑定的队列。



## 路由模式 Routing
需要设置类型为 direct 的交换机，交换机和队列进行绑定，并且指定 routing key，当发送消息到交换机
后，交换机会根据 routing key 将消息发送到对应的队列。



## 通配符模式 Topic
需要设置类型为 topic 的交换机，交换机和队列进行绑定，并且指定通配符方式的 routing key，当发送
消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。









# Work queues 工作队列模式



---

![image-20220602125406620](http://fgcy-pic.zhamao.ml/image-20220602125406620.png)

---

 

Work Queues：与入门程序的简单模式相比，多了一个或一些消费端，多个消费端共同消费同一个队列中的消息

应用场景：对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度



Work Queues 与入门程序的简单模式的代码几乎是一样的。可以完全复制，并多复制一个消费者进行多个消费者同时对消费消息的测试

## 生产者

~~~java
package com.fgcy.producer;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/*
 *
 * @since: 1.8
 * @description：发送消息
 * @author: fgcy
 * @date: 2022/6/2
 */
public class Producter_Work_Queue {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //5.创建队列
        /*
        public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare
        (String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException {
        参数：
        1.queue：队列名称
        2.duiable：是否持节化；mq重启后还在
        3.exclusive：
            - 是否独占；即：只能有一个消费者监听此队列
            - 当connection关闭时，是否删除队列
        4.autoDelete：是否自动删除；当没有consumer时，自动删除
        5.arguments：参数；配置怎么删除队列的一些参数

        * */
        //若果有hello_world的队列则不会创建；否则创建
        channel.queueDeclare("work_queue", true, false, false, null);


        //6.发送消息

        /*
          public void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException {
           参数：
           1.exchange：交换机名称 简单模式下交换机会使用默认 ”“
           2. routingKey:路由名称
           3.props:配置信息
           4.body:发送那个消息
        */
        for (int i = 0; i < 9; ) {
            String body = ++i + "hello rabbitmq!~~~~";
            channel.basicPublish("", "work_queue", null, body.getBytes(StandardCharsets.UTF_8));
        }

    }
}

~~~



## 消费者

- work_queue1

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Work_Queue1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //5.创建队列
        /*
        public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare
        (String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
        参数：
        1.queue：队列名称
        2.duiable：是否持节化；mq重启后还在
        3.exclusive：
            - 是否独占；即：只能有一个消费者监听此队列
            - 当connection关闭时，是否删除队列
        4.autoDelete：是否自动删除；当没有consumer时，自动删除
        5.arguments：参数；配置怎么删除队列的一些参数

        * */
        //若果有work_queue的队列则不会创建；否则创建
        channel.queueDeclare("work_queue", true, false, false, null);


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        channel.basicConsume("work_queue", true, consumer);

    //    不要关闭资源
    }
}

~~~





- work_queue2

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Work_Queue2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //5.创建队列
        /*
        public com.rabbitmq.client.AMQP.Queue.DeclareOk queueDeclare
        (String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
        参数：
        1.queue：队列名称
        2.duiable：是否持节化；mq重启后还在
        3.exclusive：
            - 是否独占；即：只能有一个消费者监听此队列
            - 当connection关闭时，是否删除队列
        4.autoDelete：是否自动删除；当没有consumer时，自动删除
        5.arguments：参数；配置怎么删除队列的一些参数

        * */
        //若果有work_queue的队列则不会创建；否则创建
        channel.queueDeclare("work_queue", true, false, false, null);


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        channel.basicConsume("work_queue", true, consumer);

    //    不要关闭资源
    }
}

~~~



## 小结

Work queues 工作队列模式

1. 在一个队列中如果有多个消费者，那么消费者之间对于同一个消息的关系是**竞争的关系**
2. Work Queues 对于任务过重或任务较多情况使用工作队列可以提高任务处理的速度。例如：短信服务部署多个，只需要有一个节点成功发送即可。





# Pub/Sub 订阅模式





---

![image-20220602134441648](http://fgcy-pic.zhamao.ml/image-20220602134441648.png)

---



在订阅模型中，多了一个 Exchange 角色，而且过程略有变化：

- P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
- C：消费者，消息的接收者，会一直等待消息到来
-  Queue：消息队列，接收消息、缓存消息
-  Exchange：交换机（X）。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。



- Exchange有常见以下3种类型：
  ➢ Fanout：广播，将消息交给所有绑定到交换机的队列
  ➢ Direct：定向，把消息交给符合指定routing key 的队列
  ➢ Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列
  Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与 Exchange 绑定，或者没有符合
  路由规则的队列，那么消息会丢失



交换机类型设置为广播【Fanout】

- 生产者

~~~java


import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import sun.misc.Unsafe;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

public class Producter_PubSub {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //    创建交换机
        /**
         public DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable,boolean autoDelete, boolean internal, Map<String, Object> arguments)
         参数：
         1.exchange 交换机名称
         2.type 交换机类型 枚举
             DIRECT("direct"),定向
             FANOUT("fanout"),扇形（广播） 发送消息到每一个与之绑定的队列
             TOPIC("topic"),通配符
             HEADERS("headers");参数匹配
         3.durable 是否持久化
         4.autoDelete 自动删除 当没有consumer时，自动删除
         5. internal :内部使用 一般为false
         6.arguments:参数
         */
        String exchangeName = "test_fanout";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT, true, false, false, null);


        //    创建队列
        String quequeName1 = "test_fanout_queue1";
        String quequeName2 = "test_fanout_queue2";

        //String queue, boolean durable, boolean exclusive【只能有一个消费者监听】, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(quequeName1, true, false, false, null);
        channel.queueDeclare(quequeName2, true, false, false, null);

        //    绑定队列和交换机
        /*
         public com.rabbitmq.client.AMQP.Queue.BindOk queueBind(String queue, String exchange, String routingKey)
         参数：
         1.queue：队列名称
         2.交换机名称
         3.路由；绑定规则
            如果交换机的类型为fanout，routingkey设置为“”
         */
        channel.queueBind(quequeName1, exchangeName, "");
        channel.queueBind(quequeName2, exchangeName, "");
        
        //    发送信息
        String body = "日志信息：张三调用了findAll方法 日志级别：info。。。";
        channel.basicPublish(exchangeName, "", null, body.getBytes(StandardCharsets.UTF_8));
        
        //    释放资源
        channel.close();
        connection.close();
    }
}
~~~



---

![image-20220602135218887](http://fgcy-pic.zhamao.ml/image-20220602135218887.png)

---



![image-20220602135344142](http://fgcy-pic.zhamao.ml/image-20220602135344142.png)

---





- 消费者1

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_PubSub1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();



        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) 				throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息打印到控制台。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        String quequeName1 = "test_fanout_queue1";
        channel.basicConsume(quequeName1, true, consumer);

        //    不要关闭资源
    }
}

~~~

~~~
body:日志信息：张三调用了findAll方法 日志级别：info。。。
将日志信息打印到控制台。。。。
~~~



---

![image-20220602153907378](http://fgcy-pic.zhamao.ml/image-20220602153907378.png)

---





- 消费者2

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_PubSub2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();



        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息保存数据库。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */

        String quequeName2 = "test_fanout_queue2";
        channel.basicConsume(quequeName2, true, consumer);

        //    不要关闭资源
    }
}
~~~

~~~
body:日志信息：张三调用了findAll方法 日志级别：info。。。
将日志信息保存数据库。。。。
~~~



---

![image-20220602140529133](http://fgcy-pic.zhamao.ml/image-20220602140529133.png)

---





# Routing 路由模式



队列与交换机的绑定，不能是任意绑定了，而是要指定一个 RoutingKey（路由key）

消息的发送方在向 Exchange 发送消息时，也必须指定消息的 RoutingKey

Exchange 不再把消息交给每一个绑定的队列，而是根据消息的 Routing Key 进行判断，只有队列的Routingkey 与消息的 Routing key 完全一致，才会接收到消息

---

![image-20220816203440005](http://fgcy-pic.zhamao.ml/image-20220816203440005.png)

---



- P：生产者，向 Exchange 发送消息，发送消息时，会指定一个routing key
-  X：Exchange（交换机），接收生产者的消息，然后把消息递交给与 routing key 完全匹配的队列
-  C1：消费者，其所在队列指定了需要 routing key 为 error 的消息
-  C2：消费者，其所在队列指定了需要 routing key 为 info、error、warning 的消息





- 生产者

在channel中绑定交换机、队列、路由规则

将不同的信息发送给不同的交换机（而一个交换机中又有不同的routingkey）从而可以将消息发送到不同的队列中

消费者直接从队列中取消息





~~~java
package com.fgcy.producer;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/*
 *
 * @since: 1.8
 * @description：发送消息
 * @author: fgcy
 * @date: 2022/6/2
 */
public class Producter_Routing {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //    创建交换机
        /**
         public DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable,boolean autoDelete, boolean internal, Map<String, Object> arguments)
         参数：
         1.exchange 交换机名称
         2.type 交换机类型 枚举
         DIRECT("direct"),定向
         FANOUT("fanout"),扇形（广播） 发送消息到每一个与之绑定的队列
         TOPIC("topic"),通配符
         HEADERS("headers");参数匹配
         3.durable 是否持久化
         4.autoDelete 自动删除 当没有consumer时，自动删除
         5. internal :内部使用 一般为false
         6.arguments:参数
         */
        String exchangeName = "test_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true, false, false, null);


        //    创建队列
        String quequeName1 = "test_direct_queue1";
        String quequeName2 = "test_direct_queue2";

        //String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(quequeName1, true, false, false, null);
        channel.queueDeclare(quequeName2, true, false, false, null);

        //    绑定队列和交换机
        /*
         public com.rabbitmq.client.AMQP.Queue.BindOk queueBind(String queue, String exchange, String routingKey)
         参数：
         1.queue：队列名称
         2.交换机名称
         3.路由；绑定规则
            如果交换机的类型为fanout，routingkey设置为“”
         */
        channel.queueBind(quequeName1, exchangeName, "error");

        channel.queueBind(quequeName2, exchangeName, "info");
        channel.queueBind(quequeName2, exchangeName, "error");
        channel.queueBind(quequeName2, exchangeName, "warning");
        //    发送信息
        
        // TODO: 2022/6/2 先是info级别
        //String body = "日志信息：张三调用了findAll方法 日志级别：info。。。";
        // TODO: 2022/6/2 第二次是error级别
        String body = "日志信息：张三调用了delete方法 日志级别：error。。。";
        // TODO: 2022/6/2 设置routingkey info、error
        channel.basicPublish(exchangeName, "error", null, body.getBytes(StandardCharsets.UTF_8));
        //    释放资源
        channel.close();
        connection.close();
   }

}
~~~



---

![image-20220602153611924](http://fgcy-pic.zhamao.ml/image-20220602153611924.png)

---



- 消费者1

通过channel直接从指定的队列中取消息



~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Routing1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息输出到数据库。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        String quequeName1 = "test_direct_queue1";

        channel.basicConsume(quequeName1, true, consumer);

        //    不要关闭资源
    }
}

~~~

~~~
body:日志信息：张三调用了delete方法 日志级别：error。。。
将日志信息输出到数据库。。。。
~~~



- 生产者2

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Routing2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) 				throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息打印到控制台。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */

        String quequeName2 = "test_direct_queue2";

        channel.basicConsume(quequeName2, true, consumer);

        //    不要关闭资源
    }
}

~~~

~~~
body:日志信息：张三调用了findAll方法 日志级别：info。。。
将日志信息打印到控制台。。。。
body:日志信息：张三调用了delete方法 日志级别：error。。。
将日志信息打印到控制台。。。。
~~~



- 小结

Routing 模式要求队列在绑定交换机时要指定 routing key，消息会转发到符合 routing key 的队列。







# Topics 通配符模式



Topic 类型与 Direct 相比，都是可以根据 RoutingKey 把消息路由到不同的队列。只不过 Topic 类型Exchange 可以让队列在绑定 Routing key 的时候使用通配符！
 Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert 

 通配符规则：**# 匹配一个或多个词**，*** 匹配不多不少恰好1个词**，例如：item.# 能够匹配 item.insert.abc 

或者 item.insert，item.* 只能匹配 item.insert



---

![image-20220602160525695](http://fgcy-pic.zhamao.ml/image-20220602160525695.png)

---





---

![image-20220602160539958](http://fgcy-pic.zhamao.ml/image-20220602160539958.png)

---

图解：
	 红色 Queue：绑定的是 usa.# ，因此凡是以 usa. 开头的 routing key 都会被匹配到
	 黄色 Queue：绑定的是 #.news ，因此凡是以 .news 结尾的 routing key 都会被匹配







- 生产者

在channel中绑定交换机、routingkey、队列

通过channel将消息发送给交换机【要指定路由规则】

~~~java
package com.fgcy.producer;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/*
 *
 * @since: 1.8
 * @description：发送消息
 * @author: fgcy
 * @date: 2022/6/2
 */
public class Producter_Topics {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();

        //    创建交换机
        /**
         public DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable,boolean autoDelete,
         boolean internal, Map<String, Object> arguments)
         参数：
         1.exchange 交换机名称
         2.type 交换机类型 枚举
         DIRECT("direct"),定向
         FANOUT("fanout"),扇形（广播） 发送消息到每一个与之绑定的队列
         TOPIC("topic"),通配符
         HEADERS("headers");参数匹配
         3.durable 是否持久化
         4.autoDelete 自动删除 当没有consumer时，自动删除
         5. internal :内部使用 一般为false
         6.arguments:参数
         */
        String exchangeName = "test_topics";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC, true, false, false, null);


        //    创建队列
        String quequeName1 = "test_topic_queue1";
        String quequeName2 = "test_topic_queue2";

        //String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(quequeName1, true, false, false, null);
        channel.queueDeclare(quequeName2, true, false, false, null);

        //    绑定队列和交换机
        /*
         public com.rabbitmq.client.AMQP.Queue.BindOk queueBind(String queue, String exchange, String routingKey)
         参数：
         1.queue：队列名称
         2.交换机名称
         3.路由；绑定规则
            如果交换机的类型为fanout，routingkey设置为“”
         */

        //routing有两部分组成 系统名称.日志级别
        //需求：所有error级别的日志都存入数据库；所有order系统的日志都存入数据库
        channel.queueBind(quequeName1, exchangeName, "#。error");// # 多个词
        channel.queueBind(quequeName1, exchangeName, "order.*");// * 一个词
        //所有的日志都在控制台打印
        channel.queueBind(quequeName2, exchangeName, "*.*");

        
        //    发送信息
        //String body = "日志信息：张三调用了findAll方法 日志级别：info。。。";
        String body = "日志信息：张三调用了delete方法 日志级别：error。。。";

        channel.basicPublish(exchangeName, "order.info", null, body.getBytes(StandardCharsets.UTF_8));
        //    释放资源
        channel.close();
        connection.close();
    }
}
~~~



---

![image-20220603225420927](http://fgcy-pic.zhamao.ml/image-20220603225420927.png)

---







- 消费者1

~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Topic1 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息输出到数据库。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */
        String quequeName1 = "test_topic_queue1";


        channel.basicConsume(quequeName1, true, consumer);

        //    不要关闭资源
    }
}

~~~

~~~
body:日志信息：张三调用了delete方法 日志级别：error。。。
将日志信息打印到控制台。。。。
~~~







- 消费者2



~~~java
package com.fgcy.comsumer;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @Author fgcy
 * @Date 2022/6/2
 */
public class Consumer_Topic2 {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1.创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();

        //2.设置参数
        factory.setHost("43.142.48.199");//默认localhost
        factory.setVirtualHost("/itcast");//默认 /
        factory.setPort(5672);//默认 5672
        factory.setUsername("fgcy");//默认 guess
        factory.setPassword("waiting");//默认guess

        //3。创建连接
        Connection connection = factory.newConnection();

        //4.创建channel
        Channel channel = connection.createChannel();


        //6.接收信息

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                /*
                参数：
                1.consumerTag 标识
                2.envelope：后去一些信息；交换机，路由key
                3.配置信息
                4.body：数据
                * */

                System.out.println("body:" + new String(body));
                System.out.println("将日志信息打印到控制台。。。。");

            }
        };

        /*
           public String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback)
           1. queue 队列名称
           2. autoAck：是否自动确定 消费者收到消息后会发送消息高数mq已收到消息
           3. callback: 回调对象
         */

        String quequeName2 = "test_topic_queue2";

        channel.basicConsume(quequeName2, true, consumer);

        //    不要关闭资源
    }
}

~~~

~~~
body:日志信息：张三调用了delete方法 日志级别：error。。。
将日志信息打印到控制台。。。。
~~~





- 更改生产者代码

```java
channel.basicPublish(exchangeName, "goods.info", null, body.getBytes(StandardCharsets.UTF_8));//更改routingkey
```



只有队列2可以接收到消息



## 小结

Topic 主题模式可以实现 Pub/Sub 发布与订阅模式和 Routing 路由模式的功能，只是 Topic 在配置routing key 的时候可以使用通配符，显得更加灵活。









# Spring整合RabbitMQ



## --------生产者

## pom文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.fgcy</groupId>
    <artifactId>spring-rabbitmq-producer</artifactId>
    <version>1.0.0</version>


    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.1.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
~~~





## spring核心配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>
    <!--定义管理交换机、队列-->
    <rabbit:admin connection-factory="connectionFactory"/>

    <!--定义持久化队列，不存在则自动创建；不绑定到交换机则绑定到默认交换机
    默认交换机类型为direct，名字为：""，路由键为队列的名称
    -->
    
    
    
    <!--
	id:bean的名称
	name：队列的名称
	auto-declare：当队列不存在时自动创建
	auto-delete:当没有消费者与该队列相连接时，自动删除
	durable：是否持久化
	exclusive：是否独占队列，只允许一个消费者使用
	-->
    <rabbit:queue id="spring_queue" name="spring_queue" auto-declare="true"/>


    <rabbit:queue id="spring_direct_queue_1" name="spring_direct_queue_1" auto-declare="true"/>
    <rabbit:queue id="spring_direct_queue_2" name="spring_direct_queue_2" auto-declare="true"/>

    <rabbit:direct-exchange name="spring_direct_exchange" id="spring_direct_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="spring_direct_queue_1" key="root"/>
            <rabbit:binding queue="spring_direct_queue_2" key="admin"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>
    
    
    
    

    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~广播；所有队列都能收到消息~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_1" name="spring_fanout_queue_1" auto-declare="true"/>

    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_2" name="spring_fanout_queue_2" auto-declare="true"/>

    <!--定义广播类型交换机；并绑定上述两个队列-->
    <rabbit:fanout-exchange id="spring_fanout_exchange" name="spring_fanout_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="spring_fanout_queue_1"/>
            <rabbit:binding queue="spring_fanout_queue_2"/>
        </rabbit:bindings>
    </rabbit:fanout-exchange>

    
    
    
    
    
    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~通配符；*匹配一个单词，#匹配多个单词 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_star" name="spring_topic_queue_star" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well" name="spring_topic_queue_well" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well2" name="spring_topic_queue_well2" auto-declare="true"/>

    <rabbit:topic-exchange id="spring_topic_exchange" name="spring_topic_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding pattern="heima.*" queue="spring_topic_queue_star"/>
            <rabbit:binding pattern="heima.#" queue="spring_topic_queue_well"/>
            <rabbit:binding pattern="itcast.#" queue="spring_topic_queue_well2"/>
        </rabbit:bindings>
    </rabbit:topic-exchange>

    <!--定义rabbitTemplate对象操作可以在代码中方便发送消息-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>

</beans>

~~~



## rabbitmq配置文件

~~~properties
rabbitmq.host=43.142.48.199
rabbitmq.port=5672
rabbitmq.username=fgcy
rabbitmq.password=waiting
rabbitmq.virtual-host=/itcast
~~~



## 通过测试启动程序**【发送简单消息】**

~~~java
package com.fgcy;

/**
 * @Author fgcy
 * @Date 2022/6/3
 */

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-producer.xml")
public class ConsumerTest {

    //1.注入rabbitTemplate
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testHelloWorld() {
        //2.发送消息
        //简单模式指定routingkey为队列名称
        rabbitTemplate.convertAndSend("spring_queue", "hello spring-rabbitmq...");
    }
}
~~~



---

![image-20220603234138789](http://fgcy-pic.zhamao.ml/image-20220603234138789.png)

---

要有test的字节码文件才能运行测试



否则

---

![image-20220603234253580](http://fgcy-pic.zhamao.ml/image-20220603234253580.png)

---







- 查看管理界面

---

![image-20220603234345492](http://fgcy-pic.zhamao.ml/image-20220603234345492.png)

---

![image-20220603234416980](http://fgcy-pic.zhamao.ml/image-20220603234416980.png)

---



---

![image-20220603234552652](http://fgcy-pic.zhamao.ml/image-20220603234552652.png)

---

![image-20220816203349065](http://fgcy-pic.zhamao.ml/image-20220816203349065.png)

---





## 通过广播的方式

~~~java
    @Test
    public void testFanoutMessage(){
    rabbitTemplate.convertAndSend("spring_fanout_exchange","","spring fanout.....");
    }
~~~



---

![image-20220603235201322](http://fgcy-pic.zhamao.ml/image-20220603235201322.png)

---









## 通过通配符方式

~~~java
    @Test
    public void testTopicMessage(){
    rabbitTemplate.convertAndSend("spring_topic_exchange","heima.aaa.bbb","spring topic 2");
    }
~~~





---

![image-20220603235505964](http://fgcy-pic.zhamao.ml/image-20220603235505964.png)

---







## 通过direct

~~~java
    @Test
    public void testDirectMessage() {
        rabbitTemplate.convertAndSend("spring_direct_exchange", "root", "spring direct 1");
    }
~~~



---

![image-20220604000601469](http://fgcy-pic.zhamao.ml/image-20220604000601469.png)

---











## ------消费者



## pom

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.fgcy</groupId>
    <artifactId>spring-rabbitmq-consumer</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>

        <!--        只是spring-container中的jar包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>

        <!--        spring整合rabbitmq-->
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.1.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

~~~





## rabbitmq配置文件

~~~properties
rabbitmq.host=43.142.48.199
rabbitmq.port=5672
rabbitmq.username=fgcy
rabbitmq.password=waiting
rabbitmq.virtual-host=/itcast
~~~



## spring核心配置文件



~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>

    <bean id="springQueueListener" class="com.fgcy.listener.SpringQueueListener"/>
    <bean id="fanoutListener1" class="com.fgcy.listener.FanoutListener1"/>
    <bean id="fanoutListener2" class="com.fgcy.listener.FanoutListener2"/>
    <bean id="topicListenerStar" class="com.fgcy.listener.TopicListenerStar"/>
    <bean id="topicListenerWell" class="com.fgcy.listener.TopicListenerWell"/>
    <bean id="topicListenerWell2" class="com.fgcy.listener.TopicListenerWell2"/>
    <bean id="directqueue1" class="com.fgcy.listener.Directqueue1"/>
    <bean id="directqueue2" class="com.fgcy.listener.Directqueue2"/>


    <rabbit:listener-container connection-factory="connectionFactory" auto-declare="true">
        <rabbit:listener ref="springQueueListener" queue-names="spring_queue"/>
        <rabbit:listener ref="fanoutListener1" queue-names="spring_fanout_queue_1"/>
        <rabbit:listener ref="fanoutListener2" queue-names="spring_fanout_queue_2"/>
        <rabbit:listener ref="topicListenerStar" queue-names="spring_topic_queue_star"/>
        <rabbit:listener ref="topicListenerWell" queue-names="spring_topic_queue_well"/>
        <rabbit:listener ref="topicListenerWell2" queue-names="spring_topic_queue_well2"/>
        <rabbit:listener ref="topicListenerWell2" queue-names="spring_topic_queue_well2"/>
        <rabbit:listener ref="directqueue1" queue-names="spring_direct_queue_1"/>
        <rabbit:listener ref="directqueue2" queue-names="spring_direct_queue_2"/>

    </rabbit:listener-container>

</beans>

~~~





## 定义监听器逻辑

~~~java
package com.fgcy.listener;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;

/**
 * @Author fgcy
 * @Date 2022/6/4
 */
public class Directqueue1 implements MessageListener {
    @Override
    public void onMessage(Message message) {
        //打印消息
        System.out.println(new String(message.getBody()));
    }
}

~~~



---

![image-20220604002847182](http://fgcy-pic.zhamao.ml/image-20220604002847182.png)

---

上面的监听器全部实现MessageListener接口

实现onMessage方法

~~~java
    @Override
    public void onMessage(Message message) {
        //打印消息
        System.out.println(new String(message.getBody()));
    }
~~~



- 回顾生产者向mq发送的消息

~~~java
package com.fgcy;

/**
 * @Author fgcy
 * @Date 2022/6/3
 */

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-producer.xml")
public class ConsumerTest {

    //1.注入rabbitTemplate
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testHelloWorld() {
        //2.发送消息
        //简单模式指定routingkey为队列名称
        rabbitTemplate.convertAndSend("spring_queue", "hello spring-rabbitmq...");
    }


    @Test
    public void testFanoutMessage() {
        rabbitTemplate.convertAndSend("spring_fanout_exchange", "", "spring fanout.....");
    }

    @Test
    public void testTopicMessage() {
        rabbitTemplate.convertAndSend("spring_topic_exchange", "heima.aaa.bbb", "spring topic 2");
    }

    @Test
    public void testDirectMessage() {
        rabbitTemplate.convertAndSend("spring_direct_exchange", "root", "spring direct 1");
    }
}

~~~







- 编写消费者测试代码

~~~java
package com.fgcy;

/**
 * @Author fgcy
 * @Date 2022/6/4
 */

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-consumer.xml")
public class ConsumerTest {


    @Test
    public void test1() {
        while (true) {

        }
    }
}
~~~





- 结果

---

![image-20220816000751302](http://fgcy-pic.zhamao.ml/image-20220816000751302.png)

---

