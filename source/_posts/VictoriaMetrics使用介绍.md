---
title: VictoriaMetrics 介绍
author: baixiaozhou
categories:
  - Technology
tags:
  - Programming
  - Hexo
description: A brief description of the content
repo: baixiaozhou/SysStress
leftbar: welcome
rightbar: statement
date: 2025-02-19 17:07:48
references:
cover:
banner:
---

# {{ title }}

This article was written by {{ author }} on {{ date }}.

<!-- Your content starts here -->

官方文档: https://docs.victoriametrics.com/

# 1. 简介
我们先看一下VictoriaMetrics 官网的介绍: VictoriaMetrics 是一个开源的、高可用的、分布式、时间序列数据库，用于存储和查询时间序列数据。
从简述来看，我们其实可以发现，VictoriaMetrics 其实和 Prometheus 有很多相似之处，那么我们为什么还要使用 VictoriaMetrics 呢？
在了解这些之前，我们先来了解一下 Prometheus 的一些特点:
1. 单实例，不可扩展: Prometheus 的作者及社区核心开发者都秉承一个理念：Prometheus 只聚焦核心的功能，扩展性的功能留给社区解决，所以 Prometheus 自诞生至今都是单实例不可扩展的。
2. 无法处理大规模数据: Prometheus 作为时序数据库，其核心功能就是存储和查询时序数据，但是 Prometheus 本身并没有提供分布式存储和查询的能力。默认情况下，prometheus 存储数据时长默认保留 15 天，超过 15 天的数据会被自动清理掉（可修改）。

在长期存储方案出现之前，用户如果需要跨集群聚合计算数据时，社区提供了Federation 方式。
2017 年，Prometheus 加⼊ Remote Read/Write API，自此之后社区涌现出大量长期存储的方案，如 Thanos、Grafana Cortex/Mimir、VictoriaMetrics、Wavefront、Splunk、Sysdig、SignalFx、InfluxDB、Graphite 等。 而我们这篇文章要介绍的 VictoriaMetrics 就是其中之一。

## 1.1 VictoriaMetrics 的特点

