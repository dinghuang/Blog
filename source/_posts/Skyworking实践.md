---
title: SkyWalking实践
date: 2020-12-11 15:47:00
tags:
    - JAVA
categories: JAVA
---

# 简介
SkyWalking是一个开源的可观察性平台，用于收集，分析，聚合和可视化来自服务和云本机基础结构的数据。SkyWalking提供了一种简便的方法来维护您的分布式系统的清晰视图，即使在整个云中也是如此。它是一种现代APM，专门为基于云的基于容器的分布式系统而设计。

## 为什么要使用SkyWalking
SkyWalking提供了用于在许多不同情况下观察和监视分布式系统的解决方案。首先，与传统方法一样，SkyWalking为服务提供自动仪器代理，例如Java，C＃，Node.js，Go，PHP和Nginx LUA。（并呼吁提供Python和C ++ SDK贡献）。在多语言，持续部署的环境中，云本机基础架构变得越来越强大，但也越来越复杂。SkyWalking的服务网格接收器使SkyWalking能够从Istio / Envoy和Linkerd等服务网格框架接收遥测数据，从而使用户能够了解整个分布式系统。

SkyWalking为服务，服务实例，端点提供可观察性功能。服务，实例和端点这些术语在当今到处都有使用，因此值得在SkyWalking的上下文中定义它们的特定含义：

- 服务。表示一组/一组工作负载，这些工作负载为传入请求提供相同的行为。您可以在使用乐器代理或SDK时定义服务名称。SkyWalking也可以使用您在Istio等平台中定义的名称。
- 服务实例。服务组中的每个单独工作负载都称为实例。像pods在Kubernetes中一样，它不必是单个OS进程，但是，如果您使用仪器代理，则实例实际上是一个真正的OS进程。
- 端点。服务中用于传入请求的路径，例如HTTP URI路径或gRPC服务类+方法签名。
通过SkyWalking，用户可以了解服务和端点之间的拓扑关系，查看每个服务/服务实例/端点的指标并设置警报规则。

此外，可以整合

- 其他使用SkyWalking本机代理程序和SDK以及Zipkin，Jaeger和OpenCensus进行的分布式跟踪。
- 其他度量系统，例如Prometheus，Sleuth（Micrometer）。

## 架构设计
SkyWalking分为四个部分：
- 探针：收集数据并重新格式化以符合SkyWalking要求（不同的探针支持不同的来源）。
- 平台后端：支持数据聚合，分析并驱动从探针到UI的流程。该分析包括SkyWalking本机跟踪和度量标准，包括Istio和Envoy遥测的第三方，Zipkin跟踪格式等。您甚至可以通过使用针对本机度量标准的Observability Analysis Language和针对扩展度量标准的Meter System来定制聚合和分析。
- 存储：存储设备通过开放/可插入的界面存储SkyWalking数据。您可以选择现有的实现，例如ElasticSearch，H2或由Sharding-Sphere管理的MySQL集群，也可以实现自己的实现。欢迎新存储实现者使用补丁！
- UI：是一个高度可定制的基于Web的界面，允许SkyWalking最终用户可视化和管理SkyWalking数据。


