---
title: "开源数据集成平台SeaTunnel：MySQL实时同步到es"
date: 2023-07-04T23:05:22+08:00
draft: false
description: "免费支持 MySQL 实时同步到 ElasticSearch 的工具很少，Apache SeaTunnel 是一个高性能开源大数据集成工具，提供灵活易用、易扩展并支持千亿级数据集成的解决方案，已经在B站、腾讯云、字节等数百家公司使用。"

tags: ["Java","seatunnel","同步"]
categories: ["Java","seatunnel"]

lightgallery: true

repost:
  enable: true
  url: "https://www.890808.xyz/data-sync-apache-seatunnel/"
---

<!--more-->

## 一、前言
- 最近，项目有几个表要从 MySQL 实时同步到 另一个 MySQL，也有同步到 ElasticSearch 的。
- 目前，公司生产环境同步，用的是 阿里云的 DTS，每个同步任务每月 500多元，有点小贵。
- 其他环境：MySQL同步到ES，用的是 CloudCanal，不支持 数据转换，添加同步字段比较麻烦，社区版限制5个任务，不够用；MySQL同步到MySQL，用的是 debezium，不支持写入 ES。
- 恰好3年前用过 SeaTunnel 的 前身 WaterDrop，那就开始吧。本文以 2.3.1 版本，Ubuntu 系统为例

