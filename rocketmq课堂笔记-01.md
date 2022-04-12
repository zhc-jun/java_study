## RokcetMQ，第一天

# 今日目标

- 能够熟悉MQ特点
- 能够掌握rocketmq的环境搭建
- 能够掌握rocketmq消息发送
- 能够完成rocketmq单服务器的搭建



## 1.1 MQ简介

### 1.1.1【MQ概念】问题

1.问题 1：什么是MQ？

2.问题 2：为什么要队列？

3.问题 2：什么叫消息？

### 1.1.2【MQ概念】答案

1.问题 1：什么是MQ？

- 消息队列，一般我们会简称它为MQ(Message Queue)，就是很直白的简写。

2.问题 2：为什么要有队列？

- 队列是一种**先进先出**的数据结构

  
  
  ![img](https://pic2.zhimg.com/80/v2-607a5661776bdf018ed26cdfca23cf11_720w.jpg?source=1940ef5c)

3.问题 2：什么叫消息？

- 服务A和服务B之间需要传输的数据，就可以理解为一种消息。

## 1.2 MQ作用

### 1.2.1【MQ作用】问题

1.问题 1：什么叫应用解耦？

2.问题 2：什么叫流量削峰？

### 1.2.2【MQ作用】答案

1.问题1：什么叫应用解耦？

- 应用

  就是服务

- 解耦

  服务之间的调用十分复杂，为了降低服务与服务之间调用依赖，使用MQ将两系统分开,不直接调用系统接口,减轻两系统依赖关系

2.问题 2：什么叫流量削峰？

- 流量

  主要是还是来自于互联网的业务场景，例如，大家熟知的阿里双11秒杀， 短时间上亿的用户涌入，瞬间流量巨大（高并发）

- 削峰

  把本来需要同步的直接调用转换成异步的间接推送，中间通过一个队列在一端承接瞬时的流量洪峰，在另一端平滑地将消息推送出去。

  

## 1.3 MQ的优缺点

### 1.3.1【MQ缺点】问题

1.问题 1：MQ的不足有哪些？怎么解决的？



### 1.3.2【MQ缺点】答案

1.问题1：MQ的不足有哪些？

- 系统的可用性降低

  单MQ服务情况下，系统出现问题。

  解决方案：集群部署

- 系统的复杂度提高

  需要单独开发及部署

- 消息的顺序性

  因为是集群部署，不能保证消息有序

- 消息的丢失

  某个MQ服务挂了，会造成消息丢失

- 消息的一致性

- 消息的重复使用

  

## 1.4 MQ产品介绍

### 1.4.1【MQ产品介绍】问题

1.问题 1：目前使用的MQ产品有哪些？

2.问题 2：MQ产品的区别？

### 1.4.2【MQ产品介绍】答案

1.问题1：目前使用的MQ产品有哪些？

- ActiveMQ 
- RabbitMQ
- RocketMQ
- kafka

2.问题 2：MQ产品的区别？

-  ActiveMQ：java语言实现，万级数据吞吐量，处理速度ms级，主从架构，成熟度高

-  RabbitMQ ：erlang语言实现，万级数据吞吐量，处理速度us级，主从架构， 

-  RocketMQ ：java语言实现，十万级数据吞吐量，处理速度ms级，分布式架构，功能强大，扩展性强 

-  kafka ：scala语言实现，十万级数据吞吐量，处理速度ms级，分布式架构，功能较少，应用于大数据较多

  

## 1.5 RocketMQ基础概念

### 1.5.1【RocketMQ基础概念】

RocketMQ是一款分布式消息中间件

-  生产者
-  消费者 
-  消息服务器 
-  命名服务器 
-  消息 

​      ◆ 主题 

​      ◆ 标签

### 1.5.2【RocketMQ模块间数据流转】



![img](https://user-gold-cdn.xitu.io/2019/6/9/16b3bddaba7b1e50?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 1.6 RocketMQ服务器安装

安装过程 

⚫ 步骤1：安装JDK（1.8）

⚫ 步骤2：上传压缩包（zip） 

安装上传插件

```
yum -y install lrzsz 
```

安装完成后，输入rz命令，上传rocketmq-all-4.5.2-bin-release.zip

```
rz
```

⚫ 步骤3：解压缩 

```
unzip rocketmq-all-4.5.2-bin-release.zip
```

⚫ 步骤4：修改目录名称

```
mv rocketmq-all-4.5.2-bin-release rocketmq
```

启动服务器 

进入 rocketmq的bin目录

```
cd rocketmq/bin/
```



⚫ 步骤5：启动命名服务器（bin目录下） 

```
sh mqnamesrv
```

⚫ 步骤6：启动消息服务器（bin目录下）,先修改runbroker.sh内设置的服务内存大小,推荐256m-128m

```
vim runbroker.sh
```

```
sh mqbroker -n localhost:9876
```

## 1.7 RocketMQ生产及消费，消息模式（ One-To-One）

#### 1.7.1【RocketMQ生产及消费】问题

⚫ 消息模式 

问题 1：什么是 One-To-One？

问题 2：什么是 One-To-Many？

问题 3：什么是  Many-To-Man？

1.创建maven项目

2.导入rocketMQ客户端坐标

```
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.5.2</version>
</dependency
```

3.生产者

```
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

//生产者，产生消息
public class Producer {
    public static void main(String[] args) throws Exception {
        //1.创建一个发送消息的对象Producer
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        //2.设定发送的命名服务器地址
        producer.setNamesrvAddr("192.168.184.128:9876");
        //3.1启动发送的服务
        producer.start();
        //4.创建要发送的消息对象,指定topic，指定内容body
        Message msg = new Message("topic1","hello rocketmq".getBytes("UTF-8"));
        //3.2发送消息
        SendResult result = producer.send(msg);
        System.out.println("返回结果："+result);
        //5.关闭连接
        producer.shutdown();
    }
}
SendResult result = producer.send(msg);
System.out.println("返回结果："+result);
//5.关闭连接
producer.shutdown();
```

4.消费者

```
package com.itheima.base;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

//消费者，接收消息
public class Consumer {
    public static void main(String[] args) throws Exception {
        //1.创建一个接收消息的对象Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2.设定接收的命名服务器地址
        consumer.setNamesrvAddr("192.168.184.128:9876");
        //3.设置接收消息对应的topic,对应的sub标签为任意*
        consumer.subscribe("topic1","*");
        //3.开启监听，用于接收消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                //遍历消息
                for(MessageExt msg : list){
//                    System.out.println("收到消息："+msg);
                    System.out.println("消息："+new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //4.启动接收消息的服务
        consumer.start();
        System.out.println("接收消息服务已开启运行");
    }
}

```

## 1.8 RocketMQ生产及消费消息（OneToMany）

⚫ 消费者（负载均衡模式：默认模式）

```
package com.itheima.one2many;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

//消费者，接收消息
public class Consumer {
    public static void main(String[] args) throws Exception {
        //1.创建一个接收消息的对象Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2.设定接收的命名服务器地址
        consumer.setNamesrvAddr("192.168.184.128:9876");
        //3.设置接收消息对应的topic,对应的sub标签为任意*
        consumer.subscribe("topic1","*");

        //设置当前消费者的消费模式（默认模式：负载均衡）
       consumer.setMessageModel(MessageModel.CLUSTERING);
        //3.开启监听，用于接收消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                //遍历消息
                for(MessageExt msg : list){
//                    System.out.println("收到消息："+msg);
                    System.out.println("消息："+new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //4.启动接收消息的服务
        consumer.start();
        System.out.println("接收消息服务已开启运行");
    }
}

```

⚫ 消费者（广播模式）

```
package com.itheima.one2many;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

//消费者，接收消息
public class Consumer {
    public static void main(String[] args) throws Exception {
        //1.创建一个接收消息的对象Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        //2.设定接收的命名服务器地址
        consumer.setNamesrvAddr("192.168.184.128:9876");
        //3.设置接收消息对应的topic,对应的sub标签为任意*
        consumer.subscribe("topic1","*");

        //设置当前消费者的消费模式为广播模式：所有客户端接收的消息都是一样的
        consumer.setMessageModel(MessageModel.BROADCASTING);

        //3.开启监听，用于接收消息
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                //遍历消息
                for(MessageExt msg : list){
//                    System.out.println("收到消息："+msg);
                    System.out.println("消息："+new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //4.启动接收消息的服务
        consumer.start();
        System.out.println("接收消息服务已开启运行");
    }
}

```

## 1.9 RocketMQ生产及消费消息（ManyToMany）

1.多起几个生产者

```
package com.itheima.many2many;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

//多生产者对多消费者
//生产者，产生消息
public class Producer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        producer.start();
        for (int i = 1; i <= 10; i++) {
            Message msg = new Message("topic1",("生产者2： hello rocketmq "+i).getBytes("UTF-8"));
            SendResult result = producer.send(msg);
            System.out.println("返回结果："+result);
        }
        producer.shutdown();
    }
}

```

2.多起几个消费者

```
package com.itheima.many2many;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

//消费者，接收消息
public class Consumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        consumer.setNamesrvAddr("192.168.184.128:9876");
        consumer.subscribe("topic1","*");

        consumer.registerMessageListener(new MessageListenerConcurrently() {
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for(MessageExt msg : list){
                    System.out.println("消息："+new String(msg.getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("接收消息服务已开启运行");
    }
}

```

## 2.0 RocketMQ消息类别

#### 2.0.1【RocketMQ消息类别】问题

问题 1：消息类别有哪些？

问题 2：各消息类别的特性？

#### 2.0.2 各消息类别的代码，主要是在生产者中实现

⚫ 同步消息

```
SendResult result = producer.send(msg);
```

⚫ 异步消息（回调处理结果必须在生产者进程结束前执行，否则回调无法正确执行）

```
producer.send(msg, new SendCallback() {
//表示成功返回结果
public void onSuccess(SendResult sendResult) {
System.out.println(sendResult);
}
//表示发送消息失败
public void onException(Throwable t) {
System.out.println(t);
}
});
```

⚫ 单向消息

```
producer.sendOneway(msg);
```



#### 2.0.3【RocketMQ消息类别】答案

问题 1：消息类别有哪些？

⚫ 同步消息 ⚫ 异步消息 ⚫ 单向消息

问题 2：各消息类别的特征？

⚫ 同步消息 

 特征：即时性较强，重要的消息，且必须有回执的消息，例如短信，通知（转账成功）

⚫异步消息 

 特征：即时性较弱，但需要有回执的消息，例如订单中的某些信息

⚫单向消息  

特征：不需要有回执的消息，例如日志类消

## 2.1 RocketMQ消息延时

⚫ 消息发送时并不直接发送到消息服务器，而是根据设定的等待时间到达，起到延时到达的缓冲作用

```
Message msg = new Message("topic3",("延时消息：hello rocketmq "+i).getBytes("UTF-8"));
//设置当前消息的延时效果，这边的3表示30秒，按下标来计算
msg.setDelayTimeLevel(3);
SendResult result = producer.send(msg);
System.out.println("返回结果："+result);
```

⚫ 目前支持的消息时间 

◆ 秒级：1，5，10，30

◆ 分级：1~10，20，30 

◆ 时级：1，2 

◆ 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

## 2.2 RocketMQ批量消息

⚫把多个消息对象放到一个集合里面发送

```
package com.itheima.mul;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.util.ArrayList;
import java.util.List;

//测试批量消息
public class Producer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        producer.start();

        //创建一个集合保存多个消息
        List<Message> msgList = new ArrayList<Message>();

        Message msg1 = new Message("topic5",("批量消息：hello rocketmq "+1).getBytes("UTF-8"));
        Message msg2 = new Message("topic5",("批量消息：hello rocketmq "+2).getBytes("UTF-8"));
        Message msg3 = new Message("topic5",("批量消息：hello rocketmq "+3).getBytes("UTF-8"));

        msgList.add(msg1);
        msgList.add(msg2);
        msgList.add(msg3);

        //发送批量消息(每次发送的消息总量不得超过4M）
        //消息的总长度包含4个信息：topic,body,消息的属性,日志（20字节）
        SendResult send = producer.send(msgList);

        System.out.println(send);

        producer.shutdown();
    }
}
```

⚫ 消息内容总长度不超过4M

⚫ 消息内容总长度包含如下：

 ◆ topic（字符串字节数） 

◆ body （字节数组长度）

◆ 消息追加的属性（key与value对应字符串字节数） 

◆ 日志（固定20字节）

## 2.3 RocketMQ消息过滤

#### 2.3.1 分类过滤

⚫ 生产者 ，添加tag

```
Message msg = new Message("topic6","tag2",("消息过滤按照tag：hello rocketmq 2" 标题 Tag ).getBytes("UTF-8"));
```

⚫ 消费者 

```
//接收消息的时候，除了制定topic，还可以指定接收的tag,*代表任意tag
consumer.subscribe("topic6","tag1 || tag2");
```

#### 2.3.2 语法过滤（SQL过滤）

⚫ 生产者

```
//为消息添加属性
msg.putUserProperty("vip","1");
msg.putUserProperty("age","20");

```

⚫ 消费者

```
//使用消息选择器来过滤对应的属性，语法格式为类SQL语法
consumer.subscribe("topic7", MessageSelector.bySql("age >= 18"));
```

⚫ 注意：SQL过滤需要依赖服务器的功能支持，conf目录下，在broker.conf配置文件中添加对应的功能项，并开启对应功能 

```
 enablePropertyFilter=true 
```

⚫ 启动服务器使启用对应配置文件

```
sh mqbroker -n localhost:9876 -c ../conf/broker.conf
```

## 2.4 RocketMQ消息顺序

生产者

```
package com.itheima.order;

import com.itheima.order.domain.Order;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;

import java.util.ArrayList;
import java.util.List;

//测试顺序消息
public class Producer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        producer.start();

        //创建要执行的业务队列
        List<Order> orderList = new ArrayList<Order>();

        Order order11 = new Order();
        order11.setId("a");
        order11.setMsg("主单-1");
        orderList.add(order11);

        Order order12 = new Order();
        order12.setId("a");
        order12.setMsg("子单-2");
        orderList.add(order12);

        Order order13 = new Order();
        order13.setId("a");
        order13.setMsg("支付-3");
        orderList.add(order13);

        Order order14 = new Order();
        order14.setId("a");
        order14.setMsg("推送-4");
        orderList.add(order14);

        Order order21 = new Order();
        order21.setId("b");
        order21.setMsg("主单-1");
        orderList.add(order21);

        Order order22 = new Order();
        order22.setId("b");
        order22.setMsg("子单-2");
        orderList.add(order22);

        Order order31 = new Order();
        order31.setId("c");
        order31.setMsg("主单-1");
        orderList.add(order31);

        Order order32 = new Order();
        order32.setId("c");
        order32.setMsg("子单-2");
        orderList.add(order32);

        Order order33 = new Order();
        order33.setId("c");
        order33.setMsg("支付-3");
        orderList.add(order33);

        //设置消息进入到指定的消息队列中
        for(final Order order : orderList){
            Message msg = new Message("orderTopic",order.toString().getBytes());
            //发送时要指定对应的消息队列选择器
            SendResult result = producer.send(msg, new MessageQueueSelector() {
                //设置当前消息发送时使用哪一个消息队列
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                    //根据发送的信息不同，选择不同的消息队列
                    //根据id来选择一个消息队列的对象，并返回->id得到int值
                    int mqIndex = order.getId().hashCode() % list.size();
                    return list.get(mqIndex);
                }
            }, null);

            System.out.println(result);
        }

        producer.shutdown();
    }
}
```

消费者

```
package com.itheima.order;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.*;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class Consumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group1");
        consumer.setNamesrvAddr("192.168.184.128:9876");
        consumer.subscribe("orderTopic","*");

        //使用单线程的模式从消息队列中取数据，一个线程绑定一个消息队列
        consumer.registerMessageListener(new MessageListenerOrderly() {
            //使用MessageListenerOrderly接口后，对消息队列的处理由一个消息队列多个线程服务，转化为一个消息队列一个线程服务
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
                for(MessageExt msg : list){
                    System.out.println(Thread.currentThread().getName()+"  消息："+new String(msg.getBody()));
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });

        consumer.start();
        System.out.println("接收消息服务已开启运行");
    }
}

```

## 2.5 RocketMQ事务消息

事务消息状态 

⚫ 提交状态：允许进入队列，此消息与非事务消息无区别

⚫ 回滚状态：不允许进入队列，此消息等同于未发送过 

⚫ 中间状态：完成了half消息的发送，未对MQ进行二次状态确认 

注意：事务消息仅与生产者有关，与消费者无关

生产者代码

```
package com.itheima.transaction;

import org.apache.rocketmq.client.producer.*;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

//测试事务消息
public class Producer {
    public static void main(String[] args) throws Exception {
        //事务消息使用的生产者是TransactionMQProducer
        TransactionMQProducer producer = new TransactionMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        //添加本地事务对应的监听
        producer.setTransactionListener(new TransactionListener() {
            //正常事务过程
            public LocalTransactionState executeLocalTransaction(Message message, Object o) {
                //中间状态
                return LocalTransactionState.UNKNOW;
            }
            //事务补偿过程
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                System.out.println("事务补偿过程执行");
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });
        producer.start();

        Message msg = new Message("topic11",("事务消息：hello rocketmq ").getBytes("UTF-8"));
        SendResult result = producer.sendMessageInTransaction(msg,null);
        System.out.println("返回结果："+result);
        //事务补偿过程必须保障服务器在运行过程中，否则将无法进行正常的事务补偿
//      producer.shutdown();
    }

    public static void main1(String[] args) throws Exception {
        //事务消息使用的生产者是TransactionMQProducer
        TransactionMQProducer producer = new TransactionMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        //添加本地事务对应的监听
        producer.setTransactionListener(new TransactionListener() {
            //正常事务过程
            public LocalTransactionState executeLocalTransaction(Message message, Object o) {
                //事务提交状态
                return LocalTransactionState.COMMIT_MESSAGE;
            }
            //事务补偿过程
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                return null;
            }
        });
        producer.start();

        Message msg = new Message("topic8",("事务消息：hello rocketmq ").getBytes("UTF-8"));
        SendResult result = producer.sendMessageInTransaction(msg,null);
        System.out.println("返回结果："+result);
        producer.shutdown();
    }

    public static void main2(String[] args) throws Exception {
        //事务消息使用的生产者是TransactionMQProducer
        TransactionMQProducer producer = new TransactionMQProducer("group1");
        producer.setNamesrvAddr("192.168.184.128:9876");
        //添加本地事务对应的监听
        producer.setTransactionListener(new TransactionListener() {
            //正常事务过程
            public LocalTransactionState executeLocalTransaction(Message message, Object o) {
                //事务回滚状态
                return LocalTransactionState.ROLLBACK_MESSAGE;
            }
            //事务补偿过程
            public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
                return null;
            }
        });
        producer.start();

        Message msg = new Message("topic9",("事务消息：hello rocketmq ").getBytes("UTF-8"));
        SendResult result = producer.sendMessageInTransaction(msg,null);
        System.out.println("返回结果："+result);
        producer.shutdown();
    }
}
```

