---
layout: post
title:  "Docker 安装 RocketMQ"
date:   2020-07-09 00:00:00
categories: RocketMQ
tags: RocketMQ
author: poazy
---

* content
{:toc}
> Docker 方式安装 RocketMQ 4.4.0 版本，采用 `伪` 集群 `两主两从异步` `4个` 实例方式安装；
>
> Docker 方式安装 RocketMQ 4.6.0 版本，采用 `dledger` 集群 `3个` 实例方式安装；
>
> `RocketMQ 4.4.0` 版本不支持 `dledger` 集群方式，从 `RocketMQ 4.5.0` 开始支持`dledger` 集群方式！







# RocketMQ 4.4.0 版本

> **两主两从异步**

## 安装 Name Server

> 这里以 `伪集群` 安装三个 `名字服务（Name Server）` 实例；
>
> 名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。
>
> Name Server 的默认端口为 `9876`

```bash
# 实例一，9876
docker run -d --name mqnamesrv-0 \
    -p 9876:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/store:/home/rocketmq/store \
    rocketmqinc/rocketmq:4.4.0 sh mqnamesrv

# 实例二，9877
docker run -d --name mqnamesrv-1 \
    -p 9877:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/store:/home/rocketmq/store \
    rocketmqinc/rocketmq:4.4.0 sh mqnamesrv

# 实例三，9878
docker run -d --name mqnamesrv-2 \
    -p 9878:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/store:/home/rocketmq/store \
    rocketmqinc/rocketmq:4.4.0 sh mqnamesrv

```

## 安装 Broker Server

>这里以 `伪集群` 安装 `两主两从异步刷盘` 四个 `代理服务器（Broker Server）` 实例；
>
>消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。
>
>Broker 启动时，（默认）实际上会监听3个端口：10909、10911、10912，这三个端口由Broker内部不同的组件使用，作用分别如下（http://www.tianshouzhi.com/api/tutorials/rocketmq/417）：
>
>* **remotingServer：**监听 listenPort 配置项指定的监听端口，默认为 10911
>
>* **fastRemotingServer：**监听端口值 listenPort - 2，即默认为 10909
>
>* **HAService：**监听端口为值为 listenPort + 1，即默认为 10912，该端口用于 Broker 的主从同步
>
>`4.4.0 版本` 不支持 `dledger` 群集模式！

### 创建目录并配置文件

* 1) 创建安装目录

> 宿主机上创建目录并配置

```bash
mkdir -p /boazy/data/dockerdata/rocketmq/broker-n0
```