由于官网中关于 VictoriaMetrics 的特点描述比较多，这里我们主要展示一下核心特点或者日常使用中比较关键的一些点:
- 它可以用作 Prometheus 的长期存储。这里的长期存储指的就是**Prometheus 通过 remote_write url 将数据写入到 vm 中，以 vm作为数据存储的载体**
- 也可以用做 Prometheus 的直接替代品，因为vm 支持 prometheus 语法。
- 数据压缩率高，与 TimescaleDB 相比，可以在有限存储中存储多达 70 倍的数据点，与 Prometheus、Thanos 或 Cortex 相比，所需的存储空间最多可减少 7 倍。（官网提供的数据）
- 实现了一种类似 PromQL 的查询语言 - [MetricsQL，](https://docs.victoriametrics.com/metricsql/) 在 PromQL 的基础上提供了改进的功能
- 可以用作 Grafana 中 Graphite 的直接替代品，因为它支持 [Graphite API](https://docs.victoriametrics.com/#graphite-api-usage)

## 1.2 VictoriaMetrics 的架构和组成

从部署上来说，VictoriaMetrics 有两种部署方式，一种是单机部署，一种是集群部署。 在实际的线上环境中，还是以集群部署为主，因为集群部署可以提供更高的可用性，同时也可以提供更好的性能。因此我们这里可以看下集群部署的架构图。

VictoriaMetrics 的架构图如下所示:
![VictoriaMetrics 架构图](https://docs.victoriametrics.com/Cluster-VictoriaMetrics_cluster-scheme.webp)

集群的核心角色包括以下几个组件:

- vminsert: 接收提取的数据，并根据指标名称及其所有标签的一致哈希将其分布在 `VMStstorage` 节点之间
- vmstorage: 存储原始数据并返回给定标签筛选条件的给定时间范围内的查询数据
- vmselect： 通过从所有已配置的 `VMStlayer` 节点获取所需数据来执行传入查询

除了上面的这几个组件，VictoriaMetrics 还提供了其他的一些组件

- vmagent: 一个**可选的独立组件**，主要用于数据采集和转发
- vmalert: 用于执行给定[警报](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)的列表 或[录制](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/)针对配置的 `-datasource.url` 的规则
- vmauth: 是一个 HTTP 代理，可以[授权](https://docs.victoriametrics.com/vmauth/#authorization)、[路由](https://docs.victoriametrics.com/vmauth/#routing) 以及跨 [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics) 组件或任何其他 HTTP 后端的[负载均衡](https://docs.victoriametrics.com/vmauth/#load-balancing)请求。
- vmbackup/vmrestore: 数据备份/恢复
- vmctl: 数据迁移工具，用于从其他时序数据库（TSDB）导入数据到 VictoriaMetrics。
- vmgateway: 是 VictoriaMetrics 时间序列数据库 （TSDB） 的代理。
- vmbackupmanager: 备份管理器自动执行常规备份程序
- vmalert-tool:  是 **VictoriaMetrics 提供的工具**，用于测试、调试和管理 **vmalert 规则**，主要用于 **验证 Prometheus Alerting 规则** 以及 **预处理 VictoriaMetrics 的告警配置**。

# 2. 部署安装

## 2.1 下载

```shell
# 从 github 上下载
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.111.0/victoria-metrics-linux-arm64-v1.111.0-cluster.tar.gz
# 解压
tar -xzvf victoria-metrics-linux-arm64-v1.111.0-cluster.tar.gz
# 可以看到集群的包中包括三个文件
vminsert-prod  vmselect-prod  vmstorage-prod
```

一个最小化的集群必须包含以下节点:

- 一个带有`-storageDataPath` 标志的 `vmstorage` 节点
- 一个带有 `-storageNode=<vmstorage_host>` 参数的 `vminsert` 节点
- 一个带有 `-storageNode=<vmstorage_host>` 参数的 `vmselect` 节点

如果完全以`VictoriaMetrics`作为存储系统，此外还需要一个 `vmagent` 节点

```shell
# 从 github 上下载
wget https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.111.0/vmutils-linux-arm64-v1.111.0.tar.gz
# 解压
tar xzvf vmutils-linux-arm64-v1.111.0.tar.gz
# 包括其他的一些组件包
vmagent-prod vmalert-prod vmalert-tool-prod vmauth-prod vmbackup-prod vmrestore-prod vmctl-prod
```

## 2.2 部署

我们以单个节点完成`victoriametrics`的部署，将 `vmagent`、`vmstorage`、`vmselect`、`vminsert`的二进制文件放到`/usr/local/bin`目录下。

我们通过`systemd`的形式分别启动这四个程序。

### vmstorage 配置

```service
# /usr/lib/systemd/system/vmstorage.service
[Unit]
Description=vmstorage
Documentation=https://docs.victoriametrics.com/
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/vmstorage-prod \
  -retentionPeriod=1d \
  -storageDataPath=/root/victoriametrics/data
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

这里主要给服务添加了两个参数，分别是`retentionPeriod`和 `storageDataPath`参数，分别代表数据保留周期和数据存储路径。在服务启动后，我们可以一下服务的端口占用:

```shell
[root@localhost ~]# ss -anltp | grep vmstorage
LISTEN 0      4096         0.0.0.0:8482       0.0.0.0:*    users:(("vmstorage-prod",pid=2247,fd=11))
LISTEN 0      4096         0.0.0.0:8400       0.0.0.0:*    users:(("vmstorage-prod",pid=2247,fd=9))
LISTEN 0      4096         0.0.0.0:8401       0.0.0.0:*    users:(("vmstorage-prod",pid=2247,fd=10))
```

服务默认占用了三个端口，其中 8400 和 8401 端口分别是用于接受 `vminsert` 和 `vmselect`请求的端口，可以通过参数控制:

```
-vminsertAddr string
    	TCP address to accept connections from vminsert services (default ":8400")
-vmselectAddr string
    	TCP address to accept connections from vmselect services (default ":8401")
```

### vminsert 配置

```service
# /usr/lib/systemd/system/vminsert.service
[Unit]
Description=vminsert
Documentation=https://docs.victoriametrics.com/
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/vminsert-prod \
  -storageNode=192.168.141.129:8400
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

这里给服务只配置了一个参数:`storageNode`指定了存储节点。

这里引申出一个问题，假如我有多个sotrage 节点应该如何配置呢？数据又是如何存储的？

其实在上文中，我们介绍vminsert 组件的时候就已经提示过:`accepts the ingested data and spreads it among vmstorage nodes according to consistent hashing over metric name and all its labels`。也就是说当我们有多个 storage 节点的时候，数据会分布在多个节点上。那这样一来就会有个很明显的问题，一旦节点宕机异常等，就会导致数据丢失或者无法查询等情况，为此就需要有数据复制机制,`vminsert`提供了`-replicationFactor`参数，用于控制数据复制来保证可用性。

### vmselect配置

```service
# /usr/lib/systemd/system/vmselect.service
[Unit]
Description=vmselect
Documentation=https://docs.victoriametrics.com/
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/vmselect-prod \
  -storageNode=192.168.141.129:8401
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

这里也是指定了存储节点的 8401端口

### vmagent配置

```service
# /usr/lib/systemd/system/vmagent.service
[Unit]
Description=vmagent
Documentation=https://docs.victoriametrics.com/
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/vmagent-prod \
  -promscrape.config=/root/prometheus/prometheus/config/vm.yml \
  -remoteWrite.url=http://192.168.141.129:8480/insert/0/prometheus/
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

需要指定配置文件和写入的 url。这里的配置文件类似于`prometheus`的配置文件，我这里采集的的是`node_exporter`的指标:

```yaml
global:
  scrape_interval: 15s  # 每 15 秒采集一次
  scrape_timeout: 10s   # 超时时间 10 秒

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']  # 采集本地 node_exporter
```

### 2.3 启动

```shell
# systemctl daemon-reload
# systemctl start vmagent vmselect vminsert vmstorage
```

## 2.4 监控查看

http://192.168.141.129:8481/select/0/ 可以通过这样的方式访问 vmui 进行监控指标的查询
