# Redis高级

### 学习目标：

​		1）过期数据，删除过期数据的几种方式。

​		2）主从复制，哨兵，集群动手做一做，概念性的知识看一看。

​        3）缓存预热，缓存雪崩，缓存击穿，缓存穿透知道导致问题的原因，如何解决即可，面试时多看看。

​        4）性能检测工具玩一玩，知道有哪些性能指标即可，感兴趣多看看。





### 1. 过期数据 (概念)

#### 1.1 Redis中的数据特征

⚫ Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过**TTL**指令获取其状态 

​		⚫ XX ：具有时效性的数据 

​		⚫ -1 ：永久有效的数据 

​		⚫ -2 ：已经过期的数据 或 被删除的数据 或 未定义的数据

**思考:** 过期的数据真的立马删除了吗？

**答：** 没有，redis 会维护一个hash数据结构，数据地址作为key 过期时间为value，当删除策略触发时，再对过期的数据进行删除。



**问：**数据删除策略有哪些？

1. 定时删除 2. 惰性删除 3. 定期删除



#### 1.2 定时删除

​	⚫ 创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作 

​	⚫ 优点：节约内存，到时就删除，快速释放掉不必要的内存占用 

​	⚫ 缺点：**CPU压力很大**，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量 

​	⚫ 总结：用处理器性能换取存储空间（拿时间换空间）



#### 1.3 惰性删除

​	⚫ 数据到达过期时间，不做处理。等下次客户端访问该数据时 

​			⚫ 如果未过期，返回数据 

​			⚫ 发现已过期，删除，返回不存在 

​	⚫ 优点：节约CPU性能，发现必须删除的时候才删除 

​	⚫ 缺点：内存压力很大，出现长期占用内存的数据 

​	⚫ 总结：用存储空间换取处理器性能（拿空间换时间）



#### 1.4 定期删除

​	⚫ Redis启动服务器初始化时，读取配置server.hz的值，默认为10（redis.conf/ hz 10）。 

​	⚫ 每秒钟执行server.hz次（100ms触发一次）serverCron() --> databasesCron()--> activeExpireCycle() 
​	⚫ activeExpireCycle()对每个expires[*]逐一进行检测，每次执行250ms/server.hz = 25ms。<font color=red>总结: 默认hz 10情况下: 每100ms触发一次，每次执行25ms.</font>

​	⚫ 对某个expires[*]检测时，随机挑选W个key检测 

​				⚫ 如果key超时，删除key 

​				⚫ 如果一轮中删除的key的数量>W*25%，循环该过程 *

​				⚫ 如果一轮中删除的key的数量≤W*25%，检查下一个expires[*]，0-15循环 

​				⚫ W取值=ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP属性值

⚫ 参数current_db用于记录activeExpireCycle() 进入哪个expires[*]执行 

⚫ 如果activeExpireCycle()执行时间到期，下次从current_db继续向下执行

----------------------------------------------------------------------------------------------

⚫ 周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度 

⚫ 特点1：CPU性能占用设置有峰值，检测频度可自定义设置 

⚫ 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理 

⚫ 总结：周期性抽查存储空间（ 不用粗暴抢夺cpu，也减少内存冗余 ）



#### 1.4 三种方案对比

1. 定时删除 
  - 节约内存，无占用 
  - 不分时段占用CPU资源，频度高
  - 拿时间换空间 

2. 惰性删除 
   - 内存占用严重 
   - 延时执行，CPU利用率高 
   - 拿空间换时间 
3. 定期删除
   - 内存定期随机清理
   - 每秒花费固定的CPU资源维护内存
   - 随机抽查，重点抽查



<font color=red>**总结：**定时删除，频率太高，太粗暴，一般高并发设计没人敢用，所以redis用了定期+惰性删除，比较温柔； </font>        



#### 1.5 淘汰策略

##### 新数据进入检测

当新数据进入redis时，如果内存不足怎么办将会触发淘汰策略。

- Redis使用内存存储数据，在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足。如果内存不满足新加入数据的最低存储要求，redis要临时删除一些数据为当前指令清理存储空间。清理数据的策略称为逐出算法。 

- 注意：逐出数据的过程不是100%能够清理出足够的可使用的内存空间，如果不成功则反复执行。当对所有数据尝试完毕， 如不能达到内存清理的要求，将出现错误信息。

  ```
  (error) OOM command not allowed when used memory >'maxmemory'
  ```



##### 影响数据淘汰的相关配置

-  最大可使用内存，即占用物理内存的比例，默认值为0，表示不限制。生产环境中根据需求设定，通常设置在50%以上

  ```
  maxmemory ?mb
  ```

- 每次选取待删除数据的个数，采用随机获取数据的方式作为待检测删除数据。

  ```
  maxmemory-samples count
  ```

- 对数据进行删除的选择策略

  ```
  maxmemory-policy policy  #取值如下
  ```

  - ⚫ 检测易失数据（可能会过期的数据集server.db[i].expires ） 

    ① volatile-lru：挑选最近最少使用的数据淘汰 

    ② volatile-lfu：挑选最近使用次数最少的数据淘汰 

    ③ volatile-ttl：挑选将要过期的数据淘汰 

    ④ volatile-random：任意选择数据淘汰 

  - ⚫ 检测全库数据（所有数据集server.db[i].dict ） 

    ⑤ allkeys-lru：挑选最近最少使用的数据淘汰 

    ⑥ allkeys-lfu：挑选最近使用次数最少的数据淘汰 

    ⑦ allkeys-random：任意选择数据淘汰 

  - ⚫ 放弃数据驱逐 

    ⑧ no-enviction（驱逐）：禁止驱逐数据（redis4.0/5.0中默认策略），会引发错误OOM（Out Of Memory）

  

- redis提供了info命令可以监控redis使用信息，比如：查询缓存命中hit 和 错过miss 的次数，架构师会根据业务需求调优Redis配置 。

<font color=red>总结: 我们的数据应该我们来控制删除，通常在企业环境中，内存、磁盘、CPU使用量超过阈值时，会进行报警, 更好的办法是加服务器节点，来提升整体的存储容量，在企业环境中，删除数据是很危险的操作，建议采用默认值。</font>



### 2. 主从复制

redis server可以部署在多台机器上，我们选择其中一台作为主节点，其他未从节点，通过redis.conf 配置，可以实现从节点 同步 主节点数据，专业名词：**“主从复制”**。



#### 2.1 主从复制简介 （概念）

##### 高可用

​	⚫ 高并发 : 描述流量大

​	⚫ 高性能 : 描述系统性能好

​	⚫ 高可用

一般企业要求软件服务至少能够达到四个9（99.99%），就可以称之为高可用,  计算如下 : 

```shell
#年度可用性：90%折合计算，365天x90% =328.5天。全年故障36.5天，取最容易看的单位超过1个月

#年度可用性：99%折合计算，365天x99% =361.35天。全年故障3.65天，取最容易看的单位87.6小时

#年度可用性：99.9%折合计算，365天x99.9% =364.635天。全年故障0.365天，取最容易看的单位8.76小时

#年度可用性：99.99%折合计算，365天x99.99% =364.9635天。全年故障0.0365天，取最容易看的单位52.56分钟

#年度可用性：99.999%折合计算，365天x99.999% =364.99635天。全年故障0.000365天，取最容易看的单位5.256分钟
```

