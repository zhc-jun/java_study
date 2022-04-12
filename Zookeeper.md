# Zookeeper

### 学习目标

- 说出zookeeper中 节点的种类？ zookeeper能做哪些事情？
- 敲一遍zookeeper命令，练一练 Curator API
- 仿写一遍 分布式锁，并通过测试
- 搭建zookeeper集群（看个人兴趣选做）



### 1. Zookeeper 概念

- • Zookeeper 是 Apache Hadoop 项目下的一个子项目，是一个树形目录服务。
-  • Zookeeper 翻译过来就是 动物园管理员，他是用来管Hadoop（大象）、Hive(蜜蜂)、Pig(小猪)的管理员。简称zk 
- • Zookeeper 是一个分布式的、开源的分布式应用程序的协调服务。 
- • Zookeeper 提供的主要功能包括：
  - • 配置管理 
  - • 分布式锁
  -  • 集群管理
  - • 注册中心



### 2. zookeeper安装与配置


##### Windows

```java
//下载zookeeper  win 64位压缩包
//解压缩就能用
//F:\worksoft\apache-zookeeper-3.5.6-bin\bin  双击：zkServer.cmd
```

##### Centos7

```shell
#root登录后, 进入到安装目录
[root@192 /]$ cd itcast/install/

#jdk环境 web核心已提供安装教程, zookeeper 需要java环境, 另外zookeeper 默认监听2181端口
#上传zookeeper linux版本压缩包
[root@192 install]$ rz

#解压缩 apache-zookeeper-3.5.6-bin.tar.gz
[root@192 install]$ tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz

#进入zookeeper核心配置目录
[root@192 install]$ cd apache-zookeeper-3.5.6-bin/conf/

#拷贝 zk 核心配置文件
[root@192 conf]$ cp zoo_sample.cfg zoo.cfg

#创建zooKeeper存储目录
[root@192 conf]$ mkdir ../zkdata

#修改zoo.cfg  dataDir=/itcast/install/apache-zookeeper-3.5.6-bin/zkdata
[root@192 conf]$ vim zoo.cfg 

#启动ZooKeeper
[root@192 conf]$ cd ../bin/
[root@192 bin]$ ./zkServer.sh start

#查看ZooKeeper状态, 如果存在：not running 启动失败
[root@192 bin]$ ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /itcast/install/apache-zookeeper-3.5.6-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone

#查看进程获取端口监听都可以监测zk 启动是否成功
[root@192 bin]$ netstat -ant|grep 2181

[root@192 bin]$ ps -ef|grep zookeeper

#查看状态    停止   重启
[root@192 bin]$ ./zkServer.sh status 
[root@192 bin]$ ./zkServer.sh stop
[root@192 bin]$ ./zkServer.sh restart
```



### 3. 命令操作

**Zookeeper 数据模型**

- • ZooKeeper是一个树形目录服务,其数据模型和Unix的文件系统目录树很类似，拥有一个层次化结构。 

- • 这里面的每一个节点都被称为：ZNode，每个节点上都会保存自己的数据和节点信息。 

- • 节点可以拥有子节点，同时也允许少量（1MB）数据存储在该节点之下。 

- • 节点可以分为**四大类**： 

  - • **PERSISTENT 持久化节点** 

       **介绍**： 一旦创建，除非主动调用删除操作，否则一直存储在zk上 

  - • **EPHEMERAL 临时节点** ：-e 

       **介绍**：与客户端会话绑定，一旦客户端会话失效，这个客户端所创建的所有临时节点都会被移除； 

  - • **PERSISTENT_SEQUENTIAL 持久化顺序节点** ：-s

  - • **EPHEMERAL_SEQUENTIAL 临时顺序节点** ：-es

#### 3.1 ZK Server 常用命令

```shell
#启动 ZooKeeper 服务
[root@192 bin]$ ./zkServer.sh start 

#查看 ZooKeeper 服务状态
[root@192 bin]$ ./zkServer.sh status 

#停止 ZooKeeper 服务
[root@192 bin]$ ./zkServer.sh stop 

#重启 ZooKeeper 服务
[root@192 bin]$ ./zkServer.sh restart

```



#### 3.2 ZK Client 常用命令

- 节点可以有子节点
- 临时节点一点客户端断开，该客户端创建所有临时节点都删除
- <font color=red>注意: 右上角关闭客户端黑窗口,  并不是断开连接，或者跟quit  有本质上区别</font>
- 临时节点和永久节点的顺序**共用一个计数器**，当临时节点消失时，计数器记录的当前数值并不会做-1操作，只会继续做+1操作

```shell
#连接ZooKeeper服务端
[root@192 bin]$ ./zkCli.sh –server ip:port

#断开连接
[zk: localhost:2181]$ quit 

#查看命令帮助
[zk: localhost:2181]$ help

##################### 永久节点操作 ###################

#显示指定目录下节点  ls / 查看根节点
[zk: localhost:2181]$ ls 节点名称

#创建节点
[zk: localhost:2181]$ create /节点名称 value 

#获取节点值 
[zk: localhost:2181]$ get /节点名称

#设置节点值 value 选填项
[zk: localhost:2181]$ set /节点名称  [value]
#删除单个节点
[zk: localhost:2181]$ delete /节点名称 

#删除带有子节点的节点 
[zk: localhost:2181]$ deleteall /节点名称

##################### 临时节点操作 ###################

#创建临时节点 
[zk: localhost:2181]$ create -e /节点名称  [value]
#创建顺序节点
[zk: localhost:2181]$ create -s /节点名称  [value]

#创建临时顺序节点
[zk: localhost:2181]$ create -es /节点名称  [value]

#查询节点详细信息
[zk: localhost:2181]$ ls –s /节点名称
• czxid：节点被创建的事务ID 
• ctime: 创建时间 
• mzxid: 最后一次被更新的事务ID 
• mtime: 修改时间 
• pzxid：子节点列表最后一次被更新的事务ID 
• cversion：子节点的版本号
• dataversion：数据版本号 
• aclversion：权限版本号 
• ephemeralOwner：用于临时节点，代表临时节点的事务ID，如果为持 久节点则为0 • dataLength：节点存储的数据的长度 
• numChildren：当前节点的子节点个数
```



