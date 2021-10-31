title: ClickHouse系列之集群副本与分片
tags:

  - 大数据
  - 数据库
  - OLAP
  - ClickHouse

category: 大数据
comments: true

## 概述

​		本文主要为大家介绍ClickHouse三板斧（集群、副本与分片）

#### 集群

​	集群是副本和分片的基础，它将ClickHouse的服务拓扑由当前节点延伸到多个节点，但它并不像Hadoop生态某系统那样，要求所有节点组成一个单一的大集群（服务级别）。ClickHouse的集群配置比较灵活，用户既可以组成一个单一集群，也可以按照业务划分为多个小集群（表级别），它们的节点、副本和分片数量可各不相同。

​	从作用来看，ClickHouse集群的工作可以理解为是针对逻辑层面的，而执行层面的具体工作则交给副本和分片来执行。

#### 区别

​	简言之，副本与分片区分可以理解为，副本间的数据是完全相同，而分片间的数据是不同的，如下：

## 数据副本

#### 描述

​	MergeTree + Replicated即可理解为支持数据副本，两者逻辑关系如下：

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/ReplicateMergeTree%E9%80%BB%E8%BE%91.png)

​	在MergeTree中，一个数据分区由开始创建到全部完成，会历经两类存储区域。

1. 内存：数据首先会被写入内存缓存区。

2. 本地磁盘：数据接着会被写入tmp临时目录分区，待全部完成后再将临时目录重命名为正式分区。

    ReplicatedMergeTree只是在此基础上添加了Zookeeper的部分，这里并不会涉及表数据的传输。

#### 特点

* 依赖Zookeeper，主要借助ZK的分布式协同能力
* 表级别的副本，包括副本的数量、集群内分布位置等
* 多主架构，即在任意副本上执行INSERT和ALTER查询效果是一致的
* Block数据块，这是数据写入的基本单位，具有写入的原子性和唯一性
* 原子性，要么全部成功，要么全部失败
* 唯一性，会按数据顺序、数据大小和数据行等指标Hash，重复则忽略

#### 定义形式

> ENGINE = ReplicatedMergeTree('/clickhouse/{cluster}/tables/dwd_shard/dw_user_tag_all/{layer}', '{replica}')

其中主要包括两部分：zk_path和replica_name

* zk_path，数据表路径，名称可自定义（**同一张表的同一分片的不同副本，应该定义相同的路径**）
* replica_name，副本名称（**同一张表的同一分片的不同副本，应该定义不同的名称**）

## ReplicatedMergeTree 原理

#### 数据结构

> ReplicatedMergeTree的核心逻辑主要都是运用ZK的能力，包括副本协同、主副本选举、状态感知、操作日志分发、任务队列、BlockID去重判断等

##### 元数据

* /metadata: 保存元数据信息，包括主键、分区键、采样表达式等
* /columns: 保存列字段信息，包括列名称和数据类型
* /replicas: 保存副本名称

##### 判断标识

* /leader_election: 用于主副本选举工作，主副本会主要MERGE 和 MUTATION操作
* /blocks: 记录Block数据块的Hash信息摘要及对应的partition_id
* /blocks_numbers: 按分区的写入顺序以相同的顺序记录partition_id
* /quorum: 记录quorum的数量，由insert_quorum参数控制，默认值0

##### 操作日志

