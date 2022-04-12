## RokcetMQ，第二天

# 今日目标

- 能够完成rocketmq集群环境的特点
- 能够完成rocketmq主从集群环境的搭建
- 能够理解rocketmq的高级特性



## 1.1 RocketMQ集群结构和特征

### 1.1.1【RocketMQ集群结构】问题

1.问题 1：RocketMQ集群结构是什么样的？

2.问题 2：RocketMQ集群工作流程？

### 1.1.2【RocketMQ集群结构】答案

1.问题 1：RocketMQ集群结构是什么样的？

 ◆ 多个broker提供服务 

 ◆ 多个master多个slave 

​		◼ master到slave消息同步方式为同步 （较异步方式性能略低，消息无延迟）

​		◼ master到slave消息同步方式为异步 （较同步方式性能略高，数据略有延迟）

2.问题 2：RocketMQ集群工作流程？

⚫ 步骤1：NameServer启动，开启监听，等待broker、producer与consumer连接 

⚫ 步骤2：broker启动，根据配置信息，连接所有的NameServer，并保持长连接

⚫ 步骤2补充：如果broker中有现存数据， NameServer将保存topic与broker关系 

⚫ 步骤3：producer发信息，连接某个NameServer，并建立长连接

⚫ 步骤4：producer发消息 ◆ 步骤4.1若果topic存在，由NameServer直接分配 ◆ 步骤4.2如果topic不存在，由NameServer创建topic与broker关系，并分配

⚫ 步骤5：producer在broker的topic选择一个消息队列（从列表中选择） 

⚫ 步骤6：producer与broker建立长连接，用于发送消息

⚫ 步骤7：producer发送消息 

 comsumer工作流程同producer一样

## 1.2 RocketMQ集群环境搭建

#### 1.2.1基础环境设置

安装过程 

⚫ 步骤1：安装JDK（1.8）

⚫ 步骤2：上传压缩包（zip） 

同学安装上传插件

```
yum -y install lrzsz 
```

安装完成后，输入rz命令，上传rocketmq-all-4.5.2-bin-release.zip，

注意：文件上传的路径，我们是上传到根目录下面

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

 ⚫步骤5:配置服务器环境(两台服务器都要配置)

```
vim /etc/hosts
```

#nameserver 
192.168.200.120 rocketmq-nameserver1 
192.168.200.121  rocketmq-nameserver2
#broker 
192.168.200.120 rocketmq-master1 
192.168.200.121 rocketmq-slave2 
192.168.200.120 rocketmq-master2 
192.168.200.121 rocketmq-slave1

 ⚫ 步骤6:配置完毕后重启网卡，并关闭防火墙

```
systemctl restart network
systemctl stop firewalld
systemctl disable firewalld
```

⚫步骤7: 配置环境信息

```
 vim /etc/profile
```

#set rocketmq 
ROCKETMQ_HOME=/rocketmq
PATH=$PATH:$ROCKETMQ_HOME/bin
export ROCKETMQ_HOME PATH

```
source /etc/profile
```

#### 1.2.2 master服务器环境设置

⚫ 创建集群服务器的数据存储目录 注意master与slave如果在同一个虚拟机中部署，需要将存储目录区分开

```
mkdir -p /rocketmq/store
mkdir -p /rocketmq/store/commitlog
mkdir -p /rocketmq/store/consumequeue
mkdir -p /rocketmq/store/index


mkdir -p /rocketmq/store-slave
mkdir -p /rocketmq/store-slave/commitlog
mkdir -p /rocketmq/store-slave/consumequeue
mkdir -p /rocketmq/store-slave/index

```

⚫ 进入rocketMQ配置目录，编辑master的broker配置文件

```
cd /rocketmq/conf/2m-2s-sync

vim broker-a.properties
```

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
#SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
#ASYNC_FLUSH 异步刷盘
#SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

#### 1.2.3 slave服务器环境设置

编辑slave的broker配置文件

