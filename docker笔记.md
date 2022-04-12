## Docker

## 学习目标

- 能够描述Docker有什么作用
- 能够熟悉Docker常用命令
- 能够熟悉及使用Dockerfile
- 能够使用Docker部署应用
- 能够搭建并使用Docker的私有仓库



## 1.初识 Docker 

-  • Docker 是开源的应用容器**软件**，诞生于 2013 年初，基于Go 语言实现， dotCloud 公司出品（后改名为Docker Inc）。
-  • Docker 类似tomcat, 我们将软件安装在docker 中，而不是操作系统上，就像：java代码运行在JVM上，而不是直接运行在操作系统上。
-  • Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和  EE（Enterprise Edition: 企业版），企业版更多功能偏向于企业所需。
- • Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），企业版更多功能偏向于企业所需。



**优点**

  由于不同的机器有不同的操作系统，以及不同的库和组件，将一个应用程序部署到多台机器上需要进行大量的环境配置操作 

- 部署方便
  - docker 帮我们解决了环境问题，使得我们可以不考虑环境情况下，完成软件的部署。

- 操作简单
  - 同学们可以自行对比redis、tomcat、mysql、nginx基于docker安装与传统安装相比谁的操作更简单，下面会交给大家。
- 移植性好
  - 类似java 编译一次，处处运行，而docker 制作一次镜像，处处可运行
-  管理成本低 
  - 简单，便捷，使得维护和管理都大大提高了工作效率

### 1.1 docker安装

- docker yum 源有两个：用哪个都行，能顺利下载安装就行

  - #（中央仓库）
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    #（阿里仓库）<font color=red>推荐</font>
    yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```shell
# 1、yum 包更新到最新 需要连接外网, 有点慢，耐心等待
    [root@localhost /]$ yum update

# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
[root@localhost /]$ yum install -y yum-utils device-mapper-persistent-data lvm2

# 3、 设置yum源,  有中央 和 阿里的，那个好用用那个，如果失败：yum clean all 命令清理后在尝试下载
#（中央仓库）
[root@localhost /]$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 4、 安装docker，出现输入的界面都按 y 
[root@localhost /]$ yum install -y docker-ce

# 5、 查看docker版本，验证是否验证成功
[root@localhost /]$ docker -v
Docker version 19.03.13, build 4484c46d9d


####################   卸载   ###################
[root@localhost /]$ yum remove yum remove docker-ce


```





### 1.2 Docker 架构

- **镜像**（Image）： **镜像（Image）是docker概念中的一个重要成员**
  - 我们可以将一个 或者 多个java项目 制作为 一个 docker镜像。
  - 镜像有了以后，我们可以在任何一台linux系统中，下载并运行此镜像
  - docker也提供了命令来管理镜像，比如：查看，删除等。
- **容器**（Container）：**容器（Contain er）也是docker概念中的一个重要成员**
  - docker run 命令会帮助我们创建一个容器，创建容器时一定要指定 一个镜像
  - 容器更像是docker创建出来的虚拟linux系统，我们可以使用命令进入到容器中，对系统内文件和文件夹进行操作，也可以使用命令对容器本身，进行 查看 删除 启动 停止等管理。
- **仓库**（Repository）：仓库是用来保存镜像,   就像 git/maven 都有自己的私人仓库一样，我们可以搭建自己的docker私人镜像仓库，来存储和管理我们公司所有的镜像。



#### 1.3 Docker 镜像站

默认情况下，将来从docker hub（https://hub.docker.com/）（国外）上下载 docker镜像，太慢。

**国内镜像站：**

 • USTC：中科大镜像加速器（https://docker.mirrors.ustc.edu.cn）

 • 阿里云 

• 网易云 

• 腾讯云

```shell
#1. 创建docker 加速器所需目录
[root@localhost /]$ mkdir -p /etc/docker

#2. 创建核心文件 daemon.json
[root@localhost /]$ tee /etc/docker/daemon.json <<-'EOF'
{
   "registry-mirrors": ["https://45fv9fd6.mirror.aliyuncs.com"]
 }
EOF

#3. 重新加载docker配置文件
[root@localhost /]$ systemctl daemon-reload

#4. 重新启动docker
[root@localhost /]$ systemctl restart docker

```