### 4.  JavaAPI 操作（练一练）

#### Curator 介绍

- • Curator 是 Apache ZooKeeper的Java客户端库。
- • 常见的ZooKeeper Java API ：
  -  • 原生Java API 
  - • ZkClient 
  - • Curator ：Curator 项目的目标是简化 ZooKeeper 客户端的使用。
-  • Curator 最初是 Netfix 研发的,后来捐献了Apache 基金会,目前是Apache 的顶级项目。 • 官网：http://curator.apache.org/



#### 4.1 Curator API 操作一

-  • **建立连接 • 添加节点 • 删除节点 • 修改节点 • 查询节点** • Watch事件监听 • 分布式锁实现

##### 建立连接

```java
    public void testConnect() {

        /*
         *
         * @param connectString       连接字符串。zk server 地址和端口 "192.168.149.135:2181,192.168.149.136:2181"
         * @param sessionTimeoutMs    会话超时时间 单位ms
         * @param connectionTimeoutMs 连接超时时间 单位ms
         * @param retryPolicy         重试策略
         */

        //重试策略
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000,10);
        
        //1.第一种方式
        /*CuratorFramework client = CuratorFrameworkFactory.newClient("192.168.149.135:2181",
                60 * 1000, 15 * 1000, retryPolicy);*/

        //2.第二种方式
        //CuratorFrameworkFactory.builder();
        client = CuratorFrameworkFactory.builder()
                .connectString("127.0.0.1:2181")
//                .connectString("192.168.149.135:2181")
                .sessionTimeoutMs(60 * 1000)
                .connectionTimeoutMs(15 * 1000)
                .retryPolicy(retryPolicy)
                .namespace("itheima")
                .build();

        //开启连接
        client.start();

    }
```

#####  添加节点

- 永久节点: PERSISTENT
- 永久顺序节点: PERSISTENT_SEQUENTIAL
- 临时节点: EPHEMERAL, 
- 临时顺序节点: EPHEMERAL_SEQUENTIAL

```java
    /**
     * 创建节点：create 持久 临时 顺序 数据
     * 1. 基本创建 ：create().forPath("")
     * 2. 创建节点 带有数据:create().forPath("",data)
     * 3. 设置节点的类型：create().withMode().forPath("",data)
     * 4. 创建多级节点  /app1/p1 ：create().creatingParentsIfNeeded().forPath("",data)
     */
    @Test
    public void testCreate() throws Exception {
        //2. 创建节点 带有数据
        //如果创建节点，没有指定数据，则默认将当前客户端的ip作为数据存储
        String path = client.create().forPath("/app2", "hehe".getBytes());
        System.out.println(path);
    }

    @Test
    public void testCreate2() throws Exception {
        //1. 基本创建
        //如果创建节点，没有指定数据，则默认将当前客户端的ip作为数据存储
        String path = client.create().forPath("/app1");
        System.out.println(path);
    }

    @Test
    public void testCreate3() throws Exception {
        //3. 设置节点的类型
        //默认类型：
        // 永久节点: PERSISTENT，
        // 永久顺序节点: PERSISTENT_SEQUENTIAL,
        // 临时节点: EPHEMERAL, 
        // 临时顺序节点: EPHEMERAL_SEQUENTIAL
        String path = client.create().withMode(CreateMode.EPHEMERAL).forPath("/app3");
        System.out.println(path);
    }

    @Test
    public void testCreate4() throws Exception {
        //4. 创建多级节点  /app1/p1
        //creatingParentsIfNeeded():如果父节点不存在，则创建父节点
        String path = client.create().creatingParentsIfNeeded().forPath("/app4/p1");
        System.out.println(path);
    }
```

##### 查询节点

```java
    /**
     * 查询节点：
     * 1. 查询数据：get: getData().forPath()
     * 2. 查询子节点： ls: getChildren().forPath()
     * 3. 查询节点状态信息：ls -s:getData().storingStatIn(状态对象).forPath()
     */

    @Test
    public void testGet1() throws Exception {
        //1. 查询数据：get
        byte[] data = client.getData().forPath("/app1");
        System.out.println(new String(data));
    }

    @Test
    public void testGet2() throws Exception {
        // 2. 查询子节点： ls
        List<String> path = client.getChildren().forPath("/");

        System.out.println(path);
    }

    @Test
    public void testGet3() throws Exception {

        Stat status = new Stat();
        System.out.println(status);
        //3. 查询节点状态信息：ls -s
        client.getData().storingStatIn(status).forPath("/app1");

        System.out.println(status);

    }
```

##### 修改节点