* 2) 配置 conf 文件

  * 下载 [rocketmq-all-4.5.0-bin-release.zip](http://archive.apache.org/dist/rocketmq/4.5.0/rocketmq-all-4.5.0-bin-release.zip) 文件， 用于复制里面的 conf 目录文件

  * 复制 `rocketmq-all-4.5.0-bin-release.zip` 中的 conf 目录及上下的文件上传到 `/boazy/data/dockerdata/rocketmq/broker-n0` 目录下

    >上传后查询目录结构如下：
    >
    >```bash
    >[root@slave6 conf]# pwd
    >/boazy/data/dockerdata/rocketmq/broker-n0/conf
    >[root@slave6 conf]# tree
    >.
    >├── 2m-2s-async
    >│   ├── broker-a.properties
    >│   ├── broker-a-s.properties
    >│   ├── broker-b.properties
    >│   └── broker-b-s.properties
    >├── 2m-2s-sync
    >│   ├── broker-a.properties
    >│   ├── broker-a-s.properties
    >│   ├── broker-b.properties
    >│   └── broker-b-s.properties
    >├── 2m-noslave
    >│   ├── broker-a.properties
    >│   ├── broker-b.properties
    >│   └── broker-trace.properties
    >├── broker.conf
    >├── logback_broker.xml
    >├── logback_namesrv.xml
    >├── logback_tools.xml
    >├── plain_acl.yml
    >└── tools.yml
    >
    >3 directories, 17 files
    >[root@slave6 conf]#
    >```

  * 配置修改 `conf/2m-2s-async` 目录下的的四个 `properties` 文件

    * broker-a.properties

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/2m-2s-async/broker-a.properties
    ```

    > `broker-a.properties` 配置文件未尾添加 `namesrvAddr、listenPort、brokerIP1、brokerIP2` 配置，如下：

    ```bash
    # 。。。这里的注释省略
    brokerClusterName=DefaultCluster
    brokerName=broker-a
    brokerId=0
    deleteWhen=04
    fileReservedTime=48
    brokerRole=ASYNC_MASTER
    flushDiskType=ASYNC_FLUSH
    # 添加以下配置：命名服务地址、监听端口、多网卡配置 brokerIP1 和 brokerIP2
    namesrvAddr=192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878
    listenPort=30911
    brokerIP1=192.168.0.79
    brokerIP2=192.168.0.79
    ```

    * broker-a-s.properties

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/2m-2s-async/broker-a-s.properties
    ```

    > `broker-a-s.properties` 配置文件未尾添加 `namesrvAddr、listenPort、brokerIP1、brokerIP2` 配置，如下：

    ```bash
    # 。。。这里的注释省略
    brokerClusterName=DefaultCluster
    brokerName=broker-a
    brokerId=1
    deleteWhen=04
    fileReservedTime=48
    brokerRole=SLAVE
    flushDiskType=ASYNC_FLUSH
    # 添加以下配置：命名服务地址、监听端口、多网卡配置 brokerIP1 和 brokerIP2
    namesrvAddr=192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878
    listenPort=30921
    brokerIP1=192.168.0.79
    brokerIP2=192.168.0.79
    ```

    * broker-b.properties

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/2m-2s-async/broker-b.properties
    ```

    > `broker-b.properties` 配置文件未尾添加 `namesrvAddr、listenPort、brokerIP1、brokerIP2` 配置，如下：

    ```bash
    # 。。。这里的注释省略
    brokerClusterName=DefaultCluster
    brokerName=broker-b
    brokerId=0
    deleteWhen=04
    fileReservedTime=48
    brokerRole=ASYNC_MASTER
    flushDiskType=ASYNC_FLUSH
    # 添加以下配置：命名服务地址、监听端口、多网卡配置 brokerIP1 和 brokerIP2
    namesrvAddr=192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878
    listenPort=30931
    brokerIP1=192.168.0.79
    brokerIP2=192.168.0.79
    ```

    * broker-b-s.properties

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/2m-2s-async/broker-b-s.properties
    ```

    > `broker-b-s.properties` 配置文件未尾添加 `namesrvAddr、listenPort、brokerIP1、brokerIP2` 配置，如下：

    ```bash
    # 。。。这里的注释省略
    brokerClusterName=DefaultCluster
    brokerName=broker-b
    brokerId=1
    deleteWhen=04
    fileReservedTime=48
    brokerRole=SLAVE
    flushDiskType=ASYNC_FLUSH
    # 添加以下配置：命名服务地址、监听端口、多网卡配置 brokerIP1 和 brokerIP2
    namesrvAddr=192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878
    listenPort=30941
    brokerIP1=192.168.0.79
    brokerIP2=192.168.0.79
    ```

* 3) 创建 broker-n1、broker-n3、broker-n3 目录

> 根据我们上面 创建并配置好的 `broker-n0` 目录复制出 `broker-n1、broker-n3、broker-n3` 目录

```bash
cp -r broker-n0 broker-n1 && cp -r broker-n0 broker-n2 && cp -r broker-n0 broker-n3
```

> 查询创建 broker-n1、broker-n3、broker-n3 目录结果如下：

```bash
[root@slave6 rocketmq]# pwd
/boazy/data/dockerdata/rocketmq
[root@slave6 rocketmq]# ll -h
总用量 28K
d-wx--x--x. 5 root root 4.0K 7月  10 09:32 broker-n0
d-wx--x--x. 5 root root 4.0K 7月  10 09:44 broker-n1
d-wx--x--x. 5 root root 4.0K 7月  10 09:44 broker-n2
d-wx--x--x. 5 root root 4.0K 7月  10 09:44 broker-n3
[root@slave6 rocketmq]#
```

* 4) 授权目录

> 容器中的用户ID和组ID分别是 3000 和 3000，如果不给宿主机目录授权限那么容器将会运行不成功的

```bash
 chown -R 3000:3000 /boazy/data/dockerdata/rocketmq/
```

> 授后，结果如下：

