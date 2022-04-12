# Redis入门

### 1. Redis 简介

#### 1.1 Nosql

NoSQL：即 Not-Only SQL（ 泛指非关系型的数据库），作为关系型数据库的补充。 作用：应对基于海量用户和海量数据前提下的数据处理问题。一种不同于关系型数据库的新型数据结构。

常见的Nosql:  ⚫Redis ⚫ memcache ⚫ HBase ⚫ MongoDB



#### 1.2 Redis特点

- redis 一种基于内存提供读写服务的Nosql数据库, 适合在系统中做缓存数据库。
- redis 具有五种数据结构，这五种数据结构都是key-value 格式的简单数据结构
- redis 虽然是单线程的，但你无需为它的性能和担忧
- redis 可以搭建集群来实现海量数据的缓存存储服务
- redis 支持两种持久化手段，RDB和AOF，一般企业中会同时开启这种方案
- redis 并不是用来取代mysql这种关系型数据库，仅仅为了解决高并发下的数据读写问题，采用redis 对MySQL表中数据进行缓存。
- redis 存储的数据可以设置过期时间，也就是我们所说的“时效性”。

#### 1.3 Redis 应用

- redis 是解决存在高并发，大流量的手段之一，因为它具备简单的数据结构，并且基于内存存储，单线程快速访问，所以读写性能高。往往存在热点数据的系统，流量都很大。
- 场景一:  电商系统的商品详情页缓存。
- 场景二:  新浪微博-明星的博客和八卦。
- 场景三:  小学生课后辅导APP，将考试试卷进行redis缓存，因为下午6.30小学生放学后，都会做作业和考试等，存在流量高峰期。
- 场景四: 小学生课后辅导APP, 也可以有排行榜功能，使用redis 实现排行榜要比Mysql简单的多。
- 场景五....... 只要存在流量的地方都可以考虑redis 来做数据的缓存。
- redis 存储的数据可以设置过期时间，所以类似**短信验证码** 和 **Session 会话** 这种有时效性的业务场景，都可以使用redis。
- redis 还可以：分布式锁，秒杀活动商品库存问题，消息队列等应用。



### 2. Redis 的下载与安装

#### 2.1 下载和安装

```shell
[root@localhost ~]$ cd /itcast/install    #进入到自己创建的安装目录下
#下载压缩包
[root@localhost ~]$ wget http://download.redis.io/releases/redis-5.0.0.tar.gz    
#解压缩
[root@localhost ~]$ tar -zxvf redis-5.0.0.tar.gz -C /itcast/install/

#安装gcc基础环境, 可以先查看是否安装过:  gcc --version
[root@localhost ~]$ yum -y install gcc
[root@localhost ~]$ yum -y install gcc-c++

#编译redis
[root@localhost ~]$ cd /itcast/install/redis-5.0.0
[root@localhost redis-5.0.0]$ make MALLOC=libc

#安装
[root@localhost redis-5.0.0]$ cd src
[root@localhost src]$ make install

#查看是否安装成功, 输入命令: "ll |grep redis-"  如果显示信息如下所示, 就是安装成功
[root@localhost src]$ ll |grep redis-
-rwxr-xr-x. 1 root root  353848 6月  26 18:30 redis-benchmark
-rw-rw-r--. 1 root root   29605 10月 17 2018 redis-benchmark.c
-rw-r--r--. 1 root root  109104 6月  26 18:30 redis-benchmark.o
-rwxr-xr-x. 1 root root 4016272 6月  26 18:30 redis-check-aof
-rw-rw-r--. 1 root root    7143 10月 17 2018 redis-check-aof.c
-rw-r--r--. 1 root root   28744 6月  26 18:30 redis-check-aof.o
-rwxr-xr-x. 1 root root 4016272 6月  26 18:30 redis-check-rdb
-rw-rw-r--. 1 root root   13541 10月 17 2018 redis-check-rdb.c
-rw-r--r--. 1 root root   65872 6月  26 18:30 redis-check-rdb.o
-rwxr-xr-x. 1 root root  771056 6月  26 18:30 redis-cli
-rw-rw-r--. 1 root root  249486 10月 17 2018 redis-cli.c
-rw-r--r--. 1 root root  871040 6月  26 18:30 redis-cli.o
-rwxr-xr-x. 1 root root 4016272 6月  26 18:30 redis-sentinel
-rwxr-xr-x. 1 root root 4016272 6月  26 18:30 redis-server
-rwxrwxr-x. 1 root root    3600 10月 17 2018 redis-trib.rb



```

#### 2.2 配置文件修改

```shell
#进入到 /itcast/install/redis-5.0.0/ 目录下
[root@localhost /]$ cd /itcast/install/redis-5.0.0/ 

#创建配置文件的文件夹 和 数据的文件夹
[root@localhost redis-5.0.0]$ mkdir conf
[root@localhost redis-5.0.0]$ mkdir data

#将默认配置文件复制一份到conf文件夹下, 同时备份一份
[root@localhost redis-5.0.0]$ cp redis.conf conf/
[root@localhost redis-5.0.0]$ cp redis.conf conf/redis.conf.back

```

#### 2.3 redis启动

```shell
#进入到 redis-5.0.0/src 目录下, 启动redis服务, &符号表示后台运行, 能够看到如下图所示，启动成功
[root@localhost redis-5.0.0]$ cd src
[root@localhost src]$ ./redis-server ../redis.conf  &

9818:M 26 Jun 2020 19:02:20.243 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 9818
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

9818:M 26 Jun 2020 19:02:20.244 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.


#ctrl + c 回退到命令行出, 查看进程, 如下所示就证明
[root@localhost src]$ ps -ef|grep redis
root       9818   7878  0 19:02 pts/0    00:00:00 ./redis-server 0.0.0.0:6379
root       9832   7878  0 19:02 pts/0    00:00:00 grep --color=auto redis
```

#### 2.4 客户端连接

```shell
#本地连接, 如下表示连接成功
[root@localhost conf]$ redis-cli 
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> 

#连接远程redis 
[root@localhost conf]$ redis-cli –h 61.129.65.248 –p 6384
```

#### 2.5 配置文件修改和再次启动

```shell
#将redis-6379.conf中的注释等都处理掉
[root@localhost redis-5.0.0]$ cd conf
[root@localhost conf]$ cat redis.conf|grep -Ev '^$|#' > redis-6379.conf

#进入到conf文件夹对redis-6379.conf文件进行修改
[root@localhost conf]$ vim redis-6379.conf

#bind 127.0.0.1 允许外网访问
#bind 0.0.0.0
bind 192.168.23.129

#后台启动
daemonize yes

#日志文件
logfile "redis-6379.log"

#数据文件夹设置
dir /itcast/install/redis-5.0.0/data

#:wq 保存退出

#关闭redis 服务
[root@localhost conf]$ ps -ef|grep redis
root       9818   7878  0 19:02 pts/0    00:00:02 ./redis-server 0.0.0.0:6379
root      10675   7878  0 19:45 pts/0    00:00:00 grep --color=auto redis
[root@localhost conf]$ 
[root@localhost conf]$ kill -9 9818

#再次启动并加载修改后的配置 redis-6379.conf
[root@localhost conf]$ redis-server redis-6379.conf

#查看进程是否存在, 如下启动成功
[root@localhost conf]$ ps -ef|grep redis
root      10734      1  0 19:47 ?        00:00:00 redis-server 0.0.0.0:6379
root      10739   7878  0 19:47 pts/0    00:00:00 grep --color=auto redis
[root@localhost conf]$ 


```



