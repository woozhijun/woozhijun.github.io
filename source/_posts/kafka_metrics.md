---
title: 洞悉 Kafka metrics
tags: 大数据 kafka
category: kafka
---

## 摘要
> 本次主要介绍关键的Kafka性能指标，Kafka metrics不同收集方式及数据上报的实现，最后确保达到有效监控Kafka工作状态的目的.

## Kafka metrics
Kafka使用Yammer Metrics来上报服务端和客户端的Metric信息，通过配置采集相应数据上报监控系统，展示可视化结果。
Yammer Metrics提供6种形式的Metrics收集 —— Gauges，Counters，Histograms，Timers，Health Checks，Metrics Annotation。
Yammer Metrics将Metrics收集与上报分离，可以根据需要自由组合。目前支持的Reporter有Console Reporter，JMX Reporter，CSV Reporter，SLF4J Reporter，HTTP Reporter，Ganglia Reporter，Graphite Reporter。
因此，Kafka将可以通过以上组合输出我们想要的Metrics数据。

### metrics收集
#### 注册

```
final MetricRegistry metrics = new MetricRegistry();
```

#### Gauges
gauge是一个数值的瞬时测量。比如我们可能需要衡量队列中待处理作业的数量：

```
public class QueueManager {
    private final Queue queue;
    public QueueManager(MetricRegistry metrics, String name) {
        this.queue = new Queue();
        // name(QueueManager.class, name, "size") 中间按.分隔
        metrics.register(MetricRegistry.name(QueueManager.class, name, "size"),
                         new Gauge<Integer>() {
                             @Override
                             public Integer getValue() {
                                 return queue.size();
                             }
                         });
    }
}
```

#### Counters
counter是AtomicLong类型的gauge，你可以增加或减少其值。比如我们可能需要更有效的统计阻塞在队列里job的数量：

```
private final Counter pendingJobs = metrics.counter(name(QueueManager.class, "pending-jobs"));
public void addJob(Job job) {
    // 自增
    pendingJobs.inc();
    queue.offer(job);
}
public Job takeJob() {
    pendingJobs.dec();
    return queue.take();
}

MetricRegistry.name(QueueManager.class, "jobs", "size")
```

#### Histograms
histogram统计数据的分布，比如min,max,mean,stddev,median,P75,P90,P95,P98,P99,P99.9等的值

```
private final Histogram responseSizes = metrics.histogram(name(RequestHandler.class, "response-sizes"));

public void handleRequest(Request request, Response response) {
    // 增加上报的值
    responseSizes.update(response.getContent().length);
}
```

#### Timers
Timer统计一个特定的代码片段被调用的速率和其持续时间的分布，比如统计response的耗时：

```
private final Timer responses = metrics.timer(name(RequestHandler.class, "responses"));

public String handleRequest(Request request, Response response) {
    final Timer.Context context = responses.time();
    try {
        // 具体执行逻辑
        return "OK";
    } finally {
        context.stop();
    }
}
```

#### Health Checks
主要用户可以自己判断系统的健康状态，比如判断数据库是否连接正常：

```
final HealthCheckRegistry healthChecks = new HealthCheckRegistry();

public class DatabaseHealthCheck extends HealthCheck {
    private final Database database;

    public DatabaseHealthCheck(Database database) {
        this.database = database;
    }

    @Override
    public HealthCheck.Result check() throws Exception {
        if (database.isConnected()) {
            return HealthCheck.Result.healthy();
        } else {
            return HealthCheck.Result.unhealthy("Cannot connect to " + database.getUrl());
        }
    }
}

healthChecks.register("postgres", new DatabaseHealthCheck(database));
```

#### Metrics Annotation
注解方式，简单的实现统计某个方法、某个值的数据：

```
/**
* 统计调用的次数和时间
*/
@Timed
public void call() {
}
    
/**
* 统计登陆的次数
*/
@Counted
public void userLogin(){
}

//other
@CachedGauge @Gauge @ExceptionMetered @Metered @Metric
``` 

### metrics上报
#### ConsoleReporter
对于简单指标的计算，可以使用定期向控制台报告：