```bash
[root@slave6 rocketmq]# pwd
/boazy/data/dockerdata/rocketmq
[root@slave6 rocketmq]# ll -h
总用量 28K
d-wx--x--x. 5 3000 3000 4.0K 7月  10 09:32 broker-n0
d-wx--x--x. 5 3000 3000 4.0K 7月  10 09:44 broker-n1
d-wx--x--x. 5 3000 3000 4.0K 7月  10 09:44 broker-n2
d-wx--x--x. 5 3000 3000 4.0K 7月  10 09:44 broker-n3
[root@slave6 rocketmq]#
```

### 创建并运行 Broker 容器

> 创建 两主两从 实例
>
> 官方 RocketMQ 4.4.0 版本镜像名为 `rocketmqinc/rocketmq`，而 RocketMQ 4.5.0 版本+镜像名为 `apacherocketmq/rocketmq`，这里我们用的是 RocketMQ 4.4.0 版本

```bash
# 实例一，主一
docker run -d --name mqbroker-n0 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/conf:/opt/rocketmq-4.4.0/conf \
    rocketmqinc/rocketmq:4.4.0 sh mqbroker -c ../conf/2m-2s-async/broker-a.properties

# 实例二，主二
docker run -d --name mqbroker-n1 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/conf:/opt/rocketmq-4.4.0/conf \
    rocketmqinc/rocketmq:4.4.0 sh mqbroker -c ../conf/2m-2s-async/broker-b.properties

# 实例三，从一
docker run -d --name mqbroker-n2 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/conf:/opt/rocketmq-4.4.0/conf \
    rocketmqinc/rocketmq:4.4.0 sh mqbroker -c ../conf/2m-2s-async/broker-a-s.properties
# 实例四，从二
docker run -d --name mqbroker-n3 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n3/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n3/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n3/conf:/opt/rocketmq-4.4.0/conf \
    rocketmqinc/rocketmq:4.4.0 sh mqbroker -c ../conf/2m-2s-async/broker-b-s.properties
```

> 运行结果如下：

```bash
[root@slave6 rocketmq]# docker ps -a | grep mqbroker-n
5c83d3c35ed6  rocketmqinc/rocketmq:4.4.0  "sh mqbroker -c ../c   58 minutes ago      Up 58 minutes                                                          mqbroker-n3          
d838f0d2ef4b  rocketmqinc/rocketmq:4.4.0  "sh mqbroker -c ../c   59 minutes ago      Up 59 minutes                                                          mqbroker-n2          
5b11804297df  rocketmqinc/rocketmq:4.4.0  "sh mqbroker -c ../c   59 minutes ago      Up 59 minutes                                                          mqbroker-n1          
b75789bbd473  rocketmqinc/rocketmq:4.4.0  "sh mqbroker -c ../c   About an hour ago   Up About an hour                                                       mqbroker-n0 
```

> 选取 `mqbroker-n2` 查看端口情况，结果如下：

```bash
[root@slave6 rocketmq]# ps -ef|grep java |grep broker-a-s.properties
3000     22331 22281  2 09:49 ?        00:01:34 /bin/java -server -Xms7159173611 -Xmx7159173611 -Xmn3579586805 -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:MaxDirectMemorySize=7159173611 -XX:-UseLargePages -XX:-UseBiasedLocking -Djava.ext.dirs=/jre/lib/ext:/opt/rocketmq-4.4.0/bin/../lib -cp .:/opt/rocketmq-4.4.0/bin/../conf: org.apache.rocketmq.broker.BrokerStartup -c ../conf/2m-2s-async/broker-a-s.properties
[root@slave6 rocketmq]# netstat -anp |grep 22331
tcp        0      0 :::30919                    :::*                        LISTEN      22331/java
tcp        0      0 :::30921                    :::*                        LISTEN      22331/java
tcp        0      0 :::30922                    :::*                        LISTEN      22331/java
tcp        0      0 ::ffff:192.168.0.79:48768   ::ffff:192.168.0.79:30909   ESTABLISHED 22331/java
tcp        0      0 ::ffff:192.168.0.79:47498   ::ffff:192.168.0.79:30911   ESTABLISHED 22331/java
tcp        0      0 ::ffff:192.168.0.79:40572   ::ffff:192.168.0.79:9876    ESTABLISHED 22331/java
tcp        0      0 ::ffff:192.168.0.79:49684   ::ffff:192.168.0.79:30912   ESTABLISHED 22331/java
tcp        0      0 ::ffff:192.168.0.79:40392   ::ffff:192.168.0.79:9877    ESTABLISHED 22331/java
tcp        0      0 ::ffff:192.168.0.79:53808   ::ffff:192.168.0.79:9878    ESTABLISHED 22331/java
unix  2      [ ]         STREAM     CONNECTED     12753118 22331/java
unix  2      [ ]         STREAM     CONNECTED     12753075 22331/java
[root@slave6 rocketmq]#
```