### 3. Redis 的基本操作

中文网：https://www.redis.net.cn/tutorial/3508.html

#### 3.1 数据存储类型介绍 

- redis存储的是：key,value格式的数据，其中key都是字符串，value有5种不同的数据类型

- 数据类型：                

  > 1) 字符串类型 string                
  >
  > 2) 哈希类型 hash ： map格式                  
  >
  > 3) 列表类型 list ： linkedlist格式。支持重复元素                
  >
  > 4) 集合类型 set  ： 不允许重复元素                
  >
  > 5) 有序集合类型 sortedset：不允许重复元素，且元素有顺序 

- 当命令为验证某个数据是否存在时0为false , 1 为true

  > ⚫ (integer) 0  →  false 失败 
  >
  > ⚫ (integer) 1  →  true 成功 

- 如果想清屏可以输入命令 : clear
- 如果想退出可以尝试ctrl+c



#### 3.2 字符串类型 string

- 存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型 

- 存储数据的格式：一个存储空间保存一个数据 

- 存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用

- 结构

  > key  vlaue

##### 基本命令 

```shell
#1. 存储： set key value  多次set同一个key则为修改该key的value          
127.0.0.1:6379> set username zhangsan            
OK        

#2. 获取： get key            
127.0.0.1:6379> get username            
"zhangsan"        

#3. 删除： del key            
127.0.0.1:6379> del age            
(integer) 1

#4. 判定性添加数据, 如果key存在, 则不修改
127.0.0.1:6379> setnx agui zls
(integer) 1

#5. 添加/修改多个数据
127.0.0.1:6379> mset key1 value1 key2 value2
OK

#6. 获取多个数据
127.0.0.1:6379> mget key1 key2
1) "value1"
2) "value2"

#7. 获取数据字符个数（字符串value长度）
127.0.0.1:6379> strlen key1
(integer) 6

#8. 追加信息到原始信息后部（如果原始信息存在就追加，否则新建）
127.0.0.1:6379> append key1 aaa
(integer) 9
127.0.0.1:6379> get key1
"value1aaa"

```

##### 基本命令总结

> string存储结构
>
> 数据操作 ◆ set ◆ mset ◆ del ◆ setnx ◆ append 
>
> 查询操作 ◆ get ◆ mget ◆ strlen



##### 扩展命令

```shell
#创建key为number, value为1
192.168.23.129:0>incr number
"1"

#自增+1
192.168.23.129:0>incr number
"2"

#自减-1
192.168.23.129:0>decr number
"1"

#获取number的value
192.168.23.129:0>get number
"1"

#原值上加100;  incrby key increment
192.168.23.129:0>incrby number 100
"101"

#原值上减100;  decrby key increment
192.168.23.129:0>decrby number 100
"1"

#设置数据具有指定的生命周期, 默认没有过期时间
#setex key seconds value  单位秒
#psetex key milliseconds value  单位毫秒
#name zhagnsan  10秒过期
192.168.23.129:0>setex name 10 zhangsan 
"OK"

#name zhagnsan  10秒过期
192.168.23.129:0>setex name 10000 zhangsan 
"OK"
```



##### string 类型注意事项

- 数据未获取到时，对应的数据为（nil），等同于null 
- 数据最大存储量：512MB 
- string在redis内部存储默认就是一个字符串，当遇到增减类操作incr，decr时会转成数值型进行计算 
- 按数值进行操作的数据，如果原始数据不能转成数值，或超越了redis 数值上限范围，将报错 9223372036854775807（java中Long型数据最大值，Long.MAX_VALUE） 
- redis所有的操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行的，因此无需考虑并发带来的数据影响



##### 扩展命令总结

⚫ 数值操作 ◆ incr ◆ incrby ◆ incrbyfloat ◆ desc ◆ descby 

⚫ 时效性操作 ◆ setex ◆ psetex



##### 应用场景

- 明星的新浪微博主页的**粉丝数量**和**博客数量**

- 在redis中为明星设定用户信息，以用户主键和属性值作为key，后台设定定时刷新策略即可 

  ```powershell
  #id为3506728370的明星用户  粉丝数量 200个
  192.168.23.129:0>set user:id:3506728370:fans  200
  "OK"
  
  #id为3506728370的用户  博客数量 100个
  192.168.23.129:0>set user:id:3506728370:blogs  100
  "OK"
  
  ```

-  也可以使用json格式保存数据 

  ```shell
  192.168.23.129:0>set user:id:3506728371  '{"fans":12210947, "blogs":6164,"focuses":83}'
  "OK"
  ```

##### key 的设置约定

数据库中的热点数据key命名惯例

| 表名  | 主键名 | 主键值    | 字段名     |
| ----- | ------ | --------- | ---------- |
| order | id     | 29437595  | pay_status |
| equip | id     | 390472345 | type       |
| news  | id     | 202004150 | title      |

```shell
#存储订单的支付状态
192.168.23.129:0>set order:id:29437595:pay_status  已支付 
"OK"

#获取
192.168.23.129:0>get order:id:29437595:pay_status
"已支付"
```



#### 3.3 哈希类型 hash

-  新的存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息 
- 需要的存储结构：一个存储空间保存多个键值对数据 
- hash类型：底层使用哈希表结构实现数据存储

- 结构

  > key      filed1   value1
  >
  > ​            filed2   value2
  >
  > ​             ...         ...



##### 基本命令

```shell
#添加/修改数据  hset key field value
localhost:0>hset zhangsan  name  '张三'
"1"

localhost:0>hset zhangsan  age  23
"1"

#获取数据 hget key field    hgetall key
localhost:0>hget zhangsan name
"张三"

localhost:0>hget zhangsan age
"23"

localhost:0>hgetall zhangsan
 1)  "name"
 2)  "张三"
 3)  "age"
 4)  "23"
localhost:0>

#删除数据 hdel key field1 [field2]/  del key
localhost:0>hdel zhangsan name
"1"
#删除整个hash  key
localhost:0>del zhangsan
"1"

#设置field的值，如果该field存在则不做任何操作 hsetnx key field value
localhost:0>hset zhangsan  name  '张三'
"1"
localhost:0>hsetnx zhangsan  name  '张三'
"0"
localhost:0>

#添加/修改多个数据
#hmset key field1 value1 field2 value2 … 
localhost:0>hmset lisi name '李四'  age  35  address '回龙观'
"OK"

#获取多个数据
#hmget key field1 field2 … 
localhost:0>hmget lisi name age address
 1)  "李四"
 2)  "35"
 3)  "回龙观"
localhost:0>

#获取哈希表中字段的数量
#hlen key
localhost:0>hlen lisi
"3"
localhost:0>

#获取哈希表中是否存在指定的字段
#hexists key field。 1表示有0表示没有
localhost:0>hexists lisi address
"1"
localhost:0>
```

