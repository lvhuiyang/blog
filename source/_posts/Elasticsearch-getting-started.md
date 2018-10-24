---
title: Elasticsearch 01 - Getting Started
date: 2018-10-15 14:11:57
tags: [Elasticsearch, Translation]
---

学习并翻译 Elasticsearch 官方文档 [Installation and Upgrade Guide [6.4]](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)，并进行相关笔记。

# Getting Started

Elasticsearch 是一个高度可扩展的开源全文搜索和分析引擎。它允许你快速、近实时地存储、搜索和分析大量数据。它通常用作底层引擎或技术给复杂搜索功能和要求的应用程序提供支持。

<!-- more -->

以下是可使用 Elasticsearch 的一些示例用例：

+ 在线商城，提供商品和库存的查询。

+ 收集日志或交易数据，并进行相关的数据分析和挖掘，用以分析趋势，数据统计或捕获异常。这种情况下可以使用 ELK（Elasticsearch / Logstash / Kibana）技术栈进行实现。

+ 价格分析与提醒平台。

在本教程的其余部分中，将引导完成启动和运行 Elasticsearch ，并进行相关增删改查操作。在本教程结束时，你应该很好地了解 Elasticsearch 是什么，是如何工作的，并能够使用他来构建复杂的搜索应用或者是进行相关数据挖掘与分析。

## 1. 基本概念

### 近实时(NRT, Near Realtime)

Elasticsearch 是一个近乎实时的搜索平台。 这意味着从索引文档到可搜索文档的时间有一点延迟（通常是一秒）。

### 集群(Cluster)

集群是一个或多个节点（服务器）的集合，它们共同保存整个数据，并提供跨所有节点的联合索引和搜索功能。群集由唯一名称标识，默认情况下为“elasticsearch”。 确保不要在不同的环境中重用相同的群集名称，否则最终会导致节点加入错误的群集。例如，可以将命名为 logging-dev, logging-stage 和 logging-prod 用于开发、灰度和生产环境。

注意，如果群集中只有一个节点，那么它是完全 OK 的。 此外，您还可以拥有多个独立的集群，每个集群都有自己唯一的集群名称。

### 节点(Node)

节点是作为集群中的单个服务器，存储数据并参与群集的索引和搜索功能。就像集群一样，节点由名称标识，默认情况下，该名称是在启动时分配给节点的随机通用唯一标识符（UUID）。如果不需要默认值，可以定义所需的任何节点名称。此名称对于集群的管理非常重要，您可以在其中识别网络中哪些服务器与 Elasticsearch 集群中的哪些节点相对应。

### 索引(Index)

索引是具有某些类似特征的文档集合。 例如，你拥有客户数据的索引、产品目录的索引以及订单数据的索引。 索引由名称标识（必须全部小写），此名称用于在对其中的文档执行索引搜索、更新和删除操作时引用的操作。在单个群集中，你可以根据需要定义任意数量的索引。

### 类型(Type)

于 6.0.0 版本中弃用。

### 文档(Document)

文档是可以被索引的基本信息单元。例如，你可以为单个客户提供文档，为单个产品设置为一个文档，为单个订单设置为另一个文档，文档以 JSON 类型表示。

### 碎片 & 副本(Shards & Replicas)

## 2. 安装

Elasticsearch 需要运行在 Java 8或者更新版本的 Java 环境中，具体到写该篇文章是，建议您使用 Oracle JDK 1.8.0_131 版本。Java 的安装在此不做详细介绍，在安装 Elasticsearch 之前，请先运行检查 Java版本：

```bash
java -version
echo $JAVA_HOME
```

### tar 安装示例（适用于 Mac OS 和 Linux）

如下命令下载 Elasticsearch 6.4.2 的 tar 文件：

```bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.2.tar.gz
```

提取（解压）文件

```bash
tar -xvf elasticsearch-6.4.2.tar.gz
```

会在当前目录下创建一系列文件夹，然后进入 bin 目录

```bash
cd elasticsearch-6.4.2/bin
```

启动节点和单个集群：

```bash
./ elasticsearch
```

### 使用 MSI Windows Installer 的安装示例（适用于 Windows）

略

### 成功运行节点

如果安装顺利，应该能看到如下所示的消息：

```bash
[2018-10-24T19:37:26,684][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [aggs-matrix-stats]
[2018-10-24T19:37:26,685][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [analysis-common]
[2018-10-24T19:37:26,685][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [ingest-common]
[2018-10-24T19:37:26,686][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [lang-expression]
[2018-10-24T19:37:26,686][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [lang-mustache]
[2018-10-24T19:37:26,686][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [lang-painless]
[2018-10-24T19:37:26,687][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [mapper-extras]
[2018-10-24T19:37:26,687][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [parent-join]
[2018-10-24T19:37:26,687][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [percolator]
[2018-10-24T19:37:26,687][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [rank-eval]
[2018-10-24T19:37:26,688][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [reindex]
[2018-10-24T19:37:26,688][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [repository-url]
[2018-10-24T19:37:26,688][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [transport-netty4]
[2018-10-24T19:37:26,689][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [tribe]
[2018-10-24T19:37:26,689][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-core]
[2018-10-24T19:37:26,689][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-deprecation]
[2018-10-24T19:37:26,690][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-graph]
[2018-10-24T19:37:26,690][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-logstash]
[2018-10-24T19:37:26,690][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-ml]
[2018-10-24T19:37:26,691][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-monitoring]
[2018-10-24T19:37:26,691][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-rollup]
[2018-10-24T19:37:26,691][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-security]
[2018-10-24T19:37:26,691][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-sql]
[2018-10-24T19:37:26,692][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-upgrade]
[2018-10-24T19:37:26,692][INFO ][o.e.p.PluginsService     ] [6A3cdtb] loaded module [x-pack-watcher]
```

在不详细讨论的情况下，我们可以看到名为 “6A3cdtb” 的节点（可能将是一组不同的字符）已经启动并选择自己作为单个集群中的主节点。当前不需要担心它们的具体含义，重点是知道当前启动了集群的一个节点就 OK。

如前所述，我们可以覆盖集群或节点名称。这可以在启动 Elasticsearch 时从命令行完成，如下所示：

```bash
./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

注意标有 http 的行，其中包含有关可从中访问节点的 HTTP 地址（192.168.8.112）和端口（9200）的信息。默认情况下，Elasticsearch 使用 9200 端口（可配置）来提供对其 REST API 的访问。