### 查询集群状态

> 这里在 windows 10 环境下执行命令（本 windows 10 安装过 RocketMQ）

```bash
Microsoft Windows [版本 10.0.18363.900]
(c) 2019 Microsoft Corporation。保留所有权利。

D:\duanbo\Desktop>mqadmin.cmd clusterList -n "192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878"
RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.PlatformDependent0).
RocketMQLog:WARN Please initialize the logger system properly.
#Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE
DefaultCluster    broker-a                0     192.168.0.79:30911     V4_4_0                   0.00(0,0ms)         0.00(0,0ms)          0 442875.02 -1.0000
DefaultCluster    broker-a                1     192.168.0.79:30921     V4_4_0                   0.00(0,0ms)         0.00(0,0ms)          0 442875.02 0.0305
DefaultCluster    broker-b                0     192.168.0.79:30931     V4_4_0                   0.00(0,0ms)         0.00(0,0ms)          0 442875.02 -1.0000
DefaultCluster    broker-b                1     192.168.0.79:30941     V4_4_0                   0.00(0,0ms)         0.00(0,0ms)          0 442875.02 0.0305

D:\duanbo\Desktop>
```



## 安装控件台

> 官方 RocketMQ Console 控制台镜像名为 `styletang/rocketmq-console-ng`

```bash
docker run -d --name rocketmq-console-ng \
    -p 8280:8080 \
    -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878 -Dlogging.level.root=info -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
    styletang/rocketmq-console-ng:1.0.0
```

> 安装运行正常后，可通过如下地址访问控制台：

```bash
http://192.168.0.79:8280/
```



## 安装后环境

> RocketMQ 4.4.0 版本测试（192.168.0.79）环境信息

| 服务名称      | 服务地址                                                     | 说明                               |
| ------------- | ------------------------------------------------------------ | ---------------------------------- |
| Name Server   | 192.168.0.79:9876;192.168.0.79:9877;192.168.0.79:9878        | 伪集群（4.4.0 版本）               |
| Broker Server | 192.168.0.79:30911<br/>192.168.0.79:30921<br/>192.168.0.79:30931<br/>192.168.0.79:30941 | 伪集群（4.4.0 版本；两主两从异步） |
| Console       | http://192.168.0.79:8280                                     | 1.0.0 版本                         |



# RocketMQ 4.6.0 版本

> `dledger` 集群

## 安装 Name Server

> 这里以 `伪集群` 安装3个 `名字服务（Name Server）` 实例；
>
> 名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。多个Namesrv实例组成集群，但相互独立，没有信息交换。
>
> Name Server 的默认端口为 `9876`

```bash
# 实例一，9876
docker run -d --name mqnamesrv-0 \
    -p 9876:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.6.0 sh mqnamesrv

# 实例二，9877
docker run -d --name mqnamesrv-1 \
    -p 9877:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.6.0 sh mqnamesrv

# 实例三，9878
docker run -d --name mqnamesrv-2 \
    -p 9878:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.6.0 sh mqnamesrv

```

## 安装 Broker Server

>这里以 `dledger` 集群方式3个实例安装；
>
>消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。
>
>Broker 启动时，（默认）实际上会监听3个端口：10909、10911、10912，这三个端口由Broker内部不同的组件使用，作用分别如下（http://www.tianshouzhi.com/api/tutorials/rocketmq/417）：
>
>* **remotingServer：**监听 listenPort 配置项指定的监听端口，默认为 10911
>
>* **fastRemotingServer：**监听端口值 listenPort - 2，即默认为 10909
>
>* **HAService：**监听端口为值为 listenPort + 1，即默认为 10912，该端口用于 Broker 的主从同步
>
>`4.5.0 版本` 开始支持 `dledger` 群集模式！

