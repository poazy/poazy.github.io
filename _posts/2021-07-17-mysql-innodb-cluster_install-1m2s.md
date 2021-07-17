---
layout: post
title:  "Docker 安装 MySQL InnoDB Cluster(8.0.25)"
date:   2020-07-12 23:30:00
categories: MySQL
tags: MySQL
author: poazy
---

* content
{:toc}
> Docker 下使用 MySQL Shell + MySQL Router 安装 MySQL InnoDB Cluster;
>
> 版本为：8.0.25，采用伪集群方式（在一台机器上部署）。





# 前置准备

* 拉取 Docker 镜像

```bash
# 先拉取要使用的 Docker 镜像
docker pull mysql/mysql-server:8.0.25
docker pull mysql/mysql-router:8.0.25
```

* 创建 Docker 网络

```bash
# 创建一个 Docker 网络给后面安装 MySQL InnoDB Cluster 使用
docker network create innodbnet
```



# 运行 MySQL 实例

## 启动 MySQL 容器

* 安装运行3个 MySQL 8.0 数据库实例

```bash
for N in 1 2 3
do docker run -d --restart=always --name mysql$N --hostname=mysql$N --net=innodbnet \
  -p 33${N}6:3306 -p 33${N}60-33${N}61:33060-33061 \
  -v /boazy/data/dockerdata/mysql$N/data:/var/lib/mysql \
  -v /boazy/data/dockerdata/mysql$N/log:/var/log/mysql \
  -v /etc/localtime:/etc/localtime:ro \
  -e MYSQL_ROOT_PASSWORD=root@123 \
  -e "TZ=Asia/Shanghai" \
  mysql/mysql-server:8.0.25 \
  --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
done
```

* 查询 MySQL 容器状态

```bash
docker ps -a
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# docker ps
> CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                    PORTS                                                                        NAMES
> 8fe36a5522bf        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   38 seconds ago      Up 37 seconds (healthy)   0.0.0.0:3336->3306/tcp, 0.0.0.0:33360->33060/tcp, 0.0.0.0:33361->33061/tcp   mysql3
> c8284f4ad8a2        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   39 seconds ago      Up 38 seconds (healthy)   0.0.0.0:3326->3306/tcp, 0.0.0.0:33260->33060/tcp, 0.0.0.0:33261->33061/tcp   mysql2
> f9d8d0b0c9bb        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   40 seconds ago      Up 39 seconds (healthy)   0.0.0.0:3316->3306/tcp, 0.0.0.0:33160->33060/tcp, 0.0.0.0:33161->33061/tcp   mysql1
> [root@centos7-qscft dockerdata]#
> ```



## 创建用户并授权

* 创建用户并对用户授权

```bash
for N in 1 2 3
do docker exec -it mysql$N mysql -uroot -proot@123 \
  -e "CREATE USER 'inno'@'%' IDENTIFIED BY 'inno';" \
  -e "GRANT ALL privileges ON *.* TO 'inno'@'%' with grant option;" \
  -e "reset master;"
done
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# for N in 1 2 3
> > do docker exec -it mysql$N mysql -uroot -proot@123 \
> >   -e "CREATE USER 'inno'@'%' IDENTIFIED BY 'inno';" \
> >   -e "GRANT ALL privileges ON *.* TO 'inno'@'%' with grant option;" \
> >   -e "reset master;"
> > done
> mysql: [Warning] Using a password on the command line interface can be insecure.
> mysql: [Warning] Using a password on the command line interface can be insecure.
> mysql: [Warning] Using a password on the command line interface can be insecure.
> [root@centos7-qscft dockerdata]#
> ```

* 检查用户是否被成功创建

```bash
for N in 1 2 3
do docker exec -it mysql$N mysql -uinno -pinno \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SELECT user FROM mysql.user where user = 'inno';"
done
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# for N in 1 2 3
> > do docker exec -it mysql$N mysql -uinno -pinno \
> >   -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
> >   -e "SELECT user FROM mysql.user where user = 'inno';"
> > done
> mysql: [Warning] Using a password on the command line interface can be insecure.
> +---------------+--------+
> | Variable_name | Value  |
> +---------------+--------+
> | hostname      | mysql1 |
> +---------------+--------+
> +------+
> | user |
> +------+
> | inno |
> +------+
> mysql: [Warning] Using a password on the command line interface can be insecure.
> +---------------+--------+
> | Variable_name | Value  |
> +---------------+--------+
> | hostname      | mysql2 |
> +---------------+--------+
> +------+
> | user |
> +------+
> | inno |
> +------+
> mysql: [Warning] Using a password on the command line interface can be insecure.
> +---------------+--------+
> | Variable_name | Value  |
> +---------------+--------+
> | hostname      | mysql3 |
> +---------------+--------+
> +------+
> | user |
> +------+
> | inno |
> +------+
> [root@centos7-qscft dockerdata]#
> ```



# 配置 InnoDB Cluster

## 进入 MySQL Shell 

* 进入 MySQL Shell 

> 在 mysql1 实例上运行 MySQL Shell

