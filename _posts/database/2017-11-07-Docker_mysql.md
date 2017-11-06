---
layout: post
title: "Docker 构建 MySQL 主从"
date: 2017-11-07 01:13:00 +0800 
categories: Docker 
tag: [Docker, MySQL]
---
* content
{:toc}

### 前言

做了挺长时间的 OLAP 分布式数据库，没有深入研究过 MySQL，近期对 MySQL 主从复制原理研究了一下，并结合 Docker 完成了 MySQL 主从集群的构建。

首先，从几个概念进行理解：

**主从复制**，主库一般是准实时业务数据库，从数据库实际上用数据库复制技术来构建一个和主数据库完全一样的数据库环境。

**主从复制的优点**，个人理解，MySQL 主从复制作用有三：

1. 数据热备，主库服务器宕机后，可以切换到从库继续工作，避免数据丢失。
2. 读写分离，主数据保障业务写入，从数据库分散数据读压力，使数据库能够支撑较高的并发访问。
3. 横向扩展，伴随业务量的逐渐增大，单机瓶颈越来月明显，此时可构建多个从库，降低磁盘 IO 访问频度。

<!-- more -->

**主从复制的原理**，总体上来说，MySQL 有个 bin-log 二进制文件，记录所有语句操作记录，我们目标就是将主库的 bin-log 文件进行复制重做。在 MySQL 里面，参与主从复制的主要有三个线程，bin-log 输出线程，作用就是当从库配置连接到主库时，主库都会创建一个线程 bin-log dump thread 发送 bin-log 内容；从库 IO 线程，从库执行 start slave 语句后会创建一个 IO 线程，该线程连接到主库并请求主库发送 bin-log 里面的更新记录到从库上，从库 IO 线程读取主库的 bin-log 输出线程，并将相关内容更新到本地文件（其中包含 relay log）; 从库的 SQL 线程，读取从库 IO 线程写到 relay log	 中的内容并执行。

### 主从复制构建

1.从 Docker 官方 pull 一份 MySQL Docker 镜像到本地。

```
[root@dbtest ~]# docker pull mysql:latest

[root@dbtest ~]# docker images

REPOSITORY                  TAG                 IMAGE ID            CREATED                  SIZE

docker.io/mysql             latest              5709795eeffa        Less than a second ago   408.2 MB

```

2.本地定义 MySQL 主从数据库配置文件：

`mysql-master-my.cnf` 配置如下：

```
[mysqld]
server_id=1
## 二进制日志功能启用
log-bin=mysql-bin  
binlog_cache_size=2M  
binlog_format=mixed  
expire_logs_days=3 
slave_skip_errors=1062

```

`mysql-slave-my.cnf` 配置如下：

```
[mysqld]
server_id=2 
binlog-ignore-db=mysql  
log-bin=mysql-slave1-bin  
binlog_cache_size=2M  
binlog_format=mixed  
expire_logs_days=3
slave_skip_errors=1062  
relay_log=mysql-relay-bin  
log_slave_updates=1  
read_only=1
```

3.用 Docker 启用 master 和 slave 两个容器

```
docker run --name mysql_master -p 3306:3306 -v /root/temp/mysql-master-my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=111111 -d mysql:latest

docker run --name mysql_slave -p 3306:3306 -v /root/temp/mysql-slave-my.cnf:/etc/mysql/my.cnf -e MYSQL_ROOT_PASSWORD=111111 -d mysql:latest

```

查看两个容器运行的服务进程： `docker top mysql_master `

4.连接 master 和 slave

登陆 mysql_master 容器

> docker exec -it mysql_master /bin/bash

查看 MySQL 服务运行状态

> service mysql status

登陆 MySQL master 查看 bin-log 信息

> mysql -uroot -p111111 -e "show master status"

> 记录下 File mysql-bin.000001 和 Position 字段的值 767

登陆 MySQL slave 执行 SQL 如下：

> hange master to master_host='192.168.56.12', master_user='slave', master_password='111111', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos=767, master_connect_retry=30;

查看 slave 数据主从同步的状态

> show slave status \G;
> 重点关注 SlaveIORunning 和 SlaveSQLRunning

执行如下命令，开启主从同步

> start slave;

5.测试主从同步是否成功

登陆 master 库，进行库、表的增删改操作即可观察到数据和结构的同步过程。如果没有成功，结合数据库日志进行具体分析。
