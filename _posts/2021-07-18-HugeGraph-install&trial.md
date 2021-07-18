---
layout: post
title:  "HugeGraph 安装及试用"
date:   2020-07-17 23:00:00
categories: HugeGraph
tags: HugeGraph
author: poazy
---

* content
{:toc}
> HugeGraph 安装及试用（hugegraph、hugegraph-studio、hugegraph-hubble、hugegraph-tools、hugegraph-loader）






# 前置准备

* JDK 环境要配置好

* 创建目录

```bash
mkdir -p /boazy/middleware/hugegraph && cd /boazy/middleware/hugegraph
```

* 配置约定

  rest 服务端口设置为：18080

  gremlin 服务端口设置为：18182

  所有本地实例 host 均设置为：0.0.0.0

# hugegraph

首先要进入指定目录

```bash
cd /boazy/middleware/hugegraph
```

我们这里要安装完行 1 个 hugegraph-studio 实例

## 安装 hugegraph

* 下载 hugegraph

```bash
wget https://github.com/hugegraph/hugegraph/releases/download/v0.11.2-b3/hugegraph-0.11.2-b3.tar.gz
```

* 解压 hugegraph

```bash
tar -zxvf hugegraph-0.11.2-b3.tar.gz
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> 总用量 0
> drwxr-xr-x. 10  501 games 113 7月  17 16:51 hugegraph-0.11.2
> -rw-r--r--. 1 root root 200M 7月  17 16:26 hugegraph-0.11.2-b3.tar.gz
> [root@localhost hugegraph]#
> ```

## 配置 hugegraph

* 配置 rest-server.properties

> 配置 host 、端口和图实例（这里配置 9 个图实例 gdb01~gdb09）

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2 \
&& sed -i "s/restserver.url=http:\/\/127.0.0.1:8080/restserver.url=http:\/\/0.0.0.0:18080/g" conf/rest-server.properties \
&& sed -i "s/#gremlinserver.url=http:\/\/127.0.0.1:8182/gremlinserver.url=http:\/\/127.0.0.1:18182/g" conf/rest-server.properties \
&& sed -i "s/hugegraph:conf\/hugegraph.properties/gdb01:conf\/gdb01.properties,gdb02:conf\/gdb02.properties,gdb03:conf\/gdb03.properties,gdb04:conf\/gdb04.properties,gdb05:conf\/gdb05.properties,gdb06:conf\/gdb06.properties,gdb07:conf\/gdb07.properties,gdb08:conf\/gdb08.properties,gdb09:conf\/gdb09.properties/g" conf/rest-server.properties
```

> 查看结果：
>
> ```bash
> cat conf/rest-server.properties |grep ".url=http" \
> && cat conf/rest-server.properties |grep "graphs="
> ```
>
> ```bash
> [root@centos7-qscft hugegraph-0.11.2]# cat conf/rest-server.properties |grep ".url=http" \
> > && cat conf/rest-server.properties |grep "graphs="
> restserver.url=http://0.0.0.0:18080
> gremlinserver.url=http://127.0.0.1:18182
> graphs=[gdb01:conf/gdb01.properties,gdb02:conf/gdb02.properties,gdb03:conf/gdb03.properties,gdb04:conf/gdb04.properties,gdb05:conf/gdb05.properties,gdb06:conf/gdb06.properties,gdb07:conf/gdb07.properties,gdb08:conf/gdb08.properties,gdb09:conf/gdb09.properties]
> [root@centos7-qscft hugegraph-0.11.2]#
> ```

* 配置 gremlin-server.yaml

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2 \
&& sed -i "s/#host: 127.0.0.1/host: 0.0.0.0/g"  conf/gremlin-server.yaml \
&& sed -i "s/#port: 8182/port: 18182/g" conf/gremlin-server.yaml \
&& sed -i "s/ hugegraph: conf\/hugegraph.properties/ gdb01: conf\/gdb01.properties\n,gdb02: conf\/gdb02.properties\n,gdb03: conf\/gdb03.properties\n,gdb04: conf\/gdb04.properties\n,gdb05: conf\/gdb05.properties\n,gdb06: conf\/gdb06.properties\n,gdb07: conf\/gdb07.properties\n,gdb08: conf\/gdb08.properties\n,gdb09: conf\/gdb09.properties/g" conf/gremlin-server.yaml
```