```bash
docker exec -it mysql1 mysqlsh -uroot -proot@123 -S/var/run/mysqld/mysqlx.sock
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# docker exec -it mysql1 mysqlsh -uroot -proot@123 -S/var/run/mysqld/mysqlx.sock
> Cannot set LC_ALL to locale en_US.UTF-8: No such file or directory
> MySQL Shell 8.0.25
> 
> Copyright (c) 2016, 2021, Oracle and/or its affiliates.
> Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
> Other names may be trademarks of their respective owners.
> 
> Type '\help' or '\?' for help; '\quit' to exit.
> WARNING: Using a password on the command line interface can be insecure.
> Creating a session to 'root@/var%2Frun%2Fmysqld%2Fmysqlx.sock'
> Fetching schema names for autocompletion... Press ^C to stop.
> Your MySQL connection id is 13 (X protocol)
> Server version: 8.0.25 MySQL Community Server - GPL
> No default schema selected; type \use <schema> to set one.
>  MySQL  localhost+ ssl  JS >
> ```

## 检查配置

* 检查配置

```shell
dba.checkInstanceConfiguration("inno@mysql1:3306")
```

> 输入密码 inno
>
> 结果：
>
> ```shell
>  MySQL  localhost+ ssl  JS > dba.checkInstanceConfiguration("inno@mysql1:3306")
> Please provide the password for 'inno@mysql1:3306': ****
> Validating local MySQL instance listening at port 3306 for use in an InnoDB cluster...
> 
> This instance reports its own address as mysql1:3306
> Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
> 
> Checking whether existing tables comply with Group Replication requirements...
> No incompatible tables detected
> 
> Checking instance configuration...
> 
> NOTE: Some configuration options need to be fixed:
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> | Variable                               | Current Value | Required Value | Note                                             |
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> | binlog_transaction_dependency_tracking | COMMIT_ORDER  | WRITESET       | Update the server variable                       |
> | enforce_gtid_consistency               | OFF           | ON             | Update read-only variable and restart the server |
> | gtid_mode                              | OFF           | ON             | Update read-only variable and restart the server |
> | server_id                              | 1             | <unique ID>    | Update read-only variable and restart the server |
> | slave_parallel_type                    | DATABASE      | LOGICAL_CLOCK  | Update the server variable                       |
> | slave_preserve_commit_order            | OFF           | ON             | Update the server variable                       |
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> 
> Some variables need to be changed, but cannot be done dynamically on the server.
> NOTE: Please use the dba.configureInstance() command to repair these issues.
> 
> {
>     "config_errors": [
>         {
>             "action": "server_update",
>             "current": "COMMIT_ORDER",
>             "option": "binlog_transaction_dependency_tracking",
>             "required": "WRITESET"
>         },
>         {
>             "action": "server_update+restart",
>             "current": "OFF",
>             "option": "enforce_gtid_consistency",
>             "required": "ON"
>         },
>         {
>             "action": "server_update+restart",
>             "current": "OFF",
>             "option": "gtid_mode",
>             "required": "ON"
>         },
>         {
>             "action": "server_update+restart",
>             "current": "1",
>             "option": "server_id",
>             "required": "<unique ID>"
>         },
>         {
>             "action": "server_update",
>             "current": "DATABASE",
>             "option": "slave_parallel_type",
>             "required": "LOGICAL_CLOCK"
>         },
>         {
>             "action": "server_update",
>             "current": "OFF",
>             "option": "slave_preserve_commit_order",
>             "required": "ON"
>         }
>     ],
>     "status": "error"
> }
>  MySQL  localhost+ ssl  JS >
> ```

> 依次以同样的方式检查 mysql2 和 mysql 3
>
> dba.checkInstanceConfiguration("inno@mysql2:3306")
>
> dba.checkInstanceConfiguration("inno@mysql3:3306")

## 配置实例

* 配置 InnoDB Cluster 实例（mysql1）

```shell
dba.configureInstance("inno@mysql1:3306")
```

> 输入密码 inno 和两次都是 y
>
> 结果：
>
> ```shell
>  MySQL  localhost+ ssl  JS > dba.configureInstance("inno@mysql1:3306")
> Please provide the password for 'inno@mysql1:3306': ****
> Configuring local MySQL instance listening at port 3306 for use in an InnoDB cluster...
> 
> This instance reports its own address as mysql1:3306
> Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
> 
> applierWorkerThreads will be set to the default value of 4.
> 
> NOTE: Some configuration options need to be fixed:
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> | Variable                               | Current Value | Required Value | Note                                             |
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> | binlog_transaction_dependency_tracking | COMMIT_ORDER  | WRITESET       | Update the server variable                       |
> | enforce_gtid_consistency               | OFF           | ON             | Update read-only variable and restart the server |
> | gtid_mode                              | OFF           | ON             | Update read-only variable and restart the server |
> | server_id                              | 1             | <unique ID>    | Update read-only variable and restart the server |
> | slave_parallel_type                    | DATABASE      | LOGICAL_CLOCK  | Update the server variable                       |
> | slave_preserve_commit_order            | OFF           | ON             | Update the server variable                       |
> +----------------------------------------+---------------+----------------+--------------------------------------------------+
> 
> Some variables need to be changed, but cannot be done dynamically on the server.
> Do you want to perform the required configuration changes? [y/n]: y
> Do you want to restart the instance after configuring it? [y/n]: y
> Configuring instance...
> The instance 'mysql1:3306' was configured to be used in an InnoDB cluster.
> Restarting MySQL...
> NOTE: MySQL server at mysql1:3306 was restarted.
>  MySQL  localhost+ ssl  JS >
> ```