|           **描述**           | **通俗叫法** | **可用性级别** | **年度停机时间** |
| :--------------------------: | :----------: | :------------: | :--------------: |
|          基本可用性          |     2个9     |      99%       |     87.6小时     |
|          较高可用性          |     3个9     |     99.9%      |     8.8小时      |
| 具有故障自动恢复能力的可用性 |     4个9     |     99.99%     |      53分钟      |
|          极高可用性          |     5个9     |    99.999%     |      5分钟       |



###### 思考：

**你的“单机Redis”是否高可用？**

- 问题1.机器故障 

  ◼ 现象：硬盘故障、系统崩溃，数据丢失，系统处于不可用状态。

  ◼ 结论：基本上会放弃使用redis.

- 问题2.容量瓶颈 

  ◼ 现象：内存不足，从16G升级到64G，从64G升级到128G，无限升级内存，停机加内存这段时间，又是不可以用状态。

  ◼ 本质：穷，硬件条件跟不上

  ◼ 结论：放弃使用redis 



###### 解决思路

​        Redis 为了解决这个单一节点的问题，也会把数据复制多个副本部署到其他节点上进行复制，实现 Redis的高可用，实现对数据的冗余备份，从而保证数据和服务的高可用。 





##### 主从复制的概念

- **概念**：主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave),数据的复制是单向的，只能由主节点到从节点。
- **特征**：默认情况下，redis是不分主从的，一旦对多个redis配置了主从，一个master可以拥有多个slave也可以没有slave，一个slave只对应一个master 

​	⚫ 职责划分： 

​       **方式一：**

​			◆ master: 

​				◼ 只负责写数据

​				◼ 执行写操作时，将出现变化的数据自动同步到slave 

​			◆ slave: 

​				◼ 只负责读数据

​				◼ 同时在线时刻备份master同步过来的数据

​        **方式二：**

​      	   ◆ master:  负责读写

​			 ◆ slave: 只做数据的备份，当master出现故障时，slave在提供读写服务。



##### 主从复制的作用

​	⚫ 读写分离：master写、slave读，提高服务器的读写负载能力 

​	⚫ 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数 量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量 

​	⚫ 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复 

​	⚫ 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式 

​	⚫ 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案





#### 2.2 主从复制工作流程

​	⚫ 建立连接阶段

​	⚫ 数据同步阶段 

​	⚫ 命令传播阶段 

##### 建立连接阶段（动手）

- 步骤如下：

  ​	① slave基于连接配置，发送指令给master，确认master是否在线。

  ​	②master接收到指令，响应给slave，证明自己在线。

  ​	③slave保存master的IP与端口 

  ​	④slave使用master的IP与端口创建连接socket 

  ​    ⑤slave周期性发送命令：ping，检测master时刻在线。

  ​    ⑥master每次接收到ping, 都要做出响应pong, 来证明自己活着。

  ​    ⑦slave发送指令：auth password，请求master验证密码。

  ​    ⑧master验证授权，响应给salve结果

  ​    ⑨密码正确，slave发送指令：replconf listening-port <port-number> ，告知master同步数据时用哪个端口。

  ​    ⑩master保存slave的端口号，需要同步数据给salve时，使用此端口号。

- **slave连接如何建立**：

  ```shell
  ################## 建立连接操作
  #方式一：客户端发送命令
  slaveof masterip masterport
  
  #方式二：启动服务器参数
  redis-server --slaveof masterip masterport
  
  #方式三(推荐)：slave redis.conf配置
  #slave启动时，自动跟master建立同步连接
  slaveof masterip masterport
  
  
  ################## 断开连接操作
  # slave客户端执行命令
  slaveof no one
  
  ################## 授权访问
  # master  redis.conf 配置文件设置密码
  requirepass password
  
  # slave redis.conf 配置文件设置master密码
  masterauth password
  
  ```

- slave系统信息 

  ​	◆ master_link_down_since_seconds 

  ​	◆ masterhost &  masterport 

- master系统信息 

  ​	◆ slave_listening_port (多个)



##### 数据同步阶段（概念）

- **步骤如下：**

  <img src="https://img-blog.csdnimg.cn/20201109155258408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70" alt="img" style="zoom: 80%;" />



- **同步阶段master说明**

​            1）如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执行 

​            2）复制缓冲区大小设定不合理，会导致数据溢出。如进行全量复制周期太长，进行部分复制时发现数据已 经存在丢失的情况，必须进行第二次全量复制，致使slave陷入死循环状态。

​            3）master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下50%-30%的内存用于执 行bgsave命令和创建复制缓冲区

```shell
# master/redis.conf 
# 设置同步缓冲区大小, 根据3）中的百分比建议设置即可
repl-backlog-size ?mb
```



- **同步阶段slave说明**

  ​	 1）为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务

```shell
# slave/redis.conf 
# yes，主从复制中，从服务器可以响应客户端请求
# no，主从复制中，从服务器将阻塞所有请求，有客户端请求时返回“SYNC with master in progress”;
slave-serve-stale-data yes|no
```

​          2）数据同步阶段，master已客户端的身份，通过slave告知的同步 端口，主动将数据发送给slave。

​          3）多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果 master带宽不足，因此数据同步需要根据业务需求，适量错峰 

​		  4）slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是master，也是 slave。注意：使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟 较大，数据一致性变差，应谨慎选择。



##### 命令传播阶段（概念）

​	该阶段master每次接收到客户端传来对数据有变更的指令，都会通过**“命令传播程序”**，分发给所有的从节点来完成数据同步。	



###### offset 偏移量

​	◆ **概念**：master和slave都会维护一个偏移量, 用于标识主从复制的现状；（可使用[info replication]命令查看偏移量）

​	◆ **master端**：每次接收到java客户端发来命令（字节数据），记录当前数据offset。

​	◆ **slave端**：每次master同步数据时，slave都能收到当前数据offset，并且slave存在心跳机制，每1秒都发送指令给master上报自己当前的offset值，“REPLCONF ACK {offset} ”。

​    ◆ **作用**：master会根据salve上报的offset和自己当前记录的offset做比对：

​           1）先检验，slave的offset在不在master offset范围内

​                   ◆ 假如：slave offset=20，master offset范围 [1-10]，则slave的offset非当前master提供的，进行**全量复制**，slave会将之前数据全部清理掉。

一致：则没有数据变更不同步数据  

​           2）通过检验，一致，不同步

​           3）通过检验，不一致，部分复制

​				  ◆ 假如：slave offset=5，master offset范围 [1-10]，则slave的offset在当前master offset范围内，进行**部分复制**，master会将复制缓冲区中的数据发送给slave进行数据的同步。



###### 服务器运行ID（runid）

​	◆ 每个redis服务每次启动时，都会随机生成一个40的id来代表自己，可通过[info server]命令查询run_id。