##### 基本命令总结

⚫ hash存储结构 

⚫ 数据操作 ◆ hset ◆ hmset ◆ hdel ◆ hsetnx 

⚫ 查询操作 ◆ hget ◆ hmget ◆ hgetall ◆ hlen ◆ hexists



##### 扩展命令

```shell
#获取哈希表中所有的字段名或字段值
#hkeys key
#hvals key
localhost:0>hkeys lisi
 1)  "name"
 2)  "age"
 3)  "address"
localhost:0>

localhost:0>hvals lisi
 1)  "李四"
 2)  "35"
 3)  "回龙观"

#设置指定字段的原来的数值在加上increment的数值, 只能为integer类型提供此操作
#hincrby key field increment
#hincrbyfloat key field increment
localhost:0>hincrby lisi age 100
"135"

localhost:0>hvals lisi
 1)  "李四"
 2)  "135"
 3)  "回龙观"


```

##### hash 类型注意事项

- 如果数据未获取到，对应的值为（nil）
- 每个 hash 可以存储 232 - 1 个键值对 
- hash类型十分贴近对象的数据存储形式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计 的，切记不可滥用，更不可以将hash作为对象列表使用 
- hgetall 操作可以获取全部属性，如果内部field过多，遍历整体数据效率就很会低，因为redis单线程处理模式，所以后面的线程就会在等待

##### 扩展命令总结

⚫ 数值操作 ◆ hincrby ◆ hincrbyfloat 

⚫ 特殊查询操作 ◆ hkeys ◆ hvals 

⚫ 注意事项



##### 应用场景

双11活动日，销售手机充值卡的商家对移动、联通、电信的30元、50元、100元商品推出抢购活动，每种商品抢购上限1000 张

- 解决方案

  > ⚫ 以商家id作为key 
  >
  > ⚫ 将参与抢购的商品id作为field 
  >
  > ⚫ 将参与抢购的商品数量作为对应的value 
  >
  > ⚫ 抢购时使用降值的方式控制产品数量

```shell
127.0.0.1:6379> hmset id:001 c30 1000 c50 1000 c100 1000
OK
127.0.0.1:6379> hincrby id:001 c50 -2
(integer) 998
127.0.0.1:6379>  hincrby id:001 c30 -5
(integer) 995
127.0.0.1:6379>  hincrby id:001 c100 -20
(integer) 980
127.0.0.1:6379> hgetall id:001
1) "c30"
2) "995"
3) "c50"
4) "998"
5) "c100"
6) "980"
127.0.0.1:6379>
```



#### 3.4 list 类型

- 数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分

- 需要的存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序

- list类型：保存多个数据，数据可以重复，因为采用双向链表，左右两侧都可以插入value

- 结构

  > key  [value1, value2, value3.....]

##### 基本命令

```shell
# 添加/修改数据
#从左添加 lpush key value1 [value2] ……
#从右添加 rpush key value1 [value2] ……
127.0.0.1:6379> lpush myList a
(integer) 1
127.0.0.1:6379> lpush myList b
(integer) 2
127.0.0.1:6379> rpush myList c
(integer) 3

#获取数据
#lrange key start stop  范围获取 0  -1为获取所有
127.0.0.1:6379> lrange myList 0 -1
1) "b"
2) "a"
3) "c"
#lindex key index
127.0.0.1:6379> lindex myList 0
"b"
127.0.0.1:6379> lindex myList 2
"c"
127.0.0.1:6379> lindex myList 56
(nil)
127.0.0.1:6379>
#llen key 
127.0.0.1:6379> llen myList
(integer) 3
127.0.0.1:6379>

#获取并移除数据
#lpop key
#rpop key
127.0.0.1:6379> lpop myList
"b"
127.0.0.1:6379> rpop myList
"c"
127.0.0.1:6379> lrange myList 0 -1
1) "a"
127.0.0.1:6379>
```

##### 基本命令总结

⚫ list存储结构 

⚫ 数据操作 ◆ lpush ◆ rpush ◆ lpop ◆ rpop 

⚫ 查询信息 ◆ lrange ◆ lindex ◆ llen



##### 扩展命令

```shell
#移除指定数据
#lrem key count value
127.0.0.1:6379> lpush list a a a a a
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "a"
2) "a"
3) "a"
4) "a"
5) "a"
127.0.0.1:6379> lrem list 2 a
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "a"
2) "a"
3) "a"

#规定时间内获取并移除数据
#blpop key1 [key2] timeout
127.0.0.1:6379> blpop list 10
1) "list"
2) "a"
127.0.0.1:6379> lrange list 0 -1
1) "a"
127.0.0.1:6379>
127.0.0.1:6379> blpop list 10
1) "list"
2) "a"
127.0.0.1:6379> blpop list b 10
(nil)
(10.02s)
127.0.0.1:6379>

#brpop key1 [key2] timeout
127.0.0.1:6379> brpop list b 10
(nil)
(16.69s)
127.0.0.1:6379>
#brpoplpush = brpop + lpush 命令的组合命令 单条命令具有两种命令的功能
#此命令的主要功能为:    将一个元素从列表中取出 并放入另一个列表中
#brpoplpush source destination timeout
127.0.0.1:6379>brpoplpush list1 list2 10
#注意事项:
#  1 listKeyName1 listKeyName2 必须为list(列表)或不存在redis数据库中
#  2 timeOut   必须为一个大于等于0的整数
#  3 当timeout 等于0 挂起等待时间为无限大
#  4 当timeout 大于0 挂起等待时间为timeout秒数
#  5 如果客户端命令等待时间大于超时时间后,并且listKeyName1为空,此时会返回(nil)
#  6 当在超时时间内，listKeyName1 存在数据时,此命令会从listKeyName1中最先插入的元素中获取一个元素插入至listKeyName2中，并返回操作的元素值。

```

##### list类型注意事项

- list中保存的数据都是string类型的，数据总容量是有限的，最多232 - 1 个元素 (4294967295)。 
- list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或以栈的形式进行入栈出栈操作 
-  获取全部数据操作结束索引设置为-1 
- list可以对数据进行分页操作，通常第一页的信息来自于list，第2页及更多的信息通过数据库的形式加载

##### 扩展命令总结

⚫ 数据操作 ◆ lrem ◆ blpop ◆ brpop ◆ brpoppush 

⚫ 注意事项



##### 应用场景

企业运营过程中，系统将产生出大量的运营数据，如何保障多台服务器操作日志的统一顺序输出？

- 解决方案

  > ⚫ 依赖list的数据具有顺序的特征对信息进行管理 
  >
  > ⚫ 使用队列模型解决多路信息汇总合并的问题 
  >
  > ⚫ 使用栈模型解决最新消息的问题