> **依次以同样的方式配置 mysql2 和 mysql 3**
>
> ```shell
> dba.configureInstance("inno@mysql2:3306")
> ```
>
> ``` shell
> dba.configureInstance("inno@mysql3:3306")
> ```

## 退出 MySQL Shell

* 退出 MySQL Shell

```shell
\quit
```

>结果：
>
>```shell
> MySQL  localhost+ ssl  JS > \quit
>Bye!
>[root@centos7-qscft dockerdata]#
>```

## 重启 MySQL

* 重新启动数据库

```bash
docker restart mysql1 mysql2 mysql3
```

> 结果：
>
> ```shell
> [root@centos7-qscft dockerdata]# docker restart mysql1 mysql2 mysql3
> mysql1
> mysql2
> mysql3
> [root@centos7-qscft dockerdata]# docker ps -a
> CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                    PORTS                                                                        NAMES
> 8fe36a5522bf        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   5 minutes ago       Up 31 seconds (healthy)   0.0.0.0:3336->3306/tcp, 0.0.0.0:33360->33060/tcp, 0.0.0.0:33361->33061/tcp   mysql3
> c8284f4ad8a2        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   5 minutes ago       Up 32 seconds (healthy)   0.0.0.0:3326->3306/tcp, 0.0.0.0:33260->33060/tcp, 0.0.0.0:33261->33061/tcp   mysql2
> f9d8d0b0c9bb        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   5 minutes ago       Up 35 seconds (healthy)   0.0.0.0:3316->3306/tcp, 0.0.0.0:33160->33060/tcp, 0.0.0.0:33161->33061/tcp   mysql1
> [root@centos7-qscft dockerdata]#
> ```



# 创建 InnoDB Cluster

* 进入 MySQL Shell 

> 在 mysql1 实例上运行 MySQL Shell

```bash
docker exec -it mysql1 mysqlsh -uroot -proot@123 -S/var/run/mysqld/mysqlx.sock
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# docker exec -it mysql1 mysqlsh -uroot -proot@123 -S/var/run/mysqld/mysqlx.sock
> Cannot set LC_ALL to locale en_US.UTF-8: No such file or directory
> MySQL Shell 8.0.25
> 
> Copyright (c) 2016, 2021, Oracle and/or its affiliates.
> Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
> Other names may be trademarks of their respective owners.
> 
> Type '\help' or '\?' for help; '\quit' to exit.
> WARNING: Using a password on the command line interface can be insecure.
> Creating a session to 'root@/var%2Frun%2Fmysqld%2Fmysqlx.sock'
> Fetching schema names for autocompletion... Press ^C to stop.
> Your MySQL connection id is 41 (X protocol)
> Server version: 8.0.25 MySQL Community Server - GPL
> No default schema selected; type \use <schema> to set one.
>  MySQL  localhost+ ssl  JS >
> ```

* 连接（mysql1）用户实例，并输入用户密码 inno 回车

```shell
\c inno@mysql1:3306
```

> 输入密码 inno
>
> 结果：
>
> ```shell
>  MySQL  localhost+ ssl  JS > \c inno@mysql1:3306
> Creating a session to 'inno@mysql1:3306'
> Please provide the password for 'inno@mysql1:3306': ****
> Fetching schema names for autocompletion... Press ^C to stop.
> Closing old connection...
> Your MySQL connection id is 44
> Server version: 8.0.25 MySQL Community Server - GPL
> No default schema selected; type \use <schema> to set one.
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 创建集群

```shell
var cluster = dba.createCluster("mycluster")
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > var cluster = dba.createCluster("mycluster")
> A new InnoDB cluster will be created on instance 'mysql1:3306'.
> 
> Validating instance configuration at mysql1:3306...
> 
> This instance reports its own address as mysql1:3306
> 
> Instance configuration is suitable.
> NOTE: Group Replication will communicate with other members using 'mysql1:33061'. Use the localAddress option to override.
> 
> Creating InnoDB cluster 'mycluster' on 'mysql1:3306'...
> 
> Adding Seed Instance...
> Cluster successfully created. Use Cluster.addInstance() to add MySQL instances.
> At least 3 instances are needed for the cluster to be able to withstand up to
> one server failure.
> 
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 查看集群状态