#### <font color=red>1.4 小结</font>

在这一节中，同学们不必较真镜像和容器的关系，只需要知道

- 我们为什么使用docker ? 好在哪？答：简化我们的运维成本
- docker镜像：是可以制作出来的，也可以通过docker命令进行管理
- docker容器：是由docker创建出来，我们只需要关注如何使用命令，对容器进行管理
- docker仓库：可以搭建自己的私人仓库，用来存储和管理公司所有镜像。

- 网络不好，可以考虑配置一下docker国内的镜像站
- docker 镜像 和 容器的关系：创建容器时，需要指定一个镜像，没有镜像没法创建容器，使用与被使用的关系。



### 2. docker 命令

在线命令参考网页：https://www.runoob.com/docker/docker-command-manual.html

#### 2.1 进程相关命令

• 启动docker服务 • 停止docker服务 • 重启docker服务 • 查看docker服务状态 • 开机启动docker服务

```shell
#启动docker服务:
[root@localhost /]$ systemctl start docker 

#停止docker服务:
[root@localhost /]$ systemctl stop docker 

#重启docker服务:
[root@localhost /]$ systemctl restart docker

#查看docker服务状态:
[root@localhost /]$ systemctl status docker 

#设置开机启动docker服务:
[root@localhost /]$ systemctl enable docker

#关闭开机启动docker服务:
[root@localhost /]$ systemctl disable docker

```

####  2.2 镜像相关命令 

• 查看镜像 • 搜索镜像 • 拉取镜像 • 删除镜像

```shell
#查看镜像: 查看本地所有的镜像
[root@localhost /]$ docker images 

# 查看所用镜像的id
[root@localhost /]$ docker images –q 

#搜索镜像:从网络中查找需要的镜像,
[root@localhost /]$ docker search 镜像名称 

#拉取镜像: 从Docker仓库下载镜像到本地
#镜像名称格式为: "名称:版本号", 版本号不指定则是最新的版本,
#如果不知道镜像版本，可以去docker hub 搜索对应镜像查看。地址: https://hub.docker.com/_/redis?tab=tags
[root@localhost /]$ docker pull 镜像名称

# 删除指定本地镜像
[root@localhost /]$ docker rmi 镜像id

# 删除所有本地镜像, 先查后删除
[root@localhost /]$ docker rmi `docker images -q` 

```

#### 2.3 容器相关命令

• 查看容器 • 创建容器 • 进入容器 • 启动容器 • 停止容器 • 删除容器 • 查看容器信息

```shell
# 查看正在运行的容器
[root@localhost /]$ docker ps  

# 查看所有容器
[root@localhost /]$ docker ps –a 

# 创建并启动容器
[root@localhost /]$ docker run 参数
#参数说明： 
• -i：保持容器运行。通常与-t 同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。 
• -t：为容器重新分配一个伪输入终端，通常与-i 同时使用。 
• -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭。 
• -it 创建的容器一般称为交互式容器，-id 创建的容器一般称为守护式容器 
• --name：为创建的容器命名。

```

```shell
#进入容器
# 退出容器，容器不会关闭, 243c32535da7 是 容器id, 需要查看自己容器id
[root@localhost /]$ docker exec -it 243c32535da7 /bin/bash 

#停止容器
[root@localhost /]$ docker stop 容器名称 

#启动容器
[root@localhost /]$ docker start 容器名称

#删除容器：如果容器是运行状态则删除失败，需要停止容器才能删除
[root@localhost /]$ docker rm 容器名称

#查看容器信息
[root@localhost /]$ docker inspect 容器名称

#修改容器名称
[root@localhost /]$ docker rename 当前名称  新名称

#复制容器内文件夹到本地
#容器id:/容器内文件夹路径  /本地文件夹路径
[root@localhost /]$ docker cp 126d58471a08:/etc/nginx/nginx.conf /opt/nginx/conf/
```



### 3.Docker 容器数据卷

- docker 具备数据卷(volume) 机制。数据卷是存在于一个或多个容器中的特定文件或文件夹，这个文件或文件夹以独立于 docker 文件系统的形式存在于宿主机中。数据卷的最大特定是：其生存周期独立于容器的生存周期。