```
cd /rocketmq/conf/2m-2s-sync
vim broker-b-s.properties
```

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示 Master，>0 表示 Slave
brokerId=1
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=11011
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/rocketmq/store-slave
#commitLog 存储路径
storePathCommitLog=/rocketmq/store-slave/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/rocketmq/store-slave/consumequeue
#消息索引存储路径
storePathIndex=/rocketmq/store-slave/index
#checkpoint 文件存储路径
storeCheckpoint=/rocketmq/store-slave/checkpoint
#abort 文件存储路径
abortFile=/rocketmq/store-slave/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#ASYNC_MASTER 异步复制Master
#SYNC_MASTER 同步双写Master
#SLAVE
brokerRole=SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=12
```

修改broker的内存使用

```
vim /rocketmq/bin/runbroker.sh
```

```
# 开发环境配置 JVM Configuration
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

#### 1.2.4  启动服务器

进入rocketmq的bin目录

```
cd  /rocketmq/bin
```

启动namesrv,&表示后台运行，nohup表示不挂断的意思

```
nohup sh mqnamesrv &
```

启动broker

```
nohup sh mqbroker -c ../conf/2m-2s-sync/broker-b.properties &
nohup sh mqbroker -c ../conf/2m-2s-sync/broker-a-s.properties &
```

#### 1.2.5 rocketmq-console集群监控

 ⚫ incubator-rocketmq-externals是一个基于rocketmq的基础之上扩展开发的开源项目

 ⚫ 获取地址：https://github.com/apache/rocketmq-externals

 ⚫ rocketmq-console是一款基于java环境开发的（springboot）的管理控制台工具

## 1.3 RocketMQ高级特性（存储）

### 1.3.1【RocketMQ高级特性】问题

1.问题 1：RocketMQ消息的存储方式？

2.问题 2：RocketMQ消息的存储介质？

### 1.3.2【RocketMQ高级特性】答案

1.问题 1：RocketMQ消息的存储方式？

 	持久化存储

2.问题 2：RocketMQ消息的存储介质？

⚫ 数据库 

◆ ActiveMQ 

◆ 缺点：数据库瓶颈将成为MQ瓶颈

 ⚫ 文件系统 

◆ RocketMQ/Kafka/RabbitMQ

◆ 解决方案：采用消息刷盘机制进行数据存储 

◆ 缺点：硬盘损坏的问题无法避免

## 1.4 RocketMQ高级特性（顺序写，零拷贝）

### 1.4.1【RocketMQ高级特性】问题

1.问题 1：RocketMQ是随机写，还是顺序写？

2.问题 2：RocketMQ怎么实现顺序写的？



### 1.4.2【RocketMQ高级特性】问题

1.问题 1：顺序写

2.问题 2：先在磁盘开辟一块空间，然后在开辟的空间里面模拟顺序写

## 1.5 RocketMQ高级特性（消息存储结构）

![image-20210406190610721](C:\Users\wuli-\AppData\Roaming\Typora\typora-user-images\image-20210406190610721.png)

 ⚫ MQ数据存储区域包含如下内容 

◆ 消息数据存储区域 

​		◼ topic 

​		◼ queueId

​		◼ message 

◆ 消费逻辑队列

​		◼ minOffset

​		◼ maxOffset 

​		◼ consumerOffset 

◆ 索引 

​		◼ key索引 

​		◼ 创建时间索

## 1.6 RocketMQ高级特性（刷盘机制）

⚫ 同步刷盘 

1. 生产者发送消息到MQ，MQ接到消息数据 
2. MQ挂起生产者发送消息的线程 
3. MQ将消息数据写入内存 
4. 内存数据写入硬盘 
5. 磁盘存储后返回SUCCESS 
6. MQ恢复挂起的生产者线程 
7. 发送ACK到生产者

⚫ 异步刷盘 

1. 生产者发送消息到MQ，MQ接到消息数据 
2. MQ将消息数据写入内存 
3. 发送ACK到生产者

优缺点：

⚫ 同步刷盘：安全性高，效率低，速度慢（适用于对数据安全要求较高的业务）

⚫ 异步刷盘：安全性低，效率高，速度快（适用于对数据处理速度要求较高的业务）

怎么操作：配置文件修改参数

```
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
```

## 1.7RocketMQ高级特性（高可用与主从复制）

####  1.7.1 怎么保证高可用

⚫ nameserver 

​	◆ 无状态+全服务器注册 

⚫ 消息服务器 

​	◆ 主从架构（2M-2S）

⚫ 消息生产 

