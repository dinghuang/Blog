---
title: Seata部署使用(1.4.2)
date: 2021-06-08 15:47:00
tags:
    - JAVA
categories: JAVA
---

源码地址： https://github.com/seata/seata.git 

# 前期准备
## xa模式
确保mysql版本>5.7 并且开启了XA事务支持

## at模式
每个mysql库执行脚本
```
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```
## saga模式
```
-- -------------------------------- The script used for sage  --------------------------------


CREATE TABLE IF NOT EXISTS `seata_state_machine_def`
(
    `id`               VARCHAR(32)  NOT NULL COMMENT 'id',
    `name`             VARCHAR(128) NOT NULL COMMENT 'name',
    `tenant_id`        VARCHAR(32)  NOT NULL COMMENT 'tenant id',
    `app_name`         VARCHAR(32)  NOT NULL COMMENT 'application name',
    `type`             VARCHAR(20)  COMMENT 'state language type',
    `comment_`         VARCHAR(255) COMMENT 'comment',
    `ver`              VARCHAR(16)  NOT NULL COMMENT 'version',
    `gmt_create`       DATETIME(3)  NOT NULL COMMENT 'create time',
    `status`           VARCHAR(2)   NOT NULL COMMENT 'status(AC:active|IN:inactive)',
    `content`          TEXT COMMENT 'content',
    `recover_strategy` VARCHAR(16) COMMENT 'transaction recover strategy(compensate|retry)',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `seata_state_machine_inst`
(
    `id`                  VARCHAR(128)            NOT NULL COMMENT 'id',
    `machine_id`          VARCHAR(32)             NOT NULL COMMENT 'state machine definition id',
    `tenant_id`           VARCHAR(32)             NOT NULL COMMENT 'tenant id',
    `parent_id`           VARCHAR(128) COMMENT 'parent id',
    `gmt_started`         DATETIME(3)             NOT NULL COMMENT 'start time',
    `business_key`        VARCHAR(48) COMMENT 'business key',
    `start_params`        TEXT COMMENT 'start parameters',
    `gmt_end`             DATETIME(3) COMMENT 'end time',
    `excep`               BLOB COMMENT 'exception',
    `end_params`          TEXT COMMENT 'end parameters',
    `status`              VARCHAR(2) COMMENT 'status(SU succeed|FA failed|UN unknown|SK skipped|RU running)',
    `compensation_status` VARCHAR(2) COMMENT 'compensation status(SU succeed|FA failed|UN unknown|SK skipped|RU running)',
    `is_running`          TINYINT(1) COMMENT 'is running(0 no|1 yes)',
    `gmt_updated`         DATETIME(3) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `unikey_buz_tenant` (`business_key`, `tenant_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `seata_state_inst`