```shell
127.0.0.1:6379> rpush logs a:out
(integer) 1
127.0.0.1:6379> rpush logs b:out
(integer) 2
127.0.0.1:6379> rpush logs a:print
(integer) 3
127.0.0.1:6379> rpush logs c:out
(integer) 4
127.0.0.1:6379> lrange logs 0 -1
1) "a:out"
2) "b:out"
3) "a:print"
4) "c:out"
127.0.0.1:6379>
```



#### 3.5 set 类型

- 新的存储需求：存储大量的数据，在查询方面提供更高的效率 

- 需要的存储结构：能够保存大量的数据，高效的内部存储机制，便于查询 

- set类型：与hash存储结构完全相同，仅存储键，不存储值（nil），并且值是不允许重复的, 存储的数据时无序的

- 结构

  > key  [ value1, value2, value3......]

##### 基本命令

```shell
#添加数据
#sadd key member1 [member2]
127.0.0.1:6379> sadd myset a
(integer) 1
127.0.0.1:6379> sadd myset b
(integer) 1
127.0.0.1:6379> sadd myset c
(integer) 1
127.0.0.1:6379> sadd myset a
(integer) 0


#获取全部数据
#smembers key 
127.0.0.1:6379> smembers myset
1) "a"
2) "c"
3) "b"
#删除数据
#srem key member1 [member2]
127.0.0.1:6379> srem myset b c
(integer) 2
127.0.0.1:6379> smembers myset
1) "a"
127.0.0.1:6379>

#获取集合数据总量
#scard key
127.0.0.1:6379> scard myset
(integer) 1


#判断集合中是否包含指定数据
#sismember key member
127.0.0.1:6379> smembers myset
1) "a"
127.0.0.1:6379>sismember myset a
(integer) 1
#随机获取集合中指定数量的数据
#srandmember key [count]
127.0.0.1:6379> srandmember myset 2
1) "a"
2) "b"
127.0.0.1:6379> srandmember myset 2
1) "f"
2) "a"
127.0.0.1:6379>

#随机获取集合中的某个数据并将该数据移出集合
#spop key [count]
127.0.0.1:6379> spop myset 2
1) "e"
2) "c"
127.0.0.1:6379> spop myset 2
1) "b"
2) "d"
127.0.0.1:6379> smembers myset
1) "f"
2) "a"
127.0.0.1:6379>

```

##### 基本命令总结

⚫ set存储结构 

⚫ 数据操作 ◆ sad ◆ srem ◆ spop

⚫ 查询操作 ◆ smember ◆ scard ◆ sismember ◆ srandmember



##### 扩展命令

```shell
#求两个集合的交、并、差集
#sinter key1 [key2 …] 
#sunion key1 [key2 …] 
#sdiff key1 [key2 …]
127.0.0.1:6379> sadd set1 100 600 666 itheima
(integer) 4
127.0.0.1:6379> sadd set2 100 itheima
(integer) 2
127.0.0.1:6379> sadd set3 666
(integer) 1
127.0.0.1:6379> sadd set4 itcast
(integer) 1
127.0.0.1:6379> sinter set1 set3
1) "666"
127.0.0.1:6379>
127.0.0.1:6379> sinter set1 set2
1) "itheima"
2) "100"
127.0.0.1:6379> sinter set1 set2 set3
(empty list or set)
127.0.0.1:6379> sunion set1 set4
1) "666"
2) "600"
3) "itheima"
4) "100"
5) "itcast"
127.0.0.1:6379> sdiff set1 set2
1) "600"
2) "666"
127.0.0.1:6379> sdiff set2 set1
(empty list or set)
#求两个集合的交、并、差集并存储到指定集合中
#sinterstore destination key1 [key2 …] 
#sunionstore destination key1 [key2 …] 
#sdiffstore destination key1 [key2 …] 
127.0.0.1:6379> sinterstore youset set1 set2
(integer) 2
127.0.0.1:6379> smembers youset
1) "itheima"
2) "100"
127.0.0.1:6379>
#将指定数据从原始集合中移动到目标集合中
#smove source destination member 
127.0.0.1:6379> smembers youset
1) "itheima"
2) "100"
127.0.0.1:6379> smembers myset
1) "f"
2) "a"
127.0.0.1:6379> smove myset youset f
(integer) 1
127.0.0.1:6379> smembers youset
1) "f"
2) "itheima"
3) "100"
127.0.0.1:6379> smembers myset
1) "a"
127.0.0.1:6379>

```

##### set 类型注意事项

- set 类型不允许数据重复，如果添加的数据在set 中已经存在，将只保留一份 
- set 虽然与hash的存储结构相同，但是无法启用hash中存储值的空间

##### 扩展命令总结

⚫ 集合操作 ◆ sinter ◆ sunion ◆ sdiff ◆ sinterstore ◆ sunionstore ◆ sdiffstore ◆ smove ⚫ 注意事项



##### 应用场景

- 黑名单 

  > 1)  资讯类信息类网站追求高访问量，但是由于其信息的价值，往往容易被不法分子利用，通过爬虫技术， 快速获取信息，个别特种行业网站信息通过爬虫获取分析后，可以转换成商业机密进行出售。例如第三方火 车票、机票、酒店刷票代购软件，电商刷评论、刷好评。 
  >
  > 2)  同时爬虫带来的伪流量也会给经营者带来错觉，产生错误的决策，有效避免网站被爬虫反复爬取成为每 个网站都要考虑的基本问题。在基于技术层面区分出爬虫用户后，需要将此类用户进行有效的屏蔽，这就是 **黑名单**的典型应用。 
  >
  > 3）ps:  不是说爬虫一定做摧毁性的工作，有些小型网站需要爬虫为其带来一些流量。白名单 对于安全性更高的应用访问，仅仅靠黑名单是不能解决安全问题的，此时需要设定可访问的用户群体， 依赖白名单做更为苛刻的访问验证。

- 白名单 

  > 对于安全性更高的应用访问，仅仅靠黑名单是不能解决安全问题的，此时需要设定可访问的用户群体， 依赖白名单做更为苛刻的访问验证。

  

- 解决方案

  > ⚫ 基于经营战略设定问题用户发现、鉴别规则 
  >
  > ⚫ 周期性更新满足规则的用户黑名单，加入set集合 
  >
  > ⚫ 用户行为信息达到后与黑名单进行比对，确认行为去向 
  >
  > ⚫ 黑名单过滤IP地址：应用于开放游客访问权限的信息源 
  >
  > ⚫ 黑名单过滤设备信息：应用于限定访问设备的信息源 
  >
  > ⚫ 黑名单过滤用户：应用于基于访问权限的信息源



#### 3.6 数据类型实践案例

1）利用list 数据类型实现最新消息在前面