* /log: 常规操作日志节点（INSERT、MERGE 和 DROP PARTITION）
* /mutations: MUTATION操作日志节点，与log日志类似（ALTER DELETE 和 ALTER UPDATE查询）
* /replicas/{replica_name}/*: 每个副本各自节点的一组监听节点，指导副本在本地执行具体任务指令
    * /queue: 任务队列节点，执行具体的操作任务
    * /log_pointer: log日志指针节点，记录最后一次执行log日志的下标信息
    * /mutation_pointer: mutations日志指针节点，记录最后一次执行mutations日志的下标信息

##### Entry对象

> 上述提到的/log 和 /mutations 节点在CK中被统一的抽象为Entry对象，具体实现则由LogEntry 和 MutationEntry对象承载

* LogEntry: 核心属性包括source replica,type,block_id,partition_name
* MutationEntry: 核心属性包括source replica,commands,mutation_id,partiltion_id

#### 副本协同

> 核心流程主要INSERT、MERGE、MUTATION和ALTER四种，分别对应数据写入、分区合并、数据修改和元数据修改。

备注：以下执行主要按1分片、1副本讲解，若大于1个副本的场景流程可类推

##### INSERT执行流程

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/INSERT%E6%A0%B8%E5%BF%83%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

备注：

1. CK1被选举为主副本
1. 在写入过程中，ZK不会进行任何实质性的数据传输，本着谁执行谁负责的原则

##### MERGE执行流程

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/MERGE%E6%A0%B8%E5%BF%83%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

备注：

1. 分区合并时，无论MERGE操作从哪个副本发起，都将由主副本来制定
1. 分区合并时，主副本会锁住执行线程，监听可由replication_alter_partitions_sync参数控制，默认值为1，即只等待主副本

##### MUTATION执行流程

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/MUTATION%E6%A0%B8%E5%BF%83%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

##### ALTER执行流程

> 主要进行元数据的修改

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/ALTER%E6%A0%B8%E5%BF%83%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

备注：

1. 各个副本均将监听共享元数据的变更
1. 谁执行谁负责确定所有副本修改完成状态

## 数据分片

#### 描述

​	CK中每个服务节点都可以称为一个shard分片，一般数据分片需要结合Distributed表引擎一同使用。

#### **集群配置**

> 主要介绍三种形式的配置

1. 不定义副本的分片

```xml
<cluster>
    <node>
        <host></host>
        <port></port>
        <user></user>
        <password></password>
    </node>
    <node>
    ...
    </node>
</cluster>
```

2. 定义副本不含副本的分片

> 如咱们日志集群配置，效果同不定义副本的分片

```xml
<ch_cluster2>
<shard>
    <replica>
        <host>clickhouse13</host>
        <port>9000</port>
        <user>clickhouse_cluster</user>
        <password></password>
    </replica>
</shard>
<shard>
    <replica>
        <host>clickhouse14</host>
        <port>9000</port>
        <user>clickhouse_cluster</user>
        <password></password>
     </replica>
</shard>
<shard>
...
</shard>
</ch_cluster2>
```

3. 定义副本的分片

> 如咱们业务集群配置

```xml
<ch_cluster1>
<shard>
    <weight>1</weight>
    <internal_replication>true</internal_replication>
    <replica>
        <host>clickhouse1</host>
        <port>9000</port>
        <user>clickhouse_cluster</user>
        <password></password>
    </replica>
    <replica>
        <host>clickhouse10</host>
        <port>9000</port>
        <user>clickhouse_cluster</user>
        <password></password>
    </replica>
</shard>
<shard>
...
</shard>
</ch_cluster1>
```

#### **分布式DDL**

> 即执行 ON CLUSTER cluster_name 操作

##### **数据结构**

1. 分布式DDL语句同样需要借助ZK，默认情况下ZK内使用的根路径为/clickhouse/task_queue/ddl，支持distributed_ddl配置
1. DDL操作日志使用ZK顺序节点，指令名称以query- 为前缀，包含两个状态节点 active、finished
1. /query-[seq] 下记录的日志信息由DDLLogEntry承载，核心属性包括：
    * query - 记录了DDL查询的执行语句
    * hosts - 记录了指定集群的hosts主机列表
    * initator - 记录了初始化 host主机的名称

##### **执行流程**

![](https://raw.githubusercontent.com/woozhijun/images/main/imgs/%E5%88%86%E5%B8%83%E5%BC%8FDDL%E6%A0%B8%E5%BF%83%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)

备注：

1. 确认执行时，客户端默认会阻塞等待180s，若超过该值则会装入后台线程继续等待
1. 等待时间可由distributed_ddl_task_timeout参数指定

## **Distributed原理**

#### 描述

​	Distributed表引擎自身不存储任何数据，可理解为一层透明代理，在集群内部开展数据的写入、分发、查询路由等，故它需要和其他数据表引擎一同工作。

​	从实体表层面看，一张分布式表主要有本地表（xxx_local）和分布式表（xxx_all）两部分组成。

​	Distributed表不支持任何MUTATION类型的操作。

#### **定义形式**

> ENGINE = Distributed(cluster, database, table, [, sharding_key])

其中，各参数含义如下：

* cluster: 集群名称
* database/table: 对应的数据库和本地表的名称
* sharding_key: 分片键，分布式表会依据规则将数据分布到各host节点的本地表上（注：虽然该参数为选填，但通过我们都将设置，避免失去分片的意义）
    * uid等业务字段
    * rand()等随机数函数
    * sipHash64(uid)等散列值函数

#### **写入流程**

> 主要介绍两种常见思路

##### 通过本地表

> 若要求更好的写入性能的情况下，优先推荐



**优点：**写入性能更好，读写分离

**缺点：**需要依赖外部系统

##### 通过分布式表

> 依赖Distributed表引擎代理写入，便于理解这里将拆分为分片写入、副本复制两个过程

###### 分片写入流程



备注：

1. Distributed表负责向远端发送数据时，有异步写和同步写两种模式
1. 通过insert_distributed_sync参数控制，默认false异步写，即写完本地分片后成功就返回
1. 若insert_distributed_sync设置true，可额外通过insert_distributed_timeout控制等待超时时长

###### 副本复制

1. 通过Distributed复制数据

> 该种方案Distributed节点需要通过负责分片和副本的数据写入工作，它很可能会遇到写入的单点瓶颈。

2. 通过ReplicatedMergeTree复制数据

> 该种方案则需要在shard配置中添加<internal_replication>true</internal_replication>(默认为false)，那么Distributed表只会选择一个合适的replica写入数据，其它的副本数据交由ReplicatedMergeTree自己处理，从而减负。

#### **查询流程**

> 主要介绍两种场景，如多副本表，多分片表

##### 多副本表查询

1. 若集群中是一个shard多个replica，那么Distributed表需要选择副本，故通过负载均衡算法从众多中选择一个
1. 负载均衡算法由load_balancing参数控制，主要包括random（默认）、nearest_hostname、in_order、first_or_random四种

##### 多分片表查询

1. 本着谁执行谁负责的原则，向CK10发起分布式表执行SELECT uid FROM distributed_table
1. 它会转为如下形式SELECT uid FROM local_table，先执行本地分片，再发送远端各分片执行
1. 合并结果为临时表返回
