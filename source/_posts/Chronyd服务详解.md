---
title: Chronyd服务详解
author: baixiaozhou
categories:
  - 运维工具
tags:
  - Linux
  - Chronyd
  - 时钟同步
description: 时钟同步服务 chronyd 介绍
repo: baixiaozhou/SysStress
date: 2024-09-09 14:31:17
references:
  - '[Why does "chronyc sources" output unexpected NTP servers](https://access.redhat.com/solutions/4072781)'
  - ‘[官方文档](https://chrony-project.org/)’
cover: /images/chrony.jpeg
banner: /images/chrony.jpeg
---

<!-- Your content starts here -->
# 介绍

我们先看一下官网的介绍：
```
chrony is a versatile implementation of the Network Time Protocol (NTP). It can synchronise the system clock with NTP servers, reference clocks (e.g. GPS receiver), and manual input using wristwatch and keyboard. It can also operate as an NTPv4 (RFC 5905) server and peer to provide a time service to other computers in the network.
```

简而言之，chronyd 就是用来校准时间的，基于 ntp 协议（ntp 加强版），既可以做服务器，也可以做客户端（既能为其他机器提供时钟同步服务，也能从其他机器上同步时间）

主要功能:

  - 时间同步：chronyd 主要负责将系统时间与网络时间服务器进行同步。它通过网络时间协议（NTP）或精确时间协议（PTP）从外部时间源获取时间信息，并调整本地系统时间。
  - 系统时间校准：Chrony 能够处理系统时钟漂移问题，尤其是在系统启动时或在虚拟化环境中，Chrony 能够在更短的时间内校准系统时间。
  - 网络时钟漂移：chronyd 能够处理网络延迟和时钟漂移，使得系统时间更加准确。

# 部署安装

现在很多 linux 发行版默认都会安装 chronyd 服务，如果没有安装，我们需要手动进行安装:
``` bash
yum install -y chrony
```
安装完成后，我们可以看下 chrony 包中都提供了哪些东西:
``` shell
[root@node2 ~]# rpm -ql chrony
/etc/NetworkManager/dispatcher.d/20-chrony
/etc/chrony.conf
/etc/chrony.keys
/etc/dhcp/dhclient.d/chrony.sh
/etc/logrotate.d/chrony
/etc/sysconfig/chronyd
/usr/bin/chronyc
/usr/lib/systemd/ntp-units.d/50-chronyd.list
/usr/lib/systemd/system/chrony-dnssrv@.service
/usr/lib/systemd/system/chrony-dnssrv@.timer
/usr/lib/systemd/system/chrony-wait.service
/usr/lib/systemd/system/chronyd.service
/usr/libexec/chrony-helper
/usr/sbin/chronyd
/usr/share/doc/chrony-3.4
/usr/share/doc/chrony-3.4/COPYING
/usr/share/doc/chrony-3.4/FAQ
/usr/share/doc/chrony-3.4/NEWS
/usr/share/doc/chrony-3.4/README
/usr/share/man/man1/chronyc.1.gz
/usr/share/man/man5/chrony.conf.5.gz
/usr/share/man/man8/chronyd.8.gz
/var/lib/chrony
/var/lib/chrony/drift
/var/lib/chrony/rtc
/var/log/chrony
```
经常涉及的主要包括:

1. /etc/chrony.conf: 配置文件
2. /usr/bin/chronyc: 命令行工具
3. /usr/sbin/chronyd: 启动程序

## 配置文件详解

下面是 chronyd 配置文件中提供的参数及每项含义:
| 配置项                       | 解释                                                                            |
|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `server <ntp-server> iburst`  | 指定 NTP 服务器地址，`iburst` 选项表示在启动时快速发送一系列请求，以加快初始同步过程。  |
| `driftfile /var/lib/chrony/drift` | 指定时钟漂移文件的位置，该文件记录系统时钟的增益或损失率，以帮助 Chrony 在重启后更快地调整时钟。  |
| `makestep 1.0 3`              | 允许系统时钟在前 3 次更新中，若时钟偏差超过 1 秒，可以进行大步调整（即直接调至正确时间），而不是逐渐调整。    |
| `rtcsync`                     | 启用内核与实时时钟（RTC）的同步功能，以确保系统时钟与硬件时钟保持一致。 |
| `hwtimestamp *`               | 启用所有支持的接口上的硬件时间戳（该行被注释掉，未启用）。 |
| `minsources 2`                | 增加可选时间源的最小数量，当达到该数量时，才会调整系统时钟（该行被注释掉，未启用）。    |
| `allow 192.168.0.0/16`        | 允许从指定的局域网（如 192.168.0.0/16）访问 NTP 客户端，这通常用于允许局域网内的其他设备与本机进行时间同步（该行被注释掉，未启用）。                                                                                           |
| `local stratum 10`            | 即使本地时间未与时间源同步，也允许本地系统作为时间服务器。`stratum` 指定了时间层级，`stratum 10` 表示较低的层次，用于避免对外部 NTP 服务器的依赖（该行被注释掉，未启用）。                                                |
| `keyfile /etc/chrony.keys`    | 指定 NTP 认证的密钥文件路径，用于确保 NTP 客户端和服务器之间的通信是安全的（该行被注释掉，未启用）。  |
| `logdir /var/log/chrony`      | 指定日志文件的目录，Chrony 会将日志存储在该目录下。  |
| `log measurements statistics tracking` | 选择哪些信息需要被记录到日志中，包含测量结果、统计信息和跟踪信息（该行被注释掉，未启用）。 |

## chronyc 命令详解

| 命令              | 说明                                                                                     |
|-------------------|------------------------------------------------------------------------------------------|
| `tracking`        | 显示当前时间跟踪状态，包括时间源、时钟偏移、频率偏移等信息。                             |
| `sources`         | 列出当前配置的 NTP 时间源及其状态。                                                       |
| `sourcestats`     | 显示各个 NTP 时间源的统计信息，包括时间源的延迟、偏移、抖动等。                           |
| `ntpdata`         | 显示与各个时间源相关的低级 NTP 数据，如参考时钟 ID 和偏移量等。                           |
| `makestep`        | 立即将系统时间同步到时间源（即大幅度调整系统时间，而非逐步调整）。                        |
| `burst`           | 向所有时间源发送一组时间请求，以加快同步速度。                                             |
| `reload sources`  | 重新加载配置文件中的时间源，而不需要重新启动 `chronyd`。                                  |
| `clients`         | 列出通过 `chronyd` 进行时间同步的客户端。                                                  |
| `settime`         | 手动设置系统时间。                                                                        |
| `rtcdata`         | 显示实时时钟（RTC）的状态信息。                                                           |
| `manual`          | 启动手动时间输入模式，用于手动设置时间（如通过键盘或手表）。                               |
| `dump`            | 输出 `chronyd` 内存中时间源的状态信息。                                                   |
| `waitsync`        | 等待系统时钟与时间源同步，通常用于确保系统启动时时间同步完成。                            |
| `activity`        | 显示当前活动的时间源数量和状态。                                                           |
| `reselect`        | 强制 `chronyd` 重新选择最佳的时间源。                                                     |
| `serverstats`     | 显示 `chronyd` 的服务器统计信息，如请求数量、响应延迟等。                                 |
| `manual list`     | 列出手动设置的时间值。                                                                    |
| `trimrtc`         | 通过调整实时时钟（RTC）的频率，使其更接近系统时钟的频率。                                 |
| `quit`            | 退出 `chronyc` 交互模式。                                                                 |


### 常用命令

我们先看一下常用的一些命令:

#### chronyc sources
``` bash
[root@node2 ~]# chronyc sources
210 Number of sources = 3
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ node1                        3   6   377    46   +330us[ +400us] +/- 3523us
^+ 8.8.8.8                      2   6   377    47   +570us[ +640us] +/- 2602us
^* 8.8.8.9                      2   6   377    45   +366us[ +436us] +/- 3478us
```
这里显示了当前这个机器同步的一些源信息，包含
1. 源节点: `node1`、`8.8.8.8`、`9.9.9.9`, 最前面的*表示最优先的时钟源，
2. Stratum: NTP 时间源的层级（stratum），表示时间源的准确性层次。Stratum 0 是参考时钟（如 GPS），Stratum 1 是直接从参考时钟获取时间的服务器，以此类推。较高的层级表示离参考时钟的距离越远，时钟的准确性也会降低。
3. Poll: 系统与时间源之间的轮询间隔（以秒为单位）。该值表示在上一次时间同步之后，系统等待多久再次向该时间源请求同步。Poll 值会根据网络状况自动调整，通常范围是 64 到 1024 秒。
4. Reach: 到达值（reach），是一个 8 位的八进制值，表示最近 8 次对该时间源的请求是否成功。该值通常为 377（所有 8 次请求都成功），较低的值表示有请求失败。
5. LastRx: 最近一次从该时间源接收 NTP 数据包的时间（以秒为单位），表示距离上次成功接收数据包的时间长度。值越大表示该时间源未及时响应，可能存在问题。
6. Last sample:最近一次时间样本的偏差值，表示系统时钟与时间源时钟之间的偏移量。正数表示系统时钟快于时间源，负数表示系统时钟慢于时间源。偏差通常以毫秒（ms）或微秒（µs）为单位显示。

我们可以看下这个节点的配置文件:
``` conf
server node1
commandkey 1
keyfile /etc/chrony.keys
```
这里大家会好奇，我这里明明只配置了 node1 为源服务器，为啥会多出两个 ip? 正好 Redhat 官方给了解释:
[Why does "chronyc sources" output unexpected NTP servers](https://access.redhat.com/solutions/4072781)

但是有点烦的是这个文档需要开通红帽订阅才能看，这里我直接粘贴一下原因:
```
Resolution:

Add "PEERNTP=no" entry to /etc/sysconfig/network will prevent dhclient from receiving a list of NTP servers from the DHCP server.
If you already set the network connection down but still gets the NTP server in chronyc sources, delete /var/lib/dhclient/chrony.servers.* file and restart chronyd service.

Root Cause:

If a network connection is set to use DHCP to get IP address, when NetworkManager starts or network connection is up, dhclient receives a list of NTP servers from the DHCP server and generates /var/lib/dhclient/chrony.servers.* file (since chrony 4.1-1, the file is /run/chrony-helper/nm-dhcp.*).
Chronyd not only reads /etc/chrony.conf file, but also reads /var/lib/dhclient/chrony.servers.* file to get NTP server list.

If an NTP server has already been configured in /etc/chrony.conf file, it won't appear in /var/run/chrony-helper/added_servers.
Thus, user can confirm the added servers in /var/run/chrony-helper/added_servers.

Note:
The environment variable, PEERNTP is used in /etc/dhcp/dhclient.d/chrony.sh(chrony rpm ) and /etc/dhcp/dhclient.d/ntp.sh(ntp rpm)
```
简单来说就是 chronyd 不仅会从 chrony.conf中去获取源，也会去 dhcp client 生成的信息中去获取源服务器

#### chronyc tracking
``` bash
[root@\node2 ~]# chronyc tracking
Reference ID    : xxx (8.8.8.8)
Stratum         : 3
Ref time (UTC)  : Mon Sep 09 07:58:14 2024
System time     : 0.000109912 seconds fast of NTP time
Last offset     : +0.000090043 seconds
RMS offset      : 0.000686857 seconds
Frequency       : 11.188 ppm slow
Residual freq   : +0.026 ppm
Skew            : 0.786 ppm
Root delay      : 0.003966943 seconds
Root dispersion : 0.000785699 seconds
Update interval : 64.8 seconds
Leap status     : Normal
```
这里能看到更加细致的同步信息，比如时间的便宜量、更新间隔等

#### chronyc makestep
``` bash
[root@node2 ~]# chronyc makestep
200 OK
```
这个命令主要用于时间跨度太大一次性直接进行同步的，由于时钟的调整是非常微妙要求精确的，时间跨度太大的话同步完成的周期可能比较久，所以可以通过这个命令直接同步。