> 查看结果：
>
> ```bash
> cat conf/gremlin-server.yaml |grep "host: " \
> && cat conf/gremlin-server.yaml |grep "port: " \
> && cat conf/gremlin-server.yaml |grep "graphs: " \
> && cat conf/gremlin-server.yaml |grep ": conf/"
> ```
>
> ```bash
> [root@centos7-qscft hugegraph-0.11.2]# cat conf/gremlin-server.yaml |grep "host: " \
> > && cat conf/gremlin-server.yaml |grep "port: " \
> > && cat conf/gremlin-server.yaml |grep "graphs: " \
> > && cat conf/gremlin-server.yaml |grep ": conf/"
> host: 0.0.0.0
> port: 18182
> graphs: {
>   gdb01: conf/gdb01.properties
> ,gdb02: conf/gdb02.properties
> ,gdb03: conf/gdb03.properties
> ,gdb04: conf/gdb04.properties
> ,gdb05: conf/gdb05.properties
> ,gdb06: conf/gdb06.properties
> ,gdb07: conf/gdb07.properties
> ,gdb08: conf/gdb08.properties
> ,gdb09: conf/gdb09.properties
> [root@centos7-qscft hugegraph-0.11.2]#
> ```

## 配置图实例配置文件

* 切换到 hugegraph-0.11.2 目录

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2
```

* 复制图实例配置文件（9 个）

```bash
for N in 1 2 3 4 5 6 7 8 9
do cp conf/hugegraph.properties conf/gdb0${N}.properties
done
```

> 查看复制结果：
>
> ```bash
> [root@localhost hugegraph-0.11.2]# ll -h conf
> 总用量 80K
> -rw-r--r--. 1  501 games  505 3月  29 20:57 computer.yaml
> -rw-r--r--. 1 root root  2.0K 7月  17 16:45 gdb01.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:47 gdb02.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:47 gdb03.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:48 gdb04.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:48 gdb05.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:48 gdb06.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:48 gdb07.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:49 gdb08.properties
> -rw-r--r--. 1 root root  2.0K 7月  17 16:49 gdb09.properties
> -rw-r--r--. 1  501 games  245 3月  29 20:57 gremlin-driver-settings.yaml
> -rw-r--r--. 1  501 games 4.4K 7月  17 16:50 gremlin-server.yaml
> -rw-r--r--. 1  501 games  720 3月  29 20:57 hugegraph-community.license
> -rw-r--r--. 1  501 games 2.0K 3月  29 20:57 hugegraph.properties
> -rw-r--r--. 1  501 games 1.7K 3月  29 20:57 hugegraph-server.keystore
> -rw-r--r--. 1  501 games 1.8K 3月  29 20:57 log4j2.xml
> -rw-r--r--. 1  501 games  245 3月  29 20:57 remote-objects.yaml
> -rw-r--r--. 1  501 games  245 3月  29 20:57 remote.yaml
> -rw-r--r--. 1  501 games  593 7月  17 16:51 rest-server.properties
> [root@localhost hugegraph-0.11.2]#
> ```

* 配置图实例文件（9 个）

```bash
for N in 1 2 3 4 5 6 7 8 9
do sed -i "s/store=hugegraph/store=gdb0${N}/g" conf/gdb0${N}.properties \
&& sed -i "s/#rocksdb.data_path=\/path\/to\/disk/rocksdb.data_path=rocksdb-data\/gdb0${N}/g" conf/gdb0${N}.properties \
&& sed -i "s/#rocksdb.wal_path=\/path\/to\/disk/rocksdb.wal_path=rocksdb-data\/gdb0${N}/g" conf/gdb0${N}.properties
done
```

> 查看配置结果：
>
> ```bash
> for N in 1 2 3 4 5 6 7 8 9
> do cat conf/gdb0${N}.properties |grep "store=" \
> && cat conf/gdb0${N}.properties |grep "rocksdb." |grep -v "# rocksdb "
> done
> ```
>
> ```bash
> [root@localhost hugegraph-0.11.2]# for N in 1 2 3 4 5 6 7 8 9
> > do cat conf/gdb0${N}.properties |grep "store=" \
> > && cat conf/gdb0${N}.properties |grep "rocksdb." |grep -v "# rocksdb "
> > done
> store=gdb01
> rocksdb.data_path=./rocksdb-data/gdb01
> rocksdb.wal_path=./rocksdb-data/gdb01
> store=gdb02
> rocksdb.data_path=./rocksdb-data/gdb02
> rocksdb.wal_path=./rocksdb-data/gdb02
> store=gdb03
> rocksdb.data_path=./rocksdb-data/gdb03
> rocksdb.wal_path=./rocksdb-data/gdb03
> store=gdb04
> rocksdb.data_path=./rocksdb-data/gdb04
> rocksdb.wal_path=./rocksdb-data/gdb04
> store=gdb05
> rocksdb.data_path=./rocksdb-data/gdb05
> rocksdb.wal_path=./rocksdb-data/gdb05
> store=gdb06
> rocksdb.data_path=./rocksdb-data/gdb06
> rocksdb.wal_path=./rocksdb-data/gdb06
> store=gdb07
> rocksdb.data_path=./rocksdb-data/gdb07
> rocksdb.wal_path=./rocksdb-data/gdb07
> store=gdb08
> rocksdb.data_path=./rocksdb-data/gdb08
> rocksdb.wal_path=./rocksdb-data/gdb08
> store=gdb09
> rocksdb.data_path=./rocksdb-data/gdb09
> rocksdb.wal_path=./rocksdb-data/gdb09
> [root@localhost hugegraph-0.11.2]#
> ```

## 初始化图实例

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/init-store.sh
```