​	◆ 在初次主从同步时 (**全量复制**) ，master会将自己的**runid**和**offset**以及**RDB数据文件**一起发送给slave，salve记录自己的master [runid]。

​	◆ 当slave断线重连时，从节点会将这个runid发送给主节点；主节点根据runid判断能否进行部分复制：

​           1）如果从节点保存的runid与主节点现在的runid相同，说明主从节点之前同步过，主节点会继续尝试使用部分复制  (到底能不能部分复制还要看offset和复制积压缓冲区的情况)；

​           2）如果从节点保存的runid与主节点现在的runid不同，说明从节点在断线前同步的Redis节点并不是当前的主节点，只能进行全量复制。

```shell
#命令info server
run_id:409b6e9ea2e5c32958de8f365711598c98489f13
```



###### 复制缓冲区

- **概念**：复制缓冲区，又名复制积压缓冲区，是一个固定长度的FIFO的队列，大小由配置参数[repl-backlog]指定，这个缓冲区只存在master，用于备份最近master同步slave的数据。 

  ​		◆ 默认存储空间大小是1M 

  ​		◆ 当入队元素的数量大于队列长度时，最先入队的元素会被弹出，而新元素会被放入队列

   	   ◆ 命令传播阶段，master除了将命令同步给slave外还会在这个缓冲区中备份，而且还会存储每个命令（实际是每个字节）对用的offset值。

- **作用** ：

  - 通过offset区分不同的slave当前数据传播的差异 
  - master记录已发送的信息对应的offset
  - slave记录已接收的信息对应的offset