#### 3.1 数据卷概念

##### 思考

- • Docker 容器删除后，在容器中产生的数据也会随之销毁？
-  • Docker 容器和外部机器可以直接交换文件吗？ 
- • 容器之间想要进行数据交互？

##### 数据卷 

- • 数据卷是宿主机中的一个目录或文件 
- • 当容器目录和数据卷目录绑定后，对方的修改会立即同步 
- • 一个数据卷可以被多个容器同时挂载 
- • 一个容器也可以被挂载多个数据卷



#### 3.2 数据卷作用 

1. 解决：不能在宿主机上很方便地访问容器中的文件。
2. 解决：无法在多个容器之间共享数据。
3. 解决：当容器删除时，容器中产生的数据将丢失。
4. 解决：容器数据在不同宿主机之间备份、恢复或迁移 
5. 解决：容器中的数据存储在宿主机之外的地方，比如远程主机上和云存储上 



#### 3.3 配置数据卷

 **注意事项：** 1. 目录必须是绝对路径    2. 如果目录不存在，会自动创建 3. 可以挂载多个数据卷

```shell
#创建启动容器时，使用 –v 参数 设置数据卷
#如下为示例代码, 使用--privileged=true参数，容器内的root拥有真正的root权限。
#如下只作演示
[root@localhost /]$ docker run --name itcast-nginx -p 80:80 
-v /itcast/docker/nginx/www:/usr/share/nginx/html 
-v /itcast/docker/nginx/logs:/var/log/nginx
-v /itcast/docker/nginx/conf.d:/etc/nginx/conf.d --privileged=true 

```

#### 3.4 数据卷容器

多容器进行数据交换 1. 多个容器挂载同一个数据卷 2. 数据卷容器 

```shell
#1. 创建启动c3数据卷容器，使用 –v 参数 设置数据卷
[root@localhost /]$ docker run –it --name=c3 –v /volume centos:7 /bin/bash 

#2.   创建启动 c1 c2 容器，使用–-volumes-from 参数 设置数据卷
[root@localhost /]$ docker run –it --name=c1 --volumes-from c3 centos:7 /bin/bash
[root@localhost /]$ docker run –it --name=c2 --volumes-from c3 centos:7 /bin/bash  
```



### 4.Docker 部署软件

  • mysql  • tomcat   • nginx   • redis

#### 4.1 部署mysql

- 案例：在Docker容器中部署MySQL，并通过外部mysql客户端操作MySQL Server。
- docker run 参数说明：
  - **-p 3307:3306**：将容器的 3306 端口映射到宿主机的 3307 端口。
  - **-v /itcast/docker/conf:/etc/mysql/conf.d**：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。配置目录
  - **-v /itcast/docker/logs:/logs**：将主机当前目录下的 logs 目录挂载到容器的 /logs。日志目录
  - **-v /itcast/docker/data:/var/lib/mysql** ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
  - **-e MYSQL_ROOT_PASSWORD=123：**初始化 root 用户的密码。
  - **-d:** 后台运行容器，并返回容器ID； 

```shell
#1. 选择版本, 推荐: 5.7.18, 其他版本自行搜索:https://hub.docker.com/_/mysql?tab=tags
[root@localhost /]$ docker pull mysql:5.7.18

#2. 创建数据卷目录
[root@localhost /]$ mkdir -p /itcast/docker/mysql/data
[root@localhost /]$ mkdir -p /itcast/docker/mysql/logs
[root@localhost /]$ mkdir -p /itcast/docker/mysql/conf

#3. 启动mysql容器, 指定镜像 和 数据卷, 监听端口3307
[root@localhost /]$ docker run -d -p 3307:3306 --privileged=true \
-v /etc/localtime:/etc/localtime:ro  \
-v /itcast/docker/mysql/conf:/etc/mysql  \
-v /itcast/docker/mysql/data:/var/lib/mysql \
-v /itcast/docker/mysql/logs:/logs \
-e MYSQL_ROOT_PASSWORD=root --name itcast-mysql mysql:5.7.18

#4. 查看正在运行的容器, 判断是否创建成功
[root@localhost /]$ docker ps

#5. 没问题, sqlyog 小海豚连接3307端口进行测试
```