```java
    /**
     * 修改数据
     * 1. 基本修改数据：setData().forPath()
     * 2. 根据版本修改: setData().withVersion().forPath()
     * * version 是通过查询出来的。目的就是为了让其他客户端或者线程不干扰我。
     *
     * @throws Exception
     */
    @Test
    public void testSet() throws Exception {
        client.setData().forPath("/app1", "itcast".getBytes());
    }


    @Test
    public void testSetForVersion() throws Exception {

        Stat status = new Stat();
        //3. 查询节点状态信息：ls -s
        client.getData().storingStatIn(status).forPath("/app1");


        int version = status.getVersion();//查询出来的 3
        System.out.println(version);
        client.setData().withVersion(version).forPath("/app1", "hehe".getBytes());
        
    }
```

##### 删除节点

```java
    /**
     * 删除节点： delete deleteall
     * 1. 删除单个节点:delete().forPath("/app1");
     * 2. 删除带有子节点的节点:delete().deletingChildrenIfNeeded().forPath("/app1");
     * 3. 必须成功的删除:为了防止网络抖动。本质就是重试。  client.delete().guaranteed().forPath("/app2");
     * 4. 回调：inBackground
     * @throws Exception
     */
    @Test
    public void testDelete() throws Exception {
        // 1. 删除单个节点
        client.delete().forPath("/app1");
    }

    @Test
    public void testDelete2() throws Exception {
        //2. 删除带有子节点的节点
        client.delete().deletingChildrenIfNeeded().forPath("/app4");
    }
    @Test
    public void testDelete3() throws Exception {
        //3. 必须成功的删除
        client.delete().guaranteed().forPath("/app2");
    }

    @Test
    public void testDelete4() throws Exception {
        //4. 回调
        client.delete().guaranteed().inBackground(new BackgroundCallback(){

            @Override
            public void processResult(CuratorFramework client, CuratorEvent event) throws Exception {
                System.out.println("我被删除了~");
                System.out.println(event);
            }
        }).forPath("/app1");
        
        
    }
```







#### 4.2 Curator API 操作二

##### Watch事件监听 

- ZooKeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通 知到感兴趣的客户端上去，该机制是ZooKeeper实现分布式协调服务的重要特性。
- ZooKeeper中引入了Watcher机制来实现了发布/订阅功能能，能够让多个订阅者同时监听某一个对象，当一个对象自身 状态变化时，会通知所有订阅者。 
- ZooKeeper原生支持通过注册Watcher来进行事件监听，但是其使用并不是特别方便 需要开发人员自己反复注册Watcher，比较繁琐。
- Curator引入了Cache 来实现对 ZooKeeper服务端事件的监听。 
-  ZooKeeper提供了**三种Watcher**： 
  - • NodeCache: 只是监听某一个特定的节点 
  - • PathChildrenCache : 监控一个ZNode的子节点. 
  - • TreeCache : 可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合 

###### NodeCache

- NodeCache：对一个节点进行注册监听器，监听事件包括指定路径的增删改操作

```java
    /**
     * 演示 NodeCache：对一个节点进行注册监听器
     * @deprecated 监听事件包括指定路径的增删改操作
     */
    @Test
    public void testNodeCache() throws Exception {
        //1. 创建NodeCache对象
        final NodeCache nodeCache = new NodeCache(client,"/app1");
        //2. 注册监听
        nodeCache.getListenable().addListener(new NodeCacheListener() {

            @Override
            public void nodeChanged() throws Exception {
                System.out.println("节点变化了~");

                //获取修改节点后的数据
                byte[] data = nodeCache.getCurrentData().getData();
                System.out.println(new String(data));
            }
        });

        //3. 开启监听.如果设置为true，则开启监听是，加载缓冲数据
        nodeCache.start(true);

        while (true){

        }

    }
```

###### PathChildrenCache

- PathChildrenCache：监听某个节点的所有子节点们，对指定路径节点的一级子目录监听，不对该节点的操作监听，对其子目录的增删改操作监听

```java
    /**
     * 演示 PathChildrenCache：监听某个节点的所有子节点们
     * @deprecated 对指定路径节点的一级子目录监听，不对该节点的操作监听，对其子目录的增删改操作监听
     */
    @Test
    public void testPathChildrenCache() throws Exception {
        //1.创建监听对象
        PathChildrenCache pathChildrenCache = new PathChildrenCache(client,"/app2",true);

        //2. 绑定监听器
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                System.out.println("子节点变化了~");
                System.out.println(event);
                //监听子节点的数据变更，并且拿到变更后的数据
                //1.获取类型
                PathChildrenCacheEvent.Type type = event.getType();
                //2.判断类型是否是update
                if(type.equals(PathChildrenCacheEvent.Type.CHILD_UPDATED)){
                    System.out.println("数据变了！！！");
                    byte[] data = event.getData().getData();
                    System.out.println(new String(data));

                }
            }
        });
        //3. 开启
        pathChildrenCache.start();

        while (true){

        }
    }
```

###### TreeCache

- TreeCache：监听某个节点自己和所有子节点们，综合NodeCache和PathChildrenCahce的特性，是对整个目录进行监听，可以设置监听深度。

```java
    /**
     * 演示 TreeCache：监听某个节点自己和所有子节点们
     * @deprecated 综合NodeCache和PathChildrenCahce的特性，是对整个目录进行监听，可以设置监听深度。
     */
    @Test
    public void testTreeCache() throws Exception {
        //1. 创建监听器
        TreeCache treeCache = new TreeCache(client,"/app2");

        //2. 注册监听
        treeCache.getListenable().addListener(new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {
                System.out.println("节点变化了");
                System.out.println(event);
            }
        });

        //3. 开启
        treeCache.start();

        while (true){

        }
    }
```