```shell
cluster.status()
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > cluster.status()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "primary": "mysql1:3306",
>         "ssl": "REQUIRED",
>         "status": "OK_NO_TOLERANCE",
>         "statusText": "Cluster is NOT tolerant to any failures.",
>         "topology": {
>             "mysql1:3306": {
>                 "address": "mysql1:3306",
>                 "memberRole": "PRIMARY",
>                 "mode": "R/W",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             }
>         },
>         "topologyMode": "Single-Primary"
>     },
>     "groupInformationSourceMember": "mysql1:3306"
> }
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 查看集群描述

```shell
cluster.describe()
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > cluster.describe()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "topology": [
>             {
>                 "address": "mysql1:3306",
>                 "label": "mysql1:3306",
>                 "role": "HA"
>             }
>         ],
>         "topologyMode": "Single-Primary"
>     }
> }
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 不用退出 MySQL Shell，接着往下执行将其他 MySQL 添加到 InnoDB Cluster！



# 添加到 InnoDB Cluster

* 将 mysql2 添加到 InnoDB Cluster

```shell
cluster.addInstance("inno@mysql2:3306")
```

> 提示输入时，输入 I
>
> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > cluster.addInstance("inno@mysql2:3306")
> 
> NOTE: The target instance 'mysql2:3306' has not been pre-provisioned (GTID set is empty). The Shell is unable to decide whether incremental state recovery can correctly provision it.
> The safest and most convenient way to provision a new instance is through automatic clone provisioning, which will completely overwrite the state of 'mysql2:3306' with a physical snapshot from an existing cluster member. To use this method by default, set the 'recoveryMethod' option to 'clone'.
> 
> The incremental state recovery may be safely used if you are sure all updates ever executed in the cluster were done with GTIDs enabled, there are no purged transactions and the new instance contains the same GTID set as the cluster or a subset of it. To use this method by default, set the 'recoveryMethod' option to 'incremental'.
> 
> 
> Please select a recovery method [C]lone/[I]ncremental recovery/[A]bort (default Clone): I
> Validating instance configuration at mysql2:3306...
> 
> This instance reports its own address as mysql2:3306
> 
> Instance configuration is suitable.
> NOTE: Group Replication will communicate with other members using 'mysql2:33061'. Use the localAddress option to override.
> 
> A new instance will be added to the InnoDB cluster. Depending on the amount of
> data on the cluster this might take from a few seconds to several hours.
> 
> Adding instance to the cluster...
> 
> Monitoring recovery process of the new cluster member. Press ^C to stop monitoring and let it continue in background.
> State recovery already finished for 'mysql2:3306'
> 
> The instance 'mysql2:3306' was successfully added to the cluster.
> 
>  MySQL  mysql1:3306 ssl  JS >
> ```

> 依次同样方法将 mysql3 添加到 InnoDB Cluster
>
> ```shell
> cluster.addInstance("inno@mysql3:3306")
> ```

* 查看集群状态

```shell
cluster.status()
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > cluster.status()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "primary": "mysql1:3306",
>         "ssl": "REQUIRED",
>         "status": "OK",
>         "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
>         "topology": {
>             "mysql1:3306": {
>                 "address": "mysql1:3306",
>                 "memberRole": "PRIMARY",
>                 "mode": "R/W",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             },
>             "mysql2:3306": {
>                 "address": "mysql2:3306",
>                 "memberRole": "SECONDARY",
>                 "mode": "R/O",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             },
>             "mysql3:3306": {
>                 "address": "mysql3:3306",
>                 "memberRole": "SECONDARY",
>                 "mode": "R/O",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             }
>         },
>         "topologyMode": "Single-Primary"
>     },
>     "groupInformationSourceMember": "mysql1:3306"
> }
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 查看集群描述

```shell
cluster.describe()
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > cluster.describe()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "topology": [
>             {
>                 "address": "mysql1:3306",
>                 "label": "mysql1:3306",
>                 "role": "HA"
>             },
>             {
>                 "address": "mysql2:3306",
>                 "label": "mysql2:3306",
>                 "role": "HA"
>             },
>             {
>                 "address": "mysql3:3306",
>                 "label": "mysql3:3306",
>                 "role": "HA"
>             }
>         ],
>         "topologyMode": "Single-Primary"
>     }
> }
>  MySQL  mysql1:3306 ssl  JS >
> ```

* 退出 MySQL Shell

```shell
\quit
```

> 结果：
>
> ```shell
>  MySQL  mysql1:3306 ssl  JS > \quit
> Bye!
> [root@centos7-qscft dockerdata]#
> ```



# 运行 MySQL Router 实例

## 启动 MySQL Router 容器

* 安装运行 MySQL Router 实例

```bash
for N in 1 2
do docker run -d --restart=always --name mysql-router$N --net=innodbnet \
   -p ${N}8443:8443 -p ${N}6446-${N}6449:6446-6449 \
   -e MYSQL_HOST=mysql1 -e MYSQL_PORT=3306 -e MYSQL_USER=inno -e MYSQL_PASSWORD=inno \
   -e MYSQL_INNODB_CLUSTER_MEMBERS=3 \
   -v /etc/localtime:/etc/localtime:ro \
   -e "TZ=Asia/Shanghai" \
   mysql/mysql-router:8.0.25
done
```