#### 4.2 部署tomcat

- 案例：在Docker容器中部署Tomcat，并通过外部机器访问Tomcat部署的项目。
- 拉取的镜像中自带jdk环境依赖, 不需要自行安装

```shell
#1. 选择版本, 推荐: tomcat:8.5.59-jdk11-adoptopenjdk-openj9
#其他版本自行搜索:https://hub.docker.com/_/tomcat?tab=tags&page=1
[root@localhost /]$ docker pull tomcat:8.5.59-jdk11-adoptopenjdk-openj9

#2. 创建数据卷目录
[root@localhost /]$ mkdir -p /itcast/docker/tomcat8.5/webapps

#3. 创建测试静态页面 & 默认tomcat首页被作者干掉了
[root@localhost /]$ cd /itcast/docker/tomcat8.5/webapps
[root@localhost webapps]$ mkdir helloword
[root@localhost webapps]$ vim helloword/hello.html
<font color='red' size='150px'> hello word tomcat!</font>

#4. 启动mysql容器, 指定镜像 和 数据卷, 监听端口3307
#命令要在一行执行
[root@localhost /]$ docker run -d -p 8080:8080  -v /itcast/docker/tomcat8.5/webapps:/usr/local/tomcat/webapps --name itcast-tomcat
tomcat:8.5.59-jdk11-adoptopenjdk-openj9

#5. 查看正在运行的容器, 判断是否创建成功
[root@localhost /]$ docker ps

#浏览器: http://192.168.23.129:8080/helloword/hello.html
```



#### 4.3 部署nginx

- 案例：在Docker容器中部署Nginx，并通过外部机器访问Nginx。

```shell
#选择版本, 推荐: nginx 最新 latest版本
#其他版本自行搜索:https://hub.docker.com/_/nginx?tab=tags&page=1
[root@localhost /]$ docker pull nginx

#创建数据卷目录
[root@localhost /]$ mkdir -p /itcast/docker/nginx-latest/conf
[root@localhost /]$ mkdir -p /itcast/docker/nginx-latest/logs
[root@localhost /]$ mkdir -p /itcast/docker/nginx-latest/html

#创建nginx.conf 核心配置文件
[root@localhost /]$ vim /itcast/docker/nginx-latest/conf/nginx.conf
```

nginx.conf

```shell

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

```shell
#启动nginx容器, 指定镜像 和 数据卷, 监听端口81
#命令要在一行执行
[root@localhost /]$ docker run -id --name itcast-nginx -p 82:80 -v /etc/localtime:/etc/localtime:ro -v /itcast/docker/nginx-latest/conf/nginx.conf:/etc/nginx/nginx.conf -v /itcast/docker/nginx-latest/logs:/var/log/nginx -v /itcast/docker/nginx-latest/html:/usr/share/nginx/html nginx

#创建测试静态文件
#3. 创建测试静态页面
[root@localhost /]$ cd /itcast/docker/nginx-latest/html
[root@localhost html]$ vim hello.html
<font color='red' size='150px'> hello word nginx!</font>

#查看正在运行的容器, 判断是否创建成功
[root@localhost /]$ docker ps

#浏览器: http://192.168.23.129:82/hello.html
```



#### 4.4 部署redis

- 案例：在Docker容器中部署Redis，并通过外部机器访问Redis

```shell
#1. 选择版本, 推荐:redis:5.0
#其他版本自行搜索:https://hub.docker.com/_/redis?tab=tags&page=1
[root@localhost /]$ docker pull redis:5.0

#2. 创建数据卷目录
[root@localhost /]$ mkdir -p /itcast/docker/redis5.0/conf
[root@localhost /]$ mkdir -p /itcast/docker/redis5.0/data

#创建redis-6379.conf 核心配置文件
[root@localhost /]$ vim /itcast/docker/redis5.0/conf/redis-6379.conf
```

**redis-6379.conf**

```
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile "redis-6379.log"
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes

```

```shell
#4. 启动redis容器, 指定镜像 和 数据卷, 监听端口6379
#命令要在一行执行
[root@localhost conf]$ docker run -d --privileged=true -p 6379:6379 -v /etc/localtime:/etc/localtime:ro -v /itcast/docker/redis5.0/conf/redis-6379.conf:/etc/redis/redis.conf  -v /itcast/docker/redis5.0/data:/data --name itcast-redis redis:5.0

