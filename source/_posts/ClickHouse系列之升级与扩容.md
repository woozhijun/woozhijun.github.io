title: ClickHouse系列之升级与扩容
tags:

  - 大数据
  - 数据库
  - OLAP
  - clickhouse

category: 大数据
comments: true

------

## 集群

#### 描述

​		ClickHouse的集群配置比较灵活，用户既可以组成一个单一集群，也可以按照业务划分为多个小集群（表级别），它们的节点、副本和分片数量可各不相同。

​		我们当前主要将集群划分为两个，其中一个是日志集群（写多读少），存储各类原始日志，另一个是业务集群（读多写少），存储数仓体系表。

#### 规模

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/ck%E9%9B%86%E7%BE%A4%E8%A7%84%E5%88%92.png)

#### 配置

​		实例安装成功后，几个重要目录分布如下：

1. /etc/clickhouse-server  服务端的配置文件目录，包括config.xml和users.xml等
2. /etc/clickhouse-client  客户端的配置文件目录
3. /var/lib/clickhouse  默认数据存储目录，调整至/ssd/clickhouse/data
4. /var/log/clickhouse-server  默认日志目录，调整至/home/data/logs/clickhouse-server
5. /etc/init.d/clickhouse-server  启动shell脚本，主要方便启动服务的，以统一通过systemd管理
6. /etc/security/limits.d/clickhouse.conf  配置最大文件打开数
7. /etc/cron.d/clickhouse-server 定时任务配置，默认没有开启任务，但如果文件不存在启动会异常
8. /usr/bin/clickhouse-*  编译好的可执行文件，常用的如server/client/benchmark/copier等

## 升级

#### 版本

#### 步骤

## 扩容

### 新增分片

### 新增副本