![img](https://img-blog.csdnimg.cn/20201109170516424.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)





##### 完整版本 +心跳机制

###### 流程图一

<img src="https://img-blog.csdnimg.cn/20201109155258408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70" alt="img" style="zoom: 80%;" />



![img](https://img-blog.csdnimg.cn/2020110917050514.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



###### 流程图二

- 下图建立连接步骤9存在描述错误，正确：“master 保存 slave IP 和 Port”。



![img](https://img-blog.csdnimg.cn/20201109181200244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE2NjgxMjc5,size_16,color_FFFFFF,t_70)



###### 心跳机制

- 进入命令传播阶段候，master与slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线 

- **master心跳**： 

  ​	◆ 内部指令：PING 

  ​	◆ 周期：由repl-ping-slave-period决定，默认10秒 

  ​	◆ 作用：判断slave是否在线 

  ​	◆ 查询：INFO replication 获取slave最后一次连接时间间隔，lag项维持在0或1视为正常 

- **slave心跳任务** 

  ​	◆ 内部指令：REPLCONF ACK {offset} 

  ​	◆ 周期：1秒 

  ​	◆ 作用1：汇报slave自己的复制偏移量，获取最新的数据变更指令 

  ​	◆ 作用2：判断master是否在线

- **注意事项：**

  -  当slave多数掉线，或延迟过高时，master为保障数据稳定性，将拒绝所有信息同步操作

    ```shell
    #slave数量少于2个 或者 所有slave的延迟都大于等于8秒时，强制关闭master写功能，停止数据同步 
    min-slaves-to-write 2
    min-slaves-max-lag 8
    ```

  - slave数量由slave发送REPLCONF ACK {offset} 命令做确认 

  - slave延迟由slave发送REPLCONF ACK {offset} 命令做确认



#### 2.3 主从复制常见问题

##### 全量复制

 redis 2.8 之前使用 sync 只能执行全量复制，2.8 之后同时支持全量复制和部分复制。

###### 问题一 

- 伴随着系统的运行，master的数据量会越来越大，一旦master重启，runid将发生变化，会导致全部slave的全量复制操作 

- 内部优化调整方案： 

  1. 想要关闭master时，执行命令 shutdown save，进行RDB持久化, 将runid与offset保存到RDB文件中 。

  2. master重启后加载RDB文件，恢复runid的值，此时runid未发生改变。

     ​	◼ 通过info命令可以查看该信息 

  3. 作用：本机保存上次runid，重启后恢复该值，使所有slave认为还是之前的masterid, 避免因重启导致runid不同，进行耗时的全量复制。

-  **最优方法**：对于这类问题，我们做一些故障转移的手段，例如master发生故障宕掉，我们选举一台slave晋升为master（哨兵或集群） ，下面会讲。



###### 问题二

- 问题现象：网络环境不佳，出现网络中断，slave不提供服务 

- 问题原因：复制缓冲区过小，断网时间过长，导致断网后slave的offset越界，触发全量复制 。

- 最终结果：slave反复进行全量复制 

- 解决方案：修改复制缓冲区大小

  ```shell
  # master/redis.conf
  repl-backlog-size ?mb
  ```

-  建议设置如下： 
  1. 测算从master到slave的重连平均时长second 
  2. 获取master平均每秒产生写命令数据总量write_size_per_second 
  3. 最优复制缓冲区空间 = 2 * second * write_size_per_second

<font color=red>详细解释:</font>  网络中断会导致slave无法接收到master同步的offset，通过刚才学习，我们知道：“复制积压缓冲区” 满了后，旧数据会移除，新数据进队列，长时间的网络中断会导致slave的offset已经不再master缓冲区的offset范围内，当slave网络恢复后，发送REPLCONF ACK {offset} 命令到master，此时master会做全量复制。



###### 问题三

 第一次建立连接后，主从同步不可以避免“全量复制”，那我们可以选在集群低峰的时间（凌晨）进行slave的挂载。



##### 频繁的网络中断

###### 问题一

- 问题现象：master的CPU占用过高 或 slave频繁断开连接 

- 问题原因 

  ​	◆ slave每1秒发送REPLCONF ACK命令到master 

  ​	◆ 当slave接到了慢查询时（keys * ，hgetall等），会大量占用CPU性能 

  ​	◆ master每1秒调用复制定时函数replicationCron()，比对slave发现长时间没有进行响应 

- 最终结果：master各种资源（输出缓冲区、带宽、连接等）被严重占用 

- 解决方案：

  ​    1）通过设置合理的超时时间，确认是否释放slave。

  ```shell
  #slave/redis.conf
  #该参数定义了超时时间的阈值（默认60秒）, 超过该值，slave断开主从连接
  repl-timeout seconds
  ```

  ​    2）抛弃读写分离，slave不对java客户端开放，只提供在线备份数据功能。



###### 问题二

- 问题现象：slave与master连接断开 

- 问题原因 

  ​	◆ slave 向 master发送ping指令频度较低 

  ​	◆ 对master响应设定超时时间较短 

  ​	◆ ping指令在网络中存在丢包 

- 解决方案：提高ping指令发送的频度

  ```shell
  #slave/redis.conf
  #Slave按照repl-ping-slave-period的间隔（默认10秒）, 向Master发送ping
  #repl-timeout 至少是ping指令频度的5到10倍，否则slave很容易判定为超时
  repl-ping-slave-period  seconds
  ```

  

##### 数据不一致

- 问题现象：多个slave获取相同数据不同步
- 问题原因：网络信息不同步，数据发送有延迟 

- 解决方案 

  ​	◆ 优化主从间的网络环境，通常放置在同一个机房部署，如使用阿里云等云服务器时要注意此现象 

  ​	◆ 监控主从节点延迟（通过offset）判断，如果slave延迟过大，人工干预，暂时屏蔽java客户端对该slave的数据访问

  ```shell
  # slave/redis.conf 
  # yes，主从复制中，从服务器可以响应客户端请求
  # no，主从复制中，从服务器将阻塞所有请求，有客户端请求时返回“SYNC with master in progress”;
  slave-serve-stale-data yes|no
  ```



### 3. 哨兵模式

#### 3.1 哨兵简介 

##### 哨兵

哨兵(sentinel) 是一个分布式系统，跟redis server一样需要我们人为启动才可工作，用于对主从结构中的每台redis服务器进行监控，当出现故障时通过投票机制选择新的master并 将所有slave连接到新的master。



#### 3.2 哨兵的作用

##### 保障redis高可用

- 监控

  ◆ 不断的检查master和slave是否正常运行 

  ◆ master存活检测、master与slave运行情况检测 

- 通知（提醒）：当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知

- 自动故障转移：断开master与slave连接，选取一个slave作为master，将其他slave连接新的master，并告知客户端新的服 务器地址

- 当旧的master起死回生后，会变成新master的slave从节点

  

<font color=red>注意：哨兵也是一台redis提供的server，需要我们自行启动，才可以提供监管工作，通常哨兵的数量配置为奇数，要不然选举投票时会出现1:1，造成选不出来新master问题。</font>



#### 3.3 启用哨兵模式（动手）

##### 配置哨兵

- 配置一主两从三哨兵结构 

- 三个哨兵（配置相同，端口不同），参看sentinel.conf 

- 启动哨兵命令

  ```shell
  #window 启动哨兵命令
  $ redis-server.exe  sentinel-26379.conf  --sentinel 
  
  #linux 启动哨兵命令 
  $ redis-sentinel  sentinel-26380.conf 
  
  ```



##### redis server

###### redis-master-6401.conf

```shell
port 6401
dir "/redis/data"
dbfilename "dump-6401.rdb"
```

###### redis-slave-6402.conf

```shell
port 6402
dir "/redis/data"
dbfilename "dump-6402.rdb"
slaveof 127.0.0.1 6401
```

###### redis-slave-6403.conf

```shell
port 6403
dir "/redis/data"
dbfilename "dump-6403.rdb"
slaveof 127.0.0.1 6401
```



##### redis sentinel server

- 生产环境下，127.0.0.1 要换成 外网ip。
- 三个哨兵监听都是主节点ip, 并且哨兵有办法获取从节点ip。

###### sentinel-26401

```shell
port 26401
dir "/redis/data"
#设置哨兵监听的主服务器信息，sentinel_number表示参与投票的哨兵数量
sentinel monitor mymaster 127.0.0.1 6401 2
#设置判定服务器宕机时长，该设置控制是否进行主从切换
sentinel down-after-milliseconds mymaster 5000
#设置故障切换的最大超时时长
sentinel failover-timeout mymaster 20000
#指定了在执行故障转移时, 最多可以有多少个从服务器同时对新的主服务器进行同步
sentinel parallel-sync mymaster 1
#设置master和slaves验证密码
sentinel auth-pass mymaster 123456 
# 安全，避免脚本重置，默认值yes
sentinel deny-scripts-reconfig yes
```

###### sentinel-26402.conf

```shell
port 26402
dir "/redis/data"
sentinel monitor mymaster 127.0.0.1 6401 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 20000
sentinel parallel-sync mymaster 1
sentinel auth-pass mymaster 123456 
sentinel deny-scripts-reconfig yes
```

###### sentinel-26403.conf

```shell
port 26403
dir "/redis/data"
sentinel monitor mymaster 127.0.0.1 6401 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 20000
sentinel parallel-sync mymaster 1
sentinel auth-pass mymaster 123456 
sentinel deny-scripts-reconfig yes
```



#### 3.4 哨兵演示

- 重新选举时，哨兵回去修改所有节点配置信息
- 这些配置信息都是基于内存来完成修改，磁盘配置文件并无变化

- 先停掉master，观察哨兵打印日志，可以看到其中一个slave升级为master
- 将原来的master再次启动，可以发现它已经变成新master的slave节点。





#### 3.5 哨兵工作原理

##### 监控&通知阶段

-  每个哨兵节点每10秒会向主节点和从节点发送info命令获取最新节点信息

  - 这样当有新的slave上线时，哨兵们也可以感知的到

-  每个哨兵节点每隔2秒发送自己对于主节点的判断以及自己节点的信息，给其他哨兵。

  - 当所有哨兵都确定了redis master挂了后，才会选举一个哨兵的领导者，来完成redis 的故障转移。

-  每隔1秒每个哨兵会向主节点、从节点及其余哨兵节点发送一次ping命令做一次心跳检测，这个也是哨兵用来判断redis节点是否正常的重要依据。

  - 如果从节点不正常，不会参与升级为主节点。
  - 如果redis主节点挂了，哨兵会从slave中选举一个新的主节点。
  - 哨兵在进行redis故障转移时，要从三个哨兵中选择一个领导者，来完成此操作，领导者是需要投票来确定的，如果某个哨兵挂了，不会参与投票。

  

##### 故障转移阶段

-  **主观&客观下线**：刚我知道知道哨兵节点每隔1秒对主节点和从节点、其它哨兵节点发送ping做心跳检测确认master是否存活，如果master在“一定时间范围”内不回应PONG 或者是回复了一个错误消息，那么这个sentinel会**主观地**(单方面地)认为这个master已经不可用了，会要求其他sentinel确认该节点是否丢失，当超过quorum（法定人数）个数，则认为是**客观下线**，的确不可用了，开始进行故障转移； 

- **领导者哨兵选举流程**：

  ​	1）每个在线的哨兵节点都可以成为领导者，当它确认（比如哨兵3）主节点下线时，会向其它哨兵发is-master-down-by-addr命令，征求判断并要求将自己设置为领导者，由领导者处理故障转移；

  ​	2）当其它哨兵收到此命令时，可以同意或者拒绝它成为领导者；

  ​	3）如果哨兵3发现自己在选举的票数大于等于num(sentinels)/2+1时，将成为领导者，如果没有超过，继续选举…………

- **故障转移细节操作：**

  - 由Sentinel领导者节点执行故障转移，过程和主从复制一样，但是自动执行 

  - 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点

  - 选择salve-priority从节点优先级最高（redis.conf）

  - 选择复制偏移量最大，指复制数据最完整的从节点

  - 将slave-1脱离原从节点，升级主节点

  - 将从节点slave-2指向新的主节点

  - 通知客户端(java)主节点已更换

  - 将原主节点（oldMaster）变成从节点，指向新的主节点

    ​    

#### 3.6 javaAPI操作哨兵（扩展）（动手）

##### pom.xml

```xml
<!-- redis Spring 基于注解配置 --> 
<dependency>  
    <groupId>org.springframework.data</groupId>  
    <artifactId>spring-data-redis</artifactId>  
    <version>1.8.1.RELEASE</version>  
</dependency>
<!-- Redis客户端 -->
<dependency>
    <groupId>redis.clients</groupId> 
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

##### RedisSentinelConfig.java

```java
package com.itheima.config;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisSentinelConfiguration;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.HashSet;
import java.util.Set;

@Configuration
@PropertySource("classpath:redis.properties")
public class RedisSentinelConfig {

    @Value("${redis.host}")
    private String hostName;

    @Value("${redis.port}")
    private Integer port;

//    @Value("${redis.password}")
//    private String password;

    @Value("${redis.maxActive}")
    private Integer maxActive;
    @Value("${redis.minIdle}")
    private Integer minIdle;
    @Value("${redis.maxIdle}")
    private Integer maxIdle;
    @Value("${redis.maxWait}")
    private Integer maxWait;

    @Value("${redis.sentinel.one}")
    private String sentinelOne;
    @Value("${redis.sentinel.two}")
    private String sentinelTwo;
    @Value("${redis.sentinel.three}")
    private String sentinelThree;
    @Value("${redis.sentinel.master.name}")
    private String masterName;


    @Bean
    //配置RedisTemplate
    public RedisTemplate createRedisTemplate(RedisConnectionFactory redisConnectionFactory){
        //1.创建对象
        RedisTemplate redisTemplate = new RedisTemplate();
        //2.设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //3.设置redis生成的key的序列化器，对key编码进行处理
        RedisSerializer keyStringSerializer = new StringRedisSerializer();
        RedisSerializer valueStringSerializer = new GenericJackson2JsonRedisSerializer();
        /**
         * 如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储
         * ，那么会提示错误User can't cast to String！！
         */
        redisTemplate.setKeySerializer(keyStringSerializer);
        redisTemplate.setValueSerializer(valueStringSerializer);
        redisTemplate.setHashKeySerializer(keyStringSerializer);
        redisTemplate.setHashValueSerializer(valueStringSerializer);
        //4.返回
        return redisTemplate;
    }

    @Bean
    //配置Redis连接工厂
    public RedisConnectionFactory createRedisConnectionFactory(@Autowired RedisSentinelConfiguration redisSentinelConfiguration, GenericObjectPoolConfig genericObjectPoolConfig){

        //1.创建配置构建器，它是基于池的思想管理Jedis连接的
        JedisClientConfiguration.JedisPoolingClientConfigurationBuilder builder = (JedisClientConfiguration.JedisPoolingClientConfigurationBuilder)JedisClientConfiguration.builder();
        //2.设置池的配置信息对象
        builder.poolConfig(genericObjectPoolConfig);
        //3.创建Jedis连接工厂
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisSentinelConfiguration, builder.build());

        //4.返回
        return jedisConnectionFactory;
    }

    @Bean
    //配置spring提供的Redis连接池信息
    public GenericObjectPoolConfig createGenericObjectPoolConfig(){
        //1.创建Jedis连接池的配置对象
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        //2.设置连接池信息
        genericObjectPoolConfig.setMaxTotal(maxActive);
        genericObjectPoolConfig.setMinIdle(minIdle);
        genericObjectPoolConfig.setMaxIdle(maxIdle);
        genericObjectPoolConfig.setMaxWaitMillis(maxWait);
        //3.返回
        return genericObjectPoolConfig;
    }


    @Bean
    //配置Redis标准连接配置对象
    public RedisSentinelConfiguration createRedisSentinelConfiguration(){
        Set<String> setRedisNode = new HashSet<String>();
        setRedisNode.add(sentinelOne);
        setRedisNode.add(sentinelTwo);
        setRedisNode.add(sentinelThree);
/*
        setRedisNode.add("121.42.157.181:26379");
        setRedisNode.add("121.42.157.181:26380");
        setRedisNode.add("121.42.157.181:26381");
*/
        RedisSentinelConfiguration redisSentinelConfiguration = new RedisSentinelConfiguration(masterName, setRedisNode);
//        RedisSentinelConfiguration redisSentinelConfiguration = new RedisSentinelConfiguration("mymaster", setRedisNode);
        return redisSentinelConfiguration;
    }


}
```

##### redis.properties

```properties
# redis服务器主机地址
redis.host=192.168.40.130
#redis服务器主机端口
redis.port=6378

#redis服务器登录密码
#redis.password=itheima

#最大活动连接
redis.maxActive=20
#最大空闲连接
redis.maxIdle=10
#最小空闲连接
redis.minIdle=0
#最大等待时间
redis.maxWait=-1

#哨兵配置
#哨兵的ip:端口
redis.sentinel.one=192.168.23.129:26401
redis.sentinel.two=192.168.23.129:26402
redis.sentinel.three=192.168.23.129:26403
#要跟sentinel.conf配置中 “mymaster”一致
redis.sentinel.master.name=mymaster
```

##### RedisTest.java

```java
package com.itheima.service;

import com.itheima.config.RedisConfig;
import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import redis.clients.jedis.Jedis;

//设定spring专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
//设定加载的spring上下文对应的配置
@ContextConfiguration(classes = RedisConfig.class)
public class RedisTest {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void changeMoney() {
        //等同于redis中set account:id:1 100
        redisTemplate.opsForValue().set("account:id:"+3, 200);
    }

    @Test
    public void findMondyById() {
        //等同于redis中get account:id:1
        Object money = redisTemplate.opsForValue().get("account:id:" + 3);
        System.out.println(money);
    }
}
//        redisTemplate.type()
//        redisTemplate.persist()
//        redisTemplate.move()
//        redisTemplate.hasKey()
//        redisTemplate.getExpire()
//        redisTemplate.expire()
//        redisTemplate.delete()
//        redisTemplate.rename();
//
//        redisTemplate.opsForValue().;
//        redisTemplate.opsForHash().;
//        redisTemplate.opsForList().;
//        redisTemplate.opsForSet().;
//        redisTemplate.opsForZSet();
//
//
//        redisTemplate.boundValueOps().;
//
//        redisTemplate.slaveOf();
//        redisTemplate.slaveOfNoOne();
//
//        redisTemplate.opsForCluster()

```





### 4. 集群模式

#### 4.1 集群简介 

##### 现状问题

​	⚫ redis提供的服务OPS可以达到10万/秒，当前业务OPS已经达到10万/秒 

​	⚫ 内存单机容量达到256G，当前业务需求内存容量1T 

​	⚫ 使用集群的方式可以快速解决上述问题

**问：**哨兵可以解决上面问题吗？

**答：**哨兵是主节点提供客户端读写服务，所以还是一台机器在干活，解决不了问题。



##### 集群架构

​	⚫ 集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，集群中多台主节点都可以对外提供读写服务。

##### 集群作用

​	⚫ 分散单台服务器的访问压力，实现负载均衡 

​	⚫ 分散单台服务器的存储压力，实现可扩展性 

​	⚫ 降低单台服务器宕机带来的业务灾难



#### 4.2 Cluster集群设计

##### 数据存储设计

​	⚫ 通过算法设计，计算出key应该保存的位置 

​	⚫ 将所有的存储空间计划切割成16384个槽，每台主机保存一部分 

​			◆ 注意：每份代表的是一个存储空间，不是一个key的保存空间 

​	⚫ 将key按照计算出的结果放到对应的存储空间

​			◆ 注意：统一key根据此算法，算出来的值永远固定。



##### 集群内部通讯设计

​	⚫ 各个数据库相互通信，保存各个库中槽的编号数据 

​	⚫ 一次命中，直接返回 

​	⚫ 一次未命中，告知具体位置





#### 4.3 Cluster集群搭建 （动手）

- redis cluster不光可以具备哨兵模式下的高可用（自动完成故障迁移），还能对master进行水平扩容，多台master对外提供读写服务，这不仅提高了redis整体的并发量，整体的内存存储容量也大大提升了。
- redis cluster 可以很方便的实现水平扩容，在原有模式下，继续加机器。

##### 搭建方式

​	⚫ 配置服务器（3主3从） 

​	⚫ 建立通信（Meet） 

​	⚫ 分槽（Slot） 

​	⚫ 搭建主从（master-slave）



##### Cluster配置

- 是否启用cluster，加入cluster节点

  ```
  cluster-enabled yes|no
  ```

- cluster配置文件名，该文件属于自动生成，仅用于快速查找文件并查询文件内容

  ```
  cluster-config-file filename
  ```

- 节点服务响应超时时间，用于判定该节点是否下线或切换为从节点

  ```
  cluster-node-timeout milliseconds
  ```

- master连接的slave最小数量

  ```
  cluster-migration-barrier min_slave_number
  ```

  

#####   开始搭建

- --cluster-replicas     n

  ◆ master与slave的数量要匹配，一个master对应n个slave，由最后的参数n决定 

  ◆ master与slave的匹配顺序为第一个master与前n个slave分为一组，形成主从结构

```shell
#安装目录
$ cd /itcast/install/redis-5.0/conf

#制作文件redis-650x.conf
$ vim redis-6501.conf 

port 6501
dir "/redis/data"
dbfilename "dump-6501.rdb"
cluster-enabled yes
cluster-config-file "cluster-6501.conf"
cluster-node-timeout 5000

# 复制5份配置文件  
$ cp redis-6501.conf redis-6502.conf
$ cp redis-6501.conf redis-6503.conf
$ cp redis-6501.conf redis-6504.conf
$ cp redis-6501.conf redis-6505.conf
$ cp redis-6501.conf redis-6506.conf


# 修改端口号  
$ sed -i "s/6501/6502/g" redis-6502.conf
$ sed -i "s/6501/6503/g" redis-6503.conf
$ sed -i "s/6501/6504/g" redis-6504.conf
$ sed -i "s/6501/6505/g" redis-6505.conf
$ sed -i "s/6501/6506/g" redis-6506.conf

# 启动所有节点6次 
$ redis-server redis-6501.conf  
$ redis-server redis-6502.conf  
$ redis-server redis-6503.conf
$ redis-server redis-6504.conf
$ redis-server redis-6505.conf
$ redis-server redis-6506.conf 

# 创建集群 --cluster-replicas 1 表示: 一主一从, 6个节点就是三主三从
# 在不同机器上装集群要配置外网ip, 本地127.0.0.1也可以
# 该步骤会打印出分槽 和 6个节点谁是主, 谁是从的信息
$ redis-cli --cluster create 19:6501 192.168.23.129:6502 192.168.23.129:6503 192.168.23.129:6504 192.168.23.129:6505 192.168.23.129:6506 --cluster-replicas 1
# 等一会 然后输入
yes  

# 检查集群状态
$ redis-cli --cluster check 192.168.23.129:6501  #填写任意节点即可 会带出所有的  


# 客户端连接并查看所有节点信息
$ redis-cli -h 192.168.23.129 -c -p 6501
$ >cluster nodes


```

##### 集群扩容

```shell
################# 添加新的master、slave节点到集群中########
$ cp redis-6501.conf redis-6507.conf
$ cp redis-6501.conf redis-6508.conf


# 修改端口号
$ sed -i "s/6501/6507/g" redis-6507.conf
$ sed -i "s/6501/6508/g" redis-6508.conf

# 启动
$ redis-server redis-6507.conf
$ redis-server redis-6508.conf

# 添加master到当前集群中，连接时可以指定任意现有节点地址与端口
# redis-cli --cluster add-node new-master-host:new-master-port now-host:now-port
$ redis-cli --cluster add-node 192.168.23.129:6507 192.168.23.129:6503


# 添加slave 到 选中的主节点 6507 上
# 6507 masterid 需要自己查看
$ redis-cli --cluster add-node  192.168.23.129:6508  192.168.23.129:6507 --cluster-slave --cluster-master-id masterid


########(选择性操作) 也可以删除刚刚加的节点
# 删除节点，如果删除的节点是master，必须保障其中没有槽slot
# redis-cli --cluster del-node del-slave-host:del-slave-port del-slave-id
# 6508 的del-slave-id 自己查看
$ redis-cli --cluster del-node 192.168.23.129:6508  del-slave-id


###分槽
# 重新分槽，分槽是从具有槽的master中划分一部分给其他master，过程中不创建新的槽
# 需要查看现有槽的masterid, 和没有槽的masterid
# cluster-slots的值一般都是均等分配后的值, slots=16384/4=4096
$ redis-cli --cluster reshard  192.168.23.129:6507  --cluster-from master-id1, master-id2, master-idn --cluster-to new-master-id --cluster-slots 4096


########(可选操作) 重新分配槽
# 从具有槽的master中分配指定数量的槽到另一个master中，常用于清空指定master中的槽
# 将6507-master槽分给其他master
$ redis-cli --cluster reshard 192.168.23.129:6507 --cluster-from master-6507-id --cluster-to target-master-id --cluster-slots slots  --cluster-yes


```

##### 安装常见问题

1.关于启动集群时候出现一直等待 Waiting for the cluster to join 很久都没有反应的问题

```
Redis集群不仅需要开通redis客户端连接的端口，而且需要开通集群总线端口,集群总线端口为redis客户端连接的端口 + 10000,如redis端口为7000,则集群总线端口为17000,因此所有服务器的点需要开通redis的客户端连接端口和集群总线端口
```

2.关于集群连接验证时候连接失败,如图

![img](https://img2018.cnblogs.com/blog/1344247/201903/1344247-20190326095502757-468929776.png)

```
Redis 集群模式下连接需要已-h ip地址方式连接,命令应为./redis-cli -h 192.168.118.110 -c -p 6501
```



#### 4.4 javaAPI操作集群（扩展）（动手）

##### pom.xml

```xml
<!-- redis Spring 基于注解配置 --> 
<dependency>  
    <groupId>org.springframework.data</groupId>  
    <artifactId>spring-data-redis</artifactId>  
    <version>1.8.1.RELEASE</version>  
</dependency>
<!-- Redis客户端 -->
<dependency>
    <groupId>redis.clients</groupId> 
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

##### RedisClusterConfig.java

```java
package com.itheima.config;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.MapPropertySource;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisSentinelConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

@Configuration
@PropertySource("classpath:redis.properties")
public class RedisClusterConfig {

    @Value("${redis.host}")
    private String hostName;

    @Value("${redis.port}")
    private Integer port;

//    @Value("${redis.password}")
//    private String password;

    @Value("${redis.maxActive}")
    private Integer maxActive;
    @Value("${redis.minIdle}")
    private Integer minIdle;
    @Value("${redis.maxIdle}")
    private Integer maxIdle;
    @Value("${redis.maxWait}")
    private Integer maxWait;

    @Value("${redis.cluster.nodes}")
    private String clusterNodes;
    @Value("${redis.cluster.timeout}")
    private String clusterTimeout;
    @Value("${redis.cluster.redirects}")
    private String clusterRedirects;


    @Bean
    //配置RedisTemplate
    public RedisTemplate createRedisTemplate(RedisConnectionFactory redisConnectionFactory){
        //1.创建对象
        RedisTemplate redisTemplate = new RedisTemplate();
        //2.设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //3.设置redis生成的key的序列化器，对key编码进行处理
        RedisSerializer keyStringSerializer = new StringRedisSerializer();
        RedisSerializer valueStringSerializer = new GenericJackson2JsonRedisSerializer();
        /**
         * 如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储
         * ，那么会提示错误User can't cast to String！！
         */
        redisTemplate.setKeySerializer(keyStringSerializer);
        redisTemplate.setValueSerializer(valueStringSerializer);
        redisTemplate.setHashKeySerializer(keyStringSerializer);
        redisTemplate.setHashValueSerializer(valueStringSerializer);
        //4.返回
        return redisTemplate;
    }

    @Bean
    //配置Redis连接工厂
    public RedisConnectionFactory createRedisConnectionFactory(@Autowired RedisClusterConfiguration redisClusterConfiguration, GenericObjectPoolConfig genericObjectPoolConfig){

        //1.创建配置构建器，它是基于池的思想管理Jedis连接的
        JedisClientConfiguration.JedisPoolingClientConfigurationBuilder builder = (JedisClientConfiguration.JedisPoolingClientConfigurationBuilder)JedisClientConfiguration.builder();
        //2.设置池的配置信息对象
        builder.poolConfig(genericObjectPoolConfig);
        //3.创建Jedis连接工厂
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisClusterConfiguration, builder.build());

        //4.返回
        return jedisConnectionFactory;
    }

    @Bean
    //配置spring提供的Redis连接池信息
    public GenericObjectPoolConfig createGenericObjectPoolConfig(){
        //1.创建Jedis连接池的配置对象
        GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
        //2.设置连接池信息
        genericObjectPoolConfig.setMaxTotal(maxActive);
        genericObjectPoolConfig.setMinIdle(minIdle);
        genericObjectPoolConfig.setMaxIdle(maxIdle);
        genericObjectPoolConfig.setMaxWaitMillis(maxWait);
        //3.返回
        return genericObjectPoolConfig;
    }


    @Bean
    public RedisClusterConfiguration redisClusterConfiguration(){
        Map<String, Object> source = new HashMap<String, Object>();
        source.put("spring.redis.cluster.nodes", clusterNodes);
        source.put("spring.redis.cluster.timeout", clusterTimeout);
        source.put("spring.redis.cluster.max-redirects", clusterRedirects);
        return new RedisClusterConfiguration(new MapPropertySource("RedisClusterConfiguration", source));
    }


}
```

##### redis.properties

```properties
# redis服务器主机地址
redis.host=192.168.40.130
#redis服务器主机端口
redis.port=6378

#redis服务器登录密码
#redis.password=itheima

#最大活动连接
redis.maxActive=20
#最大空闲连接
redis.maxIdle=10
#最小空闲连接
redis.minIdle=0
#最大等待时间
redis.maxWait=-1

#集群配置
redis.cluster.nodes=127.0.0.1:6379,127.0.0.1:6380,127.0.0.1:6381,127.0.0.1:6382,127.0.0.1:6383,127.0.0.1:6384
redis.cluster.timeout=5000
#未命中时要进行重定向, 一般与主节点的数量相等即可
redis.cluster.redirects=3
```

##### RedisTest.java

```java
package com.itheima.service;

import com.itheima.config.RedisConfig;
import com.itheima.config.SpringConfig;
import com.itheima.domain.Account;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import redis.clients.jedis.Jedis;

//设定spring专用的类加载器
@RunWith(SpringJUnit4ClassRunner.class)
//设定加载的spring上下文对应的配置
@ContextConfiguration(classes = RedisConfig.class)
public class RedisTest {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void changeMoney() {
        //等同于redis中set account:id:1 100
        redisTemplate.opsForValue().set("account:id:"+3, 200);
    }

    @Test
    public void findMondyById() {
        //等同于redis中get account:id:1
        Object money = redisTemplate.opsForValue().get("account:id:" + 3);
        System.out.println(money);
    }
}
//        redisTemplate.type()
//        redisTemplate.persist()
//        redisTemplate.move()
//        redisTemplate.hasKey()
//        redisTemplate.getExpire()
//        redisTemplate.expire()
//        redisTemplate.delete()
//        redisTemplate.rename();
//
//        redisTemplate.opsForValue().;
//        redisTemplate.opsForHash().;
//        redisTemplate.opsForList().;
//        redisTemplate.opsForSet().;
//        redisTemplate.opsForZSet();
//
//
//        redisTemplate.boundValueOps().;
//
//        redisTemplate.slaveOf();
//        redisTemplate.slaveOfNoOne();
//
//        redisTemplate.opsForCluster()

```





### 5. 企业级解决方案

#### 5.1 缓存预热 

对于高流量的互联网系统来说，应用系统tomcat和数据库mysql负载压力很大，随着流量剧增甚至会导致**数据库**的崩溃。

此时为了提高更好的服务体验，最好是在系统上线之前就把要缓存的热点数据加载到redis缓存中，这种缓存预加载手段就是**缓存预热**。

 

##### 解决方案

###### 统计热点数据： 

 	1. nginx+lua将访问流量上报到kafka中
     - 要统计出来当前最新的实时的热数据是哪些，我们就得将 “明星出轨微博” 访问的请求对应的流量，日志，实时上报到kafka中 
	2.  storm从kafka中消费数据，实时统计出所有微博的访问次数，访问次数基于LRU内存数据结构的存储方案 

###### 准备工作： 

1. 将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据
2.  利用多台服务器同时进行数据读取，并加载到redis集群中 
3. redis集群中，主从同步会对热点数据同时预热 

###### 实施： 

	1. 使用脚本程序固定触发数据预热过程 
 	2.  如果条件允许，使用了CDN（网络加速），效果会更好



##### 小结：

1）缓存预热就是系统启动前，提前将统计好的热点数据直接加载到redis缓存系统。

2）流量不高时企业做法：先查询mysql数据库，再将查出来的数据，同步到redis进行缓存，但是热点数据的流量往往是突然 **“剧增”** 的，这样会导致mysql直接面对高流量查询，导致数据库崩溃。

3）避免大量用户请求热点数据时，因流量过大造成mysql数据库宕机，我们可以提前将热点数据统计出来，加载到redis集群进行缓存，这样用户可以直接查询事先被预热的热点数据！



#### 5.2 缓存雪崩

   **描述：**

   缓存雪崩是指缓存中数据大批量到过期时间时，而查询数据流量依然巨大，引起数据库mysql压力过大甚至down机。

 **解决方案**：

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
   - ⚫ 根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟 ⚫ 过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量
2. 缓存数据量不大的话，可以设计多层级缓存，一级缓存过期后，去找二级缓存。
3. 设置热点数据永远不过期。
4. 限流、降级 
   - 对访问数据库接口进行限流、降级 ，短时间范围内牺牲一些客户体验，限制一部分请求访问，降低数据库的压力，保证部分用户可正常操作。
   - 对访问应用系统层接口进行限流、降级 ，降低应用服务器压力，待业务流量峰值降低后再逐步放开访问

**小节**

缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免过期时间集中，可以有效解决雪崩现象的 出现（约40%），配合其他策略一起使用，例如：限流 + 多级缓存，就可以得到很好的预防。



#### 5.3 缓存击穿

   **描述：**

​     缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，遍去数据库去取数据，引起数据库压力瞬间增大，造成过大压力甚至宕机。

   **解决方案**：

1. 缓存数据量不大的话，可以设计多层级缓存，一级缓存过期后，大家抢锁(redis 分布式乐观锁)， 没抢到锁的找二级缓存， 抢到锁的线程去查询数据库最新数据后，在更新一二级缓存。  
2. 设置热点数据永远不过期，后期人工维护。
3. 设置高流量阈值，定时任务异步判断所有热点数据当前流量是否大于阈值，大于则刷新热点数据过期时间，否则不作处理。
4. 限流
   - 对访问数据库接口进行限流，短时间范围内牺牲一些客户体验，限制一部分请求访问，降低数据库的压力，保证部分用户可正常操作。

**小节**

​	缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量请求对数据库访问，导致对数据库服务器造成压力。

​	个人推荐预防手段，设置不过期，后期人为进行清理 或者 配合定时任务，采取策略定时检测处理。





#### 5.3 缓存穿透

   **描述：**

​       缓存穿透是指Redis中大面积出现key未命中, 从而导致大量的流量去访问了数据库，造成数据库压力过大甚至崩溃。往往是有人恶意攻击导致该事故的发生。(数据库和redis都没有响应数据)

   **举例：**

​       我们想要缓存所有商品的详情页数据，商品id作为key, 当客户端篡改数据传递一个id为-1等不存在值时，我们是无法命中redis中任何一件商品，致使流量打到mysql。

   **解决方案**：

1.  id进行规则设定，比如长度32位，前4位为标识，这样我们可以对发过来的id先做一次基础检验，检验不通过视为非法id，直接驳回请求。
2.  针对redis读取不到数据，在数据库中也没有取到，这时也可以将此非法id作为key, 其值为null，这样可以防止攻击用户反复用同一个id暴力攻击，注意：缓存有效时间可以设置短点，如30秒（设置太长这种垃圾key太多，会导致内存占有量增大。
3. 设置该业务缓存数据永远不过期，后期人工维护，只要发现没有查找到数据，直接视为非法请求，将客户端ip加入黑名单后，直接驳回，后期通过布隆过滤器，对所有非法ip进行过滤。 
4.  对id（key）进行加密，当客户端传来key无法解密时，证明被破坏直接驳回请求。
5. 实施监控 
   - 业务系统每次遇到没有命中的key时，记录系统日志，日志体现出:请求数据，客户端ip等信息，处理结果。
   - 由监控系统负责收集和分析日志，发现同一个客户端ip在单位时间内，发起多次未命中的请求（非人类正常行为），可视为恶意攻击，可加入黑名单后，在请求入口处对非法ip进行过滤拦截，情节严重可报警提供有效证据。

**小节**

​	1）缓存击穿是访问了redis中不存在的key，直接访问数据库，导致对数据库服务器造成压力。

​	2）推荐使用上述1 4 5解决方案对网站缓存穿透进行预防，因为1 4 5是在考虑网站安全时，通用的手段。

​	3）注意无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。



#### 5.4 性能指标监控

##### 性能指标：

Performance 

```shell
#响应请求的平均时间
latency

#平均每秒处理请求总数量
instantaneous_ops_per_sec

#缓存查询命中率（通过查询总次数与查询得到非nil数据总次数计算而来）
hit_rate(calculated)
```

##### 内存指标：

Memory 

```shell
#当前内存使用量
used_memory

#内存碎片率（关系到是否进行碎片整理）
mem_fragmentation_ratio

#为避免内存溢出删除的key的总数量
evicted_keys

#基于阻塞操作（BLPOP等）影响的客户端数量
blocked_clients
```

##### 基本活动指标

Basic_activity 

```shell
#当前客户端连接总数
connected_clients

#当前连接slave总数
connected_slaves

#最后一次主从信息交换距现在的秒数
master_last_io_seconds_ago

#key的总数
keyspace
```

##### 持久性指标：

Persistence 

```shell
#当前服务器最后一次RDB持久化的时间
rdb_last_save_time

#当前服务器最后一次RDB持久化后数据变化总量
rdb_changes_since_last_save
```

##### 错误指标：

Error

```shell
#被拒绝连接的客户端总数（基于达到最大连接值的因素）
rejected_connections

#key未命中的总次数
keyspace_misses

#主从断开的秒数
master_link_down_since_seconds
```



#### 5.5 性能监控工具 （动手）

##### benchmark

```shell
#测试当前服务器的并发性能
redis-benchmark [-h ] [-p ] [-c ] [-n <requests]> [-k ]

#范例1：50个连接，10000次请求对应的性能
redis-benchmark

#范例2：100个连接，5000次请求对应的性能
redis-benchmark -c 100 -n 5000
```

![img](https://images2017.cnblogs.com/blog/707331/201802/707331-20180201145503750-901697180.png)

##### monitor

- 启动服务器调试信息



##### slowlog

- 获取慢查询日志

  ```
  slowlog [operator]
  ```

  - get ：获取慢查询日志信息 
  - len ：获取慢查询日志条目数 
  - reset ：重置慢查询日志 

- 相关配置redis.conf

  ```shell
  #设置慢查询的时间下限，单位：微妙
  slowlog-log-slower-than 1000
  
  #设置慢查询命令对应的日志显示条数，单位：命令数
  slowlog-max-len 100 
  ```

  

#####    图形化页面工具

   https://blog.csdn.net/weixin_42193538/article/details/108278866

   https://www.cnblogs.com/lxwphp/p/11131433.html

