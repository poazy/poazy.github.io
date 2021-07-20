---
layout: post
title:  "Docker 安装 MySQL NDB Cluster(8.0.25)"
date:   2021-07-13 23:30:00
categories: MySQL
tags: MySQL
author: poazy
---

* content
{:toc}
> Docker 下安装 MySQL NDB Cluster;
>
> 版本为：8.0.25，采用伪集群方式（在一台机器上部署）。






# 前置准备

* 创建 Docker 网络

```bash
# 创建一个 Docker 网络给后面安装 MySQL NDB Cluster 使用
docker network create ndbnet
```

* 创建 /boazy/data/dockerdata/mnc 目录，并进入此目录中

```bash
mkdir -p /boazy/data/dockerdata/mnc && cd /boazy/data/dockerdata/mnc
```

* 下载 my.cnf 文件

```bash
wget https://raw.githubusercontent.com/mysql/mysql-docker/mysql-cluster/8.0/cnf/my.cnf
```

> 内容：
>
> ```bash
> # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
> #
> # This program is free software; you can redistribute it and/or modify
> # it under the terms of the GNU General Public License as published by
> # the Free Software Foundation; version 2 of the License.
> #
> # This program is distributed in the hope that it will be useful,
> # but WITHOUT ANY WARRANTY; without even the implied warranty of
> # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> # GNU General Public License for more details.
> #
> # You should have received a copy of the GNU General Public License
> # along with this program; if not, write to the Free Software
> # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
> 
> [mysqld]
> ndbcluster
> ndb-connectstring=192.168.0.2
> user=mysql
> 
> [mysql_cluster]
> ndb-connectstring=192.168.0.2
> ```

* 下载 mysql-cluster.cnf 文件

```bash
wet https://raw.githubusercontent.com/mysql/mysql-docker/mysql-cluster/8.0/cnf/mysql-cluster.cnf
```

> 内容：
>
> ```bash
> # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
> #
> # This program is free software; you can redistribute it and/or modify
> # it under the terms of the GNU General Public License as published by
> # the Free Software Foundation; version 2 of the License.
> #
> # This program is distributed in the hope that it will be useful,
> # but WITHOUT ANY WARRANTY; without even the implied warranty of
> # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> # GNU General Public License for more details.
> #
> # You should have received a copy of the GNU General Public License
> # along with this program; if not, write to the Free Software
> # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
> 
> [ndbd default]
> NoOfReplicas=2
> DataMemory=80M
> IndexMemory=18M
> 
> 
> [ndb_mgmd]
> NodeId=1
> hostname=192.168.0.2
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=2
> hostname=192.168.0.3
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=3
> hostname=192.168.0.4
> datadir=/var/lib/mysql
> 
> [mysqld]
> NodeId=4
> hostname=192.168.0.10
> ```

* 查看下载的文件

```bash
[root@centos7-qscft mnc]# pwd && ll -h
/boazy/data/dockerdata/mnc
total 8.0K
-rw-r--r--. 1 root root  836 Jul 13 13:44 my.cnf
-rw-r--r--. 1 root root 1018 Jul 13 13:45 mysql-cluster.cnf
[root@centos7-qscft mnc]#
```

* 拉取 mysql-cluster Docker 镜像

```bash
docker pull mysql/mysql-cluster:8.0.25
```



# 运行 NDB 实例

## 调整配置文件

* 修改 my.cnf 文件内容

```bash
# 修改 ndb-connectstring 属性值为 ndb_mgmd1
sed -i 's/ndb-connectstring=192.168.0.2/ndb-connectstring=ndb_mgmd1/g' /boazy/data/dockerdata/mnc/my.cnf
```

> 查看修改结果：
>
> ```bash
> [root@centos7-qscft mnc]# pwd && cat my.cnf
> /boazy/data/dockerdata/mnc
> # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
> #
> # This program is free software; you can redistribute it and/or modify
> # it under the terms of the GNU General Public License as published by
> # the Free Software Foundation; version 2 of the License.
> #
> # This program is distributed in the hope that it will be useful,
> # but WITHOUT ANY WARRANTY; without even the implied warranty of
> # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> # GNU General Public License for more details.
> #
> # You should have received a copy of the GNU General Public License
> # along with this program; if not, write to the Free Software
> # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
> 
> [mysqld]
> ndbcluster
> ndb-connectstring=ndb_mgmd1
> user=mysql
> 
> [mysql_cluster]
> ndb-connectstring=ndb_mgmd1
> [root@centos7-qscft mnc]#
> ```