#5. 查看正在运行的容器, 判断是否创建成功
[root@localhost /]$ docker ps

#./redis-cli.exe -h 192.168.23.129 -p 6379
```



### 5. dockerfile

- 就是一个docker文件，用于制作镜像

#### 5.1 镜像原理

**Linux文件系统由bootfs和rootfs两部分组成** 

- • bootfs：包含bootloader（引导加载程序）和kernel（内核） 
- • rootfs：root文件系统，包含的就是典型Linux 系统中的/dev， /proc，/bin，/etc等标准目录和文件
-  • 不同的linux发行版，bootfs基本一样，而rootfs不同，如ubuntu ，centos等

**Docker镜像是由特殊的文件系统叠加而成**

- • 最底端是 bootfs，并使用宿主机的bootfs 
- • 第二层是 root文件系统rootfs,称为base image（基础镜像）
- • 然后再往上可以叠加其他的镜像文件 
- • 一个镜像可以放在另一个镜像的上面。位于下面的镜像是上面镜像的**父镜像**，最底部的镜像成为**基础镜像**。 • 当从一个镜像启动容器时，Docker会在最顶层加载一个读 写文件系统作为容器

**思考：**

1. Docker 镜像本质是什么？ 

   **答：**• 是一个分层文件系统 

2. Docker 中一个centos镜像为什么只有200MB，而一个centos操作系统的iso文件要几个个G？ 

   **答：**• Centos的iso镜像文件包含bootfs和rootfs，而docker的centos镜像复用操作系统的bootfs，只有rootfs和其他镜像层 

3.  Docker 中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？ **答：**• 由于docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所有整个对外暴露的 tomcat镜像大小500多MB



#### 5.2 镜像制作

##### 容器转镜像

- 下方2  3步骤可以通过文件的方式在多台机器中共享镜像，等我们学习镜像私人仓库后，这种做法不推荐。

```shell
#1.容器转镜像
$ docker commit 容器id 镜像名称:版本号

#2.镜像保存为压缩文件
$ docker save -o 压缩文件名称 镜像名称:版本号

#3.压缩文件在加载为镜像
$ docker load –i 压缩文件名称
```



##### Dockerfile 关键字

- • Dockerfile 是一个文本文件
- • 包含了一条条的指令 ，类似shell
- • 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像 
- • 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- • 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件 构建一个新的镜像开始工作了 
- • 对于运维人员：在部署时，可以基于dockerfile 快速部署

| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |



#### 5.3 Dockerfile 案例

##### 制作centos7镜像

- 要求： 1. 默认登录路径为/usr 2. 可以使用vim

- 步骤：

   ① 定义父镜像：FROM centos:7 

   ② 定义作者信息：MAINTAINER  itheima <itheima@itcast.cn> 

   ③ 执行安装vim命令：RUN yum install -y vim

   ④ 定义默认的工作目录：WORKDIR  /usr 

   ⑤ 定义容器启动执行的命令：CMD  /bin/bash 

   ⑥ 通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本

```shell
#1.制作一个dockerfile文件
$ vim centos7-dockerfile
FROM centos:7 
MAINTAINER itheima <itheima@itcast.cn>
RUN yum install -y vim
WORKDIR /usr
CMD /bin/bash

#2.基于dockerfile创建镜像
#docker build –f dockerfile文件路径 –t 镜像名称:版本 .
$ docker build –f /itcast/docker/centos7-dockerfile –t mycentos7:7.0 .
```



##### 制作springboot项目镜像

- 步骤：

​          ① 定义父镜像：FROM java:8 

​          ② 定义作者信息：MAINTAINER itheima <itheima@itcast.cn> 

​          ③ 将jar包添加到容器：ADD springboot.jar  app.jar 

​          ④ 定义容器启动执行的命令：CMD java  –jar  app.jar 

​          ⑤ 通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本

```shell
#1.制作一个dockerfile文件
$ vim springboot-dockerfile
FROM java:8  
MAINTAINER itheima <itheima@itcast.cn>
ADD springboot.jar  app.jar
CMD java –jar  app.jar 