#### 4.3 分布式锁实现

-  在我们进行单机应用开发，涉及并发同步的时候，我们往往采用synchronized或者Lock的方式来解决多线程间的代码同步问题， 这时多线程的运行都是在同一个JVM之下，没有任何问题。 
- 但当我们的应用是分布式集群工作的情况下，属于多JVM下的工作环境，跨JVM之间已经无法通过多线程的锁解决同步问题。
- 那么就需要一种更加高级的锁机制，来处理种<font color=red>跨机器的进程之间的数据同步问题——这就是分布式锁。</font>

##### ZK 分布式锁原理

- **核心思想**：当客户端要获取锁，则先创建节点，使用完锁，则删除该节点。 
  1. 客户端获取锁时，在lock节点下创建临时顺序节点。
  2.  然后获取lock下面的所有子节点，客户端获取到所有的子节点之后，如果发现自己创建的子节点序号排在第一个（最小），那么就认为该 客户端获取到了锁。使用完锁后，将自己创建的该节点删除。 
  3. 如果发现自己创建的节点并非lock所有子节点中第一个（最小），说明自己还没有获取到锁，此时客户端需要找到比自己小的 节点，同时对该节点注册事件监听器，监听删除事件。
  4. 如果发现自己前面那个节点被删除，则客户端的 Watcher会收到相应通知，此时再次判断自己创建的节点 是否是lock子节点中序号最小的，如果是则获取到了锁， 如果不是则重复以上步骤继续获取到比自己小的一个节点 并注册监听。

##### Curator 实现

- InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）
- InterProcessReadWriteLock：分布式读写锁
- InterProcessMultiLock：将多个锁作为单个实体管理的容器
- InterProcessSemaphoreV2：共享信号量
- InterProcessMutex：分布式可重入排它锁

###### ZkCuratorLock.java

- 实现 java.util.concurrent.locks.Lock 接口, 符合规范

- 实现核心方法：

  1）lock() 加锁

  2）tryLock() 尝试加锁, 返回值true/false

  3）unlock() 释放锁

```java
package com.zou;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.data.Stat;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

public class ZkCuratorLock implements Lock {

	private static final String LOCK_PATH="/LOCK";

	private static final String ZOOKEEPER_IP_PORT ="localhost:2181"; //192.168.149.135:2181

	// InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）
	// InterProcessReadWriteLock：分布式读写锁
	// InterProcessMultiLock：将多个锁作为单个实体管理的容器
	// InterProcessSemaphoreV2：共享信号量
	// InterProcessMutex：分布式可重入排它锁
	private InterProcessMutex lock ;

	//客户端连接对象
	private CuratorFramework client;


	//判断有没有LOCK目录，没有则创建
	public ZkCuratorLock() {
		//重试策略
		RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000, 10);
		client = CuratorFrameworkFactory.builder()
				.connectString(ZOOKEEPER_IP_PORT)
				.sessionTimeoutMs(60 * 1000)
				.connectionTimeoutMs(15 * 1000)
				.retryPolicy(retryPolicy)
				.build();
		//开启连接
		client.start();

		try {
			//判断节点是否存在
			Stat stat = client.checkExists().forPath(LOCK_PATH);
			if (stat == null){
				//节点不存在，创建永久节点
				client.create().withMode(CreateMode.PERSISTENT).forPath(LOCK_PATH);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		//创建分布式可重入排它锁
		lock = new InterProcessMutex(client, LOCK_PATH);

	}

	/**
	 * 加锁
	 */
	@Override
	public void lock() {
		if(tryLock()){
			//获得 Zk 分布式锁!
			return;
		}
		//没有获取到继续尝试获取
		lock();
	}

	/**
	 * 尝试加锁
	 * @return
	 */
	@Override
	public boolean tryLock() {
		try {
			return lock.acquire(3, TimeUnit.SECONDS);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return false;
	}

	/**
	 * 释放锁
	 */
	@Override
	public void unlock() {
		//删除当前临时节点
		//释放锁
		try {
			lock.release();
			System.out.println("---------------------------->" + Thread.currentThread().getName()+" 加锁/释放锁!");
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
	
	//==========================================如下方法无需实现
	@Override
	public void lockInterruptibly() throws InterruptedException {
		// TODO Auto-generated method stub

	}

	@Override
	public boolean tryLock(long time, TimeUnit unit)
			throws InterruptedException {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public Condition newCondition() {
		// TODO Auto-generated method stub
		return null;
	}

}

```

###### Ticket12306.java

- 该类是核心业务类，负责对外提供 火车票
- getTicket() 内部实现 总票数 - 1, 方法调用时需要加锁控制

```java
package com.zou;

public class Ticket12306 {

    private int ticketsTotal = 10; //数据库的票总数

    private int ticketsCurrent = ticketsTotal; //当前剩余票数

    public Ticket12306(){
    }

    public Ticket12306(int tickets){
        this.ticketsTotal = tickets;
    }

    /**
     * 获取一张票
     * @param component 其他公司： 飞猪 携程 去哪儿 智行
     */
    public String getTicket(String component) {

        //票数量> 0 时, 继续递减
        if(ticketsCurrent > 0 ){
            ticketsCurrent--;
        }

        return Thread.currentThread().getName()+"[" + component + "]: 总票数 = " + ticketsTotal + ", 剩余票数 = " + ticketsCurrent;
    }


}

```

###### ZkLockTest.java