* 修改 mysql-cluster.cnf 文件内容

> 执行以下命名替换修改后，再执行 vi /boazy/data/dockerdata/mnc/mysql-cluster.cnf 添加一些节点

```bash
sed -i 's/NoOfReplicas=2/NoOfReplicas=3/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf 
# 修改 ndb_mgmd 的 hostname 属性值为 ndb_mgmd1
sed -i 's/hostname=192.168.0.2/hostname=ndb_mgmd1/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf 
# 修改 ndbd 的 hostname 属性值为对应值
sed -i 's/hostname=192.168.0.3/hostname=ndbd2/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf
sed -i 's/hostname=192.168.0.4/hostname=ndbd3/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf
# 修改 mysqld 的 hostname 属性值为 mysqld51
sed -i 's/NodeId=4/NodeId=9/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf
sed -i 's/hostname=192.168.0.10/hostname=mysqld9/g' /boazy/data/dockerdata/mnc/mysql-cluster.cnf
```

> 查看修改结果：
>
> ```bash
> [root@centos7-qscft mnc]# pwd && cat mysql-cluster.cnf
> /boazy/data/dockerdata/mnc
> # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
> #
> # This program is free software; you can redistribute it and/or modify
> # it under the terms of the GNU General Public License as published by
> # the Free Software Foundation; version 2 of the License.
> #
> # This program is distributed in the hope that it will be useful,
> # but WITHOUT ANY WARRANTY; without even the implied warranty of
> # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
> # GNU General Public License for more details.
> #
> # You should have received a copy of the GNU General Public License
> # along with this program; if not, write to the Free Software
> # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
> 
> [ndbd default]
> NoOfReplicas=2
> DataMemory=80M
> IndexMemory=18M
> 
> 
> [ndb_mgmd]
> NodeId=1
> hostname=ndb_mgmd1
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=2
> hostname=ndbd2
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=3
> hostname=ndbd3
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=4
> hostname=ndbd4
> datadir=/var/lib/mysql
> 
> [ndbd]
> NodeId=5
> hostname=ndbd5
> datadir=/var/lib/mysql
> 
> [mysqld]
> NodeId=9
> hostname=mysqld9
> 
> [mysqld]
> NodeId=8
> hostname=mysqld8
> [root@centos7-qscft mnc]#
> ```



## 运行集群实例

* 安装运行 1 个 MGM 节点实例（管理实例）

```bash
docker run -d --restart=always --name=ndb_mgmd1 --hostname=ndb_mgmd1 --net=ndbnet \
  -p 1186:1186 -p 2212:2212 -p:13316:3306 -p 33061:33060 \
  -v /boazy/data/dockerdata/mnc/ndb_mgmd1/mgm_data:/var/lib/mysql \
  -v /boazy/data/dockerdata/mnc/mysql-cluster.cnf:/etc/mysql-cluster.cnf \
  -v /boazy/data/dockerdata/mnc/my.cnf:/etc/my.cnf \
  -v /etc/localtime:/etc/localtime:ro \
  -e "TZ=Asia/Shanghai" \
  mysql/mysql-cluster:8.0.25 ndb_mgmd --ndb-nodeid=1
```

* 安装运行 4 个 NDB 节点实例（数据实例）

> \>=2

```bash
for N in 2 3 4 5
do docker run -d --restart=always --name=ndbd$N --hostname=ndbd$N --net=ndbnet \
  -p 1${N}86:1186 -p 22${N}2:2202 -p:133${N}6:3306 -p 3306${N}:33060 \
  -v /boazy/data/dockerdata/mnc/ndbd$N/ndbd_data:/var/lib/mysql \
  -v /boazy/data/dockerdata/mnc/my.cnf:/etc/my.cnf \
  -v /etc/localtime:/etc/localtime:ro \
  -e "TZ=Asia/Shanghai" \
  mysql/mysql-cluster:8.0.25 ndbd --ndb-nodeid=$N
done
```