#2.基于dockerfile创建镜像
#docker bulid –f dockerfile文件路径 –t 镜像名称:版本
$ docker bulid –f /itcast/docker/springboot-dockerfile –t app:1.0-release
```



### 6. Docker 服务编排

**前言：**微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停 ，维护的工作量会很大。

-  • 要从Dockerfile build image 或者去dockerhub拉取image
- • 要创建多个container • 要管理这些container（启动停止删除）

<font color=red>**服务编排**：按照一定的业务规则  **批量**  管理容器</font>



#### 6.1 Docker Compose

- Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建 ，启动和停止。**使用步骤**： 
  - 1. 利用 Dockerfile 定义运行环境镜像
    2. 使用 docker-compose.yml定义组成应用的各服务 
    3.  运行 docker-compose up 启动应用



#### 6.2 安装&卸载

**安装Docker Compose**

```shell
#1. Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。
#安装在: /usr/local/bin/ 目录下
[root@localhost /]$ curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o  /usr/local/bin/docker-compose

#2. 设置文件可执行权限 
[root@localhost /]$ chmod +x /usr/local/bin/docker-compose
#4. 查看版本信息 
[root@localhost /]$ docker-compose -version
```

**卸载Docker Compose**

```shell
# 二进制包方式安装的，删除二进制文件即可
[root@localhost /]$ rm /usr/local/bin/docker-compose
```



#### 6.3 编排容器

- 使用docker compose编排nginx+springboot项目

```shell
#创建docker-compose目录
[root@localhost /]$ mkdir -p /itcast/docker/docker-compose

#编写 docker-compose.yml 文件
[root@localhost /]$ vim /itcast/docker/docker-compose/docker-compose.yml
```

**docker-compose.yml**

- services:下的 nginx:  和  myproject: 代表要编排哪些镜像

```
version: '3'
services:
  nginx:
   image: nginx
   ports:
    - 80:80
   links:
    - app
   volumes:
    - ./nginx/conf.d:/etc/nginx/conf.d
  app:
    image: springboottest:1.0
    expose:
      - "8080"
```

```shell
#创建nginx 容器挂载目录
[root@localhost /]$ cd /itcast/docker/docker-compose
[root@localhost docker-compose]$ mkdir -p nginx/conf.d

#在./nginx/conf.d目录下 编写itheima.conf文件
[root@localhost docker-compose]$ vim nginx/conf.d/itheima.conf

```

**itheima.conf**

- proxy_pass 是将网络请求转发到容器本地8080端口

```
server {
    listen 80;
    access_log off;

    location / {
        proxy_pass http://app:8080;
    }
}
```

```shell
#在~/docker-compose 目录下 使用docker-compose 启动容器
[root@localhost docker-compose]$ docker-compose up

######测试
http://192.168.23.129/hello
```



#### 6.4 私人仓库

- Docker官方的Docker hub（https://hub.docker.com）是一个用于管理公共镜像的仓库，我们可以从上面拉取镜像 到本地，也可以把我们自己的镜像推送上去。但是，有时候我们的服务器无法访问互联网，或者你不希望将自己的镜 像放到公网当中，那么我们就需要搭建自己的私有仓库来存储和管理自己的镜像。



##### 私有仓库搭建

```shell
# 1、拉取私有仓库镜像 
[root@localhost /]$ docker pull registry

# 2、启动私有仓库容器 
[root@localhost /]$ docker run -d --name=registry -p 5000:5000 registry

# 3、打开浏览器 输入地址http://私有仓库服务器ip:5000/v2/_catalog，看到{"repositories":[]} 表示私有仓库 搭建成功

# 4、修改daemon.json   
[root@localhost /]$ vim /etc/docker/daemon.json    
# 在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip 
{"insecure-registries":["私有仓库服务器ip:5000"]} 

# 5、重启docker 服务 
[root@localhost /]$ systemctl restart docker
[root@localhost /]$ docker start registry

```

##### 镜像上传至私有仓库

```shell
# 1、标记镜像为私有仓库的镜像     
#docker tag centos:7 私有仓库服务器IP:5000/centos:7
[root@localhost /]$ docker tag redis:5.0.2 192.168.184.128:5000/itcast-redis:5.0 
 
