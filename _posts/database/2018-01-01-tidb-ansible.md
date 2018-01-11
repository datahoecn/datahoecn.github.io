---
layout: post
title: "TiDB - tidb-ansible "
date: 2018-01-01 21:22:10 +0800 
categories: tidb 
tag: [tidb, ansible]
---
* content
{:toc}

在剖析 tidb-ansible 框架之前，我们需要先了解下 Ansible，Ansible 作为近阶段优秀的自动化运维工具，实现了批量系统配置管理、批量程序部署、批量运行命令等功能。本身不具备批量部署的能力，真正具有批量部署的是 Ansible 所运行的模块，Ansible 只是提供一种框架，再了解 tidb 部署原因之前需要简单了解下整个框架的体系结构和工作原理。

<!-- more -->

![](https://github.com/datahoecn/datahoecn.github.io/raw/master/effects/images/ansible.png)


(1) 连接插件 connection plugins：负责和被监控端实现通信；

(2) host inventory：指定操作的主机，是一个配置文件里面定义监控的主机；

(3) 各种模块核心模块、command 模块、自定义模块；

(4) 借助于插件完成记录日志邮件等功能；

(5) playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务。

## tidb-ansible 体系结构

### 基本组成

TiDB-Ansible 是 PingCAP 基于 Ansible playbook 功能编写的集群部署工具，小伙伴能在这么短的时间内构建整个工具框架，能够快速部署一个完整的 TiDB 集群，并且支撑整个集群平滑的升级和扩容，让我们这些刚接触的人的确耗费一番功夫。其实理解了上面整个框架的基本组成后，整个部署和排错过程都不复杂。

整个工程可重点关注两个部分，下面我们分别说明下这两个部分。

**invertory**

invertory 作为部署入口配置文件，需要填写集群配置信息，集群版本，集群部署信息，版本管控，系统变量等，具体配置可参考官网 [https://github.com/pingcap/docs-cn/edit/master/op-guide/ansible-deployment.md](https://github.com/pingcap/docs-cn/edit/master/op-guide/ansible-deployment.md)。


### Playbook 详解