> 结果：
>
> ```bash
> [root@centos7-qscft hugegraph-0.11.2]# cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/init-store.sh
> Initializing HugeGraph Store...
> 2021-07-18 03:25:24 2099  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb01.properties
> 2021-07-18 03:25:24 2215  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb01/m
> 2021-07-18 03:25:25 2427  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb01/m' with database 'gdb01', try to init CF later
> 2021-07-18 03:25:25 2470  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb01/s
> main dict load finished, time elapsed 1161 ms
> model load finished, time elapsed 42 ms.
> 2021-07-18 03:25:26 3729  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb01/g
> 2021-07-18 03:25:27 4617  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb01' has been initialized
> 2021-07-18 03:25:27 4617  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb01]
> 2021-07-18 03:25:27 4634  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb02.properties
> 2021-07-18 03:25:27 4639  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb02/m
> 2021-07-18 03:25:27 4645  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb02/m' with database 'gdb02', try to init CF later
> 2021-07-18 03:25:27 4672  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb02/s
> 2021-07-18 03:25:27 4694  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb02/g
> 2021-07-18 03:25:27 4789  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb02' has been initialized
> 2021-07-18 03:25:27 4789  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb02]
> 2021-07-18 03:25:27 4799  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb03.properties
> 2021-07-18 03:25:27 4803  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb03/m
> 2021-07-18 03:25:27 4810  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb03/m' with database 'gdb03', try to init CF later
> 2021-07-18 03:25:27 4822  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb03/s
> 2021-07-18 03:25:27 4841  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb03/g
> 2021-07-18 03:25:27 4945  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb03' has been initialized
> 2021-07-18 03:25:27 4946  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb03]
> 2021-07-18 03:25:27 4951  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb04.properties
> 2021-07-18 03:25:27 4958  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb04/m
> 2021-07-18 03:25:27 4964  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb04/m' with database 'gdb04', try to init CF later
> 2021-07-18 03:25:27 4974  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb04/s
> 2021-07-18 03:25:27 5002  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb04/g
> 2021-07-18 03:25:27 5101  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb04' has been initialized
> 2021-07-18 03:25:27 5101  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb04]
> 2021-07-18 03:25:27 5108  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb05.properties
> 2021-07-18 03:25:27 5112  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb05/m
> 2021-07-18 03:25:27 5118  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb05/m' with database 'gdb05', try to init CF later
> 2021-07-18 03:25:27 5156  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb05/s
> 2021-07-18 03:25:27 5177  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb05/g
> 2021-07-18 03:25:28 5360  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb05' has been initialized
> 2021-07-18 03:25:28 5360  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb05]
> 2021-07-18 03:25:28 5365  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb06.properties
> 2021-07-18 03:25:28 5378  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb06/m
> 2021-07-18 03:25:28 5386  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb06/m' with database 'gdb06', try to init CF later
> 2021-07-18 03:25:28 5407  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb06/s
> 2021-07-18 03:25:28 5426  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb06/g
> 2021-07-18 03:25:28 5515  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb06' has been initialized
> 2021-07-18 03:25:28 5515  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb06]
> 2021-07-18 03:25:28 5528  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb07.properties
> 2021-07-18 03:25:28 5535  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb07/m
> 2021-07-18 03:25:28 5540  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb07/m' with database 'gdb07', try to init CF later
> 2021-07-18 03:25:28 5555  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb07/s
> 2021-07-18 03:25:28 5572  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb07/g
> 2021-07-18 03:25:28 5662  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb07' has been initialized
> 2021-07-18 03:25:28 5662  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb07]
> 2021-07-18 03:25:28 5670  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb08.properties
> 2021-07-18 03:25:28 5681  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb08/m
> 2021-07-18 03:25:28 5691  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb08/m' with database 'gdb08', try to init CF later
> 2021-07-18 03:25:28 5704  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb08/s
> 2021-07-18 03:25:28 5735  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb08/g
> 2021-07-18 03:25:28 5834  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb08' has been initialized
> 2021-07-18 03:25:28 5834  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb08]
> 2021-07-18 03:25:28 5839  [main] [INFO ] com.baidu.hugegraph.cmd.InitStore [] - Init graph with config file: conf/gdb09.properties
> 2021-07-18 03:25:28 5848  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb09/m
> 2021-07-18 03:25:28 5854  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Failed to open RocksDB 'rocksdb-data/gdb09/m' with database 'gdb09', try to init CF later
> 2021-07-18 03:25:28 5867  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb09/s
> 2021-07-18 03:25:28 5888  [db-open-1] [INFO ] com.baidu.hugegraph.backend.store.rocksdb.RocksDBStore [] - Opening RocksDB with data path: rocksdb-data/gdb09/g
> 2021-07-18 03:25:28 5969  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Graph 'gdb09' has been initialized
> 2021-07-18 03:25:28 5969  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Close graph standardhugegraph[gdb09]
> 2021-07-18 03:25:28 5991  [Thread-1] [INFO ] com.baidu.hugegraph.HugeGraph [] - HugeGraph is shutting down
> Initialization finished.
> [root@centos7-qscft hugegraph-0.11.2]# ll -h rocksdb-data/
> total 0
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb01
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb02
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb03
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb04
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb05
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb06
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb07
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb08
> drwxr-xr-x. 5 root root 33 7月  17 16:51 gdb09
> [root@centos7-qscft hugegraph-0.11.2]#
> ```