```shell
#小红 小明 小张 小王 给 邹老师发消息
#小明先给邹老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaoming
(integer) 0
127.0.0.1:6379> lpush zoulaoshi xiaoming
(integer) 1

#小红给老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaohong
(integer) 0
127.0.0.1:6379> lpush zoulaoshi xiaohong
(integer) 2

#小张给老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaozhang
(integer) 0
127.0.0.1:6379> lpush zoulaoshi xiaozhang
(integer) 3

#小王给老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaowang
(integer) 0
127.0.0.1:6379> lpush zoulaoshi xiaowang
(integer) 4

#小明再一次给老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaoming
(integer) 1
127.0.0.1:6379> lpush zoulaoshi xiaoming
(integer) 4

#小张再一次给老师发消息
127.0.0.1:6379> lrem zoulaoshi 1 xiaozhang
(integer) 1
127.0.0.1:6379> lpush zoulaoshi xiaozhang
(integer) 4

#此时的顺序为: 小张 小明 小王 小红
127.0.0.1:6379> lrange zoulaoshi 0 -1
1) "xiaozhang"
2) "xiaoming"
3) "xiaowang"
4) "xiaohong"
127.0.0.1:6379>


```

2）记录每个人给老师发了多少消息

```shell
#假设每人每次给邹老师发一条消息, 我们用hash来存储这部分数据
#hincrby 在原来来filed的基础上加上指定 数值
#小明先给邹老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaoming 1

#小红给老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaohong 1  

#小张给老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaozhang 1

#小王给老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaowang 1

#小明再一次给老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaoming 1 

#小张再一次给老师发消息
127.0.0.1:6379> hincrby u:zoulaoshi u:xiaozhang 1  


#此时展示 小张 小明 小王 小红 分别发了多少条消息
127.0.0.1:6379> hgetall u:zoulaoshi
1) "u:xiaoming"
2) "2"
3) "u:xiaohong"
4) "1"
5) "u:xiaozhang"
6) "2"
7) "u:xiaowang"
8) "1"
127.0.0.1:6379>


```



### 4. key常用指令

#### 4.1 key操作分析

⚫ key是一个字符串，通过key获取redis中保存的数据 

⚫ 对于key自身状态的相关操作，例如：删除，判定存在，获取类型等 

⚫ 对于key有效性控制相关操作，例如：有效期设定，判定是否有效，有效状态的切换等 ⚫ 对于key快速查询操作，例如：按指定策略查询key 

⚫ ……

#### 4.2 key的基本命令

```shell
#删除指定key
del key
#获取key是否存在
exists key
#获取key的类型
type key

##示例如下
127.0.0.1:6379> set username zhangsan
OK
127.0.0.1:6379>
127.0.0.1:6379> exists username
(integer) 1
127.0.0.1:6379>
127.0.0.1:6379> type username
string
127.0.0.1:6379> del username
(integer) 1
127.0.0.1:6379> exists username
(integer) 0
127.0.0.1:6379>


```

#### 4.3 key扩展命令

```powershell
#排序sort, 默认按照自然顺序进行升序排序, 自然顺序: a b c d e .....z; A....Z; 0-9
127.0.0.1:6379> lrange zoulaoshi 0 -1
1) "xiaozhang"
2) "xiaoming"
3) "xiaowang"
4) "xiaohong"
#升序排序  alpha 对字母进行排序, PS:默认只能对数字进行排序
127.0.0.1:6379> sort zoulaoshi alpha
1) "xiaohong"
2) "xiaoming"
3) "xiaowang"
4) "xiaozhang"
#降序排序  alpha 对字母进行排序, PS:默认只能对数字进行排序
127.0.0.1:6379> sort zoulaoshi alpha desc
1) "xiaozhang"
2) "xiaowang"
3) "xiaoming"
4) "xiaohong"
127.0.0.1:6379>
#改名rename 改名时, 新名字存在还是不存在都能改成功, 如果新名字key本来就存在，将会被干掉，很危险；
#改名renamenx 新名称的key如果存在不会修改成功
#rename key newkey
127.0.0.1:6379> rename zoulaoshi shuaibi
OK
#renamenx key newkey, 如下因为zhagnsan已经存在, 所以结果为0表示false失败
127.0.0.1:6379> renamenx shuaibi zhangsan
(integer) 0

#############################################################################
#为指定key设置有效期
#expire key seconds
127.0.0.1:6379> expire cc  20
#pexpire key milliseconds
127.0.0.1:6379> pexpire cc 10000

########时间戳自行完成, 在线时间戳获取工具: http://tool.aliy7.com/
#expireat key timestamp
#pexpireat key milliseconds-timestamp

#获取key的有效时间
ttl key
pttl key

#默认就是-1 没有过期时间
127.0.0.1:6379> set aa 123
OK
127.0.0.1:6379> ttl aa
(integer) -1

#-2表示不存在的key
127.0.0.1:6379> ttl bb
(integer) -2
#设置string key:cc  value:123   过期时间: 100秒
127.0.0.1:6379> setex cc 100 123
OK
#查看key:cc 的剩余过期时间
127.0.0.1:6379> ttl cc
(integer) 97
127.0.0.1:6379> ttl cc
(integer) 95
127.0.0.1:6379> ttl cc
(integer) 91
#毫秒单位显示剩余过期时间
127.0.0.1:6379> pttl cc
(integer) 87359
127.0.0.1:6379>


#切换key从时效性转换为永久性
#persist key
127.0.0.1:6379> setex cc 10 123
OK
127.0.0.1:6379> ttl cc
(integer) 8
127.0.0.1:6379> ttl cc
(integer) 5
127.0.0.1:6379> persist cc
(integer) 1
127.0.0.1:6379> ttl cc
(integer) -1
127.0.0.1:6379>

#####################################################################
*   匹配任意数量的任意符号 
?   配合一个任意符号 
[]  匹配一个指定符号

keys * 查询所有 
keys it* 查询所有以it开头 
keys *heima 查询所有以heima结尾 
keys ??heima 查询所有前面两个字符任意，后面以heima结尾 
keys user:? 查询所有以user:开头，最后一个字符任意 
keys u[st]er:1 查询所有以u开头，以er:1结尾，中间包含一个字母，s或t


#查询key  keys pattern
127.0.0.1:6379> keys *
 1) "lisi"
 2) "ad"
 3) "cc"
 4) "set2"
 5) "100"
 6) "set1"
 7) "aa"
 8) "id:001"
127.0.0.1:6379>
#查询key是 a开头的, 注意: 只能查询string 类型的key
127.0.0.1:6379> keys a?
1) "ad"
2) "aa"
#查询key是 b结尾的, 注意: 只能查询string 类型的key
127.0.0.1:6379> keys ?b

#支持正则: a开头的第二位为 a或者b
127.0.0.1:6379> keys a[ab]

#a开头的第二位为 a或者b, 在后面任意
127.0.0.1:6379> keys a[ab]*


```

#### 4.4 key命令总结

⚫ 操作指令 ◆ del ◆ rename ◆ renamenx ◆ expire/ expireat ◆ pexpire/ pexpireat ◆ persist 

⚫ 查询指令 ◆ exists ◆ type ◆ ttl ◆ pttl ◆ keys ◆ sort



### 5. db 基本操作

#### 5.1 数据库

- 思考: key 的重复问题?

  > ⚫ key是由程序员定义的 
  >
  > ⚫ redis在使用过程中，伴随着操作数据量的增加，会出现大量的数据以及对应的key 
  >
  > ⚫ 数据不区分种类、类别混杂在一起，极易出现重复或冲突