### 创建目录并配置文件

* 1) 创建安装目录

> 宿主机上创建目录并配置

```bash
mkdir -p /boazy/data/dockerdata/rocketmq/broker-n0
```

* 2) 配置 conf 文件

  * 下载 [rocketmq-all-4.6.1-bin-release.zip](http://archive.apache.org/dist/rocketmq/4.6.1/rocketmq-all-4.6.1-bin-release.zip) 文件， 用于复制里面的 conf 目录文件

  * 复制 `rocketmq-all-4.6.1-bin-release.zip` 中的 conf 目录及目录中的文件上传到 `/boazy/data/dockerdata/rocketmq/broker-n0` 目录下

    >上传后查询目录结构如下：
    >
    >```bash
    >[root@centos7-qscft conf]# pwd
    >/boazy/data/dockerdata/rocketmq/broker-n0/conf
    >[root@centos7-qscft conf]# tree
    >.
    >├── 2m-2s-async
    >│   ├── broker-a.properties
    >│   ├── broker-a-s.properties
    >│   ├── broker-b.properties
    >│   └── broker-b-s.properties
    >├── 2m-2s-sync
    >│   ├── broker-a.properties
    >│   ├── broker-a-s.properties
    >│   ├── broker-b.properties
    >│   └── broker-b-s.properties
    >├── 2m-noslave
    >│   ├── broker-a.properties
    >│   ├── broker-b.properties
    >│   └── broker-trace.properties
    >├── broker.conf
    >├── dledger
    >│   ├── broker-n0.conf
    >│   ├── broker-n1.conf
    >│   └── broker-n2.conf
    >├── logback_broker.xml
    >├── logback_namesrv.xml
    >├── logback_tools.xml
    >├── plain_acl.yml
    >└── tools.yml
    >
    >4 directories, 20 files
    >[root@centos7-qscft conf]#
    >```

  * 配置修改 `conf/dledger` 目录下的的四个 `properties` 文件

    * broker-n0.conf

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/dledger/broker-n0.conf
    ```

    > `broker-n0.conf` 修改配置如下：

    ```bash
    # 。。。这里的注释省略
    
    brokerClusterName = RaftCluster
    brokerName=RaftNode00
    listenPort=30911
    # [X]修改配置
    namesrvAddr=192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878
    storePathRootDir=/tmp/rmqstore/node00
    storePathCommitLog=/tmp/rmqstore/node00/commitlog
    enableDLegerCommitLog=true
    dLegerGroup=RaftNode00
    # [X]修改配置 127.0.0.1 为实际机器 IP
    dLegerPeers=n0-192.168.9.241:40911;n1-192.168.9.241:40912;n2-192.168.9.241:40913
    ## must be unique
    dLegerSelfId=n0
    # [X]修改配置线程数（根据 cpu 核心数设置）
    sendMessageThreadPoolNums=2
    # [X]添加以下配置：多网卡配置 brokerIP1 和 brokerIP2
    brokerIP1=192.168.9.241
    brokerIP2=192.168.9.241
    ```

    * broker-n1.conf

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/dledger/broker-n1.conf
    ```

    > `broker-n1.conf` 修改配置如下：

    ```bash
    # 。。。这里的注释省略
    
    brokerClusterName = RaftCluster
    brokerName=RaftNode00
    listenPort=30921
    # [X]修改配置
    namesrvAddr=192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878
    storePathRootDir=/tmp/rmqstore/node01
    storePathCommitLog=/tmp/rmqstore/node01/commitlog
    enableDLegerCommitLog=true
    dLegerGroup=RaftNode00
    # [X]修改配置 127.0.0.1 为实际机器 IP
    dLegerPeers=n0-192.168.9.241:40911;n1-192.168.9.241:40912;n2-192.168.9.241:40913
    ## must be unique
    dLegerSelfId=n1
    # [X]修改线程数（根据 cpu 核心数设置）
    sendMessageThreadPoolNums=2
    # [X]添加以下配置：多网卡配置 brokerIP1 和 brokerIP2
    brokerIP1=192.168.9.241
    brokerIP2=192.168.9.241
    ```

    * broker-n2.conf

    ```bash
    vi /boazy/data/dockerdata/rocketmq/broker-n0/conf/dledger/broker-n2.conf
    ```

    > `broker-n2.conf` 修改配置如下：

    ```bash
    # 。。。这里的注释省略
    
    brokerClusterName = RaftCluster
    brokerName=RaftNode00
    listenPort=30931
    # [X]修改配置
    namesrvAddr=192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878
    storePathRootDir=/tmp/rmqstore/node02
    storePathCommitLog=/tmp/rmqstore/node02/commitlog
    enableDLegerCommitLog=true
    dLegerGroup=RaftNode00
    # [X]修改配置 127.0.0.1 为实际机器 IP
    dLegerPeers=n0-192.168.9.241:40911;n1-192.168.9.241:40912;n2-192.168.9.241:40913
    ## must be unique
    dLegerSelfId=n2
    # [X]修改线程数（根据 cpu 核心数设置）
    sendMessageThreadPoolNums=2
    # [X]添加以下配置：多网卡配置 brokerIP1 和 brokerIP2
    brokerIP1=192.168.9.241
    brokerIP2=192.168.9.241
    ```

* 3) 创建 broker-n1、broker-n3、broker-n3 目录