- 测试类，模拟多平台并发抢票
  - int NUM  //线程总数
  - CountDownLatch cdl  //发令枪，可以控制多个线程同时并发执行
  - new String[]{ "飞猪", "携程", "去哪儿", "智行"};   //初始化多个平台去12306抢票
  - Ticket12306  ticket12306  //12306核心业务方法执行对象

```java
package com.zou;

import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.locks.Lock;

public class ZkLockTest implements Runnable {

    // 线程数
    private static final int NUM = 1;

    //发令枪，可以控制多个线程同时并发执行, countDown()每次调用 NUM - 1, 为0时, 自动唤醒所有cdl.await() 线程
    private static CountDownLatch cdl = new CountDownLatch(NUM);

    //12306 核心业务对象
    private static Ticket12306 ticket12306 = new Ticket12306();

    //公司列表
    private static String[] components = new String[]{ "飞猪", "携程", "去哪", "智行"};

    //创建分布式锁对象
	private Lock lock = new ZkCuratorLock();

    public static void main(String[] args) {

        for (int i = 1; i <= NUM; i++) {

            // 按照线程数迭代实例化线程
            new Thread(new ZkLockTest()).start();
            // 创建一个线程，倒计数器减1, 当值减为 0 时, 会唤醒所有wait线程
            cdl.countDown();
        }

    }

    @Override
    public void run() {

        String result = "";
        long startTime = 0L;
        //等待其他线程初始化完成后, 一起往下执行
        try {
            cdl.await();

            //随机获取一家公司
            String component = components[new Random().nextInt(components.length)];

            startTime = System.currentTimeMillis();

            lock.lock();
            //获取12306票
            result = ticket12306.getTicket(component);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();

            long runTime = (System.currentTimeMillis() - startTime);
            System.out.println(result + ", 总耗时 = " + runTime + " 毫秒!");
        }
    }


}
```

###### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>curator-zk</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>

        <!--curator-->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>4.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.0.0</version>
        </dependency>

    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```





### 5. 集群

#### Leader选举

- • Serverid：服务器ID 比如有三台服务器，编号分别是1,2,3。 编号越大在选择算法中的权重越大。 
- • Zxid：数据ID 服务器中存放的最大数据ID.值越大说明数据 越新 ，在选举算法中数据越新权重越大。
-  • 在Leader选举的过程中，如果某台ZooKeeper 获得了超过半数的选票， 则此ZooKeeper就可以成为Leader了。



#### 搭建

```shell
#root登录后, 进入到安装目录
[root@192 /]$ cd itcast/install/

#jdk环境 web核心已提供安装教程, zookeeper 需要java环境, 另外zookeeper 默认监听2181端口
#上传zookeeper linux版本压缩包
[root@192 install]$ rz

#解压缩 apache-zookeeper-3.5.6-bin.tar.gz
[root@192 install]$ tar -zxvf apache-zookeeper-3.5.6-bin.tar.gz

#创建集群zookeeper-cluster目录
[root@192 install]$ mkdir zookeeper-cluster

#将解压后的apache-zookeeper-3.5.6-bin复制到以下三个目录
[root@192 install]$ cp -r apache-zookeeper-3.5.6-bin  zookeeper-cluster/zookeeper-1
[root@192 install]$ cp -r apache-zookeeper-3.5.6-bin  zookeeper-cluster/zookeeper-2
[root@192 install]$ cp -r apache-zookeeper-3.5.6-bin  zookeeper-cluster/zookeeper-3

#创建data目录, 并且将 conf下zoo_sample.cfg 文件改名为 zoo.cfg
[root@192 install]$ mkdir zookeeper-cluster/zookeeper-1/data
[root@192 install]$ mkdir zookeeper-cluster/zookeeper-2/data
[root@192 install]$ mkdir zookeeper-cluster/zookeeper-3/data

[root@192 install]$ mv  zookeeper-cluster/zookeeper-1/conf/zoo_sample.cfg   zookeeper-cluster/zookeeper-1/conf/zoo.cfg

[root@192 install]$ mv  zookeeper-cluster/zookeeper-2/conf/zoo_sample.cfg   zookeeper-cluster/zookeeper-2/conf/zoo.cfg

[root@192 install]$ mv  zookeeper-cluster/zookeeper-3/conf/zoo_sample.cfg   zookeeper-cluster/zookeeper-3/conf/zoo.cfg


#配置每一个Zookeeper 的dataDir 和 clientPort 分别为2181  2182  2183
[root@192 install]$ vim zookeeper-cluster/zookeeper-1/conf/zoo.cfg
clientPort=2181
dataDir=/itcast/install/zookeeper-cluster/zookeeper-1/data

[root@192 install]$ vim zookeeper-cluster/zookeeper-2/conf/zoo.cfg
clientPort=2182
dataDir=/itcast/install/zookeeper-cluster/zookeeper-2/data

[root@192 install]$ vim zookeeper-cluster/zookeeper-3/conf/zoo.cfg
clientPort=2183
dataDir=/itcast/install/zookeeper-cluster/zookeeper-3/data


#在每个zookeeper的 data 目录下创建一个 myid 文件, 内容分别是1、2、3.这个文件就是记录每个服务器的ID
[root@192 install]$ echo 1 >/itcast/install/zookeeper-cluster/zookeeper-1/data/myid
[root@192 install]$ echo 2 >/itcast/install/zookeeper-cluster/zookeeper-2/data/myid
[root@192 install]$ echo 3 >/itcast/install/zookeeper-cluster/zookeeper-3/data/myid