- 解决方案

  > ⚫ redis为每个服务提供有16个数据库，编号从0到15 
  >
  > ⚫ 每个数据库之间的数据相互独立

```shell
#切换数据库 select index 默认16个, 索引0-15; 可以在redis.conf 中修改默认数量
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> set username zhangsan
OK
#其他操作 ping 返回pong, 一般用来验证redis server 是否在线
127.0.0.1:6379[1]> ping
PONG


#数据移动 move key db
#将索引1的库中 key:username 移动到 索引0号库中
127.0.0.1:6379[1]> move username 0
(integer) 1
127.0.0.1:6379[1]> keys *
(empty list or set)
127.0.0.1:6379[1]>

#数据总量 dbsize；查看数据库中 key的总数量
127.0.0.1:6379[1]> dbsize
(integer) 0
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> dbsize
(integer) 18
127.0.0.1:6379>

#数据清除 flushdb清空当前数据库中所有的key;   flushall清空所有数据库
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> flushdb
OK
127.0.0.1:6379[1]>


```

#### 5.2 命令总结

⚫ 操作指令 ◆ move ◆ flushdb ◆ flushall ◆ select ◆ ping 

⚫ 查询指令 ◆ dbsize



### 6. Java连接Redis客户端框架-Jedis

-  Jedis用于Java语言连接redis服务，并提供对应的操作API, 类似于我们的jdbc。

-  Java语言连接redis服务 Jedis / SpringDataRedis / Lettuce 
- 其他编程语言也都有redis客户端框架， C 、C++ 、C# 、Erlang、Lua .....

#### 6.1 快速入门

- 现在API文档: http://xetorthio.github.io/jedis/

-  jar包导入 下载地址：https://mvnrepository.com/artifact/redis.clients/jedis 

  > 1）commons-pool2-2.4.2.jar  jedis连接池提供jar
  >
  > 2）jedis-3.1.0.jar   jedis 客户端主要jar
  >
  > 3）log4j-1.2.17.jar   slf4j-api-1.7.5.jar   slf4j-log4j12-1.7.25.jar 这三个jar包是jedis-3.1.0.jar 它的依赖jar，用来打印日志。

**示例代码**

```java
package com.itheima;

import com.itheima.util.JedisUtils;
import redis.clients.jedis.Jedis;

import java.util.List;

public class JedisTest {

    public static void main(String[] args) {
        //1.获取连接对象
//        Jedis jedis = new Jedis("192.168.40.130",6379);
        Jedis jedis = JedisUtils.getJedis();
        //2.执行操作
        //操作string数据类型
        jedis.set("age","39");
        String hello = jedis.get("hello");
        System.out.println(hello);
        
        //操作list数据类型
        jedis.lpush("list1","a","b","c","d");
        List<String> list1 = jedis.lrange("list1", 0, -1);
        for (String s:list1 ) {
            System.out.println(s);
        }
        
        //操作set数据类型
        jedis.sadd("set1","abc","abc","def","poi","cba");
        Long len = jedis.scard("set1");
        System.out.println(len);
        //3.关闭连接
        jedis.close();
    }
}

```



#### 6.2 快速入门小结

- 操作步骤 ◆ 导入坐标 ◆ 创建连接 ◆ 数据操作 ◆ 关闭连接 
-  常用操作 ◆ string ◆ hash ◆ list ◆ set
- 从上述java代码不难看出，之前我们敲过的命令在Jedis中都有对应的方法供我们调用。



#### 6.3 jedis工具类

##### redis.properties

```properties
redis.maxTotal=50
redis.maxIdel=10
redis.host=localhost
redis.port=6379
```

##### JedisUtils.java

```java
package com.itheima.util;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.ResourceBundle;

public class JedisUtils {

    private static int maxTotal;
    private static int maxIdel;
    private static String host;
    private static int port;

    private static JedisPoolConfig jpc;

    private static JedisPool jp;

    static {
        ResourceBundle bundle = ResourceBundle.getBundle("redis");
        maxTotal = Integer.parseInt(bundle.getString("redis.maxTotal"));
        maxIdel = Integer.parseInt(bundle.getString("redis.maxIdel"));
        host = bundle.getString("redis.host");
        port = Integer.parseInt(bundle.getString("redis.port"));

        //Jedis连接池配置
        jpc = new JedisPoolConfig();
        jpc.setMaxTotal(maxTotal);
        jpc.setMaxIdle(maxIdel);
        jp = new JedisPool(jpc,host,port);
    }

    public static Jedis getJedis(){
        return jp.getResource();
    }

}


```

#### 6.4 jedis工具类总结

- Jedis连接池 

- 工具类 ◆ 加载连接池静态资源 ◆ 设置连接池参数 ◆ 通过连接池获取连接对象



6.5 扩展- 动手练一练





### 7. Redis持久化存储

#### 7.1 持久化简介 

- 什么是持久化？

  > 1) 利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化 持久化用于防止数据的意外丢失，确保数据安全性。
  >
  > 2) <font color=red>说白了就是将数据保存在电脑磁盘上。</font>

- redis 支持两种持久化的操作，一种叫**RDB**，它保存的是具体的数据到磁盘上，并且是以快照的方式**对数据进行保存**。还有一种叫**AOF**，与RDB不同的是它保**存的是操作的命令**，不去保存具体的数据。



#### 7.2 RDB

- 基于快照，对具体数据进行持久化操作的存储方式。

##### 7.2.1 save指令进行手动RDB过程

```shell
#手动执行一次保存操作
127.0.0.1:6379> save
OK
127.0.0.1:6379>

```

**save指令相关配置-redis.conf**

- 设置本地数据库文件名，默认值为 dump.rdb，通常设置为dump-端口号.rdb

  > dbfilename filename

- 设置存储.rdb文件的路径，通常设置成存储空间较大的目录中，目录名称data

  > dir path

- 设置存储至本地数据库时是否压缩数据，默认yes; 设置为no时，节省CPU 运行时间，但存储文件变大

  > rdbcompression yes|no

- 设置读写文件过程是否进行RDB格式校验，默认yes，设置为no，节约读写10%时间消耗，单存在数据损坏的风险

  > rdbchecksum yes|no

<font  color=red>注意：save指令的执行会阻塞当前Redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用。</font>



##### 7.2.2 bgsave指令进行手动RDB过程

- 数据量过大，单线程执行方式造成效率过低如何处理？

  > bgsave 命令

- bgsave是对 save的一个优化命令，因为它会让redis server 创建一个子线程去完成保存的操作。

**操作如下**

```shell
#手动启动后台保存操作，但不是立即执行
127.0.0.1:6379> bgsave
Background saving started
127.0.0.1:6379>

#[2976] 28 Jun 09:58:27.352 * Background saving started by pid 15140
#[2976] 28 Jun 09:58:27.613 # fork operation complete
#[2976] 28 Jun 09:58:27.613 * Background saving terminated with #success
```

 **bgsave指令相关配置-redis.conf**

- 后台存储过程中如果出现错误现象，是否停止保存操作，默认yes

  > stop-writes-on-bgsave-error yes|no