* 安装运行 2 个 SQL 节点实例（SQL 实例）

> \>=1，尽可能多

```bash
for N in 9 8
do docker run -d --restart=always --name=mysqld$N --hostname=mysqld$N --net=ndbnet \
  -p 1${N}86:1186 -p 22${N}2:2202 -p:133${N}6:3306 -p 3306${N}:33060 \
  -v /boazy/data/dockerdata/mnc/mysqld${N}/mysql_data:/var/lib/mysql \
  -v /boazy/data/dockerdata/mnc/my.cnf:/etc/my.cnf \
  -v /etc/localtime:/etc/localtime:ro \
  -e "TZ=Asia/Shanghai" \
  -e MYSQL_ROOT_PASSWORD=root@123 \
  mysql/mysql-cluster:8.0.25 mysqld --ndb-nodeid=$N
done
```

* 查询状态

```bash
docker ps -a
```

> 结果：
>
> ```bash
> [root@centos7-qscft mnc]# docker ps -a
> CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS                      PORTS                                                                                                         NAMES
> 38f85dfbe6bc        mysql/mysql-cluster:8.0.25   "/entrypoint.sh mysq…"   29 minutes ago      Up 26 minutes (healthy)     0.0.0.0:1886->1186/tcp, 0.0.0.0:2282->2202/tcp, 0.0.0.0:13386->3306/tcp, 0.0.0.0:33068->33060/tcp             mysqld8
> 4e70bff8c0c1        mysql/mysql-cluster:8.0.25   "/entrypoint.sh mysq…"   29 minutes ago      Up 26 minutes (healthy)     0.0.0.0:1986->1186/tcp, 0.0.0.0:2292->2202/tcp, 0.0.0.0:13396->3306/tcp, 0.0.0.0:33069->33060/tcp             mysqld9
> 48bbec36360e        mysql/mysql-cluster:8.0.25   "/entrypoint.sh ndbd…"   30 minutes ago      Up 26 minutes (unhealthy)   0.0.0.0:1586->1186/tcp, 0.0.0.0:2252->2202/tcp, 0.0.0.0:13356->3306/tcp, 0.0.0.0:33065->33060/tcp             ndbd5
> 59df067f1b0a        mysql/mysql-cluster:8.0.25   "/entrypoint.sh ndbd…"   30 minutes ago      Up 26 minutes (unhealthy)   0.0.0.0:1486->1186/tcp, 0.0.0.0:2242->2202/tcp, 0.0.0.0:13346->3306/tcp, 0.0.0.0:33064->33060/tcp             ndbd4
> c96e7a5819aa        mysql/mysql-cluster:8.0.25   "/entrypoint.sh ndbd…"   30 minutes ago      Up 26 minutes (unhealthy)   0.0.0.0:1386->1186/tcp, 0.0.0.0:2232->2202/tcp, 0.0.0.0:13336->3306/tcp, 0.0.0.0:33063->33060/tcp             ndbd3
> 50d197b19717        mysql/mysql-cluster:8.0.25   "/entrypoint.sh ndbd…"   30 minutes ago      Up 26 minutes (unhealthy)   0.0.0.0:1286->1186/tcp, 0.0.0.0:2222->2202/tcp, 0.0.0.0:13326->3306/tcp, 0.0.0.0:33062->33060/tcp             ndbd2
> 07f8600a4f44        mysql/mysql-cluster:8.0.25   "/entrypoint.sh ndb_…"   30 minutes ago      Up 27 minutes (unhealthy)   0.0.0.0:1186->1186/tcp, 0.0.0.0:2212->2212/tcp, 2202/tcp, 0.0.0.0:13316->3306/tcp, 0.0.0.0:33061->33060/tcp   ndb_mgmd1
> [root@centos7-qscft mnc]#
> ```



## 创建用户并授权

* 创建用户并授权

```bash
docker exec -it mysqld8 mysql -uroot -proot@123 \
  -e "CREATE USER 'ndb'@'%' IDENTIFIED BY 'ndb';" \
  -e "GRANT ALL privileges ON *.* TO 'ndb'@'%' with grant option;" \
  -e "reset master;"
```