> 根据我们上面 创建并配置好的 `broker-n0` 目录复制出 `broker-n1、broker-n3、broker-n3` 目录

```bash
cp -r broker-n0 broker-n1 && cp -r broker-n0 broker-n2
```

> 查询创建 broker-n0、broker-n1、broker-n3 目录结果如下：

```bash
[root@centos7-qscft rocketmq]# pwd
/boazy/data/dockerdata/rocketmq
[root@centos7-qscft rocketmq]# ll -h
total 0
drwxr-xr-x. 3 root root 18 Jul  2 01:00 broker-n0
drwxr-xr-x. 3 root root 18 Jul  2 01:18 broker-n1
drwxr-xr-x. 3 root root 18 Jul  2 01:18 broker-n2
[root@centos7-qscft rocketmq]#
```

* 4) 授权目录

> 容器中的用户ID和组ID分别是 3000 和 3000，如果不给宿主机目录授权限那么容器将会运行不成功的

```bash
 chown -R 3000:3000 /boazy/data/dockerdata/rocketmq/
```

> 授后，结果如下：

```bash
[root@centos7-qscft rocketmq]# pwd
/boazy/data/dockerdata/rocketmq
[root@centos7-qscft rocketmq]# ll -h
total 0
drwxr-xr-x. 3 3000 3000 18 Jul  2 01:00 broker-n0
drwxr-xr-x. 3 3000 3000 18 Jul  2 01:18 broker-n1
drwxr-xr-x. 3 3000 3000 18 Jul  2 01:18 broker-n2
drwxr-xr-x. 3 3000 3000 18 Jul  2 01:18 broker-n3
[root@centos7-qscft rocketmq]#
```

### 创建并运行 Broker 容器

> 创建 `dledger` 集群3个实例
>
> 官方 RocketMQ 4.6.0 版本镜像名为 `apacherocketmq/rocketmq`

```bash
# 实例一
docker run -d --name mqbroker-n0 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/conf:/home/rocketmq/rocketmq-4.6.0/conf \
    apacherocketmq/rocketmq:4.6.0 sh mqbroker -c ../conf/dledger/broker-n0.conf

# 实例二
docker run -d --name mqbroker-n1 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/conf:/home/rocketmq/rocketmq-4.6.0/conf \
    apacherocketmq/rocketmq:4.6.0 sh mqbroker -c ../conf/dledger/broker-n1.conf

# 实例三
docker run -d --name mqbroker-n2 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/conf:/home/rocketmq/rocketmq-4.6.0/conf \
    apacherocketmq/rocketmq:4.6.0 sh mqbroker -c ../conf/dledger/broker-n2.conf
```

> 运行结果如下：

```bash
[root@centos7-qscft rocketmq]# docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                                NAMES
58c3ce22ccca        apacherocketmq/rocketmq:4.6.0         "sh mqbroker -c ../c…"   51 seconds ago       Up 46 seconds                                                            mqbroker-n2
6b79645b34e6        apacherocketmq/rocketmq:4.6.0         "sh mqbroker -c ../c…"   59 seconds ago       Up 55 seconds                                                            mqbroker-n1
fbe90b11f730        apacherocketmq/rocketmq:4.6.0         "sh mqbroker -c ../c…"   About a minute ago   Up About a minute                                                        mqbroker-n0                                                    
```