- 其他

  > dbfilename filename
  > dir path
  > rdbcompression yes|no
  > rdbchecksum yes|no

<font color=red>注意： bgsave命令是针对save阻塞问题做的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式，save命令可以放弃使用。</font>



##### 7.2.3 save配置进行自动RDB过程

**配置-redis.conf**

- 设置自动持久化的条件，满足限定时间范围内key的变化数量达到指定数量即进行持久化

  > save second changes

- 参数

  > second：监控时间范围 
  >
  > changes：监控key的变化量

- 范例

  > save 900 1
  > save 300 10
  > save 60 10000

- 其他

  > dbfilename filename
  > dir path
  > rdbcompression yes|no
  > rdbchecksum yes|no
  > stop-writes-on-bgsave-error yes|no



##### 7.2.4 RDB工作原理

1) “save 900 1”表示如果900秒内至少1个key发生变化（**新增、修改和删除**），则重写rdb文件；

2) “save 300 10”表示如果每300秒内至少10个key发生变化（**新增、修改和删除**），则重写rdb文件；

3) “save 60 3600”表示如果每60秒内至少3600个key发生变化（**新增、修改和删除**），则重写rdb文件。

<font color=red>注意： save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的 save配置启动后执行的是bgsave操作</font>



##### 7.2.5 三种方式对比（优缺点）

- 自动保存触发后，执行的是bgsave 操作

|      方式      | save指令 | bgsave指令 |
| :------------: | :------: | :--------: |
|      读写      |   同步   |    异步    |
| 阻塞客户端指令 |    是    |     否     |
|  额外内存消耗  |    否    |     是     |
|   启动新进程   |    否    |     是     |

**RDB特殊启动形式**

```shell
#save当前内存中数据到rdb文件，并清空当前数据库，重新加载rdb，加载与启动时加载类似，加载过程中只能服务部分只读请求（比如info、ping等）
debug reload

#关闭服务器时指定保存数据
shutdown save

```

**RDB优点**

- RDB是一个紧凑压缩的二进制文件，存储效率较高 
- RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景 
- RDB恢复数据的速度要比AOF快很多 
- 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

**RDB缺点**

- RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据 
- bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能 
- Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象



#### 7.3 AOF

- RDB存储的弊端?

  > 1) 存储数据量较大，效率较低，基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低 
  >
  > 2) 大数据量下的IO性能较低 
  >
  > 3) 基于fork创建子进程，内存产生额外消耗 
  >
  > 4) 宕机带来的数据丢失风险

- 解决思路

  > 1) 不写全数据，仅记录部分数据 
  >
  > 2) 降低区分数据是否改变的难度，改记录数据为记录操作过程 
  >
  > 3) 对所有操作均进行记录，排除丢失数据的风险



#####  7.3.1 AOF概念

- AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令 达到恢复数据的目的。与RDB相比可以简单理解为由记录数据改为记录数据产生的变化 
- AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式



**启动AOF相关配置-redis.conf**

- 开启AOF持久化功能，默认no，即不开启状态

  > appendonly yes|no

- AOF持久化文件名，默认文件名为appendonly.aof，建议配置为appendonly-端口号.aof

  > appendfilename filename

- AOF持久化文件保存路径，与RDB持久化文件保持一致即可

  > dir

- AOF写数据策略，默认为everysec

  > appendfsync always|everysec|no



##### 7.3.2 AOF执行策略

- always(每次）：每次写入操作均同步到AOF文件中 

  > 数据零误差，性能较低，不建议使用。 

- everysec（每秒-推荐使用）：每秒将缓冲区中的指令同步到AOF文件中，在系统突然宕机的情况下丢失1秒内的数据 

  > 数据准确性较高，性能较高，**建议使用**，也是默认配置

- no（系统控制）：由操作系统控制每次同步到AOF文件的周期 

  > 整体过程不可控

配置如下：

```properties
port 6379
#daemonize no
#logfile "6379.log"
dir /redis/data
dbfilename "dump-6379.rdb"
save 10 2
#开启aof持久化
appendonly yes
#aof文件名称
appendfilename "appendonly-6379.aof"
#aof持久化策略, 一秒持久化一次命令
appendfsync everysec
```



##### 7.3.3 AOF重写

- 随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重 写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结 果转化成最终结果数据对应的指令进行记录。 <font color=red>说白了，就是对重复性对同一个key做出操作后，只需要记录最终结果值</font>

  > 举例: 
  >
  > set name zhagnsan
  >
  > set name lisi
  >
  > set name wangwu
  >
  > 或者
  >
  > incr num
  >
  > incr num
  >
  > incr num
  >
  > **重写之后**:  set name wangwu   /    set num 3

- AOF重写作用

  > 1) 降低磁盘占用量，提高磁盘利用率 
  >
  > 2) 提高持久化效率，降低持久化写时间，提高IO性能 
  >
  > 3) 降低数据恢复用时，提高数据恢复效率

- AOF重写规则

  > 1） 进程内具有时效性的数据，并且数据已超时将不再写入文件 
  >
  > 2） 非写入类的无效指令将被忽略，只保留最终数据的写入命令 
  >
  > ​            ⚫如del key1、hdel key2、sremkey3、set key4 111、set key4 222等 
  >
  > ​            ⚫如select指令虽然不更改数据，但是更改了数据的存储位置，此类命令同样需要记录 
  >
  > 3） 对同一数据的多条写命令合并为一条命令 
  >
  > ​            ⚫如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。 
  >
  > ​            ⚫为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素



**AOF重写方式**

```shell
#手动重写
127.0.0.1:6379>bgrewriteaof

```

**自动重写**

| 配置名                      | 含义                          |
| --------------------------- | ----------------------------- |
| auto-aof-rewrite-min-size   | 触发AOF文件执行重写的最小尺寸 |
| auto-aof-rewrite-percentage | 触发AOF文件执行重写的增长率   |

| 统计名           | 含义                                  |
| ---------------- | ------------------------------------- |
| aof_current_size | AOF文件当前尺寸（字节）               |
| aof_base_size    | AOF文件上次启动和重写时的尺寸（字节） |

**AOF重写自动触发机制，需要同时满足下面两个条件：**

- aof_current_size > auto-aof-rewrite-min-size
- (aof_current_size - aof_base_size)  / aof_base_size >=  auto-aof-rewrite-percentage

**假设 Redis.conf 的配置项为：**

```properties
#64m
auto-aof-rewrite-min-size 64mb
#百分百
auto-aof-rewrite-percentage 100
```

当AOF文件的体积大于64Mb，并且AOF文件的体积比上一次重写之前的体积大了至少一倍（100%）时，Redis将执行 bgrewriteaof 命令进行重写



#### 7.4 RDB与AOF区别

##### 7.4.1 RDB VS AOF

|  持久化方式  |        RDB         |        AOF         |
| :----------: | :----------------: | :----------------: |
| 占用存储空间 | 小（数据级：压缩） | 大（指令级：重写） |
|   存储速度   |         慢         |         快         |
|   恢复速度   |         快         |         慢         |
|  数据安全性  |     会丢失数据     |    依据策略决定    |
|   资源消耗   |     高/重量级      |     低/轻量级      |
|  启动优先级  |         低         |         高         |