## 二、[开源数据集成平台SeaTunnel](https://github.com/apache/seatunnel)
### 1. [简介](https://seatunnel.apache.org/docs/2.3.1/about)   
- SeaTunnel 是 Apache 软件基金会下的一个高性能开源大数据集成工具，为数据集成场景提供灵活易用、易扩展并支持千亿级数据集成的解决方案。
- Seaunnel 为实时(CDC)和批量数据提供高性能数据同步能力，[支持十种以上数据源](https://seatunnel.apache.org/docs/2.3.1/Connector-v2-release-state)，已经在B站、腾讯云、字节等数百家公司使用。 
- 可以选择 SeaTunnel Zeta 引擎上运行，也可以在 Apache Flink 或 Spark 引擎上运行。   
![seatunnel-architecture.png](https://img.890808.xyz/file/javalover123/2023/07/seatunnel-architecture.png)


### 2. [安装](https://seatunnel.apache.org/docs/2.3.1/start-v2/locally/deployment#step-1-prepare-the-environment)
- [下载，这里选择 2.3.1 版本](https://seatunnel.apache.org/download/)，执行 tar -xzvf apache-seatunnel-*.tar.gz 解压缩  
- [因为 2.3.2 版本，MySQL-CDC 找不到驱动](https://github.com/apache/seatunnel/issues/4959)，[bug修复详见](https://github.com/apache/seatunnel/pull/4945/files)   
```
Caused by: java.sql.SQLException: No suitable driver
        at java.sql/java.sql.DriverManager.getDriver(DriverManager.java:298)
        at com.zaxxer.hikari.util.DriverDataSource.<init>(DriverDataSource.java:106)
        ... 20 more

        ... 11 more

        at org.apache.seatunnel.engine.client.job.ClientJobProxy.waitForJobComplete(ClientJobProxy.java:122)
        at org.apache.seatunnel.core.starter.seatunnel.command.ClientExecuteCommand.execute(ClientExecuteCommand.java:181)
```

### 3. [安装 connectors 插件](https://seatunnel.apache.org/docs/2.3.1/start-v2/locally/deployment#step-3-install-connectors-plugin)
- ***执行 bash bin/install-plugin.sh，国内建议先配置 `maven` 镜像，不然容易失败 或者 慢***
- 官方文档写着执行 sh bin/install-plugin.sh，我在 Ubuntu 20.04.2 LTS 上执行报错(bin/install-plugin.sh: 54: Bad substitution)，[我提了PR](https://github.com/apache/seatunnel-website/pull/253)   
![seatunnel-install-connectors-error.png](https://img.890808.xyz/file/javalover123/2023/07/seatunnel-install-connectors-error.png)


### 4. 编写配置文件
- config 目录下，新建配置文件：如 mysql-es-test.conf
- [添加 env 配置](https://seatunnel.apache.org/docs/2.3.1/start-v2/locally/quick-start-seatunnel-engine#step-2-add-job-config-file-to-define-a-job)
***因为是 实时同步，这里 job.mode = "STREAMING"***，execution.parallelism 是 并发数   
```
env {
  # You can set flink configuration here
  execution.parallelism = 1
  job.mode = "STREAMING"
  checkpoint.interval = 2000
  #execution.checkpoint.interval = 10000
  #execution.checkpoint.data-uri = "hdfs://localhost:9000/checkpoint"
}
```

- [MySQL 实时同步，需开启 binlog](https://debezium.io/documentation/reference/1.6/connectors/mysql.html#setting-up-mysql)
- [添加 数据源 配置](https://seatunnel.apache.org/docs/2.3.1/connector-v2/source/MySQL-CDC#options)
result_table_name 取个 临时表名，便于后续使用。***table-names 必须是 数据库.表名，base-url 必须指定 数据库。***   
[startup.mode 默认是 INITIAL，先同步历史数据，后增量同步，详情点击](https://github.com/apache/seatunnel/blob/3cd51b6defd3ddd3b011cf0f6b48f3c209bf9d22/seatunnel-connectors-v2/connector-cdc/connector-cdc-base/src/main/java/org/apache/seatunnel/connectors/cdc/base/option/StartupMode.java#L27)   
```
source {
  MySQL-CDC {
    result_table_name = "t1"
    server-id = 5656
    username = "root"
    password = "pwd"
    table-names = ["db.t1"]
    base-url = "jdbc:mysql://host:3306/db"
  }
}
```

- [添加 转换 配置，sql 比较灵活](https://seatunnel.apache.org/docs/2.3.1/transform-v2/sql#options)。
[函数列表请点击](https://seatunnel.apache.org/docs/2.3.1/transform-v2/sql-functions)   
```
transform {
  Sql {
    source_table_name = "t1"
    query = "SELECT id, alias_name aliasName FROM t1 WHERE c1 = '1'"
  }
}
```

- [添加 输出 配置](https://seatunnel.apache.org/docs/2.3.1/connector-v2/sink/Elasticsearch#options)
***CDC 实时同步 es，必须配置 primary_keys***   
```
sink {
    Elasticsearch {
        hosts = ["host:9200"]
        username = "elastic"
        password = "pwd"
        index = "index_t1"
        # cdc required options
        primary_keys = ["id"]
    }
}
```

- 最终配置截图   
![seatunnel-mysql-cdc-es.png](https://img.890808.xyz/file/javalover123/2023/07/seatunnel-mysql-cdc-es.png)


### 5. [启动任务](https://seatunnel.apache.org/docs/2.3.1/start-v2/locally/quick-start-seatunnel-engine#step-3-run-seatunnel-application)   
[这里以 本地模式为例](https://seatunnel.apache.org/docs/2.3.1/seatunnel-engine/local-mode)，另有 [集群](https://seatunnel.apache.org/docs/2.3.1/seatunnel-engine/deployment)、spark、flink 模式。   
```shell
./bin/seatunnel.sh -e local --config ./config/mysql-es-test.conf
```

## 三、总结
- [开源数据集成平台SeaTunnel](https://github.com/apache/seatunnel) 能够比较方便的进行 MySQL 实时同步到 es 等，免费，还方便添加 同步字段。[更多强大功能，请看官方文档。](https://seatunnel.apache.org/docs/2.3.1/about)
- [新版本自带 同步引擎](https://seatunnel.apache.org/docs/2.3.1/seatunnel-engine/about)，不用依赖 spark、flink 等运行，降低了 小数据量同步场景 部署复杂度
- 新版本开始提供 [UI界面](https://github.com/apache/seatunnel-web)，目前强依赖 调度平台 Apache DolphinScheduler

**本文遵守[【CC BY-NC】协议，转载请保留原文出处及本版权声明，否则将追究法律责任。](https://creativecommons.org/licenses/by-nc/4.0/)**   
***本文首先发布于 [https://www.890808.xyz/](https://www.890808.xyz/) ，其他平台需要审核更新慢一些。***   

![javalover123](https://img.890808.xyz/file/javalover123/2023/04/688b88cfd4ed9f6fcd56828b849ce47c.jpg)