> 选取 `mqbroker-n1` 查看端口情况，结果如下：

```bash
[root@centos7-qscft rocketmq]# ps -ef|grep java |grep broker-n1.conf
3000     13693 13667 12 01:42 ?        00:00:13 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/bin/java -server -Xms1451M -Xmx1451M -Xmn200M -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:MaxDirectMemorySize=1451M -XX:-UseLargePages -XX:-UseBiasedLocking -Djava.ext.dirs=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64/jre/lib/ext:/home/rocketmq/rocketmq-4.6.0/bin/../lib -cp .:/home/rocketmq/rocketmq-4.6.0/bin/../conf: org.apache.rocketmq.broker.BrokerStartup -c ../conf/dledger/broker-n1.conf
[root@centos7-qscft rocketmq]# netstat -anp |grep 13693
tcp6       0      0 :::30919                :::*                    LISTEN      13693/java
tcp6       0      0 :::30921                :::*                    LISTEN      13693/java
tcp6       0      0 :::40912                :::*                    LISTEN      13693/java
tcp6       0      0 192.168.9.241:30921     192.168.9.241:48252     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:47662     192.168.9.241:40911     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:30919     192.168.9.241:45964     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:42608     192.168.9.241:9876      ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:39230     192.168.9.241:9877      ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:60378     192.168.9.241:9878      ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:40912     192.168.9.241:39598     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:30921     192.168.9.241:48266     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:40932     192.168.9.241:40913     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:40912     192.168.9.241:39720     ESTABLISHED 13693/java
tcp6       0      0 192.168.9.241:30919     192.168.9.241:45978     ESTABLISHED 13693/java
unix  2      [ ]         STREAM     CONNECTED     881694   13693/java
unix  2      [ ]         STREAM     CONNECTED     880530   13693/java
[root@centos7-qscft rocketmq]#
```

### 查询集群状态

> 这里在 windows 10 环境下执行命令（本 windows 10 安装过 RocketMQ）

```bash
Microsoft Windows [版本 10.0.18363.900]                                                   (c) 2019 Microsoft Corporation。保留所有权利。                                                                                                                                         D:\duanbo\Desktop>mqadmin.cmd clusterList -n "192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878"                 RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.PlatformDependent0). 
RocketMQLog:WARN Please initialize the logger system properly.                           #Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE                           RaftCluster       RaftNode00              0     192.168.9.241:30921    V4_6_0                   0.00(0,0ms)         0.00(0,0ms)          0 442879.43 -1.0000                       RaftCluster       RaftNode00              1     192.168.9.241:30911    V4_6_0                   0.00(0,0ms)         0.00(0,0ms)          0 442879.43 -1.0000                       RaftCluster       RaftNode00              3     192.168.9.241:30931    V4_6_0                   0.00(0,0ms)         0.00(0,0ms)          0 442879.43 -1.0000                       

D:\duanbo\Desktop>
```



## 安装控件台

> 官方 RocketMQ Console 控制台镜像名为 `styletang/rocketmq-console-ng`

```bash
docker run -d --name rocketmq-console-ng \
    -p 8280:8080 \
    -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878 -Dlogging.level.root=info -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
    styletang/rocketmq-console-ng:1.0.0
```

> 安装运行正常后，可通过如下地址访问控制台：

```bash
http://192.168.9.241:8280/
```



## 安装后环境

> RocketMQ 4.6.0 版本测试（192.168.9.241）环境信息

| 服务名称      | 服务地址                                                     | 说明                               |
| ------------- | ------------------------------------------------------------ | ---------------------------------- |
| Name Server   | 192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878     | 伪集群（4.6.0 版本）               |
| Broker Server | 192.168.9.241:30911<br/>192.168.9.241:30921<br/>192.168.9.241:30931 | 伪集群（4.6.0 版本，dledger 模式） |
| Console       | http://192.168.9.241:8280                                    | 1.0.0 版本                         |





# 附-SCA依赖 

> dafrwo 应用中引入 spring-cloud-starter-stream-rocketmq

```xml
<!-- 消息队列引入此依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>

<!-- 消息事件引入此依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-rocketmq</artifactId>
</dependency>
```

> 文档
>
> https://github.com/apache/rocketmq
> https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/rocketmq-example