#在每一个zookeeper的zoo.cfg 配置客户端访问端口和集群服务器IP列表。
[root@192 install]$ vim /itcast/install/zookeeper-cluster/zookeeper-1/conf/zoo.cfg
[root@192 install]$ vim /itcast/install/zookeeper-cluster/zookeeper-2/conf/zoo.cfg
[root@192 install]$ vim /itcast/install/zookeeper-cluster/zookeeper-3/conf/zoo.cfg

#如下配置在三个zoo.cfg 都一样
#解释: server.服务器ID=服务器IP地址:服务器之间通信端口:服务器之间投票选举端口
#因为我们只在一台机器上部署集群，所以下方端口都要唯一, 多台机器无所谓
server.1=192.168.149.135:2881:3881
server.2=192.168.149.135:2882:3882
server.3=192.168.149.135:2883:3883

```



#### 测试

```shell
#启动集群
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh start
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh start
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh start

#查看集群状态
#Mode为follower表示是**跟随者**（从）
#Mode为leader表示是**领导者**（主）
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh status
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh status
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh status


#模拟集群异常**
#（1）首先我们先测试如果是从服务器挂掉，会怎么样
# 把3号服务器停掉，观察1号和2号，发现状态并没有变化
# 结论，3个节点的集群，从服务器挂掉，集群正常
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh stop