​	◆ 生产者将相同的topic绑定到多个group组，保障master挂掉后，其他master仍可正常进行消 息接收 

⚫ 消息消费

​	 ◆ RocketMQ自身会根据master的压力确认是否由master承担消息读取的功能，当master繁忙 时候，自动切换由slave承担数据读取的工作

#### 1.7.2 主从数据复制

⚫ 同步复制

 ◆ master接到消息后，先复制到slave，然后反馈给生产者写操作成功

 ◆ 优点：数据安全，不丢数据，出现故障容易恢复 

 ◆ 缺点：影响数据吞吐量，整体性能低 

⚫ 异步复制 

 ◆ master接到消息后，立即返回给生产者写操作成功，当消息达到一定量后再异步复制到slave

 ◆ 优点：数据吞吐量大，操作延迟低，性能高 

 ◆ 缺点：数据不安全，会出现数据丢失的现象，一旦master出现故障，从上次数据同步到故障时间的数据将丢失 

⚫ 配置方式

```
 #Broker 的角色
#- ASYNC_MASTER 异步复制Master 
#- SYNC_MASTER 同步双写Master
#- SLAVE 从服务器配置
brokerRole=SYNC_MASTER
```

## 1.8 RocketMQ高级特性（负载均衡）

⚫ Producer负载均衡

​		 ◆ 内部实现了不同broker集群中对同一topic对应消息队列的负载均衡

 ⚫ Consumer负载均衡

​		 ◆ 平均分配

​		 ◆ 循环平均分配

⚫ 广播模式，不参与负载均衡

## 1.9 RocketMQ高级特性（消息重试，死信队列）

#### 1.9.1 当消息消费后未正常返回消费成功的信息将启动消息重试机制 

⚫ 消息重试机制

- [x]  顺序消息 

​		 ◆ 当消费者消费消息失败后，RocketMQ会自动进行消息重试（每次间隔时间为 1 秒）

​		  注意：应用会出现消息消费被阻塞的情况，因此，要对顺序消息的消费情况进行监控，避免阻塞现象的发

- [x] 无序消息

   ◆无序消息包括普通消息、定时消息、延时消息、事务消息

   ◆无序消息重试仅适用于负载均衡（集群）模型下的消息消费，不适用于广播模式下的消息消费

   ◆ 为保障无序消息的消费，MQ设定了合理的消息重试间隔时长，默认重试16次

#### 1.9.2 死信队列

 ⚫ 当消息消费重试到达了指定次数（默认16次）后，MQ将无法被正常消费的消息称为死信消息（Dead-Letter Message）

 ⚫ 死信消息不会被直接抛弃，而是保存到了一个全新的队列中，该队列称为死信队列（Dead-Letter Queue）

 ⚫ 死信队列特征 

​	◆ 归属某一个组（Gourp Id），而不归属Topic，也不归属消费者 

​	◆ 一个死信队列中可以包含同一个组下的多个Topic中的死信消息 

​	◆ 死信队列不会进行默认初始化，当第一个死信出现后，此队列首次初始化

 ⚫ 死信队列中消息特征 

​	◆ 不会被再次重复消费 

​	◆ 死信队列中的消息有效期为3天，达到时限后将被清除

 ⚫死信处理

​	 在监控平台中，通过查找死信，获取死信的messageId，然后通过id对死信进行精准消费





## 2.0 RocketMQ高级特性（消息幂等）

#### 2.0.1 消息重复消费原因 

⚫ 消息重试机制

◆ 生产者发送了重复的消息

 	◼ 网络闪断

​	 ◼ 生产者宕机 

◆ 消息服务器投递了重复的消息 

​	◼ 网络闪断

 ◆ 动态的负载均衡过程

​	 ◼ 网络闪断/抖动

 	◼ broker重启 

​	 ◼ 订阅方应用重启（消费者）

​	 ◼ 客户端扩容

 	◼ 客户端缩容

#### 2.0.2 消息幂等

⚫ 对同一条消息，无论消费多少次，结果保持一致，称为消息幂等性 

⚫ 解决方案 

​	◆ 使用业务id作为消息的key 

​	◆ 在消费消息时，客户端对key做判定，未使用过放行，使用过抛弃

 ⚫ 注意：messageId由RocketMQ产生，messageId并不具有唯一性，不能作用幂等判定条件