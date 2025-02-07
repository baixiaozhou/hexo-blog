---
title: Prometheus使用介绍
author: baixiaozhou
categories:
  - Prometheus
  - 监控方案
tags:
  - Programming
  - Hexo
description: A brief description of the content
repo: https://github.com/prometheus/prometheus
date: 2024-12-18 16:11:55
references:
cover:
banner:
---

```shell
# 安装请参考官网: https://prometheus.io/docs/prometheus/latest/installation/
# 下载地址: https://prometheus.io/download/
# github地址: https://github.com/prometheus/prometheus
```

# 概述
Prometheus 是一个开源的监控系统和告警工具包，用于收集和存储监控数据。它提供了强大的查询语言 PromQL，用于分析和可视化监控数据。Prometheus 可以与多种数据源集成，包括 Prometheus 自身、外部服务、数据库等。

下图是官方提供的架构图:
![Prometheus architecture](https://prometheus.ac.cn/assets/architecture.png)

Prometheus 由多个组件组成，包括:
- Prometheus Server: 负责收集和存储监控数据，提供查询和聚合功能。
- Alertmanager: 负责处理告警，包括分组、静默、抑制、静默规则等。
- Pushgateway: 用于临时存储和转发监控数据的中间网关。
- Exporter: 用于收集和导出监控数据的客户端。
- Service discovery: 用于自动发现和配置监控数据的客户端。
- Prometheus web UI: 用于查询和可视化监控数据的 Web 界面。

## 适用场景

Prometheus 非常适合记录任何纯粹的数字时间序列。它既适合以机器为中心的监控，也适合监控高度动态的面向服务的体系结构。在微服务的世界中，它对多维数据收集和查询的支持是一个特别的优势。

Prometheus 旨在实现可靠性，成为您在出现故障时可以求助的系统，以便您可以快速诊断问题。每个 Prometheus 服务器都是独立的，不依赖于网络存储或其他远程服务。当您的基础设施的其他部分出现故障时，您可以依靠它，并且您不需要设置大量基础设施来使用它。

## 不适用场景

Prometheus 重视可靠性。您始终可以查看有关系统的可用统计信息，即使在故障情况下也是如此。如果您需要 100% 的准确性，例如针对每个请求的计费，那么 Prometheus 不是一个好的选择，因为收集的数据可能不够详细和完整。在这种情况下，最好使用其他系统来收集和分析计费数据，并将 Prometheus 用于其他监控需求。

# 安装

## 解压安装包

```bash
[root@192 ~]# tar xzvf prometheus-3.0.1.linux-arm64.tar.gz
[root@192 ~]# ls prometheus-3.0.1.linux-arm64
LICENSE  NOTICE  prometheus  prometheus.yml  promtool
```

可以看到其中主要包含三个文件:

- promethues:  监控服务端
- prometool:  监控工具
- prometheus.yml:  配置文件

## 启动参数

prometheus 启动参数，可以通过`prometheus -h`进行查看,支持参数的列表包括:

| **参数**                                                     | **描述**                                                                                                                                                                 | **默认值**         | **示例**                                                                                      |
|------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|-----------------------------------------------------------------------------------------------|
| `--config.file="prometheus.yml"`                            | 配置 Prometheus 配置文件的路径。                                                                                                                                         | 无默认值            | `--config.file="prometheus.yml"`                                                               |
| `--config.auto-reload-interval=30s`                         | 指定 Prometheus 检测配置文件变化并自动重载的间隔时间。                                                                                                                 | 无默认值            | `--config.auto-reload-interval=30s`                                                            |
| `--web.listen-address=0.0.0.0:9090`                         | 配置 Prometheus 用于 UI、API 和遥测的监听地址。                                                                                                                          | 无默认值            | `--web.listen-address=0.0.0.0:9090`                                                           |
| `--[no-]auto-gomaxprocs`                                    | 自动设置 GOMAXPROCS 以匹配 Linux 容器的 CPU 配额。                                                                                                                     | 默认开启            | `--auto-gomaxprocs`                                                                            |
| `--[no-]auto-gomemlimit`                                    | 自动设置 GOMEMLIMIT 以匹配 Linux 容器或系统内存限制。                                                                                                                 | 默认开启            | `--auto-gomemlimit`                                                                            |
| `--auto-gomemlimit.ratio=0.9`                               | 容器或系统内存的最大可用比例，决定内存限制的比例。                                                                                                                     | `0.9`               | `--auto-gomemlimit.ratio=0.8`                                                                  |
| `--web.config.file=""`                                      | [实验性] 配置文件的路径，可以启用 TLS 或认证。                                                                                                                           | 无默认值            | `--web.config.file="web_config.yml"`                                                           |
| `--web.read-timeout=5m`                                     | 配置读取请求的最大超时时间，并关闭空闲连接。                                                                                                                             | `5m`                | `--web.read-timeout=10m`                                                                       |
| `--web.max-connections=512`                                 | 配置所有监听器的最大并发连接数。                                                                                                                                         | `512`               | `--web.max-connections=1024`                                                                   |
| `--web.max-notifications-subscribers=16`                    | 限制最大并发接收实时通知的订阅者数量。如果达到限制，新的订阅请求将被拒绝，直到现有连接关闭。                                                                                 | `16`                | `--web.max-notifications-subscribers=32`                                                      |
| `--web.external-url=<URL>`                                  | 配置 Prometheus 在外部可访问的 URL 地址。例如，如果通过反向代理提供 Prometheus 服务时使用。该 URL 用于生成相对或绝对链接。                                               | 无默认值            | `--web.external-url="https://prometheus.example.com"`                                        |
| `--web.route-prefix=<path>`                                 | 配置 Prometheus 内部路由的路径前缀。默认使用 `--web.external-url` 的路径部分。                                                                                           | 无默认值            | `--web.route-prefix="/prometheus"`                                                            |
| `--web.user-assets=<path>`                                  | 配置静态资产目录路径，访问路径为 `/user`。                                                                                                                                | 无默认值            | `--web.user-assets="/path/to/assets"`                                                         |
| `--[no-]web.enable-lifecycle`                               | 启用通过 HTTP 请求来进行 Prometheus 的关闭和重载功能。                                                                                                                 | 默认开启            | `--web.enable-lifecycle`                                                                      |
| `--[no-]web.enable-admin-api`                               | 启用用于管理操作的 API 端点。                                                                                                                                             | 默认开启            | `--web.enable-admin-api`                                                                      |
| `--[no-]web.enable-remote-write-receiver`                    | 启用接收远程写入请求的 API 端点。                                                                                                                                         | 默认开启            | `--web.enable-remote-write-receiver`                                                           |
| `--web.remote-write-receiver.accepted-protobuf-messages=prometheus.WriteRequest...` | 配置接受的远程写入的 Protobuf 消息类型。                                                                                                                                  | 默认开启            | `--web.remote-write-receiver.accepted-protobuf-messages=prometheus.WriteRequest`              |
| `--[no-]web.enable-otlp-receiver`                            | 启用接收 OTLP 写入请求的 API 端点。                                                                                                                                       | 默认关闭            | `--web.enable-otlp-receiver`                                                                  |
| `--web.console.templates="consoles"`                         | 配置控制台模板目录路径，访问路径为 `/consoles`。                                                                                                                           | 默认值为 `consoles` | `--web.console.templates="/path/to/templates"`                                               |
| `--web.console.libraries="console_libraries"`               | 配置控制台库目录路径。                                                                                                                                                    | 默认值为 `console_libraries` | `--web.console.libraries="/path/to/libraries"`                                           |
| `--web.page-title="Prometheus Time Series Collection and Processing Server"` | 配置 Prometheus 实例的文档标题。                                                                                                                                             | 默认值              | `--web.page-title="My Prometheus Server"`                                                     |
| `--web.cors.origin=".*"`                                     | 配置 CORS 的源匹配规则，完全匹配的正则表达式。                                                                                                                               | `.*`                | `--web.cors.origin="https://example.com"`                                                     |
| `--storage.tsdb.path="data/"`                               | 配置 Prometheus 存储指标的基本路径。仅在服务器模式下使用。                                                                                                              | `data/`             | `--storage.tsdb.path="/var/lib/prometheus/data"`                                             |
| `--storage.tsdb.retention.time=STORAGE.TSDB.RETENTION.TIME`  | 配置指标存储的保留时间。                                                                                                                                                   | 默认为 `15d`        | `--storage.tsdb.retention.time="30d"`                                                        |
| `--storage.tsdb.retention.size=STORAGE.TSDB.RETENTION.SIZE`  | 配置指标存储的最大大小。                                                                                                                                                  | 无默认值            | `--storage.tsdb.retention.size="500GB"`                                                      |
| `--[no-]storage.tsdb.no-lockfile`                            | 禁止在数据目录中创建锁文件。                                                                                                                                               | 默认关闭            | `--storage.tsdb.no-lockfile`                                                                  |
| `--storage.tsdb.head-chunks-write-queue-size=0`              | 配置写入磁盘的队列大小，用于块的写入操作。实验性功能。                                                                                                                     | `0`                 | `--storage.tsdb.head-chunks-write-queue-size=1024`                                           |
| `--storage.agent.path="data-agent/"`                         | 配置代理模式下指标存储的基本路径。                                                                                                                                         | `data-agent/`       | `--storage.agent.path="/path/to/agent/data"`                                                 |
| `--[no-]storage.agent.wal-compression`                       | 启用压缩代理 WAL（Write-Ahead Log）。                                                                                                                                      | 默认关闭            | `--storage.agent.wal-compression`                                                            |
| `--storage.agent.retention.min-time=STORAGE.AGENT.RETENTION.MIN-TIME` | 配置代理模式下指标数据的最小保留时间。                                                                                                                                     | 无默认值            | `--storage.agent.retention.min-time="1h"`                                                    |
| `--storage.agent.retention.max-time=STORAGE.AGENT.RETENTION.MAX-TIME` | 配置代理模式下指标数据的最大保留时间。                                                                                                                                     | 无默认值            | `--storage.agent.retention.max-time="24h"`                                                   |
| `--[no-]storage.agent.no-lockfile`                           | 禁止在代理模式下创建数据目录中的锁文件。                                                                                                                                   | 默认关闭            | `--storage.agent.no-lockfile`                                                                |
| `--storage.remote.flush-deadline=<duration>`                 | 配置在关闭或配置重载时等待刷新样本的最大时限。                                                                                                                              | 无默认值            | `--storage.remote.flush-deadline="2m"`                                                       |
| `--storage.remote.read-sample-limit=5e7`                     | 配置远程读取接口中每个查询能返回的最大样本数。                                                                                                                                 | `5e7`               | `--storage.remote.read-sample-limit=1000000`                                                 |
| `--storage.remote.read-concurrent-limit=10`                  | 配置并发远程读取调用的最大数量。                                                                                                                                           | `10`                | `--storage.remote.read-concurrent-limit=20`                                                  |
| `--storage.remote.read-max-bytes-in-frame=1048576`           | 配置每个远程读取响应中每帧的最大字节数。                                                                                                                                   | `1048576`           | `--storage.remote.read-max-bytes-in-frame=2048000`                                           |
| `--rules.alert.for-outage-tolerance=1h`                      | 配置 Prometheus 停机恢复期间容忍的最大时间。                                                                                                                                  |


## 配置文件参数

我们可以看一下配置文件默认的内容:
```yaml
# my global config 
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["localhost:9090"]
```

在配置文件中我们可以看到有多个配置块: global、alerting、rule_files、scrape_configs。 除此之外，还支持其他配置块信息，示例如下:

### global 全局配置

在 `global` 配置块中，我们可以定义全局的配置参数，这些参数将应用于所有的抓取任务。以下是一些常用的全局配置参数：
| **参数**                     | **描述**                                                                                                 | **默认值**           | **示例**                                |
|------------------------------|----------------------------------------------------------------------------------------------------------|----------------------|-----------------------------------------|
| **`scrape_interval`**          | 定义 Prometheus 每次抓取目标的间隔时间。                                                                   | `1m`                 | `15s`（每 15 秒抓取一次）               |
| **`scrape_timeout`**           | 定义每次抓取任务的最大超时时间。如果在这个时间内没有得到响应，则认为抓取失败。                           | `10s`                | `30s`（每个抓取任务最多等待 30 秒）     |
| **`scrape_protocols`**         | 配置抓取时与目标通信的协议。支持以下协议：`PrometheusProto`、`OpenMetricsText0.0.1`、`OpenMetricsText1.0.0`、`PrometheusText0.0.4`。 | 默认值为 `[OpenMetricsText1.0.0, OpenMetricsText0.0.1, PrometheusText0.0.4]` | `PrometheusProto, OpenMetricsText1.0.0` |
| **`evaluation_interval`**      | 定义 Prometheus 用来评估告警规则的频率。                                                                  | `1m`                 | `30s`（每 30 秒评估一次规则）           |
| **`rule_query_offset`**        | 偏移告警规则评估的时间戳，指定一个延迟时间，确保基础指标在评估前已被抓取。如果启用远程写入或抓取有延迟，可能需要设置此项。 | `0s`                 | `10s`（向后延迟 10 秒评估规则）         |
| **`external_labels`**          | 向 Prometheus 中的时间序列和告警添加标签，这些标签会在与外部系统（如 Alertmanager、远程存储、联邦等）通信时附加。 | 无默认值               | `monitor: 'main-monitor'`              |
| **`query_log_file`**           | 配置文件中指定的 PromQL 查询日志文件的路径，用于记录所有 PromQL 查询。重新加载配置文件时会重新打开该文件。  | 无默认值               | `/var/log/prometheus/query.log`         |
| **`body_size_limit`**          | 抓取请求的响应体大小限制。如果响应体超过此大小，则抓取会失败。0 表示没有限制。                           | `0`（无大小限制）     | `100MB`（限制为 100MB）                 |
| **`sample_limit`**             | 每次抓取中允许的最大样本数。如果在指标重标签后样本数超过此值，则该抓取任务会被视为失败。 0 表示无限制。     | `0`（无限制）        | `10000`（最多 10000 个样本）           |
| **`label_limit`**              | 每个抓取任务允许的最大标签数量。如果标签数量超过此值，抓取会失败。 0 表示没有限制。                        | `0`（无限制）        | `1000`（最多 1000 个标签）             |
| **`label_name_length_limit`**  | 每个标签名称的最大长度。如果标签名称超过此值，抓取会失败。0 表示没有限制。                             | `0`（无限制）        | `50`（标签名称最多 50 个字符）         |
| **`label_value_length_limit`** | 每个标签值的最大长度。如果标签值超过此值，抓取会失败。0 表示没有限制。                                 | `0`（无限制）        | `200`（标签值最多 200 个字符）        |
| **`target_limit`**             | 每个抓取任务允许的目标数量的最大值。如果目标数量超过此限制，Prometheus 会标记这些目标为失败并不进行抓取。 0 表示没有限制。 | `0`（无限制）        | `100`（最多 100 个目标）               |
| **`keep_dropped_targets`**     | 限制在目标重标签过程中被丢弃的目标数量，控制被丢弃的目标在内存中的保存数量。0 表示没有限制。              | `0`（无限制）        | `50`（最多保留 50 个被丢弃的目标）     |

### alerting 告警配置项

| **参数**                    | **描述**                                                     | **默认值** | **示例**                                                     |
| --------------------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`alert_relabel_configs`** | 用于定义告警标签重写规则。告警标签重写允许你修改告警的标签，以适应 Alertmanager 或其他系统的要求。 | 无默认值   | `- source_labels: [alertname]`<br>`target_label: severity`   |
| **`alertmanagers`**         | 定义 Alertmanager 配置，用于指定 Prometheus 向哪些 Alertmanager 实例发送告警。 | 无默认值   | `- static_configs:`<br>`    - targets: ['alertmanager:9093']` |

### rule_files 告警规则文件配置项

| **参数**         | **描述**                                                     | **默认值** | **示例**                                         |
| ---------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------ |
| **`rule_files`** | 定义一个包含告警规则的文件路径列表。所有匹配的文件将被读取并加载规则和告警。 | 无默认值   | `- 'alert_rules.yml'`<br>`- 'other_rules/*.yml'` |

### scrape_config_files 指标配置文件

| **参数**                  | **描述**                                                     | **默认值** | **示例**                                                     |
| ------------------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`scrape_config_files`** | 指定一个或多个文件路径或路径模式，用于加载抓取配置文件。所有匹配的文件将被读取并应用到抓取配置。 | 无默认值   | `- 'scrape_configs/*.yml'`<br>`- 'custom_scrape_config.yml'` |

### scrape_configs 指标配置

| **参数**             | **描述**                                                     | **默认值** | **示例**                                                     |
| -------------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`scrape_configs`** | 定义一个或多个抓取配置项，用于设置 Prometheus 如何抓取目标。支持多个抓取配置。 | 无默认值   | `- job_name: 'node_exporter'`<br>`  static_configs: [ { targets: ['localhost:9100'] } ]` |

### remote_write 远程写入

| **参数**           | **描述**                                                     | **默认值** | **示例**                                                     |
| ------------------ | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`remote_write`** | 配置远程写入功能，将数据写入远程存储系统。可以配置多个远程写入目标。 | 无默认值   | `- url: 'http://remote-storage/receive'`<br>`  write_relabel_configs: [...]` |

### remote_read 远程读取

| **参数**          | **描述**                                                     | **默认值** | **示例**                                                     |
| ----------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`remote_read`** | 配置远程读取功能，从远程存储系统读取数据。可以配置多个远程读取目标。 | 无默认值   | `- url: 'http://remote-storage/read'`<br>`  read_relabel_configs: [...]` |

### storage 存储

| **参数**        | **描述**                                                     | **默认值** | **示例**                                                     |
| --------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`tsdb`**      | 配置时间序列数据库（TSDB）的设置，用于存储 Prometheus 采集的数据。 | 无默认值   | `max_samples_per_tsdb_block: 1000000`<br>`  retention: '30d'` |
| **`exemplars`** | 配置用于存储示例数据的相关设置。示例数据用于增强高卡诺图的分析，提供对数据点的详细追踪。 | 无默认值   | `enable: true`<br>`  retention: '7d'`                        |

### tracing 跟踪

| **参数**      | **描述**                                                     | **默认值** | **示例**                                                     |
| ------------- | ------------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| **`tracing`** | 配置与追踪系统集成的设置。用于将追踪数据导出到外部追踪系统（例如 Jaeger 或 Zipkin）。 | 无默认值   | `- provider: 'jaeger'`<br>`  config:`<br>`    endpoint: 'http://jaeger:5775'` |

## 启动

```bash
[root@192 ~]# ./prometheus --config.file prometheus.yml
```

也可以通过配置 `systemd service` 文件进行启动

Start.sh

```shell
#!/bin/bash

set -e

ROOT=$(unset CDPATH && cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
cd $ROOT

exec $ROOT/prometheus \
    --config.file "prometheus.yml" \
    --web.listen-address "0.0.0.0:9080" \
    --web.enable-lifecycle \
    --web.enable-admin-api \
    --query.max-concurrency "1000" \
    --query.timeout "2m" \
    --storage.tsdb.retention.time "12h" \
    --storage.tsdb.retention.size "1GB" \
    --storage.tsdb.path "/root/prometheus/prometheus/data"
```

```shell
# /usr/lib/systemd/system/prometheus.service
[Unit]
Description=Prometheus Monitoring System
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/root/prometheus/prometheus/start.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

# 接入exporter

exporter 是一个收集指标的程序，Prometheus 服务器通过抓取这些指标数据来监控目标系统。Prometheus 提供了丰富的 exporter，用于监控各种系统和服务。例如，用于监控 Linux 系统的 `node_exporter`、用于监控 MySQL 数据库的 `mysqld_exporter`、用于监控 Redis 数据库的 `redis_exporter` 等。此外，用户还可以编写自己的 exporter 来监控其他系统和服务。

常见 exporter 如下:
- [node_exporter](https://github.com/prometheus/node_exporter)：用于监控 Linux 系统。
- [mysqld_exporter](https://github.com/prometheus/mysqld_exporter)：用于监控 MySQL 数据库。
- [redis_exporter](https://github.com/oliver006/redis_exporter)：用于监控 Redis 数据库。
- [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)：用于监控网络服务。
- [nginx_exporter](https://github.com/nginxinc/nginx-prometheus-exporter)：用于监控 Nginx 服务器。
- [process_exporter](https://github.com/ncabatoff/process-exporter): 监控进程服务
- [nginxlog_exporter](https://github.com/martin-helmich/prometheus-nginxlog-exporter): 监控 nginx 日志
- ......

我在本地启动了`node_exporter`、`process_exporter`、`nginxlog_exporter`，并将其配置到`prometheus.yml`中。`node_exporter`用于监控本地主机，`process_exporter`用于监控本地进程，`nginxlog_exporter`用于监控本地 nginx 日志。`prometheus.yml`配置如下：
```yaml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: "process_exporter"
    static_configs:
      - targets: ["localhost:9256"]

  - job_name: "nginxlog_exporter"
    static_configs:
      - targets: ["localhost:9302"]
```

指标接入后，我们就可以查看采集到的指标，当然也可以在通过对应端口查看`/metrics`信息。

## 监控 docker

如果我们想监控 docker 容器中的指标，我们可以使用 `docker stats` 命令来查看容器的资源使用情况。例如，我们可以使用以下命令来查看所有正在运行的容器的资源使用情况：
```bash
[root@192 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT    MEM %     NET I/O           BLOCK I/O         PIDS
5e48f91da9cd   kibana    0.60%     281MiB / 1.42GiB     19.33%    1.95GB / 3.47GB   5.04GB / 1.06GB   19
186c160bd03e   es        0.44%     401.8MiB / 1.42GiB   27.64%    3.5GB / 2.02GB    6.63GB / 43.6GB   93
```

Docker 容器的监控方案有很多，除了 Docker 自带的docker stats命令，还有很多开源的解决方案，例如 sysdig、cAdvisor。

cAdvisor 是谷歌开源的一款通用的容器监控解决方案。cAdvisor 不仅可以采集机器上所有运行的容器信息，还提供了基础的查询界面和 HTTP 接口，更方便与外部系统结合。所以，cAdvisor很快成了容器指标监控最常用组件，并且 Kubernetes 也集成了 cAdvisor 作为容器监控指标的默认工具。我们这里通过 cAdvisor 来监控 docker 容器。

安装 cAdvisor
```bash
$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/gcr.io/cadvisor/cadvisor-arm64:v0.49.1-linuxarm64
  
```
由于我个人环境是 arm 版本的，并且原始的源太慢，所以使用华为云的镜像。
安装完成后，可以通过 `http://localhost:8080` 访问 cAdvisor 的监控界面



# 查询

## 查询语法

Prometheus 的查询语法基于 [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/)，PromQL 是一种用于在 Prometheus 中查询时间序列数据的查询语言。

promql 指标和语法请参考:`https://www.prometheus.wang/promql/`

## 内置函数

部分内置函数:
| **函数**                        | **说明**                                                                                     |
|----------------------------------|----------------------------------------------------------------------------------------------|
| `avg()`                          | 计算一组时间序列的平均值。                                                                    |
| `avg_over_time()`                | 计算指定时间范围内时间序列的平均值。例如，`avg_over_time(node_cpu_seconds_total[1h])` 计算过去一小时内的平均值。 |
| `max()`                          | 计算一组时间序列的最大值。                                                                    |
| `max_over_time()`                | 计算指定时间范围内时间序列的最大值。                                                          |
| `min()`                          | 计算一组时间序列的最小值。                                                                    |
| `min_over_time()`                | 计算指定时间范围内时间序列的最小值。                                                          |
| `sum()`                          | 计算一组时间序列的总和。                                                                      |
| `sum_over_time()`                | 计算指定时间范围内时间序列的总和。                                                            |
| `count()`                        | 计算一组时间序列的数量。                                                                      |
| `count_over_time()`              | 计算指定时间范围内时间序列的数量。                                                            |
| `rate()`                         | 计算速率（每秒变化的值），通常用于计数器类型的指标。例如，`rate(http_requests_total[5m])` 计算过去 5 分钟内的每秒请求数。 |
| `irate()`                        | 计算速率（每秒变化的值），但使用的是两个最近数据点的瞬时变化。通常用于更精确的速率计算。       |
| `histogram_quantile()`           | 从直方图数据中计算分位数。                                                                      |
| `increase()`                     | 计算一个计数器在指定时间范围内的增加值。                                                        |
| `delta()`                        | 计算时间序列在指定时间范围内的差值。                                                            |
| `changes()`                      | 计算时间序列在指定时间范围内变化的次数。                                                        |
| `label_replace()`                | 替换标签值，支持正则表达式和替换功能。例如，`label_replace(metric, "new_label", "$1", "label_name", "(.*)")` 会将标签值根据正则进行替换。 |
| `clamp_max()`                    | 限制最大值，例如，`clamp_max(metric, 100)` 将 `metric` 的值限制在最大 100。                    |
| `clamp_min()`                    | 限制最小值，例如，`clamp_min(metric, 0)` 将 `metric` 的值限制在最小 0。                        |
| `avg_over_time()`                | 计算给定时间窗口内的时间序列的平均值。例如，`avg_over_time(metric[1h])`。                      |
| `stddev()`                       | 计算一组时间序列的标准差。                                                                    |
| `stddev_over_time()`             | 计算给定时间范围内时间序列的标准差。                                                            |
| `quantile_over_time()`           | 计算给定时间范围内的时间序列的分位数。                                                        |
| `topk()`                         | 返回排名前 `k` 的时间序列。                                                                  |
| `bottomk()`                      | 返回排名后 `k` 的时间序列。                                                                  |
| `sort()`                         | 对时间序列按值进行排序。                                                                      |
| `sort_desc()`                    | 对时间序列按值进行降序排序。                                                                  |
| `time()`                         | 返回当前时间戳，以秒为单位。                                                                  |
| `label_join()`                   | 将多个标签值合并成一个新的标签。                                                              |
| `label_values()`                 | 返回指定标签的所有标签值。例如，`label_values(http_requests_total, job)` 会返回所有 `http_requests_total` 指标的 `job` 标签值。 |
| `idelta()`                       | 计算时间序列的瞬时变化，类似于 `rate()`，但适用于计数器和其他类型的时间序列。                  |
| `avg_over_time()`                | 计算时间序列在指定时间范围内的平均值。                                                          |
| `reset()`                        | 重置某个计数器值，常用于重置函数的计算。                                                      |
| `histogram_quantile()`           | 根据直方图数据计算指定分位数。例如，`histogram_quantile(0.95, http_duration_seconds_bucket)` 计算 HTTP 请求响应时间的 95% 分位数。 |
| `increase()`                     | 计算计数器在时间范围内的增加量，适用于计数器类型的指标。                                      |
| `over_time()`                    | 计算时间序列在指定时间范围内的总值变化量。                                                    |
| `count_values()`                 | 计算不同标签值的出现次数。例如，`count_values("status", http_requests_total)` 会返回不同 `status` 标签值的出现次数。 |

## 查询示例
```shell
# 查询机器负载
node_load1/node_load5/node_load15
# 查询 CPU 使用率
sum by (instance)(irate(node_cpu_seconds_total{mode!="idle"}[1m])) / sum by (instance)(irate(node_cpu_seconds_total[1m]))
# 查看某个进程的CPU使用率 
sum by (instance)(irate(namedprocess_namegroup_cpu_seconds_total{groupname="sysstress"}[1m])) //该指标由 process_exporter 提供
# 查看某个进程的内存使用率
sum by (groupname)(namedprocess_namegroup_memory_bytes{groupname="sysstress"})
# 可以使用正则
http_requests_total{job=~".*server"}
# 按照应用和进程类型来获取 CPU 利用率最高的 3 个样本数据：
topk(3, sum(rate(instance_cpu_time_ns[5m])) by (app, proc))
```



# 告警配置

告警规则允许基于 Prometheus 表达式语言表达式定义告警条件，并将有关触发告警的通知发送到外部服务。每当告警表达式在给定时间点产生一个或多个向量元素时，该告警即被视为针对这些元素的标签集处于活动状态。

## 告警规则
告警规则配置文件通常使用 YAML 编写，以固定的格式进行操作，下面是告警规则支持的参数:
| 参数名                                | 类型    | 默认值      | 描述                                                                 |
|-------------------------------------|---------|------------|----------------------------------------------------------------------|
| `alert`                             | 字符串   | 必须指定     | 告警名称，必须唯一。                                                  |
| `expr`                              | 字符串   | 无          | 触发告警的 PromQL 表达式，表示监控指标的条件。                       |
| `for`                               | 时间     | 无          | 触发告警前条件保持满足的时间。例如 `1m` 表示条件满足 1 分钟后才告警。 |
| `labels`                            | 字典     | 无          | 告警附加标签，通常用来标识告警的源，例如 `severity: critical`。         |
| `annotations`                       | 字典     | 无          | 告警附加信息，用于提供额外的上下文信息，如告警的描述和解决建议。       |
| `severity`                          | 字符串   | 无          | 自定义标签，用于指示告警的严重级别。例如 `critical`, `warning` 等。      |
| `summary`                           | 字符串   | 无          | 告警的简要描述，通常用于告警通知中。                                    |
| `description`                       | 字符串   | 无          | 告警的详细描述，通常用于告警通知中。                                    |

其中，`annnotations` 下的附加信息是可以自定义的，例如 `summary` 和 `description`都是自定义的。如果需要，还可以添加其他自定义的标签

以下是一个简单的告警规则配置示例（rules/alert.yml）：

```yaml
groups:
- name: localhost alert
  rules:
  - alert: HighLoad           # 告警名称
    expr: node_load1 > 1.5    # 触发告警的 PromQL 表达式
    for: 1m                   # 触发告警前条件保持满足的时间
    labels:
      severity: critical      # 自定义标签
      resource: node          # 自定义标签
    annotations:
      summary: "High load on {{ $labels.instance }}" # 告警的简要描述（自定义）
      description: "{{ $labels.instance }} has a load average of {{ $value }}" # 告警的详细描述 (自定义)
      runbook_url: https://baixiaozhou.github.io/p/%E7%BA%BF%E4%B8%8A%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E6%96%B9%E6%B3%95%E6%B1%87%E6%80%BB/ # 告警处理文档 (自定义)
      recovery: node_load1 < 1.5 # 恢复条件 (自定义)
```
在 promethues 引用告警文件
```yaml
rule_files:
  - 'rules/alert.yml' # 也可以使用通配，例如 rules/*.yml
```

## 告警通知
alertmanager 用于接收prometheus 发出的告警并发送通知。它负责去重、分组和将告警路由到正确的接收器集成（email 等），还负责静默和抑制告警。

1. 分组:分组将性质相似的告警归类到一个通知中。这在大型故障期间尤其有用，因为此时许多系统会同时发生故障，可能会同时触发数百甚至数千个告警。
     - 示例：在您的集群中运行着数十或数百个服务实例，当网络分区发生时，一半的服务实例无法再访问数据库。Prometheus 中的告警规则被配置为在每个服务实例无法与数据库通信时发送告警。因此，会向 Alertmanager 发送数百个告警。
     - 作为一个用户，您只想收到一个页面，但仍然能够看到哪些服务实例受到了影响。因此，您可以将 Alertmanager 配置为按其集群和告警名称对告警进行分组，以便它发送一个简化的通知。
     - 告警的分组、分组通知的计时和这些通知的接收者是在配置文件中的路由树中配置的。
2. 抑制:抑制是指在某些其他告警已经触发的情况下抑制某些告警的通知。
     - 示例：一个告警正在触发，通知您整个集群不可访问。Alertmanager 可以配置为在该特定告警触发时静默有关该集群的所有其他告警。这将阻止发送与实际问题无关的数百或数千个触发告警的通知。
     - 抑制是在 Alertmanager 的配置文件中配置的。
3. 静默:静默是一种简单的方法，可以将告警静默一段时间。静默是根据匹配器配置的，就像路由树一样。传入的告警会检查它们是否与活动静默的所有相等或正则表达式匹配器匹配。如果匹配，则不会为该告警发送任何通知。
     - 静默是在 Alertmanager 的 Web 界面中配置的。

告警通知配置文件通常使用 YAML 格式编写。以下是一个简单的告警通知配置示例：
```yaml
global:
  smtp_smarthost: 'smtp.example.com:25'
  smtp_from: '<EMAIL>'
  smtp_auth_username: '<EMAIL>'
  smtp_auth_password: '<PASSWORD>'
  smtp_require_tls: false

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: '<EMAIL>'
```



# 长期存储方案对比

https://kubesphere.io/zh/blogs/prometheus-storage/