* 查询 MySQL Router 容器状态

```bash
docker ps -a
```

> 结果：
>
> ```basi
> [root@centos7-qscft dockerdata]# docker ps -a
> 071ce9a7051d        mysql/mysql-router:8.0.25   "/run.sh mysqlrouter"    38 seconds ago      Up 37 seconds (healthy)   0.0.0.0:26446->6446/tcp, 0.0.0.0:26447->6447/tcp, 0.0.0.0:26448->6448/tcp, 0.0.0.0:26449->6449/tcp, 0.0.0.0:28443->8443/tcp   mysql-router2
> 9cead9dd8f04        mysql/mysql-router:8.0.25   "/run.sh mysqlrouter"    39 seconds ago      Up 38 seconds (healthy)   0.0.0.0:16446->6446/tcp, 0.0.0.0:16447->6447/tcp, 0.0.0.0:16448->6448/tcp, 0.0.0.0:16449->6449/tcp, 0.0.0.0:18443->8443/tcp   mysql-router1
> 8fe36a5522bf        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   10 minutes ago      Up 6 minutes (healthy)    0.0.0.0:3336->3306/tcp, 0.0.0.0:33360->33060/tcp, 0.0.0.0:33361->33061/tcp                                                    mysql3
> c8284f4ad8a2        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   10 minutes ago      Up 6 minutes (healthy)    0.0.0.0:3326->3306/tcp, 0.0.0.0:33260->33060/tcp, 0.0.0.0:33261->33061/tcp                                                    mysql2
> f9d8d0b0c9bb        mysql/mysql-server:8.0.25   "/entrypoint.sh --ch…"   11 minutes ago      Up 6 minutes (healthy)    0.0.0.0:3316->3306/tcp, 0.0.0.0:33160->33060/tcp, 0.0.0.0:33161->33061/tcp                                                    mysql1
> [root@centos7-qscft dockerdata]#
> ```

* 查看日志

```bash
docker logs mysql-router1
```

> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# docker logs mysql-router1
> [Entrypoint] MYSQL_CREATE_ROUTER_USER is not set, Router will generate a new account to be used at runtime.
> [Entrypoint] Set it to 0 to reuse inno instead.
> [Entrypoint] Succesfully contacted mysql server at mysql1:3306. Checking for cluster state.
> 0
> 12
> [Entrypoint] Successfully contacted cluster with 3 members. Bootstrapping.
> [Entrypoint] Succesfully contacted mysql server at mysql1. Trying to bootstrap.
> Please enter MySQL password for inno:
> # Bootstrapping MySQL Router instance at '/tmp/mysqlrouter'...
> 
> - Creating account(s) (only those that are needed, if any)
> - Verifying account (using it to run SQL queries that would be run by Router)
> - Storing account in keyring
> - Adjusting permissions of generated files
> - Creating configuration /tmp/mysqlrouter/mysqlrouter.conf
> 
> # MySQL Router configured for the InnoDB Cluster 'mycluster'
> 
> After this MySQL Router has been started with the generated configuration
> 
>     $ mysqlrouter -c /tmp/mysqlrouter/mysqlrouter.conf
> 
> the cluster 'mycluster' can be reached by connecting to:
> 
> ## MySQL Classic protocol
> 
> - Read/Write Connections: localhost:6446
> - Read/Only Connections:  localhost:6447
> 
> ## MySQL X protocol
> 
> - Read/Write Connections: localhost:6448
> - Read/Only Connections:  localhost:6449
> 
> [Entrypoint] Starting mysql-router.
> 2021-07-12 21:43:19 http_server INFO [7f5c3dac9780] listening on 0.0.0.0:8443
> 2021-07-12 21:43:19 io INFO [7f5c3dac9780] starting 2 io-threads, using backend 'linux_epoll'
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c27fff700] Starting Metadata Cache
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c27fff700] Connections using ssl_mode 'PREFERRED'
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700] Starting metadata cache refresh thread
> 2021-07-12 21:43:19 routing INFO [7f5c0cff9700] [routing:mycluster_rw] started: routing strategy = first-available
> 2021-07-12 21:43:19 routing INFO [7f5c0d7fa700] [routing:mycluster_ro] started: routing strategy = round-robin-with-fallback
> 2021-07-12 21:43:19 routing INFO [7f5c0cff9700] Start accepting connections for routing routing:mycluster_rw listening on 6446
> 2021-07-12 21:43:19 routing INFO [7f5c0d7fa700] Start accepting connections for routing routing:mycluster_ro listening on 6447
> 2021-07-12 21:43:19 routing INFO [7f5c077fe700] [routing:mycluster_x_ro] started: routing strategy = round-robin-with-fallback
> 2021-07-12 21:43:19 routing INFO [7f5c077fe700] Start accepting connections for routing routing:mycluster_x_ro listening on 6449
> 2021-07-12 21:43:19 routing INFO [7f5c06ffd700] [routing:mycluster_x_rw] started: routing strategy = first-available
> 2021-07-12 21:43:19 routing INFO [7f5c06ffd700] Start accepting connections for routing routing:mycluster_x_rw listening on 6448
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700] Potential changes detected in cluster 'mycluster' after metadata refresh
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700] Metadata for cluster 'mycluster' has 1 replicasets:
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700] 'default' (3 members, single-primary)
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700]     mysql1:3306 / 33060 - mode=RW
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700]     mysql2:3306 / 33060 - mode=RO
> 2021-07-12 21:43:19 metadata_cache INFO [7f5c3dac0700]     mysql3:3306 / 33060 - mode=RO
> 2021-07-12 21:43:19 routing INFO [7f5c3dac0700] Routing routing:mycluster_ro listening on 6447 got request to disconnect invalid connections: metadata change
> 2021-07-12 21:43:19 routing INFO [7f5c3dac0700] Routing routing:mycluster_x_ro listening on 6449 got request to disconnect invalid connections: metadata change
> 2021-07-12 21:43:19 routing INFO [7f5c3dac0700] Routing routing:mycluster_rw listening on 6446 got request to disconnect invalid connections: metadata change
> 2021-07-12 21:43:19 routing INFO [7f5c3dac0700] Routing routing:mycluster_x_rw listening on 6448 got request to disconnect invalid connections: metadata change
> [root@centos7-qscft dockerdata]#
> ```
> 查看 mysql-router 的日志这里就不操作了：
>
> ```bash
> docker logs mysql-router
> ```


## 应用访问地址

* 从日志中可以看到

```bash
the cluster 'mycluster' can be reached by connecting to:

## MySQL Classic protocol

- Read/Write Connections: localhost:6446
- Read/Only Connections:  localhost:6447

## MySQL X protocol

- Read/Write Connections: localhost:6448
- Read/Only Connections:  localhost:6449
```



# MySQL Router 高可用

## Keepalived

* 安装多两个 MySQL Router 实例，采用 Keepalived 等工具实现？

```shell
https://www.cnblogs.com/williamzheng/p/11598475.html
```



## HAProxy

* /etc/haproxy/haproxy.cfg 文件添加内容

```bash
vi /etc/haproxy/haproxy.cfg
```

```bash
listen innodb-cluster-rw
    bind 0.0.0.0:6446
    option tcplog
    mode tcp
    balance roundrobin
    server mysql-router1 192.168.9.241:16446 check port 16446 inter 1s rise 2 fall 2
    server mysql-router2 192.168.9.241:26446 check port 26446 inter 1s rise 2 fall 2
 
listen innodb-cluster-ro
    bind 0.0.0.0:6447
    option tcplog
    mode tcp
    balance roundrobin
    server mysql-router1 192.168.9.241:16447 check port 16447 inter 1s rise 2 fall 2
    server mysql-router2 192.168.9.241:26447 check port 26447 inter 1s rise 2 fall 2
```

* 检查配置并重启

```bash
# 检查 haproxy.cfg
haproxy -f /etc/haproxy/haproxy.cfg -c
# 重启 haproxy
systemctl restart haproxy && systemctl status haproxy
```



# 中途添加 MySQL 实例

* 动态扩容从服务器（这里演示添加一台）

> 在整个集群运行后并有数据后，再创建一个新 MySQL 实例添加到集群中，数据能同步到新的 MySQL 实例中去的!

```bash
# 创建实例
for N in 4
do docker run -d --restart=always --name mysql$N --hostname=mysql$N --net=innodbnet \
  -p 33${N}6:3306 -p 33${N}60-33${N}61:33060-33061 \
  -v /boazy/data/dockerdata/mysql$N/data:/var/lib/mysql \
  -v /boazy/data/dockerdata/mysql$N/log:/var/log/mysql \
  -v /etc/localtime:/etc/localtime:ro \
  -e MYSQL_ROOT_PASSWORD=root@123 \
  -e "TZ=Asia/Shanghai" \
  mysql/mysql-server:8.0.25 \
  --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
done

# 查看实例状态，启动成功后继续后面的操作
docker ps -a

# 创建用户并授权
for N in 4
do docker exec -it mysql$N mysql -uroot -proot@123 \
  -e "CREATE USER 'inno'@'%' IDENTIFIED BY 'inno';" \
  -e "GRANT ALL privileges ON *.* TO 'inno'@'%' with grant option;" \
  -e "reset master;"
done
# 检查用户创建是否成功
for N in 4
do docker exec -it mysql$N mysql -uinno -pinno \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SELECT user FROM mysql.user where user = 'inno';"
done

# 进入 MySQL Shell
docker exec -it mysql1 mysqlsh -uinno -pinno -S/var/run/mysqld/mysqlx.sock
# 检查实例配置（提示输入密码）
dba.checkInstanceConfiguration("inno@mysql4:3306")
# 配置实例（提示输入密码、提示输入两次 y）
dba.configureInstance("inno@mysql4:3306")

# 退出 MySQL Shell
\quit

# 重启 mysql4 实例
docker restart mysql4

