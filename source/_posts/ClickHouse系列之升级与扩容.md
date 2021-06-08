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

​	ClickHouse的集群配置比较灵活，用户既可以组成一个单一集群，也可以按照业务划分为多个小集群（表级别），它们的节点、副本和分片数量可各不相同。

​	我们当前主要将集群划分为两个，其中一个是日志集群（写多读少），存储各类原始日志，另一个是业务集群（读多写少），存储数仓体系表。

#### 规模

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/ck%E9%9B%86%E7%BE%A4%E8%A7%84%E5%88%92.png)

#### 配置

​	实例安装成功后，几个重要目录分布如下：

1. /etc/clickhouse-server  服务端的配置文件目录，包括config.xml和users.xml等

    a. config.xml 核心配置

    b. users.xml 用户权限相关配置

    c. metrika.xml 集群变量别名配置

    d. profiles.xml 可通过指定profile配置服务限额参数

    e. quotas.xml 可配置用户角色限额参数

1. /etc/clickhouse-client  客户端的配置文件目录

1. /var/lib/clickhouse  默认数据存储目录，调整至/ssd/clickhouse/data

1. /var/log/clickhouse-server  默认日志目录，调整至/home/data/logs/clickhouse-server

1. /etc/init.d/clickhouse-server  启动shell脚本，主要方便启动服务的，以统一通过systemd管理

1. /etc/security/limits.d/clickhouse.conf  配置最大文件打开数

1. /etc/cron.d/clickhouse-server 定时任务配置，默认没有开启任务，但如果文件不存在启动会异常

1. /usr/bin/clickhouse-*  编译好的可执行文件，常用的如server/client/benchmark/copier等

## 升级

​	大家都知晓近年来CK社区较活跃，若要修复以往Bug或想尽早体验到新功能的，可以决策升级操作。目前社区主要有stable和lts版本，此处我们选用lts版本。

#### 版本

> 目标版本 21.3.6.55-lts，系统 CentOS Linux release 7.9.2009 (Core) 

#### 操作

##### 备份

1. 配置

```shell
cp -r /etc/clickhouse-server /backup/clickhouse/clickhouse-server 
cp -r /etc/clickhouse-client /backup/clickhouse/clickhouse-client 
cp -r /etc/metrika.xml /backup/config/
```

2. 元数据

```shell
cp -r /ssd/clickhouse/metadata /backup/clickhouse/metadata 
cp -r /ssd/clickhouse/dictionaries /backup/clickhouse/dictionaries
```

##### 镜像

> 更改官方源为国内镜像源，速度较快些

```text
--官方源
https://repo.clickhouse.tech/rpm/stable/x86_64 
--国内源
https://mirrors.tuna.tsinghua.edu.cn/clickhouse/rpm/lts/x86_64
```

##### 环境

> 检查环境

```shell
--检测环境
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"

--from RPM Packages
sudo yum install yum-utils 
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/clickhouse/rpm/lts/x86_64 
```

> 检查配置源
>
> more /etc/yum.repos.d/mirrors.tuna.tsinghua.edu.cn_clickhouse_rpm_lts_x86_64

```shell
[mirrors.tuna.tsinghua.edu.cn_clickhouse_rpm_lts_x86_64] 
name=added from: https://mirrors.tuna.tsinghua.edu.cn/clickhouse/rpm/lts/x86_64 
baseurl=https://mirrors.tuna.tsinghua.edu.cn/clickhouse/rpm/lts/x86_64 
enabled=1 
```

##### 执行

> 升级命令如下

```shell
sudo yum upgrade -y clickhouse-server clickhouse-client
cp /deploy/new_config.xml /etc/clickhouse-server/config.xml
systemctl restart clickhouse-server
```

##### 回滚

> 若升级过程中出现异常，需要回滚可执行操作如下

```shell
sudo yum downgrade -y clickhouse-server-common-xxx.x86_64 clickhouse-server-xxx-2.noarch clickhouse-common-static-xxx.x86_64 clickhouse-client-xxx.noarch
```

##### 备注

* 莫忘记 更新新版本不兼容的配置参数，如profiles.xml、quotas.xml等

* 若是从比较旧的版本更新至新版本，如像咱们从19.3直接到21.3，需要整个集群节点停服后统一启动
* dictionaries_lib 字段目录 莫忘记同步元数据
* 新版本无需额外的clickhouse_exporter，默认已封装 
* 新版本若想指定用户grant等权限操作，需启用配置access_management

## 扩容

#### 背景

​	随着业务的增长，磁盘存储不足的问题将暴露，虽然可以在单节点上增加磁盘来缓解，但集群节点扩充是迟早要面临操作的。

 	CK扩容不同于Hdfs集群那样增减节点时可以通过balance命令去自动调节，CK集群不能自动感知集群拓扑变化，也不能自动balance数据。当集群数据量较大，复制表和分布式表过多时，想做到表维度或集群间的数据平衡运维成本是不低的。

#### 方案

> 这里主要提供以下三种方案供大家参考

##### 拷贝迁移（不推荐）

