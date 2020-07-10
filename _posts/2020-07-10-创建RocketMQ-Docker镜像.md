---
layout: post
title:  "创建 RocketMQ Docker 镜像"
date:   2020-07-10 00:00:00
categories: RocketMQ
tags: RocketMQ
author: poazy
---

* content
{:toc}
> 目前官方 RocketMQ 镜像 `hub.docker.com` 只有 `4.6.0` 版本及以下版本的镜像，而 `4.6.1`、`4.7.0` 和 `4.7.1` 版本在 `hub.docker.com` 是无镜像，如果需要用 docker 安装 4.7.1 版本的 RocketMQ 那只能自己打包镜像；
>
> 这里我基于 `RocketMQ 4.7.1` 和 `Rocket Console 1.0.1` 分别打包两个镜像。







# 创建 RocketMQ 镜像

## rocketmq 镜像

> 打包当前 `(20200710)` 最新版本 4.7.1 版本镜像
>
> 镜像用于 `Name Server` 和 `Broker Server`

### 克隆 rocketmq-docker 源码

> 推使用加速克隆地址克隆代码，官方  `github` 地址很慢的

```bash
cd /boazy/tmp/
# 官方克隆地址
git clone https://github.com/apache/rocketmq-docker.git
# 加速克隆地址（推介用它，快）
git clone https://github.com.cnpmjs.org/apache/rocketmq-docker.git
```

> 下载后的位置

```bash
[root@centos7-qscft tmp]# pwd
/boazy/tmp
[root@centos7-qscft tmp]# ll -h
total 8.0K
drwxr-xr-x. 30 root root 4.0K Jul  2 02:10 rocketmq-docker
[root@centos7-qscft tmp]#
```

### 打包 rocketmq 镜像