## 启动 hugegraph

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/start-hugegraph.sh
```

> 结果：
>
> ```bash
> [root@localhost hugegraph-0.11.2]# cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/start-hugegraph.sh
> Starting HugeGraphServer...
> Connecting to HugeGraphServer (http://0.0.0.0:18080/graphs)......OK
> Started [pid 32333]
> [root@localhost hugegraph-0.11.2]# lsof -i:18080
> COMMAND  PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    6310 root  257u  IPv6 4023355      0t0  TCP *:18080 (LISTEN)
> [root@localhost hugegraph-0.11.2]# lsof -i:18182
> COMMAND  PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    6310 root  255u  IPv6 3991889      0t0  TCP *:opsec-ufp (LISTEN)
> [root@localhost hugegraph-0.11.2]#
> ```

## 关闭 hugegraph

```bash
cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/stop-hugegraph.sh
```

> 结果：
>
> ```bash
> [root@localhost hugegraph-0.11.2]# cd /boazy/middleware/hugegraph/hugegraph-0.11.2 && bin/stop-hugegraph.sh
> no crontab for root
> The HugeGraphServer monitor has been closed
> Killing HugeGraphServer(pid 32333)...OK
> [root@localhost hugegraph-0.11.2]#
> ```

## 配置防火墙

* 添加端口白名单

```bash
for N in 18080 18182
do firewall-cmd --zone=public --add-port=${N}/tcp --permanent
done
```

* 让端口白名单生效

```bash
firewall-cmd --reload && firewall-cmd --zone=public --list-ports
```

> 结果：
>
> ```bash
> [root@localhost hugegraph-0.11.2]# firewall-cmd --reload && firewall-cmd --zone=public --list-ports
> success
> 18080/tcp 18182/tcp
> [root@localhost hugegraph-0.11.2]#
> ```



# hugegraph-studio

首先要进入指定目录

```bash
cd /boazy/middleware/hugegraph
```

我们这里要安装完行多个 hugegraph-studio 实例

## 安装 hugegraph-studio

* 下载 hugegraph-studio

```bash
wget https://github.com/hugegraph/hugegraph-studio/releases/download/v0.11.0/hugegraph-studio-0.11.0.tar.gz
```

* 解压 hugegraph-studio

```bash
tar -zxvf hugegraph-studio-0.11.0.tar.gz
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> 总用量 0
> drwxr-xr-x. 10  501 games 113 7月  17 16:51 hugegraph-0.11.2
> drwxr-xr-x.  7 root root   93 7月  17 15:46 hugegraph-studio-0.11.0
> -rw-r--r--. 1 root root  43M 7月  17 16:24 hugegraph-studio-0.11.0.tar.gz
> [root@localhost hugegraph]#
> ```

* 复制多个 hugegraph-studio 实例目录

> 这里要安装 9 个 hugegraph-studio 实例

```bash
for N in 1 2 3 4 5 6 7 8 9
do cp -r hugegraph-studio-0.11.0 720${N}-hugegraph-studio-0.11.0
done
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> 总用量 0
> drwxr-xr-x. 10 root root  166 7月  17 17:24 7201-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7202-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7203-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7204-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7205-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7206-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7207-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7208-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7209-hugegraph-studio-0.11.0
> drwxr-xr-x. 10  501 games 113 7月  17 16:51 hugegraph-0.11.2
> drwxr-xr-x.  7 root root   93 7月  17 15:46 hugegraph-studio-0.11.0
> -rw-r--r--. 1 root root  43M 7月  17 16:24 hugegraph-studio-0.11.0.tar.gz
> [root@localhost hugegraph]#
> ```

## 配置 hugegraph-studio

* 配置多个  hugegraph-studio 实例的配置信息

```bash
for N in 1 2 3 4 5 6 7 8 9
do sed -i "s/studio.server.port=8088/studio.server.port=720${N}/g" 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties \
  && sed -i "s/studio.server.host=localhost/studio.server.host=0.0.0.0/g" 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties \
  && sed -i "s/graph.server.host=localhost/graph.server.host=127.0.0.1/g" 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties \
  && sed -i "s/graph.server.port=8080/graph.server.port=18080/g" 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties \
  && sed -i "s/graph.name=gdb00/graph.name=gdb0${N}/g" 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties
done
```

* 检查配置信息

```bash
for N in 1 2 3 4 5 6 7 8 9
do cat 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties |grep ".server." |grep -v ".ui" |grep -v ".api" \
  && cat 720${N}-hugegraph-studio-0.11.0/conf/hugegraph-studio.properties |grep "graph.name="
done
```

## 启动 hugegraph-studio

* 启动多个 hugegraph-studio 实例

```bash
for N in 1 2 3 4 5 6 7 8 9
do cd /boazy/middleware/hugegraph/720${N}-hugegraph-studio-0.11.0 \
  && nohup bin/hugegraph-studio.sh > /dev/null 2>nohup.out & \
done
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# for N in 1 2 3 4 5 6 7 8 9
> > do cd /boazy/middleware/hugegraph/720${N}-hugegraph-studio-0.11.0 \
> >   && nohup bin/hugegraph-studio.sh > /dev/null 2>nohup.out & \
> > done
> [1] 10267
> [2] 10268
> [3] 10269
> [4] 10270
> [5] 10272
> [6] 10274
> [7] 10276
> [8] 10278
> [9] 10280
> [root@localhost hugegraph]#
> ```

* 查看多个 hugegraph-studio 实例启动状态（端口）

```bash
for N in 1 2 3 4 5 6 7 8 9
do lsof -i:720${N}
done
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# for N in 1 2 3 4 5 6 7 8 9
> > do lsof -i:720${N}
> > done
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10271 root   80u  IPv6 3967891      0t0  TCP *:dlip (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10273 root   80u  IPv6 4009120      0t0  TCP *:7202 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10275 root   80u  IPv6 3957626      0t0  TCP *:7203 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10277 root   80u  IPv6 3967889      0t0  TCP *:7204 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10279 root   80u  IPv6 3964776      0t0  TCP *:7205 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10281 root   80u  IPv6 3955505      0t0  TCP *:7206 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10282 root   80u  IPv6 3992674      0t0  TCP *:7207 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10283 root   80u  IPv6 4008122      0t0  TCP *:7208 (LISTEN)
> COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    10284 root   80u  IPv6 4007792      0t0  TCP *:7209 (LISTEN)
> [root@localhost hugegraph]#
> ```

## 关闭 hugegraph-studio

* 关闭所有的 hugegraph-studio 实例

```bash
ps -ef |grep hugegraph-studio-0.11.0 |grep -v "grep" |awk '{print $2}' |xargs -r kill -9
```

* 关闭后再查看（已经没有 hugegraph-studio 的进程了）

```bash
[root@localhost hugegraph]# ps -ef |grep hugegraph-studio-0.11.0 |grep -v "grep"
[root@localhost hugegraph]#
```

## 配置防火墙

* 添加端口白名单

```bash
for N in 1 2 3 4 5 6 7 8 9
do firewall-cmd --zone=public --add-port=720${N}/tcp --permanent
done
```

* 让端口白名单生效

```bash
firewall-cmd --reload && firewall-cmd --zone=public --list-ports
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# firewall-cmd --reload && firewall-cmd --zone=public --list-ports
> success
> 18080/tcp 18182/tcp 7201/tcp 7202/tcp 7203/tcp 7204/tcp 7205/tcp 7206/tcp 7207/tcp 7208/tcp 7209/tcp
> [root@localhost hugegraph]#
> ```

## 实例访问地址

```bash
第一组：http://192.168.0.89:7201/
第二组：http://192.168.0.89:7202/
第三组：http://192.168.0.89:7203/
第四组：http://192.168.0.89:7204/
第五组：http://192.168.0.89:7205/
第六组：http://192.168.0.89:7206/
第七组：http://192.168.0.89:7207/
第八组：http://192.168.0.89:7208/
合并组：http://192.168.0.89:7209/
```



# hugegraph-hubble

## 安装 hugegraph-hubble

* 下载 hugegraph-hubble

```bash
wget https://github.com/hugegraph/hugegraph-hubble/releases/download/v1.5.0/hugegraph-hubble-1.5.0.tar.gz
```

* 解压 hugegraph-hubble

```bash
tar -zxvf hugegraph-hubble-1.5.0.tar.gz
```

> 查看解压结果：
>
> ```bash
> [root@c hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> total 32K
> drwxr-xr-x  8  501 games 4.0K Jun 29 21:51 hugegraph-hubble-1.5.0
> -rw-r--r--. 1 root root  117M Jun 29 21:40 hugegraph-hubble-1.5.0.tar.gz
> [root@c hugegraph]#
> ```

## 配置 hugegraph-hubble

* 配置 hugegraph-hubble.properties 文件

```bash
cd /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0 \
&& sed -i "s/server.host=localhost/server.host=0.0.0.0/g" conf/hugegraph-hubble.properties \
&& sed -i "s/server.port=8088/server.port=18088/g" conf/hugegraph-hubble.properties
```

> 查看结果：
>
> ```bash
> 
> ```

## 启动 hugegraph-hubble

* 启动 hugegraph-hubble 实例

```bash
cd /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0 && bin/start-hubble.sh
```

> 结果：
>
> ```bash
> [root@c hugegraph-hubble-1.5.0]# cd /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0 && bin/start-hubble.sh
> starting HugeGraphHubble.....OK
> logging to /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0/logs/hugegraph-hubble.log
> [root@c hugegraph-hubble-1.5.0]# lsof -i:18088
> COMMAND  PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
> java    3090 root  240u  IPv6 5524676      0t0  TCP *:radan-http (LISTEN)
> [root@c hugegraph-hubble-1.5.0]#
> ```

## 关闭 hugegraph-hubble

* 关闭  hugegraph-hubble 实例

```bash
cd /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0 && bin/stop-hubble.sh
```

> 结果：
>
> ```bash
> [root@c hugegraph-hubble-1.5.0]# cd /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0 && bin/stop-hubble.sh
> stopped HugeGraphHubble
> [root@c hugegraph-hubble-1.5.0]#
> ```

## 配置防火墙

* 添加端口白名单

```bash
for N in 18088
do firewall-cmd --zone=public --add-port=${N}/tcp --permanent
done
```

* 让端口白名单生效

```bash
firewall-cmd --reload && firewall-cmd --zone=public --list-ports
```

> 结果：
>
> ```bash
> [root@localhost hugegraph]# firewall-cmd --reload && firewall-cmd --zone=public --list-ports
> success
> 18080/tcp 18088/tcp 18182/tcp 7201/tcp 7202/tcp 7203/tcp 7204/tcp 7205/tcp 7206/tcp 7207/tcp 7208/tcp 7209/tcp
> [root@localhost hugegraph]#
> ```

## 实例访问地址

```bash
http://192.168.0.89:18088/
```



# hugegraph-tools

## 安装 hugegraph-tools

* 下载 hugegraph-tools

```bash
wget https://github.com/hugegraph/hugegraph-tools/releases/download/v1.5.0/hugegraph-tools-1.5.0.tar.gz
```

* 解压 hugegraph-tools

```bash
tar -zxvf hugegraph-tools-1.5.0.tar.gz
```

> 查看解压结果：
>
> ```bash
> [root@localhost hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> 总用量 0
> drwxr-xr-x. 10 root root  166 7月  17 17:24 7201-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7202-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7203-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7204-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7205-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7206-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7207-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7208-hugegraph-studio-0.11.0
> drwxr-xr-x. 10 root root  166 7月  17 17:32 7209-hugegraph-studio-0.11.0
> drwxr-xr-x. 10  501 games 113 7月  17 16:51 hugegraph-0.11.2
> drwxr-xr-x.  7 root root   93 7月  17 15:46 hugegraph-studio-0.11.0
> drwxrwxr-x.  5 1    1      40 11月 25 2020 hugegraph-tools-1.5.0
> -rw-r--r--. 1 root root  53M 7月  17 16:25 hugegraph-tools-1.5.0.tar.gz
> [root@localhost hugegraph]#
> ```

## 备份与恢复数据

* 要先切换到 hugegraph-tools-1.5.0 目录

```bash
cd /boazy/middleware/hugegraph/hugegraph-tools-1.5.0
```

* 备份

> 这里演示备份 gdb01~gdb09 实例的数据！

```bash
for N in 1 2 3 4 5 6 7 8 9
do bin/hugegraph --graph gdb0${N} --url http://192.168.0.89:18080 backup -t all -d gdb0${N}
done
```

* 恢复

> 这里演示将 gdb01~gdb08 实例的备份数据恢复（合并）到 gdb09 实例中！

```bash
bin/hugegraph --graph gdb09 --url http://192.168.0.89:18080 graph-mode-set -m MERGING && \
for N in 1 2 3 4 5 6 7 8
do bin/hugegraph --graph gdb09 --url http://192.168.0.89:18080 restore -t all -d gdb0${N}
done
```



# hugegraph-loader

## 安装 hugegraph-loader

* 下载 hugegraph-tools

```bash
wget https://github.com/hugegraph/hugegraph-loader/releases/download/v0.11.1/hugegraph-loader-0.11.1.tar.gz
```

* 解压 hugegraph-tools

```bash
tar -zxvf hugegraph-loader-0.11.1.tar.gz
```

> 查看解压结果：
>
> ```bash
> [root@c hugegraph]# pwd && ll -h
> /boazy/middleware/hugegraph
> total 32K
> drwxr-xr-x  8  501 games 4.0K Jun 29 21:51 hugegraph-loader-0.11.1
> -rw-r--r--. 1 root root  96M  Jun 29 21:40 hugegraph-loader-0.11.1.tar.gz
> [root@c hugegraph]#
> ```

## 数据导入

* 要先切换到 hugegraph-loader-0.11.1 目录

```bash
cd /boazy/middleware/hugegraph/hugegraph-loader-0.11.1
```

* 数据导入

> 先要将要志入的数据上传到 hugegraph-loader-0.11.1 下然后再执行导入命令

```bash
sh bin/hugegraph-loader.sh -g gdb09 -s stcok-data/schema.groovy -f stcok-data/struct.json -h 127.0.0.1 -p -18080
```



# 遇到的问题

## 执行 sh 文件错误

* 执行 sh 文件时出现错误

```bash
/bin/bash^M: bad interpreter: No such file or directory
```

* 解决方法


```bash
# 安装 dos2unix
yum -y install dos2unix
# 转换 sh 文件
dos2unix bin/hugegraph-studio.sh
```

## 设置开机启动

* 通过配置 /etc/rc.d/rc.local 实现

```bash
vi /etc/rc.d/rc.local
```

> 内容（根据应用实际情况）如下：

```bash
# su - root -c 'systemctl restart network'
su - root -c 'sh /boazy/middleware/hugegraph/hugegraph-0.11.2/bin/start-hugegraph.sh'
su - root -c 'nohup /boazy/middleware/hugegraph/hugegraph-studio-0.11.0/bin/hugegraph-studio.sh &'
su - root -c 'sh /boazy/middleware/hugegraph/hugegraph-hubble-1.5.0/bin/start-hubble.sh'
```

## 查路径

* 查最短路径

```groovy
g.V().hasId('35:严兰')
  .store('x').repeat(both().where(without('x')).aggregate('x'))
  .until(hasId('35:袁宁'))
  .path().limit(1)
```

* 查单向路径

```groovy
g.V().hasId('35:严兰')
  .repeat(out().simplePath()).until(hasId('35:袁宁'))
  .path()
```

* 查双向路径

```bash
g.V().hasId('35:严兰')
  .repeat(both().simplePath()).until(hasId('35:袁宁'))
  .path()
```

* 查路径，出度为 0 截止

```groovy
g.V()
  .has('中文名',within('小麦滋养保湿洁面乳','欧莱雅(LOREAL)青春密码酵素精华肌底液'))
  .repeat(out()).until(outE().count().is(0))
  .path()
```

## 删除点和边

```groovy
g.E().drop()
g.V().drop()
```

## 删除 Label

```groovy
// 查询所有 indexLabel
graph.schema().getIndexLabels()
// 查询所有 edgeLabel
graph.schema().getEdgeLabels()
// 查询所有 vertexLabel
graph.schema().getVertexLabels()
// 查询所有 propertyKey
graph.schema().getPropertyKeys()

// 删除 indexLabel
graph.schema().indexLabel("personByAge").remove()
// 删除 edgeLabel
graph.schema().edgeLabel("rate").remove()
// 删除 vertexLabel
graph.schema().vertexLabel("person").remove()
// 删除 propertyKey
graph.schema().propertyKey("weight").remove()
```

> 测试：
>
> ```bash
> graph.schema().edgeLabel("属于.ee").remove();
> graph.schema().edgeLabel("属于.ec").remove();
> graph.schema().edgeLabel("拥有.ee").remove();
> graph.schema().edgeLabel("有.ce").remove();
> graph.schema().edgeLabel("有.cc").remove();
> graph.schema().edgeLabel("有.ee").remove();
> graph.schema().edgeLabel("用.ee").remove();
> graph.schema().edgeLabel("开.ee").remove();
> graph.schema().edgeLabel("属于.ec").remove();
> graph.schema().edgeLabel("吃.ee").remove();
> graph.schema().edgeLabel("听.ee").remove();
> graph.schema().edgeLabel("是.ee").remove();
> ```

## hugegraph-client 文档

```bash
https://hugegraph.github.io/hugegraph-doc/clients/hugegraph-client.html
```