##### 7.4.2 RDB与AOF的选择之惑

- 对数据非常敏感，建议使用默认的AOF持久化方案 

  > ⚫ AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出 现问题时，最多丢失0-1秒内的数据。 
  >
  > ⚫ 注意：由于AOF文件存储体积较大，且恢复速度较慢 

- 数据呈现阶段有效性，建议使用RDB持久化方案 

  > ⚫ 数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段 点数据恢复通常采用RDB方案 
  >
  > ⚫ 注意：利用RDB实现紧凑的数据持久化会使Redis降的很低，慎重总结： 

- 综合比对 

  > ⚫ RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊 
  >
  > ⚫ 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF 
  >
  > ⚫ 如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB 
  >
  > ⚫ 灾难恢复选用RDB 
  >
  > ⚫ 双保险策略，同时开启RDB 和 AOF，重启后，Redis优先使用AOF 来恢复数据，降低丢失数据的量



### 8. 扩展

#### 8.1 有序集合类型 sortedset

- 不允许重复元素，且元素有顺序.每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

```shell
#存储：zadd key score value
127.0.0.1:6379> zadd mysort 60 zhangsan
(integer) 1
127.0.0.1:6379> zadd mysort 50 lisi
(integer) 1
127.0.0.1:6379> zadd mysort 80 wangwu
(integer) 1
#获取：zrange key start end [withscores] 0 -1 查询所有
127.0.0.1:6379> zrange mysort 0 -1
1) "lisi"
2) "zhangsan"
3) "wangwu"

#获取同时展示分数
127.0.0.1:6379> zrange mysort 0 -1 withscores
1) "zhangsan"
2) "60"
3) "wangwu"
4) "80"
5) "lisi"
6) "50"
#删除：zrem key value
127.0.0.1:6379> zrem mysort lisi
(integer) 1
```

#### 8.2 通用命令

​	1）keys * : 查询所有的键
​	2）type key ： 获取键对应的value的类型
​	3）del key：删除指定的key value

#### 8.3 Jedis-动手练一练

```java
package cn.itcast.jedis.test;

import cn.itcast.jedis.util.JedisPoolUtils;
import org.junit.Test;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * jedis的测试类
 */
public class JedisTest {


    /**
     * 快速入门
     */
    @Test
    public void test1(){
        //1. 获取连接
        Jedis jedis = new Jedis("localhost",6379);
        //2. 操作
        jedis.set("username","zhangsan");
        String value = jedis.get("name");
        jedis.del("username");
        //3. 关闭连接
        jedis.close();
    }


    /**
     * string 数据结构操作
     */
    @Test
    public void test2(){
        //1. 获取连接
        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
        //2. 操作
        //存储
        jedis.set("username","zhangsan");
        jedis.expire("username", 10); // 给 string  key username 设置10秒的过期时间
        //获取
        String username = jedis.get("username");
        System.out.println(username);

        //可以使用setex()方法存储可以指定过期时间的 key value
        jedis.setex("activecode",20,"hehe");//将activecode：hehe键值对存入redis，并且20秒后自动删除该键值对


        jedis.del("activecode");
        //3. 关闭连接
        jedis.close();
    }

    /**
     * hash 数据结构操作
     */
    @Test
    public void test3(){
        //1. 获取连接
        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
        //2. 操作
        // 存储hash
        jedis.hset("user","name","李四");
        jedis.hset("user","age","23");
        jedis.hset("user","gender","female");

        // 获取hash
        String name = jedis.hget("user", "name");
        System.out.println(name);


        // 获取hash的所有map中的数据
        Map<String, String> user = jedis.hgetAll("user");

        // keyset
        Set<String> keySet = user.keySet();
        for (String key : keySet) {
            //获取value
            String value = user.get(key);
            System.out.println(key + ":" + value);
        }

        //3. 关闭连接
        jedis.close();
    }


    /**
     * list 数据结构操作
     */
    @Test
    public void test4(){
        //1. 获取连接
        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口

        jedis.del("mylist");

        //2. 操作
        // list 存储--list允许数据出现重复
        jedis.lpush("mylist","a","b","c");//从左边存
        jedis.rpush("mylist","a","b","c");//从右边存

        // list 范围获取
        List<String> mylist = jedis.lrange("mylist", 0, -1);
        System.out.println(mylist);
        
        // list 弹出
        String element1 = jedis.lpop("mylist");//c
        System.out.println(element1);

        String element2 = jedis.rpop("mylist");//c
        System.out.println(element2);

        // list 范围获取
        List<String> mylist2 = jedis.lrange("mylist", 0, -1);
        System.out.println(mylist2);

        //3. 关闭连接
        jedis.close();
    }



    /**
     * set 数据结构操作
     */
    @Test
    public void test5(){
        //1. 获取连接
        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
        //2. 操作


        // set 存储
        jedis.sadd("myset","java","php","c++");

        // set 获取
        Set<String> myset = jedis.smembers("myset");
        System.out.println(myset);

        //3. 关闭连接
        jedis.close();
    }

    /**
     * sortedset 数据结构操作
     */
    @Test
    public void test6(){
        //1. 获取连接
        Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
        //2. 操作
        // sortedset 存储
        jedis.zadd("mysortedset",3,"亚瑟");
        jedis.zadd("mysortedset",30,"后裔");
        jedis.zadd("mysortedset",55,"孙悟空");

        // sortedset 获取
        Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);

        System.out.println(mysortedset);


        //3. 关闭连接
        jedis.close();
    }

    /**
     * jedis连接池使用
     */
    @Test
    public void test7(){

        //0.创建一个配置对象
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(50);
        config.setMaxIdle(10);

        //1.创建Jedis连接池对象
        JedisPool jedisPool = new JedisPool(config,"localhost",6379);

        //2.获取连接
        Jedis jedis = jedisPool.getResource();
        //3. 使用
        jedis.set("hehe","heihei");


        //4. 关闭 归还到连接池中
        jedis.close();;

    }

    /**
     * jedis连接池工具类使用
     */
    @Test
    public void test8(){
        
        //通过连接池工具类获取
        Jedis jedis = JedisUtils.getJedis();

        //3. 使用
        jedis.set("hello","world");

        //4. 关闭 归还到连接池中
        jedis.close();;

    }

}

```



### 今日重点

- 说出redis存在几种数据类型? 分别是什么？各自的数据结构长什么样？ 特点分别是？并知道其使用场景。
- 至少2遍练习对五种数据类型：增删改查命令的练习。
- 能够使用Jedis框架，通过java代码操作Redis，工具类不要求会写，但是必须要能够看懂，”邹老师提供而工具类能看懂最好, 看不懂会用也行“。
- 练习一遍，java代码对五种数据类型：增删改查操作。
- 能够说出Redis的持久化方式有哪些？他们的区别是？如何在redis.conf 核心配置文件中进行 持久化配置,  并能说出每一项配置什么意思。
- 其他命令和知识点了解即可。