```
final ConsoleReporter reporter = ConsoleReporter.forRegistry(metrics)
				                    .convertRatesTo(TimeUnit.SECONDS)
				                    .convertDurationsTo(TimeUnit.MILLISECONDS).build();
metrics.register("jvm.mem", new MemoryUsageGaugeSet());
metrics.register("jvm.gc", new GarbageCollectorMetricSet());
                                                
reporter.start(5, TimeUnit.MINUTES);
```

#### JmxReporter
使用Jmx上报数据，转化为MBean，注：不建议在生产环境中使用，JMX的RPC API是不可靠的，但为了开发和浏览可选可视化工具：

```
final JmxReporter reporter = JmxReporter.forRegistry(registry).build();
reporter.start();
```

#### CsvReporter
对于相对复杂的指标，可将同一个metric创建.csv文件，并将定期上报的数据按新行写入：

```
final CsvReporter reporter = CsvReporter.forRegistry(registry)
                                        .formatFor(Locale.US)
                                        .convertRatesTo(TimeUnit.SECONDS)
                                        .convertDurationsTo(TimeUnit.MILLISECONDS)
                                        .build(new File("~/projects/data/"));
reporter.start(1, TimeUnit.SECONDS);
```

#### Slf4jReporter
可以将上报数据记录slf4j日志：

```
final Slf4jReporter reporter = Slf4jReporter.forRegistry(registry)
                                            .outputTo(LoggerFactory.getLogger("com.example.metrics"))
                                            .convertRatesTo(TimeUnit.SECONDS)
                                            .convertDurationsTo(TimeUnit.MILLISECONDS)
                                            .build();
reporter.start(1, TimeUnit.MINUTES);
```

#### HTTP Reporter
需搭建web服务，新增Listener事件，完成相应metrics指标注册，目前支持HealthCheckServlet，ThreadDumpServlet，MetricsServlet，PingServlet四类，最后启动服务即可请求获取Json格式的监控数据。实现详情如下：

```
public class MyMetricsServletListener extends MetricsServlet.ContextListener {
	
	public static final MetricRegistry metrics = new MetricRegistry();

	@Override
	protected MetricRegistry getMetricRegistry() {
		
		metrics.register("jvm.mem", new MemoryUsageGaugeSet());
		metrics.register("jvm.gc", new GarbageCollectorMetricSet());
		System.out.println(">>.Listener success...");
		return metrics;
	}
}
```

```
public class MyHealthCheckServletListener extends HealthCheckServlet.ContextListener {

	public static final HealthCheckRegistry healthCheck = new HealthCheckRegistry();
	@Override
	protected HealthCheckRegistry getHealthCheckRegistry() {
		
		healthCheck.register("postgres", new TFHealthCheck(false));
		return healthCheck;
	}
}
```

```
	<listener>
        <listener-class>com.immomo.hubble.kafka.servlet.MyMetricsServletListener</listener-class>
	</listener>
	<listener>
        <listener-class>com.immomo.hubble.kafka.servlet.MyHealthCheckServletListener</listener-class>
	</listener>
	
	<servlet>
        <servlet-name>metrics</servlet-name>
        <servlet-class>com.codahale.metrics.servlets.AdminServlet</servlet-class>
	</servlet>
	<servlet-mapping>
	   <servlet-name>metrics</servlet-name>
	   <url-pattern>/metrics/*</url-pattern>
	</servlet-mapping>
```