> Tip: The supported RMQ-VERSIONs can be obtained from [here](https://archive.apache.org/dist/rocketmq/). The supported BASE-IMAGEs are [centos, alpine].
>
> 提示：可以从 [这里](https://archive.apache.org/dist/rocketmq/) 获得受支持的 RMQ-VERSION。 支持的 BASE-IMAGE 是[centos，alpine]。
>
> 执行build-image.sh 里，会自动下载 `curl -L https://archive.apache.org/dist/rocketmq/4.7.1/rocketmq-all-4.7.1-bin-release.zip -o rocketmq.zip`

```bash
cd /boazy/tmp/rocketmq-docker/image-build
# sh build-image.sh RMQ-VERSION BASE-IMAGE
sh build-image.sh 4.7.1 centos
```

> 打包结果如下，可以看到 4.7.1 版本已经打包成功了并且 docker images 也可以看到了：

```bash
。。。以下日志省略
Step 21/22 : USER ${user}
 ---> Running in 17f2c463f4a3
Removing intermediate container 17f2c463f4a3
 ---> dadbbf2b6082
Step 22/22 : WORKDIR ${ROCKETMQ_HOME}/bin
 ---> Running in 2d71154f0475
Removing intermediate container 2d71154f0475
 ---> 7f3c586adc46
Successfully built 7f3c586adc46
Successfully tagged apacherocketmq/rocketmq:4.7.1
[root@centos7-qscft image-build]#
[root@centos7-qscft image-build]# docker images | grep rocketmq
apacherocketmq/rocketmq                     4.7.1                            7f3c586adc46        1 minutes ago      500MB
apacherocketmq/rocketmq                     4.6.0                            1318fbff3674        5 months ago        496MB
styletang/rocketmq-console-ng               1.0.0                            7df83bb6e638        2 years ago         702MB
[root@centos7-qscft image-build]#
```



## rocketmq-console-ng 镜像

> 打包当前 `(20200710)` 最新版本 1.0.1 版本镜像
>
> 镜像用于 `RocketMQ 控制台`

### 克隆 rocketmq-externals 代码

> 推使用加速克隆地址克隆代码，官方  `github` 地址很慢的

```bash
cd /boazy/tmp/
# 官方克隆地址
git clone https://github.com/apache/rocketmq-externals.git
# 加速克隆地址（推介用它，快）
git clone https://github.com.cnpmjs.org/apache/rocketmq-externals.git
```

> 下载后的位置

```bash
[root@centos7-qscft tmp]# pwd
/boazy/tmp
[root@centos7-qscft tmp]# ll -h
total 8.0K
drwxr-xr-x. 30 root root 4.0K Jul  2 02:10 rocketmq-docker
drwxr-xr-x. 30 root root 4.0K Jul  2 02:10 rocketmq-externals
[root@centos7-qscft tmp]#
```

### 打包 rocketmq-console-ng 镜像

```bash
# 切换到 rocketmq-console 源码目录
cd /boazy/tmp/rocketmq-externals/rocketmq-console/ 
# （这里不切换分支了，默认使用 master 分支搞）
# 编译源码并打包镜像
mvn clean package -U -Dmaven.test.skip=true docker:build
```

> 以下是编译的结果

```bash
[root@centos7-qscft target]# pwd
/boazy/tmp/rocketmq-externals/rocketmq-console/target
[root@centos7-qscft target]# ll -h
total 40M
-rw-r--r--. 1 root root  11K Jul  2 04:10 checkstyle-cachefile
-rw-r--r--. 1 root root 5.8K Jul  2 04:10 checkstyle-checker.xml
-rw-r--r--. 1 root root  12K Jul  2 04:10 checkstyle-result.xml
drwxr-xr-x. 4 root root  130 Jul  2 04:10 classes
drwxr-xr-x. 2 root root   61 Jul  2 04:17 docker
drwxr-xr-x. 3 root root   25 Jul  2 04:10 generated-sources
drwxr-xr-x. 2 root root   28 Jul  2 04:10 maven-archiver
drwxr-xr-x. 3 root root   35 Jul  2 04:10 maven-status
-rw-r--r--. 1 root root  32M Jul  2 04:10 rocketmq-console-ng-1.0.1.jar
-rw-r--r--. 1 root root 3.7M Jul  2 04:10 rocketmq-console-ng-1.0.1.jar.original
-rw-r--r--. 1 root root 3.7M Jul  2 04:10 rocketmq-console-ng-1.0.1-sources.jar
drwxr-xr-x. 2 root root   29 Jul  2 04:18 test-classes
[root@centos7-qscft target]#
```

> 打包镜像结果如下，可以看到 1.0.1 版本 和 latest 版本已经打包成功了并且 docker images 也可以看到了：

```bash
。。。以上日志省略
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
Status: Downloaded newer image for java:8
 ---> d23bdf5b1b1b
Step 2/6 : VOLUME /tmp

 ---> Running in 81d20a2cb42c
Removing intermediate container 81d20a2cb42c
 ---> 81207d26f370
Step 3/6 : ADD rocketmq-console-ng-*.jar rocketmq-console-ng.jar

 ---> 81b95e01749e
Step 4/6 : RUN sh -c 'touch /rocketmq-console-ng.jar'

 ---> Running in 7c708c448d79
Removing intermediate container 7c708c448d79
 ---> dcdae99da58f
Step 5/6 : ENV JAVA_OPTS=""

 ---> Running in b3da84ba1311
Removing intermediate container b3da84ba1311
 ---> 0ea245d3f1fa
Step 6/6 : ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /rocketmq-console-ng.jar" ]

 ---> Running in 1521fd138ffc
Removing intermediate container 1521fd138ffc
 ---> eff7d2ae650f
ProgressMessage{id=null, status=null, stream=null, error=null, progress=null, progressDetail=null}
Successfully built eff7d2ae650f
Successfully tagged styletang/rocketmq-console-ng:latest
[INFO] Built styletang/rocketmq-console-ng
[INFO] Tagging styletang/rocketmq-console-ng with 1.0.1
[INFO] Tagging styletang/rocketmq-console-ng with latest
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 8:24.663s
[INFO] Finished at: Wed Jul 01 20:18:53 UTC 2020
[INFO] Final Memory: 61M/579M
[INFO] ------------------------------------------------------------------------
[root@centos7-qscft rocketmq-console]#
[root@centos7-qscft rocketmq-console]# docker images |grep rocketmq
styletang/rocketmq-console-ng               1.0.1                            eff7d2ae650f        17 minutes ago      710MB
styletang/rocketmq-console-ng               latest                           eff7d2ae650f        17 minutes ago      710MB
apacherocketmq/rocketmq                     4.7.1                            7f3c586adc46        About an hour ago   500MB
apacherocketmq/rocketmq                     4.6.0                            1318fbff3674        5 months ago        496MB
styletang/rocketmq-console-ng               1.0.0                            7df83bb6e638        2 years ago         702MB
[root@centos7-qscft rocketmq-console]#
[root@centos7-qscft rocketmq-console]# docker images |grep rocketmq
styletang/rocketmq-console-ng               1.0.1                            eff7d2ae650f        1 minutes ago      710MB
styletang/rocketmq-console-ng               latest                           eff7d2ae650f        1 minutes ago      710MB
apacherocketmq/rocketmq                     4.7.1                            7f3c586adc46        About an hour ago   500MB
apacherocketmq/rocketmq                     4.6.0                            1318fbff3674        5 months ago        496MB
styletang/rocketmq-console-ng               1.0.0                            7df83bb6e638        2 years ago         702MB
[root@centos7-qscft rocketmq-console]#
```



# 附-运行镜像

> 运行上面创建好的 `RocketMQ 4.7.0` 和 `RocketMQ Console 1.0.1` 版本镜像
>
> 按照之前安装 4.6.0 版本的方式安装即可，这里不再详讲了

```bash
# 1）创建目录并配置目录
```

```bash
# 2）安装 Name Server
# 实例二，9877
docker run -d --name mqnamesrv-0 \
    -p 9876:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-0/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.7.1 sh mqnamesrv
# 实例二，9877
docker run -d --name mqnamesrv-1 \
    -p 9877:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-1/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.7.1 sh mqnamesrv
# 实例三，9878
docker run -d --name mqnamesrv-2 \
    -p 9878:9876 \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/mqnamesrv-2/store:/home/rocketmq/store \
    apacherocketmq/rocketmq:4.7.1 sh mqnamesrv
```

```bash
# 3）安装 Rocket Console
docker run -d --name rocketmq-console-ng \
    -p 8280:8080 \
    -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878 -Dlogging.level.root=info -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
    styletang/rocketmq-console-ng:1.0.1
```

```bash
# 4）安装 Broker Server
# 实例一
docker run -d --name mqbroker-n0 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n0/conf:/home/rocketmq/rocketmq-4.7.1/conf \
    apacherocketmq/rocketmq:4.7.1 sh mqbroker -c ../conf/dledger/broker-n0.conf
# 实例二
docker run -d --name mqbroker-n1 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n1/conf:/home/rocketmq/rocketmq-4.7.1/conf \
    apacherocketmq/rocketmq:4.7.1 sh mqbroker -c ../conf/dledger/broker-n1.conf
# 实例三
docker run -d --name mqbroker-n2 \
    --net=host \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/logs:/home/rocketmq/logs \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/store:/home/rocketmq/store \
    -v /boazy/data/dockerdata/rocketmq/broker-n2/conf:/home/rocketmq/rocketmq-4.7.1/conf \
    apacherocketmq/rocketmq:4.7.1 sh mqbroker -c ../conf/dledger/broker-n2.conf
```

* docker 运行 4.7.1 版本结果，如下

```bash
[root@centos7-qscft docker]# docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS                                                NAMES
965b39df1a8c        apacherocketmq/rocketmq:4.7.1         "sh mqbroker -c ../c…"   4 minutes ago       Up 4 minutes                                                             mqbroker-n2
20e8a40483fa        apacherocketmq/rocketmq:4.7.1         "sh mqbroker -c ../c…"   5 minutes ago       Up 5 minutes                                                             mqbroker-n1
aacef0e44520        apacherocketmq/rocketmq:4.7.1         "sh mqbroker -c ../c…"   6 minutes ago       Up 6 minutes                                                             mqbroker-n0
6cd50632dc69        styletang/rocketmq-console-ng:1.0.1   "sh -c 'java $JAVA_O…"   7 minutes ago       Up 7 minutes        0.0.0.0:8280->8080/tcp                               rocketmq-console-ng
68768fc27409        apacherocketmq/rocketmq:4.7.1         "sh mqnamesrv"           8 minutes ago       Up 8 minutes        10909/tcp, 10911-10912/tcp, 0.0.0.0:9878->9876/tcp   mqnamesrv-2
7f356662a052        apacherocketmq/rocketmq:4.7.1         "sh mqnamesrv"           8 minutes ago       Up 8 minutes        10909/tcp, 10911-10912/tcp, 0.0.0.0:9877->9876/tcp   mqnamesrv-1
21a68ce8b051        apacherocketmq/rocketmq:4.7.1         "sh mqnamesrv"           9 minutes ago       Up 9 minutes        10909/tcp, 0.0.0.0:9876->9876/tcp, 10911-10912/tcp   mqnamesrv-0
```

* 查看集群结果，如下

```bash
Microsoft Windows [版本 10.0.18363.900]                                                   (c) 2019 Microsoft Corporation。保留所有权利。                                                                                                                                         D:\duanbo\Desktop>mqadmin.cmd clusterList -n "192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878"                 RocketMQLog:WARN No appenders could be found for logger (io.netty.util.internal.PlatformDependent0). 
RocketMQLog:WARN Please initialize the logger system properly.                           #Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE                           RaftCluster       RaftNode00              0     192.168.9.241:30911    V4_7_1                   0.00(0,0ms)         0.00(0,0ms)          0 442882.67 -1.0000                       RaftCluster       RaftNode00              2     192.168.9.241:30921    V4_7_1                   0.00(0,0ms)         0.00(0,0ms)          0 442882.67 -1.0000                       RaftCluster       RaftNode00              3     192.168.9.241:30931    V4_7_1                   0.00(0,0ms)         0.00(0,0ms)          0 442882.67 -1.0000                                                                                                                 D:\duanbo\Desktop>     
```

* RocketMQ 4.7.1 版本测试（192.168.9.241）环境信息

| 服务名称      | 服务地址                                                     | 说明                               |
| ------------- | ------------------------------------------------------------ | ---------------------------------- |
| Name Server   | 192.168.9.241:9876;192.168.9.241:9877;192.168.9.241:9878     | 伪集群（4.7.1 版本）               |
| Broker Server | 192.168.9.241:30911<br/>192.168.9.241:30921<br/>192.168.9.241:30931 | 伪集群（4.7.1 版本，dledger 模式） |
| Console       | http://192.168.9.241:8280                                    | 1.0.1 版本                         |





# 附-编译 RocketMQ 源码

> **[选看]**

## 克隆 rocketmq 源码

> rocketmq 镜像用于 `Name Server` 和 `Broker Server`

* 克隆源码

> 推使用加速克隆地址克隆代码，官方  `github` 地址很慢的

```bash
# 官方克隆地址
git clone https://github.com/apache/rocketmq.git
# 加速克隆地址
git clone https://github.com.cnpmjs.org/apache/rocketmq.git
```

> 下载后的位置

```bash
[root@centos7-qscft tmp]# pwd
/boazy/tmp
[root@centos7-qscft tmp]# ll -h
total 8.0K
drwxr-xr-x. 23 root root 4.0K Jul  2 02:06 rocketmq
[root@centos7-qscft tmp]#
```

## 编译 rocketmq  源码

```bash
# 切换到 rocketmq 源码目录
cd /boazy/tmp/rocketmq/ 
# 切换到 release-4.7.1 分支（这里不切换了，默认使用 master 分支搞）
# 编译源码
mvn -Prelease-all -DskipTests clean install -U
```

> 编译结果，如下：

```bash
。。。以上信息省略
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Apache RocketMQ 4.7.1 ............................. SUCCESS [22:56.642s]
[INFO] rocketmq-logging 4.7.1 ............................ SUCCESS [2:27.753s]
[INFO] rocketmq-remoting 4.7.1 ........................... SUCCESS [3:08.902s]
[INFO] rocketmq-common 4.7.1 ............................. SUCCESS [24.943s]
[INFO] rocketmq-client 4.7.1 ............................. SUCCESS [1:49.932s]
[INFO] rocketmq-store 4.7.1 .............................. SUCCESS [2:17.173s]
[INFO] rocketmq-srvutil 4.7.1 ............................ SUCCESS [5.064s]
[INFO] rocketmq-filter 4.7.1 ............................. SUCCESS [2:49.279s]
[INFO] rocketmq-acl 4.7.1 ................................ SUCCESS [31.605s]
[INFO] rocketmq-broker 4.7.1 ............................. SUCCESS [3.731s]
[INFO] rocketmq-tools 4.7.1 .............................. SUCCESS [2.183s]
[INFO] rocketmq-namesrv 4.7.1 ............................ SUCCESS [1.161s]
[INFO] rocketmq-logappender 4.7.1 ........................ SUCCESS [33.831s]
[INFO] rocketmq-openmessaging 4.7.1 ...................... SUCCESS [5.051s]
[INFO] rocketmq-example 4.7.1 ............................ SUCCESS [1.287s]
[INFO] rocketmq-test 4.7.1 ............................... SUCCESS [24.295s]
[INFO] rocketmq-distribution 4.7.1 ....................... SUCCESS [11:32.697s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 15:18.182s
[INFO] Finished at: Wed Jul 01 19:05:35 UTC 2020
[INFO] Final Memory: 64M/533M
[INFO] ------------------------------------------------------------------------
[root@centos7-qscft rocketmq]#
```

```bash
[root@centos7-qscft target]# pwd
/boazy/tmp/rocketmq/distribution/target
[root@centos7-qscft target]# ll -h
total 27M
drwxr-xr-x. 2 root root    6 Jul  2 02:55 archive-tmp
-rw-r--r--. 1 root root   87 Jul  2 03:05 checkstyle-cachefile
-rw-r--r--. 1 root root 6.0K Jul  2 03:05 checkstyle-checker.xml
-rw-r--r--. 1 root root   83 Jul  2 03:05 checkstyle-result.xml
drwxr-xr-x. 3 root root   22 Jul  2 02:54 maven-shared-archive-resources
drwxr-xr-x. 3 root root   28 Jul  2 03:05 rocketmq-4.7.1
-rw-r--r--. 1 root root  14M Jul  2 03:05 rocketmq-4.7.1.tar.gz
-rw-r--r--. 1 root root  14M Jul  2 03:05 rocketmq-4.7.1.zip
[root@centos7-qscft target]#
```