# 进入 MySQL Shell
docker exec -it mysql1 mysqlsh -uinno -pinno -S/var/run/mysqld/mysqlx.sock

# 获取 InnoDB Cluster
var cluster = dba.getCluster("mycluster")
# 将 mysql4 添加到 InnoDB Cluster
cluster.addInstance("inno@mysql4:3306")
# 到这里 mysql4 成功添加到集群中，数据成功同步到 mysql4 了
# 查看集群状态
cluster.status()

# 退出 MySQL Shell
\quit
```





# 遇到的问题

## 所有实例重起后问题？

* 当集群的所有节点都 offline，获取集群信息出错，怎么恢复集群?

```bash
docker exec -it mysql1 mysqlsh -uinno -pinno -S/var/run/mysqld/mysqlx.sock
```

```shell
var cluster = dba.getCluster("mycluster")
```

> Dba.getCluster: This function is not available through a session to a standalone instance (metadata exists, instance belongs to that metadata, but GR is not active) (MYSQLSH 51314)
>
> 结果：
>
> ```bash
> [root@centos7-qscft dockerdata]# docker exec -it mysql1 mysqlsh -uinno -pinno -S/var/run/mysqld/mysqlx.sock
> Cannot set LC_ALL to locale en_US.UTF-8: No such file or directory
> MySQL Shell 8.0.25
> 
> Copyright (c) 2016, 2021, Oracle and/or its affiliates.
> Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
> Other names may be trademarks of their respective owners.
> 
> Type '\help' or '\?' for help; '\quit' to exit.
> WARNING: Using a password on the command line interface can be insecure.
> Creating a session to 'root@/var%2Frun%2Fmysqld%2Fmysqlx.sock'
> Fetching schema names for autocompletion... Press ^C to stop.
> Your MySQL connection id is 59 (X protocol)
> Server version: 8.0.25 MySQL Community Server - GPL
> No default schema selected; type \use <schema> to set one.
>  MySQL  localhost+ ssl  JS > var cluster = dba.getCluster("mycluster")
> Dba.getCluster: This function is not available through a session to a standalone instance (metadata exists, instance belongs to that metadata, but GR is not active) (MYSQLSH 51314)
>  MySQL  localhost+ ssl  JS >
> ```

* 执行 `rebootClusterFromCompleteOutage` 命令，恢复集群

```shell
dba.rebootClusterFromCompleteOutage("mycluster")
```

> 提示中输入 n，会好些
>
> 结果：
>
> ```bash
>  MySQL  localhost+ ssl  JS > dba.rebootClusterFromCompleteOutage("mycluster")
> Restoring the cluster 'mycluster' from complete outage...
> 
> ERROR: The administrative account credentials for mysql3:3306 do not match the cluster's administrative account. The cluster administrative account user name and password must be the same on all instances that belong to it.
> ERROR: The administrative account credentials for mysql2:3306 do not match the cluster's administrative account. The cluster administrative account user name and password must be the same on all instances that belong to it.
> Could not open a connection to 'mysql3:3306': 'MySQL Error 1045: Could not open connection to 'mysql3:3306': Access denied for user 'root'@'172.21.0.2' (using password: YES)'
> Would you like to remove it from the cluster's metadata? [y/N]: y
> 
> Could not open a connection to 'mysql2:3306': 'MySQL Error 1045: Could not open connection to 'mysql2:3306': Access denied for user 'root'@'172.21.0.2' (using password: YES)'
> Would you like to remove it from the cluster's metadata? [y/N]: y
> 
> mysql1:3306 was restored.
> The cluster was successfully rebooted.
> 
> <Cluster:mycluster>
>  MySQL  localhost+ ssl  JS >
> ```

* 查看集群状态(有可能恢复正常。这里尚未正常)

```bash
var cluster = dba.getCluster("mycluster")
```

```bash
cluster.status()
```

> 结果：
>
> ```bash
>  MySQL  localhost+ ssl  JS > var cluster = dba.getCluster("mycluster")
>  MySQL  localhost+ ssl  JS > cluster.status()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "primary": "mysql1:3306",
>         "ssl": "REQUIRED",
>         "status": "OK_NO_TOLERANCE",
>         "statusText": "Cluster is NOT tolerant to any failures.",
>         "topology": {
>             "mysql1:3306": {
>                 "address": "mysql1:3306",
>                 "memberRole": "PRIMARY",
>                 "memberState": "(MISSING)",
>                 "mode": "n/a",
>                 "readReplicas": {},
>                 "role": "HA",
>                 "shellConnectError": "MySQL Error 1045 (28000): Access denied for user 'root'@'172.21.0.2' (using password: YES)",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             }
>         },
>         "topologyMode": "Single-Primary"
>     },
>     "groupInformationSourceMember": "mysql1:3306"
> }
>  MySQL  localhost+ ssl  JS >
> ```

* 将实例重新加入集群（这步不一定有！）

```bash
cluster.addInstance("inno@mysql2:3306")
```

```bash
cluster.addInstance("inno@mysql3:3306")
```

> 结果
>
> ```bash
>  MySQL  localhost+ ssl  JS > cluster.addInstance("inno@mysql2:3306")
> The safest and most convenient way to provision a new instance is through automatic clone provisioning, which will completely overwrite the state of 'mysql2:3306' with a physical snapshot from an existing cluster member. To use this method by default, set the 'recoveryMethod' option to 'clone'.
> 
> The incremental state recovery may be safely used if you are sure all updates ever executed in the cluster were done with GTIDs enabled, there are no purged transactions and the new instance contains the same GTID set as the cluster or a subset of it. To use this method by default, set the 'recoveryMethod' option to 'incremental'.
> 
> Incremental state recovery was selected because it seems to be safely usable.
> 
> Validating instance configuration at mysql2:3306...
> 
> This instance reports its own address as mysql2:3306
> 
> Instance configuration is suitable.
> NOTE: Group Replication will communicate with other members using 'mysql2:33061'. Use the localAddress option to override.
> 
> A new instance will be added to the InnoDB cluster. Depending on the amount of
> data on the cluster this might take from a few seconds to several hours.
> 
> Adding instance to the cluster...
> 
> NOTE: User 'mysql_innodb_cluster_3221311444'@'%' already existed at instance 'mysql1:3306'. It will be deleted and created again with a new password.
> Monitoring recovery process of the new cluster member. Press ^C to stop monitoring and let it continue in background.
> State recovery already finished for 'mysql2:3306'
> 
> The instance 'mysql2:3306' was successfully added to the cluster.
> 
>  MySQL  localhost+ ssl  JS > cluster.addInstance("inno@mysql3:3306")
> The safest and most convenient way to provision a new instance is through automatic clone provisioning, which will completely overwrite the state of 'mysql3:3306' with a physical snapshot from an existing cluster member. To use this method by default, set the 'recoveryMethod' option to 'clone'.
> 
> The incremental state recovery may be safely used if you are sure all updates ever executed in the cluster were done with GTIDs enabled, there are no purged transactions and the new instance contains the same GTID set as the cluster or a subset of it. To use this method by default, set the 'recoveryMethod' option to 'incremental'.
> 
> Incremental state recovery was selected because it seems to be safely usable.
> 
> Validating instance configuration at mysql3:3306...
> 
> This instance reports its own address as mysql3:3306
> 
> Instance configuration is suitable.
> NOTE: Group Replication will communicate with other members using 'mysql3:33061'. Use the localAddress option to override.
> 
> A new instance will be added to the InnoDB cluster. Depending on the amount of
> data on the cluster this might take from a few seconds to several hours.
> 
> Adding instance to the cluster...
> 
> NOTE: User 'mysql_innodb_cluster_1192341322'@'%' already existed at instance 'mysql1:3306'. It will be deleted and created again with a new password.
> Monitoring recovery process of the new cluster member. Press ^C to stop monitoring and let it continue in background.
> State recovery already finished for 'mysql3:3306'
> 
> The instance 'mysql3:3306' was successfully added to the cluster.
> 
>  MySQL  localhost+ ssl  JS >
> ```

* 再次查看集群状态(正常)

```shell
cluster.status()
```

> 结果：
>
> ```bash
>  MySQL  localhost+ ssl  JS > cluster.status()
> {
>     "clusterName": "mycluster",
>     "defaultReplicaSet": {
>         "name": "default",
>         "primary": "mysql1:3306",
>         "ssl": "REQUIRED",
>         "status": "OK",
>         "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
>         "topology": {
>             "mysql1:3306": {
>                 "address": "mysql1:3306",
>                 "memberRole": "PRIMARY",
>                 "mode": "R/W",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             },
>             "mysql2:3306": {
>                 "address": "mysql2:3306",
>                 "memberRole": "SECONDARY",
>                 "mode": "R/O",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             },
>             "mysql3:3306": {
>                 "address": "mysql3:3306",
>                 "memberRole": "SECONDARY",
>                 "mode": "R/O",
>                 "readReplicas": {},
>                 "replicationLag": null,
>                 "role": "HA",
>                 "status": "ONLINE",
>                 "version": "8.0.25"
>             }
>         },
>         "topologyMode": "Single-Primary"
>     },
>     "groupInformationSourceMember": "mysql1:3306"
> }
>  MySQL  localhost+ ssl  JS > 
> ```

* 恢复集群命令集记录

```shell
docker exec -it mysql1 mysqlsh -uinno -pinno -S/var/run/mysqld/mysqlx.sock
var cluster = dba.getCluster("mycluster")
dba.rebootClusterFromCompleteOutage("mycluster")
var cluster = dba.getCluster("mycluster")
cluster.status()

# 选用
cluster.rejoinInstance('inno@mysql2:3306')
cluster.rejoinInstance('inno@mysql3:3306')

# 选用
cluster.removeInstance('inno@mysql2:3306')
cluster.addInstance("inno@mysql2:3306")
cluster.removeInstance('inno@mysql3:3306')
cluster.addInstance("inno@mysql3:3306") 
```