[源码地址](https://github.com/dropwizard/metrics)


### Kafka性能指标
#### kafka.server
BrokerTopicMetrics,name=MessagesInPerSec: 每秒消息量
BrokerTopicMetrics,name=BytesInPerSec: 每秒输入字节数
BrokerTopicMetrics,name=BytesOutPerSec: 每秒输出字节数
ReplicaManager,name=UnderReplicatedPartitions: 复制分区的数量，默认0，|ISR|<|all replicas|
ReplicaManager,name=PartitionCount: 分区数
ReplicaManager,name=LeaderCount: Leader副本数
ReplicaManager,name=IsrShrinksPerSec: ISR回退
ReplicaManager,name=IsrExpandsPerSec: ISR超前 value is 0
ReplicaFetcherManager,name=MaxLag,clientId=Replica: 滞后follower和leader的最大消息长度
FetcherLagMetrics,name=ConsumerLag,clientId=([-.\w]+),topic=([-.\w]+),partition=([0-9]+): 滞后follower的消息长度
DelayedOperationPurgatory,name=PurgatorySize,delayedOperation=Produce: producer等待请求大小
DelayedOperationPurgatory,name=PurgatorySize,delayedOperation=Fetch: 获取等待请求大小
KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent: 平均处理线程空闲时间

#### kafka.network
kafka.network:type=RequestMetrics,name=$1,request={Produce|FetchConsumer|FetchFollower}
RequestsPerSec: 每秒请求量
TotalTimeMs: 请求总时间
RequestQueueTimeMs: 请求队列等待时间
LocalTimeMs: 请求leader处理时间
RemoteTimeMs: 请求follower等待时间
ResponseQueueTimeMs: 请求队列等待响应时间
ResponseSendTimeMs: 请求响应发送时间
kafka.network:type=SocketServer,name=NetworkProcessorAvgIdlePercent: 平均网络处理空闲时间
    
#### kafka.controller
KafkaController,name=ActiveControllerCount: 活跃broker数量
ControllerStats,name=LeaderElectionRateAndTimeMs: leader选举率
ControllerStats,name=UncleanLeaderElectionsPerSec: Unclean leader选举率

#### common
connection-close-rate: 每秒连接关闭率
connection-creation-rate: 每秒新建连接率
network-io-rate: 平均每秒IO次数（读取或写入）
outgoing-byte-rat: 平均每秒向服务器发送的字节数
request-rate: 平均每秒发送的请求数
incoming-byte-rate: 每秒读取字节数
response-rate: 每秒收到的回复
select-rate: IO切换次数
io-wait-ratio: IO线程等待时间
connection-count: 当前活跃连接数
    
#### broker
outgoing-byte-rate: 平均每秒发送字节数
request-rate: 平均每秒请求数
request-size-avg: 所有请求的平均大小
request-latency-avg: 平均请求时间(ms)
response-rate: 每秒收到的响应数
    
#### producer
waiting-threads: 缓存区排队的用户阻塞线程数
buffer-available-bytes: 可用内存字节数
batch-size-avg: 每个分区每次请求发送的平均字节数
compression-rate-avg: 批量记录平均压缩率
record-queue-time-avg: 批量记录耗费的平均时间（ms）
request-latency-avg: 平均请求时间（ms）
record-send-rate: 每秒发送的平均次数
record-retry-rate: 每秒重试发送次数
record-error-rate: 每秒错误数量次数
requests-in-flight: 目前等待响应的请求数量
metadata-age: 当前生产者数据使用周期（s）
    
#### consumer
##### Consumer Group
commit-latency-avg: 提交请求所用的平均时间
commit-rate: 每秒提交调用次数
assigned-partitions: 当前分配给该消费者的分区数（可选）
heartbeat-rate: 平均每秒心跳数
join-time-avg: 群组重新加入的平均时间
join-rate: 每秒连接组的数量
sync-time-avg: 群组同步所需的平均时间
sync-rate: 每秒同步的组数
    
##### consumer fetch
fetch-size-avg: 每次请求获取的平均字节数
bytes-consumed-rate: 每秒消耗的平均字节数
fetch-latency-avg: 请求所用的平均时间
fetch-rate: 每秒提取请求数
records-lag-max: 分区中记录的最大滞后数量
    
##### topic-level fetch
fetch-size-avg: topic请求的平均字节数
bytes-consumed-rate: topic每秒平均消耗的字节数

#### streams
##### Thread
[commit|poll|process|punctuate]-latency-avg: 平均执行时间（ms）
[commit|poll|process|punctuate]-rate: 平均每秒请求数
task-created-rate: 每秒新建任务数
task-closed-rate: 每秒关闭任务数
skipped-records-rate: 每秒跳过记录数
    
##### Task
commit-latency-avg: 平均执行时间（ms）
commit-rate: 每秒提交的平均次数

##### Processor Node
forward-rate: 每秒从源节点向下游转发的平均速率

##### State Store 
[put|put-if-absent|get|delete|put-all|all|range|flush|restore]-latency-avg: 平均执行时间（ns）
[put|put-if-absent|get|delete|put-all|all|range|flush|restore]-rate: 每秒的平均运行速度

#### others
GC、CPU、IO等

### Monitoring实现
#### 方式一：
1.提供Web服务支持，执行监听器注册我们需要监控的指标，可按Json格式输出
2.启动后台进程定期通过Http请求抓取指标，并上报数据到redis队列
3.storm消费数据进行分析计算

```
需多依赖
<dependency>
    <groupId>io.dropwizard.metrics</groupId>
    <artifactId>metrics-servlets</artifactId>
    <version>3.1.0</version>
</dependency>
```

#### 方式二：
1.重写KafkaReporter继承ScheduledReporter，构造相应指标，格式化、序列化Json，通过metrics收集统计结果
2.创建kafkaProducer将构造的结果数据按不同Topic分类发送给kafka
3.storm消费对应kafka数据，再进行分析计算

Pom依赖：

```
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.11</artifactId>
    <version>0.10.2.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
			</exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.10</version>
    <exclusions>
        <exclusion>
            <groupId>com.sun.jmx</groupId>
            <artifactId>jmxri</artifactId>
        </exclusion>
			<exclusion>
				<groupId>com.sun.jdmk</groupId>
				<artifactId>jmxtools</artifactId>
			</exclusion>
			<exclusion>
				<groupId>javax.jms</groupId>
				<artifactId>jms</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
	       <groupId>io.dropwizard.metrics</groupId>
	       <artifactId>metrics-core</artifactId>
	       <version>3.1.0</version>
	</dependency>
```

### Kafka监控工具
#### KafkaOffsetMonitor
> 是由Kafka开源社区提供的一款Web管理界面，配置操作简单，界面简单，但不能自动刷新，但功能覆盖不全
> 可以对consumer消费情况进行监控，并能列出每个consumer的offset数据
> 可以查看每个topic的partition的列表（topic，pid，offset，logsize，lag，owner等）
> 可以查看每个consumser group列表信息
> 可以查看topic的历史消费信息
> [源码地址](https://github.com/quantifind/KafkaOffsetMonitor)
    
#### kafka-web-console
> 也是kafka开源的web监控程序，Scala编写，编译搭建相对于前者更复杂，默认用的数据库是H2，但可自动刷新。
> 可以查看brokers kafka集群信息
> 可以查看每个topic的Partition，logsize，分区leader等信息
> 可以查看consumer group的partition，offset，lag，owner等信息；topic feed最新发布消息
> 可以查看consumer偏移和滞后情况，历史消息以及consumer/producer消息吞吐量历史的图表
> 控制台还提供了RAML中描述的JSON API
> [源码地址](https://github.com/claudemamo/kafka-web-console)

#### kafka-manger
> 是由yahoo构建一款基于web的kafka管理器，可以管理多个kafka集群，且容易检测集群（topics,brokers,备份,分区）分布不均的情况
> 可以选择你要运行的副本
> 可以基于当前分区状况重新分配生成分区
> 可以选择topic配置并创建topic(0.8.1.1和0.8.2+的配置不同)
> 可以删除topic(只支持0.8.2+的版本并且要在broker配置中设置delete.topic.enable=true)，Topic list会指明哪些topic被删除
> 可以为已存在的topic增加分区，更新配置
> 可以支持多个topic批量重分区，选择partition broker位置等
> 可以启用JMX轮询代理，以及broker、metrics主题等级
> 可选地筛选出zookeeper中没有ids/owner/＆offset/目录的消费者
> [源码地址](https://github.com/yahoo/kafka-manager)

## QA
**欢迎大家共同讨论、分享**