> 结果：
>
> ```bash
> [root@centos7-qscft mnc]# docker exec -it mysqld8 mysql -uroot -proot@123 \
> >   -e "CREATE USER 'ndb'@'%' IDENTIFIED BY 'ndb';" \
> >   -e "GRANT ALL privileges ON *.* TO 'ndb'@'%' with grant option;" \
> >   -e "reset master;"
> mysql: [Warning] Using a password on the command line interface can be insecure.
> [root@centos7-qscft mnc]#
> ```

* 查询创建用户结果

```bash
docker exec -it mysqld8 mysql -undb -pndb \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SELECT user FROM mysql.user where user = 'ndb';"
```

> (在通过 mysqld8 创建的用户，在 mysqld8 中也能查看得到)
>
> 结果：
>
> ```bash
> [root@centos7-qscft mnc]# docker exec -it mysqld8 mysql -undb -pndb \
> >   -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
> >   -e "SELECT user FROM mysql.user where user = 'ndb';"
> mysql: [Warning] Using a password on the command line interface can be insecure.
> +---------------+---------+
> | Variable_name | Value   |
> +---------------+---------+
> | hostname      | mysqld8 |
> +---------------+---------+
> +------+
> | user |
> +------+
> | ndb  |
> +------+
> [root@centos7-qscft mnc]# docker exec -it mysqld9 mysql -undb -pndb \
> >   -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
> >   -e "SELECT user FROM mysql.user where user = 'ndb';"
> mysql: [Warning] Using a password on the command line interface can be insecure.
> +---------------+---------+
> | Variable_name | Value   |
> +---------------+---------+
> | hostname      | mysqld9 |
> +---------------+---------+
> +------+
> | user |
> +------+
> | ndb  |
> +------+
> [root@centos7-qscft mnc]#
> ```



## 查看集群状态

* 查看集群状态

```bash
docker exec -it ndb_mgmd1 ndb_mgm -e show
```

> 结果：
>
> ```bash
> [root@centos7-qscft mnc]# docker exec -it ndb_mgmd1 ndb_mgm -e show
> Connected to Management Server at: ndb_mgmd1:1186
> Cluster Configuration
> ---------------------
> [ndbd(NDB)]     4 node(s)
> id=2    @172.23.0.2  (mysql-8.0.25 ndb-8.0.25, Nodegroup: 0)
> id=3    @172.23.0.4  (mysql-8.0.25 ndb-8.0.25, Nodegroup: 0, *)
> id=4    @172.23.0.3  (mysql-8.0.25 ndb-8.0.25, Nodegroup: 1)
> id=5    @172.23.0.5  (mysql-8.0.25 ndb-8.0.25, Nodegroup: 1)
> 
> [ndb_mgmd(MGM)] 1 node(s)
> id=1    @172.23.0.8  (mysql-8.0.25 ndb-8.0.25)
> 
> [mysqld(API)]   2 node(s)
> id=8    @172.23.0.7  (mysql-8.0.25 ndb-8.0.25)
> id=9    @172.23.0.6  (mysql-8.0.25 ndb-8.0.25)
> 
> [root@centos7-qscft mnc]#
> ```



# SQL 节点高可用(HAProxy)

* 安装 HAProxy

```bash
yum -y install haproxy
```

* 备份 haproxy.cfg

```bash
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak0
```

* 修改 haproxy.cfg 配置，添加 mysqld 的代理

```bash
vi /etc/haproxy/haproxy.cfg
```