(
    `id`                       VARCHAR(48)  NOT NULL COMMENT 'id',
    `machine_inst_id`          VARCHAR(128) NOT NULL COMMENT 'state machine instance id',
    `name`                     VARCHAR(128) NOT NULL COMMENT 'state name',
    `type`                     VARCHAR(20)  COMMENT 'state type',
    `service_name`             VARCHAR(128) COMMENT 'service name',
    `service_method`           VARCHAR(128) COMMENT 'method name',
    `service_type`             VARCHAR(16) COMMENT 'service type',
    `business_key`             VARCHAR(48) COMMENT 'business key',
    `state_id_compensated_for` VARCHAR(50) COMMENT 'state compensated for',
    `state_id_retried_for`     VARCHAR(50) COMMENT 'state retried for',
    `gmt_started`              DATETIME(3)  NOT NULL COMMENT 'start time',
    `is_for_update`            TINYINT(1) COMMENT 'is service for update',
    `input_params`             TEXT COMMENT 'input parameters',
    `output_params`            TEXT COMMENT 'output parameters',
    `status`                   VARCHAR(2)   NOT NULL COMMENT 'status(SU succeed|FA failed|UN unknown|SK skipped|RU running)',
    `excep`                    BLOB COMMENT 'exception',
    `gmt_updated`              DATETIME(3) COMMENT 'update time',
    `gmt_end`                  DATETIME(3) COMMENT 'end time',
    PRIMARY KEY (`id`, `machine_inst_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

## 部署TC
执行mysql脚本
```
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

```
如果使用源码编译的话，需要注释掉，或者idea导入插件protobuf support
```
<!--        <module>seata-serializer-protobuf</module>-->
```
修改配置``registry.conf``、``file.conf``，如果需要不同的环境区分不同的配置，新建文件``registry-dev.conf``、``file-dev.conf``，然后启动启动类``io.seata.server.Server``，启动的jvm参数加上变量``-DseataEnv=uat``，这样就可以实现多环境启动了。

到server目录下，打包部署服务器:
```
mvn clean package -Dmaven.test.skip=true -Prelease-seata
```
会生成制品，制品目录在``/distribution``，如果是部署服务器上面或者是通过docker镜像部署的话，可以参考``/distribution/Dockerfile``，或者直接通过下面的命令启动
```
sh ./bin/seata-server.sh -p 8091 -h 127.0.0.1 -n 1 -e uat
```
具体参数如下

参数 | 全写 | 作用 | 备注
---|---|---|---
-h |--host|指定在注册中心注册的 IP|不指定时获取当前的 IP，外部访问部署在云环境和容器中的 server 建议指定
-p | --port | 指定 server 启动的端口 | 默认为 8091
-m | --storeMode |事务日志存储方式 |支持file,db,redis，默认为 file 注:redis需seata-server 1.3版本及以上
-n| --serverNode |用于指定seata-server节点ID |如 1,2,3..., 默认为 1
-e|--seataEnv|指定 seata-server 运行环境|如 dev, test 等, 服务启动时会使用 registry-dev.conf 这样的配置


可以查看在我本地输出脚本的完整的jvm参数
```
/Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/bin/java -server -Xmx2048m -Xms2048m -Xmn1024m -Xss512k -XX:SurvivorRatio=10 -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m -XX:MaxDirectMemorySize=1024m -XX:-OmitStackTraceInFastThrow -XX:-UseAdaptiveSizePolicy -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2/logs/java_heapdump.hprof -XX:+DisableExplicitGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=75 -Xloggc:/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2/logs/seata_gc.log -verbose:gc -Dio.netty.leakDetectionLevel=advanced -Dlogback.color.disable-for-bat=true -classpath /Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2/conf:/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2/lib/* -Dapp.name=seata-server -Dapp.pid=23748 -Dapp.repo=/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2/lib -Dapp.home=/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2 -Dbasedir=/Users/dinghuang/Documents/workSpace/github/seata/distribution/seata-server-1.4.2 io.seata.server.Server -e uat
```
如果配置中心使用apollo，可以参考这篇文章：https://www.studying.icu/pages/94a254/#seata  （1.4.2有bug，apollo的配置不能用，建议用1.4.1）



# Spring框架使用
pom.xml引入包
```
<!-- https://mvnrepository.com/artifact/io.seata/seata-spring-boot-starter -->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>1.4.2</version>
</dependency>
```

修改application.yml配置文件（1.4.2有bug，apollo的配置不能用，建议用1.4.1）
```
seata:
  enabled: true
  application-id: applicationName
  tx-service-group: my_test_tx_group
  enable-auto-data-source-proxy: true
  data-source-proxy-mode: AT
  use-jdk-proxy: false
  excludes-for-auto-proxying: firstClassNameForExclude,secondClassNameForExclude
  client:
    rm:
      async-commit-buffer-limit: 10000
      report-retry-count: 5
      table-meta-check-enable: false
      report-success-enable: false
      saga-branch-register-enable: false
      saga-json-parser: fastjson
      saga-retry-persist-mode-update: false
      saga-compensate-persist-mode-update: false
      lock:
        retry-interval: 10
        retry-times: 30
        retry-policy-branch-rollback-on-conflict: true
    tm:
      commit-retry-count: 5
      rollback-retry-count: 5
      default-global-transaction-timeout: 60000
      degrade-check: false
      degrade-check-period: 2000
      degrade-check-allow-times: 10
    undo:
      data-validation: true
      log-serialization: jackson
      log-table: undo_log
      only-care-update-columns: true
      compress:
        enable: true
        type: zip
        threshold: 64k
    load-balance:
      type: RandomLoadBalance
      virtual-nodes: 10
  service:
    vgroup-mapping:
      my_test_tx_group: default
    grouplist:
      default: 127.0.0.1:8091
    enable-degrade: false
    disable-global-transaction: false
  transport:
    shutdown:
      wait: 3
    thread-factory:
      boss-thread-prefix: NettyBoss
      worker-thread-prefix: NettyServerNIOWorker
      server-executor-thread-prefix: NettyServerBizHandler
      share-boss-worker: false
      client-selector-thread-prefix: NettyClientSelector
      client-selector-thread-size: 1
      client-worker-thread-prefix: NettyClientWorkerThread
      worker-thread-size: default
      boss-thread-size: 1
    type: TCP
    server: NIO
    heartbeat: true
    serialization: seata
    compressor: none
    enable-client-batch-send-request: true
  config:
    type: file
    consul:
      server-addr: 127.0.0.1:8500
    apollo:
      apollo-meta: http://192.168.1.204:8801
      app-id: seata-server
      namespace: application
      apollo-accesskey-secret: ""
    etcd3:
      server-addr: http://localhost:2379
    nacos:
      namespace: ""
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      username: ""
      password: ""
    zk:
      server-addr: 127.0.0.1:2181
      session-timeout: 6000
      connect-timeout: 2000
      username: ""
      password: ""
    custom:
      name: ""
  registry:
    type: file
    file:
      name: file.conf
    consul:
      server-addr: 127.0.0.1:8500
      acl-token: ""
    etcd3:
      server-addr: http://localhost:2379
    eureka:
      weight: 1
      service-url: http://localhost:8761/eureka
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group : "SEATA_GROUP"
      namespace: ""
      username: ""
      password: ""
    redis:
      server-addr: localhost:6379
      db: 0
      password: ""
      timeout: 0
    sofa:
      server-addr: 127.0.0.1:9603
      region: DEFAULT_ZONE
      datacenter: DefaultDataCenter
      group: SEATA_GROUP
      address-wait-time: 3000
      application: default
    zk:
      server-addr: 127.0.0.1:2181
      session-timeout: 6000
      connect-timeout: 2000
      username: ""
      password: ""
    custom:
      name: ""
  log:
    exception-rate: 100
```


具体的例子可以看
https://github.com/seata/seata-samples