# 2、上传标记的镜像     
#docker push 私有仓库服务器IP:5000/centos:7
[root@localhost /]$ docker push 192.168.23.129:5000/itcast-redis:5.0

###刷新页面: http://192.168.23.129:5000/v2/_catalog
```

##### 镜像从私有仓库拉取

```shell
#拉取镜像 
#docker pull 私有仓库服务器ip:5000/centos:7
#1. 先删除
[root@localhost /]$ docker rmi 192.168.23.129:5000/itcast-redis:5.0

#2.在拉取
[root@localhost /]$ docker pull 192.168.184.128:5000/itcast-redis:
```



### 7.docker与虚拟机对比

容器就是将软件打包成标准化单元，以用于开发、交付和部署。 

- • 容器镜像是轻量的、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行时环境、系统工具、 系统库和设置。 
- • 容器化软件在任何环境中都能够始终如一地运行。 
- • 容器赋予了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软 件时的冲突。

**相同：** 

- • 容器和虚拟机具有相似的资源隔离和分配优势 

**不同：** 

- • 容器虚拟化的是操作系统，虚拟机虚拟化的是硬件。
- • 传统虚拟机可以运行不同的操作系统，容器只能运行同一类型操作系统

|    特性    |        容器        |   虚拟机   |
| :--------: | :----------------: | :--------: |
|    启动    |        秒级        |   分钟级   |
|  磁盘使用  |      一般为MB      |  一般为GB  |
|    性能    |      接近原生      |    弱于    |
| 系统支持量 | 单击支持上千个容器 | 一般几十个 |



### 8. 扩展

#### 8.1 编排工具

##### 1、Docker Compose

- 适用于单机编排（一个docker host）

##### 2、Docker Swarm

- 适用于集群编排（多个docker host）

##### 3、<font color=red>kubernetes（k8s）</font>

- 未来五年最火的编排工具，也是当前最流行编排工具

- 自动装箱，自我修复，水平扩展，服务发现和负载均衡，自动发布和回滚

秘钥和配置管理，存储编排，批量处理执行

**kubernetes集群**

- master：组件：API Server，Scheduler，Controller-Manager

- node：组件：kubelet，容器引擎（如docker），负责启动容器，从register中拉取镜像启动

- 在k8s运行的最小单元不是docker容器，而是pod，他是容器的外壳，pod可以包含一个或多个容器



#### 8.2 镜像私服-harbor

 Harbor 是由 VMware 公司中国团队为企业用户设计的 Registry server 开源项目，包括了权限管理(RBAC)、LDAP、审计、管理界面、自我注册、HA 等企业必需的功能，同时针对中国用户的特点，设计镜像复制和中文支持等功能。

- **基于角色的访问控制** - 用户与 Docker 镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。
- **镜像复制** - 镜像可以在多个 Registry 实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。
- **图形化用户界面** - 用户可以通过浏览器来浏览，检索当前 Docker 镜像仓库，管理项目和命名空间。
- **AD/LDAP 支持** - Harbor 可以集成企业内部已有的 AD/LDAP，用于鉴权认证管理。
- **审计管理** - 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。
- **国际化** - 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。
- **RESTful API** - RESTful API 提供给管理员对于 Harbor 更多的操控, 使得与其它管理软件集成变得更容易。
- **部署简单** - 提供在线和离线两种安装工具， 也可以安装到 vSphere 平台(OVA 方式)虚拟设备。

##### 安装步骤

- <font color=red>注意虚拟机磁盘空间要在30G以上</font>
- <font color=red>vmware 虚拟机存在快照时，不让扩展磁盘容量，需要删除</font>
- 需要提前装好 docker-compose
- 安装harbor的过程中，会拉取很多镜像并启动，强烈推荐，准备一台裸的centos7单独搭建harbor使用

###### 安装harbor

```shell
#1.进入到安装目录
[root@192 ssl]$ cd /itcast/docker

#2.下载harbor 安装包
[root@192 ssl]$ wget https://github.com/goharbor/harbor/releases/download/v1.10.1/harbor-offline-installer-v1.10.1.tgz 