> 修改后的内容如下：
>
> ```bash
> #---------------------------------------------------------------------
> # Example configuration for a possible web application.  See the
> # full configuration options online.
> #
> #   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
> #
> #---------------------------------------------------------------------
> 
> #---------------------------------------------------------------------
> # Global settings
> #---------------------------------------------------------------------
> global
>     # to have these messages end up in /var/log/haproxy.log you will
>     # need to:
>     #
>     # 1) configure syslog to accept network log events.  This is done
>     #    by adding the '-r' option to the SYSLOGD_OPTIONS in
>     #    /etc/sysconfig/syslog
>     #
>     # 2) configure local2 events to go to the /var/log/haproxy.log
>     #   file. A line like the following can be added to
>     #   /etc/sysconfig/syslog
>     #
>     #    local2.*                       /var/log/haproxy.log
>     #
>     log         127.0.0.1 local2
> 
>     chroot      /var/lib/haproxy
>     pidfile     /var/run/haproxy.pid
>     maxconn     4000
>     user        haproxy
>     group       haproxy
>     daemon
> 
>     # turn on stats unix socket
>     stats socket /var/lib/haproxy/stats
> 
> #---------------------------------------------------------------------
> # common defaults that all the 'listen' and 'backend' sections will
> # use if not designated in their block
> #---------------------------------------------------------------------
> defaults
>     mode                    http
>     log                     global
>     option                  httplog
>     option                  dontlognull
>     option http-server-close
>     option forwardfor       except 127.0.0.0/8
>     option                  redispatch
>     retries                 3
>     timeout http-request    10s
>     timeout queue           1m
>     timeout connect         10s
>     timeout client          1m
>     timeout server          1m
>     timeout http-keep-alive 10s
>     timeout check           10s
>     maxconn                 3000
> 
> #---------------------------------------------------------------------
> # main frontend which proxys to the backends
> #---------------------------------------------------------------------
> 
> #---------------------------------------------------------------------
> # static backend for serving up images, stylesheets and such
> #---------------------------------------------------------------------
> 
> #---------------------------------------------------------------------
> # round robin balancing between the various backends
> #---------------------------------------------------------------------
> 
> listen mysql-cluster
>     bind 0.0.0.0:3366
>     option tcplog
>     mode tcp
>     balance roundrobin
>     server mysql1 192.168.9.241:13386 check port 13386 inter 1s rise 2 fall 2
>     server mysql2 192.168.9.241:13396 check port 13396 inter 1s rise 2 fall 2
> 
> listen http_front
>     bind 0.0.0.0:8888
>     mode http
>     option httplog
>     stats uri /haproxy
>     stats auth admin:123456
>     stats refresh 5s
>     stats enable
> 
> ```

* 查询 haproxy.cfg 是否正确

```bash
haproxy -f /etc/haproxy/haproxy.cfg -c
```

> 结果：
>
> ```bash
> [root@centos7-qscft ~]# haproxy -f /etc/haproxy/haproxy.cfg -c
> [WARNING] 196/144957 (589711) : config : 'option forwardfor' ignored for proxy 'mysql-cluster' as it requires HTTP mode.
> Configuration file is valid
> [root@centos7-qscft ~]#
> ```

* 重新启动 HAProxy 并查看状态

```bash
systemctl restart haproxy && systemctl status haproxy
```

> 结果（为错误）：
>
> ```bash
> [root@centos7-qscft haproxy]# systemctl restart haproxy && systemctl status haproxy
> ● haproxy.service - HAProxy Load Balancer
>    Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
>    Active: failed (Result: exit-code) since Fri 2021-07-13 20:58:43 CST; 2s ago
>   Process: 521390 ExecStart=/usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid $OPTIONS (code=exited, status=1/FAILURE)
>  Main PID: 521390 (code=exited, status=1/FAILURE)
> 
> Jul 13 20:58:43 centos7-qscft systemd[1]: Started HAProxy Load Balancer.
> Jul 13 20:58:43 centos7-qscft haproxy-systemd-wrapper[521390]: haproxy-systemd-wrapper: executing /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
> Jul 13 20:58:43 centos7-qscft haproxy-systemd-wrapper[521390]: [WARNING] 196/135843 (521391) : config : 'option forwardfor' ignored for proxy 'mysql-cluster' as it requires HTTP mode.
> Jul 13 20:58:43 centos7-qscft haproxy-systemd-wrapper[521390]: [ALERT] 196/135843 (521391) : Starting proxy mysql-cluster: cannot bind socket [0.0.0.0:3366]
> Jul 13 20:58:43 centos7-qscft haproxy-systemd-wrapper[521390]: [ALERT] 196/135843 (521391) : Starting proxy http_front: cannot bind socket [0.0.0.0:8888]
> Jul 13 20:58:43 centos7-qscft systemd[1]: haproxy.service: main process exited, code=exited, status=1/FAILURE
> Jul 13 20:58:43 centos7-qscft haproxy-systemd-wrapper[521390]: haproxy-systemd-wrapper: exit, haproxy RC=1
> Jul 13 20:58:43 centos7-qscft systemd[1]: Unit haproxy.service entered failed state.
> Jul 13 20:58:43 centos7-qscft systemd[1]: haproxy.service failed.
> Hint: Some lines were ellipsized, use -l to show in full.
> [root@centos7-qscft haproxy]# 
> ```