> 若数据量小的情况还能接受，若表多量大的情况非常不建议，成本高且性能影响

##### 配置权重（推荐）

> 我们可以通过配置权重指定大部分的数据写入新的节点，随着时间流逝，原节点具有TTL的部分数据自动删除，数据会趋于均衡，最后调回原始权重。

如调整zookeeper里cluster相关的配置即可，为什么是zookeeper？后续会介绍通过zookeeper配置的好处。

```xml
<shard>
	<weight>1</weight> 
  <internal_replication>true</internal_replication>
  <replica></replica> 
  <replica></replica>
</shard>
<shard>
	<weight>2</weight> 
  <internal_replication>true</internal_replication>
  <replica></replica> 
  <replica></replica>
</shard>
```

备注：weight值必须大于0，否则小于0将失败，等于0将失效。

##### 直接扩容不迁移

> 横向扩容，配置简单不需要额外操作，但达到数据均衡所需的时间较长，查询没有完全发挥分布式的优势

#### Zookeeper配置

##### 目的

​	便于调优策略修改配置而无需重启CK服务，故将部分核心CK配置抽取保存在Zookeeper中。

##### 配置

1. 集群配置，抽取为cluster.xml

    ```xml
    <shard>
      <replica> 
        <host></host> 
        <port></port> 
        <user></user> 
        <password></password> 
      </replica> 
    </shard> 
    <shard>
      ...
    </shard>
    ```

1. 用户权限配置，抽取为users.xml

    ```xml
    <clickhouse_cluster>
      <password_sha256_hex></password_sha256_hex>
      <networks>
        <ip></ip>
      </networks>
      <profile>default</profile> 
      <quota>default</quota> 
    </clickhouse_cluster>
    <default>
      ...
    </default>
    ```

1. 用户角色限额配置，抽取为quotas.xml

    ```xml
    <default> 
      <interval>
        <duration></duration> 
        <queries></queries> 
        <errors></errors>
        ...
      </interval>
    </default>
    ```

1. 服务限额配置，抽取为profiles.xml

    ```xml
    <default> 
      <log_queries>1</log_queries>
      <log_queries_cut_to_length></log_queries_cut_to_length>
      <max_memory_usage></max_memory_usage>
      <load_balancing></load_balancing>
      ...
    </default>
    ```

#### 场景

##### 新增副本

> 新增节点作为新副本加入集群，生成新的配置如下，拷贝至对应zookeeper path，即可动态生效。

```xml
<shard>
  <weight>1</weight>
  <internal_replication>true</internal_replication>
  <replica>
    <host>xxx</host>
    <port></port>
    <user></user>
    <password></password>
  </replica>
  <replica>
    <host>xxx.new.add</host>
    <port></port>
    <user></user>
    <password></password>
  </replica>
</shard>
```

##### 新增分片

> 新增节点作为分片加入集群，生成新的配置如下，拷贝至对应zookeeper path，即可动态生效。

```xml
<shard>
  <weight>1</weight>
  <replica>
    <host>xxx</host>
    <port></port>
    <user></user>
    <password></password>
  </replica>
</shard>
<shard>
  <weight>1</weight>
  <replica>
    <host>xxx.new.add</host>
    <port></port>
    <user></user>
    <password></password>
  </replica>
</shard>
```

#### 步骤

1. 新增节点上安装相应版本的clickhouse-server/clickhouse-client

1. 同步集群对应的config.xml、dictionaries_lib等相关配置

1. 修改节点上macros的参数，主要确保建表ZK唯一路径

    ```xml
    <macros>
       <cluster></cluster>
       <host></host>
       <instance></instance>
       <shard></shard>
       <layer></layer>
       <replica></replica>
    </macros>
    ```

1. 选择扩容集群的某个实例，同步metadata元数据，这里我们只需同步database的sql文件即可。

    > **备注：此处要注意CK自身有个Bug，若我们的表是分布式表，直接将表结构sql文件拷贝过去，虽然CK中表已经创建成功，但是ZK上表的唯一地址并不会自动创建，故与ZK交互的表将变成Read Only模式，写入数据会失败**

1. 同步完database的sql文件后，可直接启动服务。

    ```shell
    systemctl start clickhouse-server
    ```

1. 接下来我们手动的创建相关表（针对是扩容分片还是副本，可分场景选择要同步的集群节点）

    a. 选择要同步集群的一个节点的表元数据信息，导出SQL文件

    ```shell
    clickhouse-client -h xxx --port 9000 -u xxx --ask-password --query "SELECT create_table_query FROM system.tables where database not in ('system')" --format TabSeparatedRaw > result.sql
    ```

    b. 通过结果文件创建相关表

    ```shell
    clickhouse-client -h xxx --port 9000 -u xxx --ask-password --multiquery < result.sql
    ```

1. 确实节点日志无误时，可调整ZK cluster.xml配置，动态即时生效

1. 不定期调整weight权重值

#### 备注

* 扩容后cluster.xml的调整切记分场景，是副本还是分片

* 注意观察复制表和本地表在扩容后数据增长的区别

    

---

待续...