![image](https://minios.strongsickcat.com/dinghuang-blog-picture/687474703a2f2f736b7977616c6b696e672e6170616368652e6f72672f6173736574732f6672616d652d76382e6a70673f753d3230323030343233.jpeg)


# 使用
本文章基于官方源码tag:``v8.3.0``

## 源码编译

修改``.mvn/jvm.config``
```
-Dhttp.proxyHost=proxy_ip
-Dhttp.proxyPort=proxy_port
-Dhttps.proxyHost=proxy_ip
-Dhttps.proxyPort=proxy_port 
-Dhttp.proxyUser=username
-Dhttp.proxyPassword=password
```

下载git源码
```
git clone --branch v8.3.0 https://github.com/apache/skywalking.git
git submodule init
git submodule update
```
执行编译
```
./mvnw clean package -DskipTests
```
打包后的都会在``/dist``目录下

## 下载制品
推荐直接下载[成品](https://mirror.bit.edu.cn/apache/skywalking/8.3.0/apache-skywalking-apm-8.3.0.tar.gz)

## 服务启动
进入目录``/bin``，windows和linux的启动脚本都在里面，可以直接启动。

执行命令
```
 ./dist-material/bin/startup.sh
 ```
 
现在服务端就启起来了，可以打开后台地址查看(默认是8080端口): http://localhost:8080,界面如下：
![image](https://minios.strongsickcat.com/dinghuang-blog-picture/WechatIMG8155.png)
![image](https://minios.strongsickcat.com/dinghuang-blog-picture/WechatIMG8158.png)

![image](https://minios.strongsickcat.com/dinghuang-blog-picture/WechatIMG8162.png)

![image](https://minios.strongsickcat.com/dinghuang-blog-picture/WechatIMG8153.png)

![image](https://minios.strongsickcat.com/dinghuang-blog-picture/WechatIMG8154.png)



客户端需要提前准备开墙信息，默认的启动端口是8080，数据采集的端口是11800


默认的存储是用内存数据库H2来存储的，建议改掉，不然数据会跟着服务重启丢失，而且服务占用内存会很大。修改的话在目录``/config``下有3个文件可以改：

- application.yml
- log4j.xml
- alarm-settings.yml


其中``application.yml``配置如下
```
cluster:
  selector: ${SW_CLUSTER:standalone}
  standalone:
  # Please check your ZooKeeper is 3.5+, However, it is also compatible with ZooKeeper 3.4.x. Replace the ZooKeeper 3.5+
  # library the oap-libs folder with your ZooKeeper 3.4.x library.
  zookeeper:
    nameSpace: ${SW_NAMESPACE:""}
    hostPort: ${SW_CLUSTER_ZK_HOST_PORT:localhost:2181}
    # Retry Policy
    baseSleepTimeMs: ${SW_CLUSTER_ZK_SLEEP_TIME:1000} # initial amount of time to wait between retries
    maxRetries: ${SW_CLUSTER_ZK_MAX_RETRIES:3} # max number of times to retry
    # Enable ACL
    enableACL: ${SW_ZK_ENABLE_ACL:false} # disable ACL in default
    schema: ${SW_ZK_SCHEMA:digest} # only support digest schema
    expression: ${SW_ZK_EXPRESSION:skywalking:skywalking}
  kubernetes:
    namespace: ${SW_CLUSTER_K8S_NAMESPACE:default}
    labelSelector: ${SW_CLUSTER_K8S_LABEL:app=collector,release=skywalking}
    uidEnvName: ${SW_CLUSTER_K8S_UID:SKYWALKING_COLLECTOR_UID}
  consul:
    serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
    # Consul cluster nodes, example: 10.0.0.1:8500,10.0.0.2:8500,10.0.0.3:8500
    hostPort: ${SW_CLUSTER_CONSUL_HOST_PORT:localhost:8500}
    aclToken: ${SW_CLUSTER_CONSUL_ACLTOKEN:""}
  etcd:
    serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
    # etcd cluster nodes, example: 10.0.0.1:2379,10.0.0.2:2379,10.0.0.3:2379
    hostPort: ${SW_CLUSTER_ETCD_HOST_PORT:localhost:2379}
  nacos:
    serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
    hostPort: ${SW_CLUSTER_NACOS_HOST_PORT:localhost:8848}
    # Nacos Configuration namespace
    namespace: ${SW_CLUSTER_NACOS_NAMESPACE:"public"}
    # Nacos auth username
    username: ${SW_CLUSTER_NACOS_USERNAME:""}
    password: ${SW_CLUSTER_NACOS_PASSWORD:""}
    # Nacos auth accessKey
    accessKey: ${SW_CLUSTER_NACOS_ACCESSKEY:""}
    secretKey: ${SW_CLUSTER_NACOS_SECRETKEY:""}
core:
  selector: ${SW_CORE:default}
  default:
    # Mixed: Receive agent data, Level 1 aggregate, Level 2 aggregate
    # Receiver: Receive agent data, Level 1 aggregate
    # Aggregator: Level 2 aggregate
    role: ${SW_CORE_ROLE:Mixed} # Mixed/Receiver/Aggregator
    restHost: ${SW_CORE_REST_HOST:0.0.0.0}
    restPort: ${SW_CORE_REST_PORT:12800}
    restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
    restMinThreads: ${SW_CORE_REST_JETTY_MIN_THREADS:1}
    restMaxThreads: ${SW_CORE_REST_JETTY_MAX_THREADS:200}
    restIdleTimeOut: ${SW_CORE_REST_JETTY_IDLE_TIMEOUT:30000}
    restAcceptorPriorityDelta: ${SW_CORE_REST_JETTY_DELTA:0}
    restAcceptQueueSize: ${SW_CORE_REST_JETTY_QUEUE_SIZE:0}
    gRPCHost: ${SW_CORE_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_CORE_GRPC_PORT:11800}
    maxConcurrentCallsPerConnection: ${SW_CORE_GRPC_MAX_CONCURRENT_CALL:0}
    maxMessageSize: ${SW_CORE_GRPC_MAX_MESSAGE_SIZE:0}
    gRPCThreadPoolQueueSize: ${SW_CORE_GRPC_POOL_QUEUE_SIZE:-1}
    gRPCThreadPoolSize: ${SW_CORE_GRPC_THREAD_POOL_SIZE:-1}
    gRPCSslEnabled: ${SW_CORE_GRPC_SSL_ENABLED:false}
    gRPCSslKeyPath: ${SW_CORE_GRPC_SSL_KEY_PATH:""}
    gRPCSslCertChainPath: ${SW_CORE_GRPC_SSL_CERT_CHAIN_PATH:""}
    gRPCSslTrustedCAPath: ${SW_CORE_GRPC_SSL_TRUSTED_CA_PATH:""}
    downsampling:
      - Hour
      - Day
    # Set a timeout on metrics data. After the timeout has expired, the metrics data will automatically be deleted.
    enableDataKeeperExecutor: ${SW_CORE_ENABLE_DATA_KEEPER_EXECUTOR:true} # Turn it off then automatically metrics data delete will be close.
    dataKeeperExecutePeriod: ${SW_CORE_DATA_KEEPER_EXECUTE_PERIOD:5} # How often the data keeper executor runs periodically, unit is minute
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:3} # Unit is day
    metricsDataTTL: ${SW_CORE_METRICS_DATA_TTL:7} # Unit is day
    # Cache metrics data for 1 minute to reduce database queries, and if the OAP cluster changes within that minute,
    # the metrics may not be accurate within that minute.
    enableDatabaseSession: ${SW_CORE_ENABLE_DATABASE_SESSION:true}
    topNReportPeriod: ${SW_CORE_TOPN_REPORT_PERIOD:10} # top_n record worker report cycle, unit is minute
    # Extra model column are the column defined by in the codes, These columns of model are not required logically in aggregation or further query,
    # and it will cause more load for memory, network of OAP and storage.
    # But, being activated, user could see the name in the storage entities, which make users easier to use 3rd party tool, such as Kibana->ES, to query the data by themselves.
    activeExtraModelColumns: ${SW_CORE_ACTIVE_EXTRA_MODEL_COLUMNS:false}
    # The max length of service + instance names should be less than 200
    serviceNameMaxLength: ${SW_SERVICE_NAME_MAX_LENGTH:70}
    instanceNameMaxLength: ${SW_INSTANCE_NAME_MAX_LENGTH:70}
    # The max length of service + endpoint names should be less than 240
    endpointNameMaxLength: ${SW_ENDPOINT_NAME_MAX_LENGTH:150}
    # Define the set of span tag keys, which should be searchable through the GraphQL.
    searchableTracesTags: ${SW_SEARCHABLE_TAG_KEYS:http.method,status_code,db.type,db.instance,mq.queue,mq.topic,mq.broker}
storage:
  selector: ${SW_STORAGE:h2}
  elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    trustStorePath: ${SW_STORAGE_ES_SSL_JKS_PATH:""}
    trustStorePass: ${SW_STORAGE_ES_SSL_JKS_PASS:""}
    secretsManagementFile: ${SW_ES_SECRETS_MANAGEMENT_FILE:""} # Secrets management file in the properties format includes the username, password, which are managed by 3rd party tool.
    dayStep: ${SW_STORAGE_DAY_STEP:1} # Represent the number of days in the one minute/hour/day index.
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:1} # Shard number of new indexes
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:1} # Replicas number of new indexes
    # Super data set has been defined in the codes, such as trace segments.The following 3 config would be improve es performance when storage super size data in es.
    superDatasetDayStep: ${SW_SUPERDATASET_STORAGE_DAY_STEP:-1} # Represent the number of days in the super size dataset record index, the default value is the same as dayStep when the value is less than 0
    superDatasetIndexShardsFactor: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_SHARDS_FACTOR:5} #  This factor provides more shards for the super data set, shards number = indexShardsNumber * superDatasetIndexShardsFactor. Also, this factor effects Zipkin and Jaeger traces.
    superDatasetIndexReplicasNumber: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_REPLICAS_NUMBER:0} # Represent the replicas number in the super size dataset record index, the default value is 0.
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the async bulk record data every ${SW_STORAGE_ES_BULK_ACTIONS} requests
    syncBulkActions: ${SW_STORAGE_ES_SYNC_BULK_ACTIONS:50000} # Execute the sync bulk metrics data every ${SW_STORAGE_ES_SYNC_BULK_ACTIONS} requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    profileTaskQueryMaxSize: ${SW_STORAGE_ES_QUERY_PROFILE_TASK_SIZE:200}
    advanced: ${SW_STORAGE_ES_ADVANCED:""}
  elasticsearch7:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    trustStorePath: ${SW_STORAGE_ES_SSL_JKS_PATH:""}
    trustStorePass: ${SW_STORAGE_ES_SSL_JKS_PASS:""}
    dayStep: ${SW_STORAGE_DAY_STEP:1} # Represent the number of days in the one minute/hour/day index.
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:1} # Shard number of new indexes
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:1} # Replicas number of new indexes
    # Super data set has been defined in the codes, such as trace segments.The following 3 config would be improve es performance when storage super size data in es.
    superDatasetDayStep: ${SW_SUPERDATASET_STORAGE_DAY_STEP:-1} # Represent the number of days in the super size dataset record index, the default value is the same as dayStep when the value is less than 0
    superDatasetIndexShardsFactor: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_SHARDS_FACTOR:5} #  This factor provides more shards for the super data set, shards number = indexShardsNumber * superDatasetIndexShardsFactor. Also, this factor effects Zipkin and Jaeger traces.
    superDatasetIndexReplicasNumber: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_REPLICAS_NUMBER:0} # Represent the replicas number in the super size dataset record index, the default value is 0.
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    secretsManagementFile: ${SW_ES_SECRETS_MANAGEMENT_FILE:""} # Secrets management file in the properties format includes the username, password, which are managed by 3rd party tool.
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the async bulk record data every ${SW_STORAGE_ES_BULK_ACTIONS} requests
    syncBulkActions: ${SW_STORAGE_ES_SYNC_BULK_ACTIONS:50000} # Execute the sync bulk metrics data every ${SW_STORAGE_ES_SYNC_BULK_ACTIONS} requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    profileTaskQueryMaxSize: ${SW_STORAGE_ES_QUERY_PROFILE_TASK_SIZE:200}
    advanced: ${SW_STORAGE_ES_ADVANCED:""}
  h2:
    driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
    url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
    user: ${SW_STORAGE_H2_USER:sa}
    metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
    maxSizeOfArrayColumn: ${SW_STORAGE_MAX_SIZE_OF_ARRAY_COLUMN:20}
    numOfSearchableValuesPerTag: ${SW_STORAGE_NUM_OF_SEARCHABLE_VALUES_PER_TAG:2}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:root@1234}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
    maxSizeOfArrayColumn: ${SW_STORAGE_MAX_SIZE_OF_ARRAY_COLUMN:20}
    numOfSearchableValuesPerTag: ${SW_STORAGE_NUM_OF_SEARCHABLE_VALUES_PER_TAG:2}
  tidb:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:4000/tidbswtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:""}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
      dataSource.useAffectedRows: ${SW_DATA_SOURCE_USE_AFFECTED_ROWS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
    maxSizeOfArrayColumn: ${SW_STORAGE_MAX_SIZE_OF_ARRAY_COLUMN:20}
    numOfSearchableValuesPerTag: ${SW_STORAGE_NUM_OF_SEARCHABLE_VALUES_PER_TAG:2}
  influxdb:
    # InfluxDB configuration
    url: ${SW_STORAGE_INFLUXDB_URL:http://localhost:8086}
    user: ${SW_STORAGE_INFLUXDB_USER:root}
    password: ${SW_STORAGE_INFLUXDB_PASSWORD:}
    database: ${SW_STORAGE_INFLUXDB_DATABASE:skywalking}
    actions: ${SW_STORAGE_INFLUXDB_ACTIONS:1000} # the number of actions to collect
    duration: ${SW_STORAGE_INFLUXDB_DURATION:1000} # the time to wait at most (milliseconds)
    batchEnabled: ${SW_STORAGE_INFLUXDB_BATCH_ENABLED:true}
    fetchTaskLogMaxSize: ${SW_STORAGE_INFLUXDB_FETCH_TASK_LOG_MAX_SIZE:5000} # the max number of fetch task log in a request

agent-analyzer:
  selector: ${SW_AGENT_ANALYZER:default}
  default:
    sampleRate: ${SW_TRACE_SAMPLE_RATE:10000} # The sample rate precision is 1/10000. 10000 means 100% sample in default.
    slowDBAccessThreshold: ${SW_SLOW_DB_THRESHOLD:default:200,mongodb:100} # The slow database access thresholds. Unit ms.
    forceSampleErrorSegment: ${SW_FORCE_SAMPLE_ERROR_SEGMENT:true} # When sampling mechanism active, this config can open(true) force save some error segment. true is default.
    segmentStatusAnalysisStrategy: ${SW_SEGMENT_STATUS_ANALYSIS_STRATEGY:FROM_SPAN_STATUS} # Determine the final segment status from the status of spans. Available values are `FROM_SPAN_STATUS` , `FROM_ENTRY_SPAN` and `FROM_FIRST_SPAN`. `FROM_SPAN_STATUS` represents the segment status would be error if any span is in error status. `FROM_ENTRY_SPAN` means the segment status would be determined by the status of entry spans only. `FROM_FIRST_SPAN` means the segment status would be determined by the status of the first span only.
    # Nginx and Envoy agents can't get the real remote address.
    # Exit spans with the component in the list would not generate the client-side instance relation metrics.
    noUpstreamRealAddressAgents: ${SW_NO_UPSTREAM_REAL_ADDRESS:6000,9000}
    slowTraceSegmentThreshold: ${SW_SLOW_TRACE_SEGMENT_THRESHOLD:-1} # Setting this threshold about the latency would make the slow trace segments sampled if they cost more time, even the sampling mechanism activated. The default value is `-1`, which means would not sample slow traces. Unit, millisecond.
    meterAnalyzerActiveFiles: ${SW_METER_ANALYZER_ACTIVE_FILES:} # Which files could be meter analyzed, files split by ","

receiver-sharing-server:
  selector: ${SW_RECEIVER_SHARING_SERVER:default}
  default:
    # For Jetty server
    restHost: ${SW_RECEIVER_SHARING_REST_HOST:0.0.0.0}
    restPort: ${SW_RECEIVER_SHARING_REST_PORT:0}
    contextPath: ${SW_RECEIVER_SHARING_REST_CONTEXT_PATH:/}
    restMinThreads: ${SW_RECEIVER_SHARING_JETTY_MIN_THREADS:1}
    restMaxThreads: ${SW_RECEIVER_SHARING_JETTY_MAX_THREADS:200}
    restIdleTimeOut: ${SW_RECEIVER_SHARING_JETTY_IDLE_TIMEOUT:30000}
    restAcceptorPriorityDelta: ${SW_RECEIVER_SHARING_JETTY_DELTA:0}
    restAcceptQueueSize: ${SW_RECEIVER_SHARING_JETTY_QUEUE_SIZE:0}
    # For gRPC server
    gRPCHost: ${SW_RECEIVER_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_RECEIVER_GRPC_PORT:0}
    maxConcurrentCallsPerConnection: ${SW_RECEIVER_GRPC_MAX_CONCURRENT_CALL:0}
    maxMessageSize: ${SW_RECEIVER_GRPC_MAX_MESSAGE_SIZE:0}
    gRPCThreadPoolQueueSize: ${SW_RECEIVER_GRPC_POOL_QUEUE_SIZE:0}
    gRPCThreadPoolSize: ${SW_RECEIVER_GRPC_THREAD_POOL_SIZE:0}
    gRPCSslEnabled: ${SW_RECEIVER_GRPC_SSL_ENABLED:false}
    gRPCSslKeyPath: ${SW_RECEIVER_GRPC_SSL_KEY_PATH:""}
    gRPCSslCertChainPath: ${SW_RECEIVER_GRPC_SSL_CERT_CHAIN_PATH:""}
    authentication: ${SW_AUTHENTICATION:""}
receiver-register:
  selector: ${SW_RECEIVER_REGISTER:default}
  default:

receiver-trace:
  selector: ${SW_RECEIVER_TRACE:default}
  default:

receiver-jvm:
  selector: ${SW_RECEIVER_JVM:default}
  default:

receiver-clr:
  selector: ${SW_RECEIVER_CLR:default}
  default:

receiver-profile:
  selector: ${SW_RECEIVER_PROFILE:default}
  default:

service-mesh:
  selector: ${SW_SERVICE_MESH:default}
  default:

envoy-metric:
  selector: ${SW_ENVOY_METRIC:default}
  default:
    acceptMetricsService: ${SW_ENVOY_METRIC_SERVICE:true}
    alsHTTPAnalysis: ${SW_ENVOY_METRIC_ALS_HTTP_ANALYSIS:""}
    # `k8sServiceNameRule` allows you to customize the service name in ALS via Kubernetes metadata,
    # the available variables are `pod`, `service`, f.e., you can use `${service.metadata.name}-${pod.metadata.labels.version}`
    # to append the version number to the service name.
    # Be careful, when using environment variables to pass this configuration, use single quotes(`''`) to avoid it being evaluated by the shell.
    k8sServiceNameRule: ${K8S_SERVICE_NAME_RULE:"${service.metadata.name}"}

prometheus-fetcher:
  selector: ${SW_PROMETHEUS_FETCHER:-}
  default:
    enabledRules: ${SW_PROMETHEUS_FETCHER_ENABLED_RULES:"self"}

kafka-fetcher:
  selector: ${SW_KAFKA_FETCHER:-}
  default:
    bootstrapServers: ${SW_KAFKA_FETCHER_SERVERS:localhost:9092}
    partitions: ${SW_KAFKA_FETCHER_PARTITIONS:3}
    replicationFactor: ${SW_KAFKA_FETCHER_PARTITIONS_FACTOR:2}
    enableMeterSystem: ${SW_KAFKA_FETCHER_ENABLE_METER_SYSTEM:false}
    isSharding: ${SW_KAFKA_FETCHER_IS_SHARDING:false}
    consumePartitions: ${SW_KAFKA_FETCHER_CONSUME_PARTITIONS:""}
    kafkaHandlerThreadPoolSize: ${SW_KAFKA_HANDLER_THREAD_POOL_SIZE:-1}
    kafkaHandlerThreadPoolQueueSize: ${SW_KAFKA_HANDLER_THREAD_POOL_QUEUE_SIZE:-1}

receiver-meter:
  selector: ${SW_RECEIVER_METER:default}
  default:

receiver-otel:
  selector: ${SW_OTEL_RECEIVER:-}
  default:
    enabledHandlers: ${SW_OTEL_RECEIVER_ENABLED_HANDLERS:"oc"}
    enabledOcRules: ${SW_OTEL_RECEIVER_ENABLED_OC_RULES:"istio-controlplane"}

receiver_zipkin:
  selector: ${SW_RECEIVER_ZIPKIN:-}
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
    jettyMinThreads: ${SW_RECEIVER_ZIPKIN_JETTY_MIN_THREADS:1}
    jettyMaxThreads: ${SW_RECEIVER_ZIPKIN_JETTY_MAX_THREADS:200}
    jettyIdleTimeOut: ${SW_RECEIVER_ZIPKIN_JETTY_IDLE_TIMEOUT:30000}
    jettyAcceptorPriorityDelta: ${SW_RECEIVER_ZIPKIN_JETTY_DELTA:0}
    jettyAcceptQueueSize: ${SW_RECEIVER_ZIPKIN_QUEUE_SIZE:0}

receiver_jaeger:
  selector: ${SW_RECEIVER_JAEGER:-}
  default:
    gRPCHost: ${SW_RECEIVER_JAEGER_HOST:0.0.0.0}
    gRPCPort: ${SW_RECEIVER_JAEGER_PORT:14250}

receiver-browser:
  selector: ${SW_RECEIVER_BROWSER:default}
  default:
    # The sample rate precision is 1/10000. 10000 means 100% sample in default.
    sampleRate: ${SW_RECEIVER_BROWSER_SAMPLE_RATE:10000}

query:
  selector: ${SW_QUERY:graphql}
  graphql:
    path: ${SW_QUERY_GRAPHQL_PATH:/graphql}

alarm:
  selector: ${SW_ALARM:default}
  default:

telemetry:
  selector: ${SW_TELEMETRY:none}
  none:
  prometheus:
    host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
    port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
    sslEnabled: ${SW_TELEMETRY_PROMETHEUS_SSL_ENABLED:false}
    sslKeyPath: ${SW_TELEMETRY_PROMETHEUS_SSL_KEY_PATH:""}
    sslCertChainPath: ${SW_TELEMETRY_PROMETHEUS_SSL_CERT_CHAIN_PATH:""}

configuration:
  selector: ${SW_CONFIGURATION:none}
  none:
  grpc:
    host: ${SW_DCS_SERVER_HOST:""}
    port: ${SW_DCS_SERVER_PORT:80}
    clusterName: ${SW_DCS_CLUSTER_NAME:SkyWalking}
    period: ${SW_DCS_PERIOD:20}
  apollo:
    apolloMeta: ${SW_CONFIG_APOLLO:http://localhost:8080}
    apolloCluster: ${SW_CONFIG_APOLLO_CLUSTER:default}
    apolloEnv: ${SW_CONFIG_APOLLO_ENV:""}
    appId: ${SW_CONFIG_APOLLO_APP_ID:skywalking}
    period: ${SW_CONFIG_APOLLO_PERIOD:5}
  zookeeper:
    period: ${SW_CONFIG_ZK_PERIOD:60} # Unit seconds, sync period. Default fetch every 60 seconds.
    nameSpace: ${SW_CONFIG_ZK_NAMESPACE:/default}
    hostPort: ${SW_CONFIG_ZK_HOST_PORT:localhost:2181}
    # Retry Policy
    baseSleepTimeMs: ${SW_CONFIG_ZK_BASE_SLEEP_TIME_MS:1000} # initial amount of time to wait between retries
    maxRetries: ${SW_CONFIG_ZK_MAX_RETRIES:3} # max number of times to retry
  etcd:
    period: ${SW_CONFIG_ETCD_PERIOD:60} # Unit seconds, sync period. Default fetch every 60 seconds.
    group: ${SW_CONFIG_ETCD_GROUP:skywalking}
    serverAddr: ${SW_CONFIG_ETCD_SERVER_ADDR:localhost:2379}
    clusterName: ${SW_CONFIG_ETCD_CLUSTER_NAME:default}
  consul:
    # Consul host and ports, separated by comma, e.g. 1.2.3.4:8500,2.3.4.5:8500
    hostAndPorts: ${SW_CONFIG_CONSUL_HOST_AND_PORTS:1.2.3.4:8500}
    # Sync period in seconds. Defaults to 60 seconds.
    period: ${SW_CONFIG_CONSUL_PERIOD:60}
    # Consul aclToken
    aclToken: ${SW_CONFIG_CONSUL_ACL_TOKEN:""}
  k8s-configmap:
    period: ${SW_CONFIG_CONFIGMAP_PERIOD:60}
    namespace: ${SW_CLUSTER_K8S_NAMESPACE:default}
    labelSelector: ${SW_CLUSTER_K8S_LABEL:app=collector,release=skywalking}
  nacos:
    # Nacos Server Host
    serverAddr: ${SW_CONFIG_NACOS_SERVER_ADDR:127.0.0.1}
    # Nacos Server Port
    port: ${SW_CONFIG_NACOS_SERVER_PORT:8848}
    # Nacos Configuration Group
    group: ${SW_CONFIG_NACOS_SERVER_GROUP:skywalking}
    # Nacos Configuration namespace
    namespace: ${SW_CONFIG_NACOS_SERVER_NAMESPACE:}
    # Unit seconds, sync period. Default fetch every 60 seconds.
    period: ${SW_CONFIG_NACOS_PERIOD:60}
    # Nacos auth username
    username: ${SW_CONFIG_NACOS_USERNAME:""}
    password: ${SW_CONFIG_NACOS_PASSWORD:""}
    # Nacos auth accessKey
    accessKey: ${SW_CONFIG_NACOS_ACCESSKEY:""}
    secretKey: ${SW_CONFIG_NACOS_SECRETKEY:""}

exporter:
  selector: ${SW_EXPORTER:-}
  grpc:
    targetHost: ${SW_EXPORTER_GRPC_HOST:127.0.0.1}
    targetPort: ${SW_EXPORTER_GRPC_PORT:9870}

health-checker:
  selector: ${SW_HEALTH_CHECKER:-}
  default:
    checkIntervalSeconds: ${SW_HEALTH_CHECKER_INTERVAL_SECONDS:5}
```

### 插件安装
Java代理插件都是可插入的。可以optional-plugins在代理或第三方存储库下的文件夹中提供可选插件。要使用这些插件，需要将目标插件jar文件放入/plugins。

目前已有的插件有：

- [Plugin of tracing Spring annotation beans](https://github.com/apache/skywalking/blob/8.3.0/docs-hotfix/docs/en/setup/service-agent/java-agent/agent-optional-plugins/Spring-annotation-plugin.md)
- [tracing Oracle and Resin](https://github.com/apache/skywalking/blob/8.3.0/docs-hotfix/docs/en/setup/service-agent/java-agent/agent-optional-plugins/Oracle-Resin-plugins.md)
- [Filter traces through specified endpoint name patterns](https://github.com/apache/skywalking/blob/8.3.0/docs-hotfix/docs/en/setup/service-agent/java-agent/agent-optional-plugins/trace-ignore-plugin.md)
- Plugin of Gson serialization lib in optional plugin folder.
- Plugin of Zookeeper 3.4.x in optional plugin folder. The reason of being optional plugin is, many business irrelevant traces are generated, which cause extra payload to agents and backends. At the same time, those traces may be just heartbeat(s).
- Customize enhance Trace methods based on description files, rather than write plugin or change source codes.
- Plugin of Spring Cloud Gateway 2.1.x in optional plugin folder. Please only active this plugin when you install agent in Spring Gateway. spring-cloud-gateway-2.x-plugin and spring-webflux-5.x-plugin are both required.
- Plugin of Spring Transaction in optional plugin folder. The reason of being optional plugin is, many local span are generated, which also spend more CPU, memory and network.
- Plugin of Kotlin coroutine provides the tracing across coroutines automatically. As it will add local spans to all across routines scenarios, Please assess the performance impact.
- Plugin of quartz-scheduler-2.x in the optional plugin folder. The reason for being an optional plugin is, many task scheduling systems are based on quartz-scheduler, this will cause duplicate tracing and link different sub-tasks as they share the same quartz level trigger, such as ElasticJob.
- Plugin of spring-webflux-5.x in the optional plugin folder. Please only activate this plugin when you use webflux alone as a web container. If you are using SpringMVC 5 or Spring Gateway, you don't need this plugin.


### 前端配置
前端配置在文件夹``webapp``的``webapp.yml``文件里面。


## 客户端接入
### java应用
执行命令里面加上
```
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -Dskywalking_config=/path/to/agent.config -jar yourApp.jar
 ```



这里另外设置了agent的配置，具体agent的配置可以参考[官方文档](https://github.com/apache/skywalking/blob/8.3.0/docs-hotfix/docs/en/setup/service-agent/java-agent/README.md)，这里贴上我自己的配置:
```
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# The agent namespace
agent.namespace=${SW_AGENT_NAMESPACE:LOCAL}

# The service name in UI
agent.service_name=${SW_AGENT_NAME:Activiti}

# The number of sampled traces per 3 seconds
# Negative or zero means off, by default
agent.sample_n_per_3_secs=${SW_AGENT_SAMPLE:-1}

# Authentication active is based on backend setting, see application.yml for more details.
# agent.authentication = ${SW_AGENT_AUTHENTICATION:xxxx}

# The max amount of spans in a single segment.
# Through this config item, SkyWalking keep your application memory cost estimated.
# agent.span_limit_per_segment=${SW_AGENT_SPAN_LIMIT:150}

# Ignore the segments if their operation names end with these suffix.
# agent.ignore_suffix=${SW_AGENT_IGNORE_SUFFIX:.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg}

# If true, SkyWalking agent will save all instrumented classes files in `/debugging` folder.
# SkyWalking team may ask for these files in order to resolve compatible problem.
# agent.is_open_debugging_class = ${SW_AGENT_OPEN_DEBUG:true}

# If true, SkyWalking agent will cache all instrumented classes files to memory or disk files (decided by class cache mode),
# allow other javaagent to enhance those classes that enhanced by SkyWalking agent.
# agent.is_cache_enhanced_class = ${SW_AGENT_CACHE_CLASS:false}

# The instrumented classes cache mode: MEMORY or FILE
# MEMORY: cache class bytes to memory, if instrumented classes is too many or too large, it may take up more memory
# FILE: cache class bytes in `/class-cache` folder, automatically clean up cached class files when the application exits
# agent.class_cache_mode = ${SW_AGENT_CLASS_CACHE_MODE:MEMORY}

# The operationName max length
# Notice, in the current practice, we don't recommend the length over 190.
# agent.operation_name_threshold=${SW_AGENT_OPERATION_NAME_THRESHOLD:150}

# If true, skywalking agent will enable profile when user create a new profile task. Otherwise disable profile.
# profile.active=${SW_AGENT_PROFILE_ACTIVE:true}

# Parallel monitor segment count
# profile.max_parallel=${SW_AGENT_PROFILE_MAX_PARALLEL:5}

# Max monitor segment time(minutes), if current segment monitor time out of limit, then stop it.
# profile.duration=${SW_AGENT_PROFILE_DURATION:10}

# Max dump thread stack depth
# profile.dump_max_stack_depth=${SW_AGENT_PROFILE_DUMP_MAX_STACK_DEPTH:500}

# Snapshot transport to backend buffer size
# profile.snapshot_transport_buffer_size=${SW_AGENT_PROFILE_SNAPSHOT_TRANSPORT_BUFFER_SIZE:50}

# Backend service addresses.
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}

# Logging file_name
logging.file_name=${SW_LOGGING_FILE_NAME:skywalking-api.log}

# Logging level
logging.level=${SW_LOGGING_LEVEL:INFO}

# Logging dir
# logging.dir=${SW_LOGGING_DIR:""}

# Logging max_file_size, default: 300 * 1024 * 1024 = 314572800
# logging.max_file_size=${SW_LOGGING_MAX_FILE_SIZE:314572800}

# The max history log files. When rollover happened, if log files exceed this number,
# then the oldest file will be delete. Negative or zero means off, by default.
# logging.max_history_files=${SW_LOGGING_MAX_HISTORY_FILES:-1}

# Listed exceptions would not be treated as an error. Because in some codes, the exception is being used as a way of controlling business flow.
# Besides, the annotation named IgnoredException in the trace toolkit is another way to configure ignored exceptions.
# statuscheck.ignored_exceptions=${SW_STATUSCHECK_IGNORED_EXCEPTIONS:}

# The max recursive depth when checking the exception traced by the agent. Typically, we don't recommend setting this more than 10, which could cause a performance issue. Negative value and 0 would be ignored, which means all exceptions would make the span tagged in error status.
# statuscheck.max_recursive_depth=${SW_STATUSCHECK_MAX_RECURSIVE_DEPTH:1}

# Mount the specific folders of the plugins. Plugins in mounted folders would work.
plugin.mount=${SW_MOUNT_FOLDERS:plugins,activations}

# Exclude activated plugins
# plugin.exclude_plugins=${SW_EXCLUDE_PLUGINS:}

# mysql plugin configuration
plugin.mysql.trace_sql_parameters=${SW_MYSQL_TRACE_SQL_PARAMETERS:true}

# Kafka producer configuration
# plugin.kafka.bootstrap_servers=${SW_KAFKA_BOOTSTRAP_SERVERS:localhost:9092}

# Match spring bean with regex expression for classname
# plugin.springannotation.classname_match_regex=${SW_SPRINGANNOTATION_CLASSNAME_MATCH_REGEX:}

```

### NodeJs接入
```
import Agent from 'skywalking';

Agent.start({
  serviceName: '',
  serviceInstance: '',
  collectorAddress: '',
  authorization: '',
  maxBufferSize: 1000,
});
```
参数包括：


Environment Variable | Description | Default
| --- | --- | --- |
| `SW_AGENT_NAME` | The name of the service | `your-nodejs-service` |
| `SW_AGENT_INSTANCE` | The name of the service instance | Randomly generated |
| `SW_AGENT_COLLECTOR_BACKEND_SERVICES` | The backend OAP server address | `127.0.0.1:11800` |
| `SW_AGENT_AUTHENTICATION` | The authentication token to verify that the agent is trusted by the backend OAP, as for how to configure the backend, refer to [the yaml](https://github.com/apache/skywalking/blob/4f0f39ffccdc9b41049903cc540b8904f7c9728e/oap-server/server-bootstrap/src/main/resources/application.yml#L155-L158). | not set |
| `SW_AGENT_LOGGING_LEVEL` | The logging level, could be one of `CRITICAL`, `FATAL`, `ERROR`, `WARN`(`WARNING`), `INFO`, `DEBUG` | `INFO` |
| `SW_AGENT_MAX_BUFFER_SIZE` | The maximum buffer size before sending the segment data to backend | `'1000'` |

### 前端接入
可以参考[官方文档](https://github.com/apache/skywalking-client-js)


## 高级功能

### 自己开发插件

可以参考[文章](https://insights.thoughtworks.cn/skywalking-plugin-guide/)，已经写得很好了。。。我不需要再表述了.


开发插件后，可以给官方[贡献一下](https://skywalking.apache.org/zh/2019-01-21-agent-plugin-practice/)

### 浏览器端监控、使用标签查询、指标分析语言
参考[官方文档](https://skywalking.apache.org/zh/2020-10-29-skywalking8-2-release/)

SkyWalking浏览器监视也提供以下数据: PV（page views，页面浏览量）， UV（unique visitors，独立访客数），浏览量前 N 的页面（Top N Page Views）等。这些数据可以为产品队伍优化他们的产品提供线索。


![image](https://minios.strongsickcat.com/dinghuang-blog-picture/plugin.intellij.assistant.mybaitslog-2020.1-1.0.3.jpg)


![image](https://minios.strongsickcat.com/dinghuang-blog-picture/0081Kckwly1gkl5gnpi0dj30zk0m843n.jpg)


### 服务性能指标监控
Skywalking还可以查看具体Service的性能指标，根据相关的性能指标可以分析系统的瓶颈所在并提出优化方案。

在服务调用拓扑图上点击相应的节点我们可以看到该服务的

- SLA: 服务可用性（主要是通过请求成功与失败次数来计算）
- CPM: 每分钟调用次数
- Avg Response Time: 平均响应时间

从应用整体外部来看我们可以监测到应用在一定时间段内的

- 服务可用性指标SLA
- 每分钟平均响应数
- 平均响应时间
- 服务进程PID
- 服务所在物理机的IP、HostName、Operation System


### 服务告警
通过webhook的方式让我们可以自定义我们告警信息的通知方式。诸如:邮件通知、微信通知、短信通知等。

先来看一下告警的规则配置。在``alarm-settings.xml``中可以配置告警规则，告警规则支持自定义。

一份告警配置由以下几部分组成：

- ``service_resp_time_rule``：告警规则名称 ``***_rule`` （规则名称可以自定义但是必须以``_rule``结尾
- ``indicator-name``：指标数据名称： 定义参见``http://t.cn/EGhfbmd``
- ``op``: 操作符： > , < , = 【当然你可以自己扩展开发其他的操作符】
- ``threshold``：目标值：指标数据的目标数据 如sample中的1000就是服务响应时间，配合上操作符就是大于1000ms的服务响应
- ``period``: 告警检查周期：多久检查一次当前的指标数据是否符合告警规则
- ``counts``: 达到告警阈值的次数
- ``silence-period``：忽略相同告警信息的周期
- ``message``：告警信息
- ``webhooks``：服务告警通知服务地


Skywalking通过HttpClient的方式远程调用在配置项webhooks中定义的告警通知服务地址。