* 解决以上错误，要先设置 HAProxy 连接策略

```bash
setsebool -P haproxy_connect_any=1
```

> 结果：
>
> ```bash
> [root@centos7-qscft haproxy]# setsebool -P haproxy_connect_any=1
> [root@centos7-qscft haproxy]#
> ```

* 再重新启动 HAProxy 并查看状态

```bash
systemctl restart haproxy && systemctl status haproxy
```

> 结果（正常）：
>
> ```bash
> [root@centos7-qscft haproxy]# systemctl restart haproxy && systemctl status haproxy
> ● haproxy.service - HAProxy Load Balancer
>    Loaded: loaded (/usr/lib/systemd/system/haproxy.service; disabled; vendor preset: disabled)
>    Active: active (running) since Fri 2021-07-13 21:00:10 CST; 1s ago
>  Main PID: 523406 (haproxy-systemd)
>     Tasks: 3
>    Memory: 1.7M
>    CGroup: /system.slice/haproxy.service
>            ├─523406 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
>            ├─523407 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
>            └─523408 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
> 
> Jul 13 21:00:10 centos7-qscft systemd[1]: Started HAProxy Load Balancer.
> Jul 13 21:00:10 centos7-qscft haproxy-systemd-wrapper[523406]: haproxy-systemd-wrapper: executing /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
> Jul 13 21:00:10 centos7-qscft haproxy-systemd-wrapper[523406]: [WARNING] 196/140010 (523407) : config : 'option forwardfor' ignored for proxy 'mysql-cluster' as it requires HTTP mode.
> Hint: Some lines were ellipsized, use -l to show in full.
> [root@centos7-qscft haproxy]#
> ```
>
> ```bash
> [root@centos7-qscft haproxy]# lsof -i:3366
> COMMAND    PID    USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
> haproxy 523408 haproxy    5u  IPv4 16410739      0t0  TCP *:3366 (LISTEN)
> [root@centos7-qscft haproxy]# lsof -i:8888
> COMMAND    PID    USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
> haproxy 523408 haproxy    7u  IPv4 16410741      0t0  TCP *:ddi-tcp-1 (LISTEN)
> [root@centos7-qscft haproxy]#
> ```

* 检查连通性（OK）

```bash
[root@centos7-qscft ~]# telnet 192.168.9.241 3366
Trying 192.168.9.241...
Connected to 192.168.9.241.
Escape character is '^]'.
R
8.0.25-cluster_ P3&▒>!:
                       nv1OYcaching_sha2_passwordxterm-256color^CConnection closed by foreign host.
[root@centos7-qscft ~]#
[root@centos7-qscft ~]# telnet 192.168.9.241 8888
Trying 192.168.9.241...
Connected to 192.168.9.241.
Escape character is '^]'.
^CConnection closed by foreign host.
[root@centos7-qscft ~]#
```



# 遇到的问题

## InnoDB 的表是不会被同步的？

* 这样子创建的表是 InnoDB 存储引擎，表及数据不会被同步的，因为 MySQL NDB Cluster 不支持 InnoDB 存储引擎

```sql
CREATE TABLE `bktb_table_demo` (
  `id` int NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

* 建表指定存储引擎为 ndbcluster，就支持同步啦

```sql
CREATE TABLE `ktb_table_demo` (
  `id` int NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=ndbcluster DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

## 与 InnoDB 的区别？

* 可以查看相关链接

```bash
https://blog.csdn.net/qq_42979842/article/details/107420757
https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-compared.html
https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-engines.html
https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-workloads.html
https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-usage.html
```