#3.解压缩
[root@192 ssl]$ tar -zxvf harbor-offline-installer-v1.10.1.tgz 

#4.进入到harbor 目录中
[root@192 ssl]$ cd harbor

#5.修改默认配置，保存退出
[root@192 ssl]$ vim harbor.yml  

 1) #hostname 自己外网ip 或者 /etc/hosts 中ip对应的域名
    hostname 192.168.23.129
    
 2) #https下全都注释掉
    #https:
      # https port for harbor, default is 443
      #port: 443
      # The path of cert and key files for nginx
      #certificate: /your/certificate/path
      #private_key: /your/private/key/path
  
 4) 默认密码更改 admin，默认账号admin不可修改
    harbor_admin_password: admin  
    
    
#6.配置docker 私服仓库免密登录, 默认端口80
[root@192 ssl]$ echo '{ "insecure-registries":["192.168.23.129"] }' > /etc/docker/daemon.json
 
 
#7.重启docker:
[root@192 ssl]$ systemctl daemon-reload
[root@192 ssl]$ systemctl restart docker


#8.保存退出后, 配置&安装, 一共五步, 耐心等待 successfully 表示成功
[root@192 ssl]$ ./prepare
[root@192 ssl]$ ./install.sh

..........此处省略n多行打印
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating redis         ... done
Creating registryctl   ... done
Creating harbor-db     ... done
Creating registry      ... done
Creating harbor-portal ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----


#9.docker登录私服测试：admin, admin
[root@192 ssl]$ docker login 192.168.23.129:80
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

#10.访问网址: http://192.168.23.129/   账号密码: admin、admin
#点击新建项目: 项目名任意起名称,  例如: itcast-images, 其他项默认值即可

#11. 随便拉取一个镜像或者自己制作一个, 例如: 拉取redis:5.0 
[root@localhost /]$ docker pull redis:5.0

#12.将redis:5.0 打一个tag后上传私服
# docker tag  镜像名:标签  私服地址/仓库项目名/镜像名:标签
# 注意：itcast-images 是我们在页面中创建的项目仓库
[root@localhost /]$ docker tag redis:5.0  192.168.23.129:80/itcast-images/itcast-redis:5.0 


#13. 推送到私服的命令
# docker push 私服地址/仓库项目名/镜像名:标签
[root@localhost /]$ docker push 192.168.23.129:80/itcast-images/itcast-redis:5.0
 

#14.回到页面, 点击itcast-images, 再点击仓库镜像查看是否存在, 刚才上传的itcast-redis:5.0 镜像

#15.也可以自行尝试下载
#先删除本地已有镜像
[root@192 docker]$ docker rmi 192.168.23.129:80/itcast-images/itcast-redis:5.0
#在从仓库拉取itcast-redis:5.0镜像
[root@localhost /]$ docker pull 192.168.23.129/itcast-images/itcast-redis:5.0



```

###### 常见问题

```shell
###docker login 192.168.23.129 时 出现如下错误
Error response from daemon: Get https://192.168.23.129/v2/: dial tcp 192.168.23.129:443: connect: connection refused

###解决方案如下: 
#查找docker.service 所在的位置 
[root@master01 docker]$ find / -name docker.service -type f
/etc/systemd/system/docker.service
　　

#修改配置文件， 增加  --insecure-registry=192.168.23.129 选项
[root@master01 docker]$ cat /etc/systemd/system/docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io

[Service]
Environment="PATH=/opt/kube/bin:/bin:/sbin:/usr/bin:/usr/sbin"
ExecStart=/opt/kube/bin/dockerd --insecure-registry=192.168.23.129 
ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
　　

#重新docker启动服务 
[root@master01 docker]$ systemctl daemon-reload
[root@master01 docker]$ systemctl restart docker
　　

#查看服务，已经包含了 --insecure-registry=192.168.23.129  参数
[root@master01 docker]$ ps aux|grep docker
root      6385  0.5  2.1 419248 39836 ?        Ssl  05:30   0:03 /opt/kube/bin/dockerd --insecure-registry=192.168.23.129 
root      6398  0.0  0.5 292736  9884 ?        Ssl  05:30   0:00 docker-containerd -l 
```