[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh status
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh status

#（2）我们再把1号服务器（从服务器）也停掉，查看2号（主服务器）的状态，发现已经停止运行了。
# 结论，3个节点的集群，2个从服务器都挂掉，主服务器也无法运行。因为可运行的机器没有超过集群总数量的半数
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh stop

[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh status


#（3）我们再次把1号服务器启动起来，发现2号服务器又开始正常工作了。而且依然是领导者。
#
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh start

[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh status

#（4）我们把3号服务器也启动起来，把2号服务器停掉,停掉后观察1号和3号的状态。
# 结论，当集群中的主服务器挂了，集群中的其他服务器会自动进行选举状态，然后产生新得leader 
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh start
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh stop

[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-1/bin/zkServer.sh status
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh status

#（5）我们再次测试，当我们把2号服务器重新启动起来启动后，会发生什么？2号服务器会再次成为新的领导吗？我们看结果
# 2号服务器启动后依然是跟随者（从服务器），3号服务器依然是领导者（主服务器），没有撼动3号服务器的领导地位。
# 结论，当领导者产生后，再次有新服务器加入集群，不会影响到现任领导者。
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh start

[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-2/bin/zkServer.sh status
[root@192 install]$ /itcast/install/zookeeper-cluster/zookeeper-3/bin/zkServer.sh status

```



#### 三个角色

为了避免 Zookeeper 的单点问题，zk 也是以集群的形式出现的。zk 集群中的角色主要有 以下三类： 

- Leader 领导者 ： 
  - zk 集群写请求的唯一处理者，并负责进行投票的发起和决议，更新系统状态。 Leader 是很民主的，并不是说其在接收到写请求后马上就修改其中保存的数据，而是首 先根据写请求提出一个提议，在大多数 zkServer 均同意时才会做出修改。 
- Follower 跟随者 ： 
  - 接收客户端请求，处理读请求，并向客户端返回结果；将写请求转给 Leader； 在选主(选 Leader)过程中参与投票。

- Observer 观察者： 
  - 可以理解为无选主投票权的 Flollower，其主要是为了协助 Follower 处理更多 的读请求。如果 Zookeeper 集群的读请求负载很高，或者客户端非常非常多，多到跨机 房，则可以设置一些 Observer 服务器，以提高读取的吞吐量。 

### 6.扩展

#### 6.1 三种模式 

Zookeeper 的核心是广播，这个机制保证了各个 zkServer 间数据的同步，即数据的一致 性。实现这个机制的协议叫做 ZAB 协议，即 Zookeeper Atomic Broadcast，Zookeeper 原子广 播协议。ZAB 协议有三种模式：恢复模式、同步模式和广播模式。

- 恢复模式：在服务重启过程中，或在 Leader 崩溃后，就进入了恢复模式，要恢复到 zk 集群正常的工作状态。 
- 同步模式：在所有的 zkServer 启动完毕，或 Leader 崩溃后又被选举出来时，就进入了同步模式，各个 Follower 需要马上将 Leader 中的数据同步到自己的主机中。当大多数 zkServer 完成了与 Leader 的状态同步以后，恢复模式就结束了。所以，同步模式包含在 恢复模式过程中。 
-  广播模式：当 Leader 的提议被大多数 zkServer 同意后，Leader 会修改自身数据，然后 会将修改后的数据广播给其它 Follower。 

 

#### 6.2 zoo.cfg 配置介绍

- tickTime：zk 使用的基本时间单位为 tick，该属性设置一个 tick 的时间长度，单位毫秒。 
- initLimit：Follower 初始化同步的超时时限，单位为 tickTime 的倍数。集群启动或新 Leader 选举完成后，Follower 需要从 Leader 同步数据，如果需要同步的数据量较大，则可能超 时。一旦超时，同步失效。 
- syncLimit：Follower 更新同步的超时时限，单位为 tickTime 的倍数。后续同步与初始化 同步的不同点在于，初始化同步的数据量要大于后续同步的，后续同步的数据量一般是 较小的。所以，对于 initLimit 是否超时，更多是取决于网络质量。如果超时，Leader 就 认为该 Follower 落后太多需要放弃。如果网络为高延迟环境，可适当增加该值，但不宜 增加太大，否则会掩盖某些故障。
- dataDir：ZKServer 的内存中维护着一个内存数据库，像域名服务 zk 中的服务映射表就 存放在该内存数据库中。该路径为内存数据库快照文件 snapshot 的存储路径。默认给 出的路径在/tmp 中，提示也指出，不要将快照存储路径设置到/tmp 中。 
- clientPort：客户端连接 zkServer 的端口号，每台 ZKServer 允许设置为不同的值。默认配 置文件设定的是 2181。 



#### 6.3 Leader 的选举机制 

##### 基本概念 

###### myid 

这是 zk 集群中服务器的唯一标识，称为 myid。例如，有三个 zk 服务器，那么编号分别 是 1,2,3。 

###### zxid 

zxid 为 Long 类型，其中高 32 位表示 epoch，低 32 位表示 xid。即 zxid 由两部分构成： epoch 与 xid。 每个 Leader 都会具有一个不同的 epoch 值，表示一个时期、时代。新的 Leader 产生， 则会更新所有 zkServer 的 zxid 中的 epoch。 而 xid 则为 zk 的事务 id，每一个写操作都是一个事务，都会有一个 xid。每一个写操作 都需要由 Leader 发起一个提议，由所有 Follower 表决是否同意本次写操作。 

###### 逻辑时钟 

逻辑时钟，Logicalclock，是一个整型数，该概念在选举时称为 logicalclock，而在 zxid 中 则为 epoch 的值。即 epoch 与 logicalclock 是同一个值，在不同情况下的不同名称。 

###### 选举状态 

- LOOKING，选举状态(查找 Leader 的状态)。 
- FOLLOWING，随从状态，同步 leader 状态。处于该状态的服务器称为 Follower。 
- OBSERVING，观察状态，同步 leader 状态。处于该状态的服务器称为 Observer。 
- LEADING，领导者状态。处于该状态的服务器称为 Leader



##### 选举算法 

​       各个服务器都要投票进行 Leader 的选举，大家投票的标准是：epoch 大者投之；epoch 相等，则 xid 大者投之；xid 相等，则 myid 大者投之；最终，得票最多者当选。其实，简单 来说就是 zxid(epoch, xid)大者投之，否则 myid 大者投之。 

​       对于“myid 大者投之”这条规则好理解，就是人为指定的一种标准，但为什么要有“epoch 大者投之”与“xid 大者投之”的标准呢？

​        因为 epoch 其实表示的是发起 Leader 选举的时间顺序，越靠后发起的选举，epoch 的值 越大。若 epochA 大于 epochB，则说明 epochB 的选举已经过时，有可能 epochB 时代选举出的 Leader 已经宕机，现在又要发起新的选举了。所以，epoch 大者就是最近的选举，应遵从 最近选举的结果。 

​        那么，为什么 xid 大者要投之呢？xid 即事务 id，每一个写请求都会对应一个 xid。其投 票中的 xid 大，则说明其具有最新的写操作信息，那么，其就应作为 Leader，让其它 Server 来同步该写操作。 



#####  Leader 选举时机 

- 在以下两种情况下会发生 Leader 选举。 

###### 服务器启动时 

​     现在以 3 台服务器为例来分析选举过程。这 3 台服务器的 myid 编号分别是 1、2、3， 然后按编号依次启动，它们的选择举过程为： 

- 服务器 1 启动，给自己投票，然后发布自己的投票结果(myid,zxid)，由于其它机器还没 有启动所以它收不到反馈信息，服务器 1 的状态一直属于 Looking，即属于非服务状态。 
- 服务器 2 启动，给自己投票，然后发布自己的投票结果(myid,zxid)，其 zxid 与服务器 1 的相同，在与服务器 1 交换结果后，由于服务器 2 的 myid 大，所以服务器 2 胜出，此时服 务器 2 获取到的投票数为 2，大于半数 1，所以服务器 2 成为 Leader，服务器 1 成为小弟。 
- 服务器 3 启动，同样给自己投票，同时与之前启动的服务器 1,2 交换信息，尽管服务器 3 的 myid 大，但之前服务器 2 已经胜出，所以服务器 3 只能成为小弟。 

###### 原Leader宕机后 

- 在 Zookeeper 运行期间，Leader 与非 Leader 服务器(Follower 与 Observer)各司其职，即 便当有非 Leader 服务器宕机或新加入，此时也不会影响 Leader，但是一旦 Leader 服务器宕 机，那么整个集群将暂停对外服务，进入新一轮 Leader 选举，其过程和启动时期的 Leader 选举过程基本一致。
- 假设正在运行的有 Server1、Server2、Server3 三台服务器，当前 Leader 是 Server2，若 某一时刻 Leader 挂了，此时便开始 Leader 选举。 
- 首先这些活着的 Server 会修改状态为 LOOKING，进入选举状态。需要注意，在运行期 间，每个服务器上的 ZXID 可能是不同的。此时投票，zxid 大者会作为新的 Leader。



#### 6.4 zkClient 实现分布式锁

##### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>curator-zk</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>

        <!-- zkclient依赖 -->
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>

    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```

##### ZkClientLock.java

```java 
package com.zou.zkclient;

import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.ZkClient;
import org.I0Itec.zkclient.serialize.SerializableSerializer;

import java.util.Collections;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

public class ZkClientLock implements Lock {

	private static final String LOCK_PATH="/LOCK";

	private static final String ZOOKEEPER_IP_PORT ="localhost:2181";

	private ZkClient client = new ZkClient(ZOOKEEPER_IP_PORT, 1000, 1000, new SerializableSerializer());

	private CountDownLatch cdl;

	private String beforePath;//当前请求的节点
	private String currentPath;//当前请求的节点前一个节点

	//判断有没有LOCK目录，没有则创建
	public ZkClientLock() {
		if (!this.client.exists(LOCK_PATH)) {
			this.client.createPersistent(LOCK_PATH);
		}
	}

	/**
	 * 加锁
	 */
	@Override
	public void lock() {
		if(!tryLock()){
			waitForLock();
			lock();
		}else{
			//获得 Zk 分布式锁!
		}

	}

	/**
	 * 等待加锁
	 */
	private void waitForLock() {

		/**
		 * 监听节点是否删除
		 */
		IZkDataListener listener = new IZkDataListener() {

			@Override
			public void handleDataDeleted(String dataPath) throws Exception {
				if(cdl!=null){
					cdl.countDown();
				}
			}

			@Override
			public void handleDataChange(String dataPath, Object data) throws Exception {

			}
		};
		//给之前的节点增加数据删除的watcher
		this.client.subscribeDataChanges(beforePath, listener);

		if(this.client.exists(beforePath)){
			cdl=new CountDownLatch(1);
			try {
				cdl.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		this.client.unsubscribeDataChanges(beforePath, listener);

	}

	/**
	 * 尝试加锁
	 * @return
	 */
	@Override
	public boolean tryLock() {
		//如果currentPath为空则为第一次尝试加锁，第一次加锁赋值currentPath
		if(currentPath == null || currentPath.length()<= 0){
			//创建一个临时顺序节点
			currentPath = this.client.createEphemeralSequential(LOCK_PATH + '/',"lock");
			System.out.println("---------------------------->"+currentPath);
		}
		//获取所有临时节点并排序，临时节点名称为自增长的字符串如：0000000400
		List<String> childrens = this.client.getChildren(LOCK_PATH);
		Collections.sort(childrens);

		if (currentPath.equals(LOCK_PATH + '/'+childrens.get(0))) {//如果当前节点在所有节点中排名第一则获取锁成功
			return true;
		} else {//如果当前节点在所有节点中排名中不是排名第一，则获取前面的节点名称，并赋值给beforePath
			int wz = Collections.binarySearch(childrens,
					currentPath.substring(6));
			beforePath = LOCK_PATH + '/'+childrens.get(wz-1);
		}
		return false;
	}

	/**
	 * 释放锁
	 */
	@Override
	public void unlock() {
		//删除当前临时节点
		client.delete(currentPath);
		System.out.println("---------------------------->" + Thread.currentThread().getName()+" 加锁/释放锁!");
	}


	//==========================================如下方法无需实现
	@Override
	public void lockInterruptibly() throws InterruptedException {
		// TODO Auto-generated method stub

	}

	@Override
	public boolean tryLock(long time, TimeUnit unit)
			throws InterruptedException {
		// TODO Auto-generated method stub
		return false;
	}

	@Override
	public Condition newCondition() {
		// TODO Auto-generated method stub
		return null;
	}

}
```

##### ZkClientLockTest.java

```java
package com.zou.zkclient;

import com.zou.Ticket12306;

import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.locks.Lock;

public class ZkClientLockTest implements Runnable {

	// 线程数
	private static final int NUM = 20;

	//发令枪，可以控制多个线程同时并发执行, countDown()每次调用 NUM - 1, 为0时, 自动唤醒所有cdl.await() 线程
	private static CountDownLatch cdl = new CountDownLatch(NUM);

	//12306 核心业务对象
	private static Ticket12306 ticket12306 = new Ticket12306();

	//公司列表
	private static String[] components = new String[]{ "飞猪", "携程", "去哪", "智行"};

	private Lock lock = new ZkClientLock();


	public static void main(String[] args) {

		for (int i = 1; i <= NUM; i++) {
			// 按照线程数迭代实例化线程
			new Thread(new ZkClientLockTest()).start();
			// 创建一个线程，倒计数器减1
			cdl.countDown();
		}

	}

	
	@Override
	public void run() {

		String result = "";
		long startTime = 0L;
		//等待其他线程初始化完成后, 一起往下执行
		try {
			cdl.await();

			//随机获取一家公司
			String component = components[new Random().nextInt(components.length)];

			startTime = System.currentTimeMillis();

			lock.lock();
			//获取12306票
			result = ticket12306.getTicket(component);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}finally {
			lock.unlock();

			long runTime = (System.currentTimeMillis() - startTime);
			System.out.println(result + ", 总耗时 = " + runTime + " 毫秒!");
		}
	}

	
}
```

##### Ticket12306.java

```java
package com.zou;

public class Ticket12306 {

    private int ticketsTotal = 50; //数据库的票总数

    private int ticketsCurrent = ticketsTotal; //当前剩余票数

    public Ticket12306(){
    }

    public Ticket12306(int tickets){
        this.ticketsTotal = tickets;
    }

    /**
     * 获取一张票
     * @param component 其他公司： 飞猪 携程 去哪儿 智行
     */
    public String getTicket(String component) {

        //票数量> 0 时, 继续递减
        if(ticketsCurrent > 0 ){
            ticketsCurrent--;
        }

        return Thread.currentThread().getName()+"[" + component + "]: 总票数 = " + ticketsTotal + ", 剩余票数 = " + ticketsCurrent;
    }


}

```

