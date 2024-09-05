---
title: Pacemaker+Corosync使用简介
author: baixiaozhou
categories:
  - 高可用
tags:
  - Pacemaker
  - Corosync
  - Linux
  - 高可用方案
description: Pacemaker+Corosync使用简介
repo: baixiaozhou/SysStress
date: 2024-08-30 17:59:26
references:
  - '[红帽官方文档](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)'
cover: https://th.bing.com/th/id/OIP.Rm6ykLUtQ5OOT3x4JTrWNAAAAA?rs=1&pid=ImgDetMain
banner: https://th.bing.com/th/id/OIP.Rm6ykLUtQ5OOT3x4JTrWNAAAAA?rs=1&pid=ImgDetMain
---


{% quot 参考文档 %}

[红帽官方文档](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index)

# 介绍

pacemaker 和 corosync是两种开源软件组件，通常结合使用以构建高可用性（HA）集群。

## Pacemaker

Pacemaker 是一个集群资源管理器，负责管理集群中所有资源的启动、停止、迁移等操作。它通过与 Corosync 协作，确保在节点故障或服务异常时，资源能够自动在其他健康节点上接管。从这里我们就可以发现，pacemaker 的核心在于管理

### 组件

pacemaker 主要包括以下组件:

-	CIB (Cluster Information Base)：存储集群的配置信息，包括资源、约束、节点等。
-	CRM (Cluster Resource Manager)：决定如何在集群中分配和管理资源。
-	PEngine (Policy Engine)：根据集群状态和配置策略做出决策。
-	Fencing：通过 STONITH（Shoot The Other Node In The Head）机制来隔离失效的节点，防止脑裂。

### 使用场景

-	管理集群中的各种资源（如虚拟 IP、数据库服务、文件系统等）。
-	确保服务的高可用性，在故障发生时自动切换资源到其他节点。

## Corosync

Corosync 是一个集群通信引擎，负责在集群节点之间提供消息传递、组成员资格管理、心跳检测等功能。它确保集群中所有节点之间的信息同步，监控节点的健康状况，并在节点故障时通知 pacemaker。

###	组件

-	组通信：用于确保集群中所有节点保持一致的视图。
-	故障检测：通过心跳机制监控节点状态，当节点失联时，通知 Pacemaker。
-	配置管理：管理集群节点的配置和成员资格。

### 使用场景

-	集群中节点间的实时通信。
-	监控节点的可用性，并在节点失效时做出响应。

# 安装部署

## 安装依赖

1. 在集群的所有节点上安装相关依赖:
```
yum install -y pcs pacemaker corosync # Centos
```
2. 启动相关服务并设置服务开机自启动:
```
systemctl start pcsd 
systemctl enable pcsd

```
3. 设置`hacluster`用户的密码(此用户在包安装的过程中会自动创建)
```
sudo passwd hacluster
```
4. 在 `/etc/hosts` 中加入节点配置，例如:
```
192.168.1.2 node2
192.168.1.3 node3
```

## 命令操作

集群的命令行操作基本上都是通过 pcs 进行，pcs 提供了如下一些命令:

| 命令        | 说明                                | 示例命令                              |
|-------------|-------------------------------------|---------------------------------------|
| `cluster`   | 配置集群选项和节点                  | `pcs cluster start` 启动集群          |
| `resource`  | 管理集群资源                        | `pcs resource create myresource ocf:heartbeat:IPaddr2 ip=192.168.1.1` 创建一个资源 |
| `stonith`   | 管理 fence 设备                     | `pcs stonith create myfence fence_ipmilan ipaddr=192.168.1.100 login=admin passwd=password lanplus=1` 创建 STONITH 设备 |
| `constraint`| 管理资源约束                        | `pcs constraint location myresource prefers node1=100` 设置资源约束 |
| `property`  | 管理 Pacemaker 属性                 | `pcs property set stonith-enabled=false` 禁用 STONITH |
| `acl`       | 管理 Pacemaker 访问控制列表         | `pcs acl role create readonly` 创建只读角色 |
| `qdevice`   | 管理本地主机上的仲裁设备提供程序    | `pcs qdevice add model net` 添加网络仲裁设备 |
| `quorum`    | 管理集群仲裁设置                    | `pcs quorum status` 查看仲裁状态      |
| `booth`     | 管理 booth (集群票据管理器)         | `pcs booth status` 查看 booth 状态    |
| `status`    | 查看集群状态                        | `pcs status` 查看集群运行状态         |
| `config`    | 查看和管理集群配置                  | `pcs config show` 显示集群配置        |
| `pcsd`      | 管理 pcs 守护进程                   | `pcs pcsd status` 查看 pcsd 服务状态  |
| `node`      | 管理集群节点                        | `pcs node standby node1` 将节点设置为备用 |
| `alert`     | 管理 Pacemaker 警报                 | `pcs alert create node=node1 severity=critical` 创建警报 |
| `client`    | 管理 pcsd 客户端配置                | `pcs client cert-key-gen --force` 生成新的客户端证书 |

此外，packmaker 还提供了其他的命令，比如 crm 的一系列工具:

| 工具名                | 解释                                                                                     | 示例                                                                                   |
|-----------------------|------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| `crm_attribute`        | 管理集群属性，包括设置、修改或删除节点属性。      | `crm_attribute --node node1 --name attr_name --update attr_value` 设置节点属性。       |
| `crm_diff`             | 比较两个 CIB 配置文件的差异，便于配置版本管理。   | `crm_diff cib_old.xml cib_new.xml` 比较两个 CIB 文件的差异。                           |
| `crm_error`            | 显示集群运行过程中遇到的错误信息，帮助排查故障。   | `crm_error -s 12345` 显示特定错误代码的详细信息。                                      |
| `crm_failcount`        | 查看或管理资源的失败计数，影响资源的自动重新调度。 | `crm_failcount --query --resource my_resource --node node1` 查看失败计数。             |
| `crm_master`           | 管理主从资源（如 DRBD）状态的工具，用于启动或停止主从资源。 | `crm_master --promote my_resource` 提升资源为主状态。                                  |
| `crm_mon`              | 实时监控集群状态，显示资源、节点、失败信息。      | `crm_mon --interval=5s --show-detail` 每5秒更新监控，显示详细信息。                    |
| `crm_node`             | 管理集群节点的工具，包括查看节点状态、删除节点等。 | `crm_node -l` 列出所有集群节点。                                                      |
| `crm_report`           | 生成集群故障报告的工具，汇总集群状态、日志和诊断信息。  | `crm_report -f report.tar.bz2` 生成详细的故障报告。                                   |
| `crm_resource`         | 管理集群资源，包括启动、停止、迁移和清除资源。     | `crm_resource --move my_resource --node node2` 将资源迁移到另一个节点。                |
| `crm_shadow`           | 允许对 CIB 进行“影子”配置，便于测试和调试。       | `crm_shadow --create shadow_test` 创建影子配置。                                       |
| `crm_simulate`         | 模拟集群运行状态的工具，用于测试集群配置的行为。   | `crm_simulate --live --save-output output.xml` 运行模拟，并保存输出。                  |
| `crm_standby`          | 将节点设置为待机状态，临时不参与资源调度，或重新激活节点。 | `crm_standby --node node1 --off` 将节点设置为待机状态。                                |
| `crm_ticket`           | 管理集群的 ticket，用于决定哪些资源在哪些位置可以运行（多站点集群）。 | `crm_ticket --grant my_ticket --node node1` 授权 ticket 给指定节点。                   |
| `crm_verify`           | 验证当前集群配置的工具，检查配置文件的完整性和正确性。 | `crm_verify --live-check` 验证当前运行中的集群配置。                                  |

pacemaker 和 crm 的命令对比:

1. pcs（Pacemaker/Corosync Shell）
  - 简介: pcs 是 Pacemaker 和 Corosync 集群管理的命令行工具。它主要用于 Red Hat 系列操作系统（例如 RHEL、CentOS 等）。pcs 提供了一个简单的命令行界面，用于管理集群、资源、节点、约束等功能。
  - 功能:
    - 管理 Pacemaker 集群、Corosync 配置、STONITH 设备、资源和约束等。
    - 提供集群的创建、启动、停止、删除、资源添加、约束设置等命令。
    - 提供简单易用的命令接口，能够将集群管理的命令封装成一步到位的操作。
    - 支持通过 pcsd 提供 Web 界面的管理。
  - 适用场景: pcs 更加适用于初学者和需要快速操作的用户，因为它提供了很多高层次的命令，简化了集群管理。

2. crm（Cluster Resource Manager Shell）
  - 简介: crm 是 Pacemaker 的原生命令行工具，提供更加底层的控制。crm 主要用于 Pacemaker 集群资源管理和调度，支持在更细粒度上配置和管理集群资源。
  - 功能:
    - 提供更细致的资源管理和集群控制功能。
    - crm 的指令可以进行更复杂的操作，比如编辑 CIB (Cluster Information Base) 的 XML 配置文件。
    - 允许更加精细的配置，适合对集群系统有深度了解的用户。
  - 适用场景: crm 更加适合高级用户，特别是那些需要精确配置、排查问题或操作底层 Pacemaker 资源的场景。

## 节点认证和集群创建

在集群中的任意一个节点上执行:
1. 认证:
``` bash
pcs cluster auth node2 node3 -u hacluster
```
2. 认证完整后创建集群
``` bash
pcs cluster setup --name mycluster node2 node3 (同时添加所有节点)
```
创建完成后，会生成 corosync 的配置文件，默认位置`/etc/corosync/corosync.conf`, 其中的内容如下:
``` json
totem {
    version: 2
    cluster_name: mycluster
    secauth: off
    transport: udpu
}

nodelist {
    node {
        ring0_addr: node2
        nodeid: 1
    }

    node {
        ring0_addr: node3
        nodeid: 2
    }
}

quorum {
    provider: corosync_votequorum
    two_node: 1
}

logging {
    to_logfile: yes
    logfile: /var/log/cluster/corosync.log
    to_syslog: yes
}
```

3. 启动集群
``` bash
pcs cluster start --all # 这里启动失败的话，可以后面加上 --debug参数查看更详细的信息，可能会因为防火墙等问题导致启动失败
pcs cluster enable --all # 设置自启动
```
4. 查看集群状态
``` bash
pcs status
```
我们看下输出情况:
```
Cluster name: mycluster

WARNINGS:
No stonith devices and stonith-enabled is not false

Stack: corosync
Current DC: node2 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Mon Sep  2 18:43:57 2024
Last change: Mon Sep  2 18:35:08 2024 by hacluster via crmd on node2

2 nodes configured
0 resource instances configured

Online: [ node2 node3 ]

No resources


Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
这里我们可以看到集群的总体情况，包括节点状态、服务状态（有两个服务还处于 disabled 状态， 通过`systemctl enable corosync pacemaker` 设置开机启动）、资源信息（还没有添加 resource）等

## stonith 配置

在上文集群的状态输出中还包括了一个告警信息: `No stonith devices and stonith-enabled is not false`, 这里的 stonith（Shoot The Other Node In The Head） 是一种防止“脑裂” (split-brain) 的机制。当集群中的一个节点失去与其他节点的连接时，stonith 设备可以强制重启或关闭这个失联的节点，避免两个或多个节点同时操作同一个资源，导致数据损坏。想要消除这个告警，有两种解决方案:
1. 禁用 stonith: 如果是自己的测试环境，那么可以禁用掉 stonith 来消除告警，操作方法为:
  ``` bash
  pcs property set stonith-enabled=false
  ```
2. 配置 stonith 设备: 在生产环境中，建议配置 stonith

我们先根据官方文档的指示看一下stonith 有哪些可用代理:
``` bash
[root@node2 corosync]# pcs stonith list
Error: No stonith agents available. Do you have fence agents installed?
```
这里提示没有代理的 agent 可用，所以我们首先需要安装`fence agent`：
``` shell
yum install -y fence-agents
```
我们再 list 一下，就可以看到支持的代理了

| Fence Agent | 描述 |
|-------------|------|
| `fence_amt_ws` | 适用于 AMT (WS) 的 Fence 代理 |
| `fence_apc` | 通过 telnet/ssh 控制 APC 的 Fence 代理 |
| `fence_apc_snmp` | 适用于 APC 和 Tripplite PDU 的 SNMP Fence 代理 |
| `fence_bladecenter` | 适用于 IBM BladeCenter 的 Fence 代理 |
| `fence_brocade` | 通过 telnet/ssh 控制 HP Brocade 的 Fence 代理 |
| `fence_cisco_mds` | 适用于 Cisco MDS 的 Fence 代理 |
| `fence_cisco_ucs` | 适用于 Cisco UCS 的 Fence 代理 |
| `fence_compute` | 用于自动复活 OpenStack 计算实例的 Fence 代理 |
| `fence_drac5` | 适用于 Dell DRAC CMC/5 的 Fence 代理 |
| `fence_eaton_snmp` | 适用于 Eaton 的 SNMP Fence 代理 |
| `fence_emerson` | 适用于 Emerson 的 SNMP Fence 代理 |
| `fence_eps` | 适用于 ePowerSwitch 的 Fence 代理 |
| `fence_evacuate` | 用于自动复活 OpenStack 计算实例的 Fence 代理 |
| `fence_heuristics_ping` | 基于 ping 进行启发式 Fencing 的代理 |
| `fence_hpblade` | 适用于 HP BladeSystem 的 Fence 代理 |
| `fence_ibmblade` | 通过 SNMP 控制 IBM BladeCenter 的 Fence 代理 |
| `fence_idrac` | 适用于 IPMI 的 Fence 代理 |
| `fence_ifmib` | 适用于 IF MIB 的 Fence 代理 |
| `fence_ilo` | 适用于 HP iLO 的 Fence 代理 |
| `fence_ilo2` | 适用于 HP iLO2 的 Fence 代理 |
| `fence_ilo3` | 适用于 IPMI 的 Fence 代理 |
| `fence_ilo3_ssh` | 通过 SSH 控制 HP iLO3 的 Fence 代理 |
| `fence_ilo4` | 适用于 IPMI 的 Fence 代理 |
| `fence_ilo4_ssh` | 通过 SSH 控制 HP iLO4 的 Fence 代理 |
| `fence_ilo5` | 适用于 IPMI 的 Fence 代理 |
| `fence_ilo5_ssh` | 通过 SSH 控制 HP iLO5 的 Fence 代理 |
| `fence_ilo_moonshot` | 适用于 HP Moonshot iLO 的 Fence 代理 |
| `fence_ilo_mp` | 适用于 HP iLO MP 的 Fence 代理 |
| `fence_ilo_ssh` | 通过 SSH 控制 HP iLO 的 Fence 代理 |
| `fence_imm` | 适用于 IPMI 的 Fence 代理 |
| `fence_intelmodular` | 适用于 Intel Modular 的 Fence 代理 |
| `fence_ipdu` | 通过 SNMP 控制 iPDU 的 Fence 代理 |
| `fence_ipmilan` | 适用于 IPMI 的 Fence 代理 |
| `fence_kdump` | 与 kdump 崩溃恢复服务一起使用的 Fence 代理 |
| `fence_mpath` | 用于多路径持久保留的 Fence 代理 |
| `fence_redfish` | 适用于 Redfish 的 I/O Fencing 代理 |
| `fence_rhevm` | 适用于 RHEV-M REST API 的 Fence 代理 |
| `fence_rsa` | 适用于 IBM RSA 的 Fence 代理 |
| `fence_rsb` | 适用于 Fujitsu-Siemens RSB 的 I/O Fencing 代理 |
| `fence_sbd` | 适用于 SBD 的 Fence 代理 |
| `fence_scsi` | 用于 SCSI 持久保留的 Fence 代理 |
| `fence_virt` | 适用于虚拟机的 Fence 代理 |
| `fence_vmware_rest` | 适用于 VMware REST API 的 Fence 代理 |
| `fence_vmware_soap` | 通过 SOAP API 控制 VMware 的 Fence 代理 |
| `fence_wti` | 适用于 WTI 的 Fence 代理 |
| `fence_xvm` | 适用于虚拟机的 Fence 代理 |

想查看代理的具体用法，可以使用:
``` bash
pcs stonith describe stonith_agent
```

使用 `fence_heuristics_ping` 作为代理，先通过`pcs stonith describe fence_heuristics_ping` 看下具体的用法和配置
```
fence_heuristics_ping - Fence agent for ping-heuristic based fencing

fence_heuristics_ping uses ping-heuristics to control execution of another fence agent on the same fencing level.

This is not a fence agent by itself! Its only purpose is to enable/disable another fence agent that lives on the same fencing level but after fence_heuristics_ping.

Stonith options:
  method: Method to fence
  ping_count: The number of ping-probes that is being sent per target
  ping_good_count: The number of positive ping-probes required to account a target as available
  ping_interval: The interval in seconds between ping-probes
  ping_maxfail: The number of failed ping-targets to still account as overall success
  ping_targets (required): A comma separated list of ping-targets (optionally prepended by 'inet:' or 'inet6:') to be probed
  ping_timeout: The timeout in seconds till an individual ping-probe is accounted as lost
  quiet: Disable logging to stderr. Does not affect --verbose or --debug-file or logging to syslog.
  verbose: Verbose mode
  debug: Write debug information to given file
  delay: Wait X seconds before fencing is started
  login_timeout: Wait X seconds for cmd prompt after login
  power_timeout: Test X seconds for status change after ON/OFF
  power_wait: Wait X seconds after issuing ON/OFF
  shell_timeout: Wait X seconds for cmd prompt after issuing command
  retry_on: Count of attempts to retry power on
  pcmk_host_map: A mapping of host names to ports numbers for devices that do not support host names. Eg. node1:1;node2:2,3 would tell the cluster to use port 1 for node1 and ports 2 and 3
                 for node2
  pcmk_host_list: A list of machines controlled by this device (Optional unless pcmk_host_check=static-list).
  pcmk_host_check: How to determine which machines are controlled by the device. Allowed values: dynamic-list (query the device via the 'list' command), static-list (check the pcmk_host_list
                   attribute), status (query the device via the 'status' command), none (assume every device can fence every machine)
  pcmk_delay_max: Enable a random delay for stonith actions and specify the maximum of random delay. This prevents double fencing when using slow devices such as sbd. Use this to enable a
                  random delay for stonith actions. The overall delay is derived from this random delay value adding a static delay so that the sum is kept below the maximum delay.
  pcmk_delay_base: Enable a base delay for stonith actions and specify base delay value. This prevents double fencing when different delays are configured on the nodes. Use this to enable a
                   static delay for stonith actions. The overall delay is derived from a random delay value adding this static delay so that the sum is kept below the maximum delay.
  pcmk_action_limit: The maximum number of actions can be performed in parallel on this device Pengine property concurrent-fencing=true needs to be configured first. Then use this to specify
                     the maximum number of actions can be performed in parallel on this device. -1 is unlimited.

Default operations:
  monitor: interval=60s
```

这里我们进行创建(其他参数都有默认值，按需修改即可):
``` bash
pcs stonith create my_ping_fence_device fence_heuristics_ping \
    ping_targets="node2,node3"
```

创建完成后`pcs status`查看状态
```
Cluster name: mycluster
Stack: corosync
Current DC: node2 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Tue Sep  3 14:40:56 2024
Last change: Tue Sep  3 14:35:39 2024 by root via cibadmin on node2

2 nodes configured
1 resource instance configured

Online: [ node2 node3 ]

Full list of resources:

 my_ping_fence_device	(stonith:fence_heuristics_ping):	Started node2

Failed Fencing Actions:
* reboot of my_apc_fence_device failed: delegate=, client=stonith_admin.40341, origin=node2,
    last-failed='Tue Sep  3 14:12:23 2024'

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```
验证 stonith 是否生效:
``` bash
[root@node2 cluster]# pcs stonith fence node3
Node: node3 fenced
```
执行完这个操作后，节点会离线，pcs 服务会停止，想要加回来的话，在停止的节点上重新启动集群即可:`pcs cluster start && pcs cluster enable`

# 实战操作

## 添加节点

上文中，我们构建了一个两节点的集群，我们可以尝试增加一个节点，构建一个三节点的集群

1. 首先在新节点上安装各种依赖，设置密码等。
2. 在原集群上认证新 node
3. 在原集群上添加 node：`pcs cluster node add node4`
4. 在 node4 上执行：`pcs cluster start && pcs cluster enable`

再通过`pcs status`就可以看到新的节点已经加入

添加完之后还需要更新新节点的一些配置，比如上文提到的 stonith：
``` bash
pcs stonith update my_ping_fence_device fence_heuristics_ping \
    ping_targets="node2,node3,node4"
```

## 配置 resource

### 资源类型

创建 resource 的基本格式为:
``` bash
pcs resource create resource-name ocf:heartbeat:apache [--options]
```
这里的 ocf:heartbeat:apache 第一个部分ocf，指明了这个资源采用的标准(类型)，第二个部分标明这个资源脚本的在ocf中的名字空间，在这个例子中是heartbeat。最后一个部分指明了资源脚本的名称。

我们先看下有哪些标准类型
``` bash
[root@node3 ~]# pcs resource standards
lsb
ocf
service
systemd
```
查看可用的ocf资源提供者:
``` bash
[root@node3 ~]# pcs resource providers
heartbeat
openstack
pacemaker
```
查看特定标准下所支持的脚本，例：ofc:heartbeat 下的脚本(列举了部分)：
``` bash
[root@node3 ~]# pcs resource agents ocf:heartbeat
aliyun-vpc-move-ip
apache
aws-vpc-move-ip
aws-vpc-route53
awseip
awsvip
azure-events
azure-lb
clvm
conntrackd
CTDB
db2
Delay
dhcpd
docker
Dummy
ethmonitor
exportfs
Filesystem
galera
garbd
iface-vlan
IPaddr
IPaddr2
```

### 设置虚拟 ip

虚拟 IP（Virtual IP）是在高可用性集群中使用的一种技术，通过为服务提供一个不依赖于特定物理节点的 IP 地址来实现服务的高可用性。当集群中的某个节点出现故障时，虚拟 IP 可以迅速转移到另一个健康的节点上，从而保证服务的连续性。

虚拟 IP 的使用场景

1. 高可用性：虚拟 IP 最常见的使用场景是高可用性集群（如 Pacemaker 或 Keepalived），它允许一个服务在集群中的多个节点之间进行切换，而不会更改客户端访问的 IP 地址。
2. 负载均衡：虚拟 IP 可以结合负载均衡器使用，将来自客户端的请求分配到多个后端服务器，以实现流量的均匀分布。
3. 灾难恢复：在灾难恢复场景中，虚拟 IP 可以用于快速恢复服务，将业务流量从故障节点转移到备用节点上

在 pcs 集群中，我们可以通过以下方式增加一个虚拟 ip:
``` bash
pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=x.x.x.x cidr_netmask=32 nic=bond1 op monitor interval=30s
```
执行完成后，通过`pcs status`就可以看到 ip 绑定在哪里:
```
virtual_ip	(ocf::heartbeat:IPaddr2):	Started node3
```
当我们关停 node3 的服务时，就会发现这个虚拟ip 绑定到了其他节点的 bond1 网卡上。

### 增加服务

我们以 httpd 服务为例，在集群中创建资源,首先安装对应服务:
``` bash
sudo yum install httpd -y
sudo systemctl start httpd # 这里可选择不启动，后续如果通过pcs 直接托管，需要先停掉，
sudo systemctl enable httpd 
```

创建 resource:
```
pcs resource create WebService ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf op monitor interval=30s
```

## 结合 LVS + ldirectord 进行使用

如果环境是一套多节点集群，在生产中我们肯定需要充分利用起这些节点，所以就要考虑流量分发。在这一层面上，我们可以使用 lvs 进行流量分发。这里首先对 lvs 对一个简单的介绍

LVS（Linux Virtual Server）是一个基于 IP 负载均衡技术的开源软件项目，主要用于构建高可用、高性能的负载均衡集群。LVS 是 Linux 内核的一部分，通过网络层的负载均衡技术，将来自客户端的请求分发到多个后端服务器，从而实现分布式处理、提高系统的处理能力和可靠性。

LVS 主要通过三种负载均衡模式（NAT 模式、DR 模式、TUN 模式）来实现流量的分发，支持大规模并发请求的处理，通常用于大型网站、电子商务平台和高访问量的 Web 应用中。

LVS 的特点
1. 高性能: LVS 工作在网络层（第4层），基于 IP 进行流量转发，性能极高。它能够处理大量的并发连接，适合高流量、大规模的网站和服务。
2. 高可用性: LVS 通常与 Keepalived、Pacemaker 等高可用性工具配合使用，以实现负载均衡器的自动故障切换，确保服务的高可用性和稳定性。
3. 多种负载均衡算法: LVS 提供了多种负载均衡算法，如轮询（Round Robin）、最小连接（Least Connection）、基于目标地址哈希（Destination Hashing）等，可以根据具体需求选择合适的算法进行流量分发。
4. 多种工作模式, LVS 支持三种主要工作模式：
	-	NAT 模式（网络地址转换模式）：LVS 充当请求和响应的中介，适用于小规模集群。
	-	DR 模式（直接路由模式）：请求由 LVS 转发，但响应直接返回给客户端，适用于大型集群，性能高。
	-	TUN 模式（IP 隧道模式）：类似于 DR 模式，但支持跨网络部署，非常适合广域网负载均衡。
5. 高扩展性: LVS 可以轻松地扩展和管理多台服务器，支持动态添加或移除后端服务器，适应业务需求的变化，且不影响服务的正常运行。
6. 透明性: 对客户端和后端服务器来说，LVS 的存在是透明的。客户端并不感知负载均衡的存在，访问体验一致。后端服务器也不需要做特殊的配置，只需处理 LVS 转发的请求。
7. 成熟且稳定: 作为一个成熟的负载均衡解决方案，LVS 被广泛应用于生产环境中，经过多年发展，功能完备，稳定性高。
8. 安全性: LVS 可以与防火墙等安全工具结合使用，增强系统的安全性。此外，LVS 还支持 IP 地址过滤、端口过滤等功能，提供一定程度的安全保护。

ldirectord 是一个守护进程，用于管理和监控由 LVS 提供的虚拟服务（Virtual Services）。其主要功能包括：

1.	监控后端服务器：ldirectord 定期检查后端服务器的健康状况，确保只有健康的服务器参与流量分配。
2.	动态配置：基于后端服务器的健康状况，ldirectord 可以动态调整 LVS 的配置。例如，当一台服务器宕机时，ldirectord 会自动将其从 LVS 配置中移除。
3.	高可用性：结合 heartbeat 等高可用性工具，ldirectord 可以确保在主节点故障时，负载均衡服务能够自动切换到备用节点，继续提供服务。

### 安装部署

安装lvs：
``` bash
yum install lvm2 ipvsadm -y
```
在这里找包有一些技巧，比如一开始 chatgpt 提供的说法是要安装`lvs` 和 `ipvsadm`，但是在我的环境上通过`yum install -y lvs` 的时候提示没有这个包，那我们可以通过 yum 提供的一些命令来简单锁定一下，比如 `yum provides lvs`,这样就会把包含了这个命令的包显示出来（适用于知道命令但是不知道是哪个包的场景），

安装 ldirector：
```
# 这里我用 yum 下载是没有找到对应包的，找了一圈也没找到安装方法，所以直接找的 rpm 包
# 下载地址: ftp://ftp.icm.edu.pl/vol/rzm3/linux-opensuse/update/leap/15.2/oss/x86_64/ldirectord-4.4.0+git57.70549516-lp152.2.9.1.x86_64.rpm
# 上传到机器上后，进行安装
rpm -Uvh --force ldirectord-4.4.0+git57.70549516-lp152.2.9.1.x86_64.rpm

# 需要依赖，先安装依赖,再装包
yum install -y perl-IO-Socket-INET6 perl-MailTools perl-Net-SSLeay perl-Socket6 perl-libwww-perl
# 操作完成之后，启动服务
systemctl start ldirectord.service
```
服务启动失败:
{% image /images/ldirectord.png ldirectord服务 fancybox:true %}

这里有点坑，缺少了依赖的文件，但是装包的时候没有提示，需要再安装: `yum install -y perl-Sys-Syslog`, 安装完成后此问题消失，但是此时配置文件还没配置，所以服务还起不来。

### pcs 结合 lvs、ldirectord

在上文中，我们创建了一个 httpd 服务和 vip 资源。 在实际生产中，要充分利用节点性能，我们可能要在多个节点上启动httpd 示例，我们在每个节点上都启动一个实例，然后将他们归到一个组中:
``` bash
pcs resource delete WebService # 移除之前创建的服务
pcs resource create WebService1 ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf op monitor interval=30s
pcs resource create WebService2 ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf op monitor interval=30s --force
pcs resource create WebService3 ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf op monitor interval=30s --force # 创建三个服务

pcs constraint location WebService1 prefers node2
pcs constraint location WebService2 prefers node3
pcs constraint location WebService3 prefers node4  # 限制对应 resource 服务只能在指定节点上运行
```

配置 ldirectord:
``` conf
checktimeout=10
checkinterval=2
autoreload=yes
logfile="/var/log/ldirectord.log"
quiescent=yes

virtual=vip:80 # 之前绑定的 VIP
    real=192.168.1.2:80 gate
    real=192.168.1.3:80 gate
    real=192.168.1.4:80 gate
    fallback=127.0.0.1:80
    service=http
    request="index.html"
    receive="HTTP/1.1 200 OK"
    scheduler=rr
    protocol=tcp
    checktype=negotiate
```

然后在通过 `ipvsadm -ln` 就可以查看到详细的信息:
``` bash
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  vip:80 rr
  -> 192.168.1.2:80               Route   0      0          0
  -> 192.168.1.3:80               Route   0      0          0
  -> 192.168.1.4:80               Route   0      0          0
  -> 127.0.0.1:80                 Route   1      0          0
```

然后我们可以在 pcs 上创建一个资源 lvs 相关的资源:
``` bash
pcs resource create my_lvs ocf:heartbeat:ldirectord \
    configfile=/etc/ha.d/ldirectord.cf \
    ldirectord=/usr/sbin/ldirectord  \
    op monitor interval=15s timeout=60s \
    op stop timeout=60s
```
这里的ocf:heartbeat:ldirectord 在有的版本中会默认安装，有的版本不会，如果没有的话需要手动下载: https://github.com/ClusterLabs/resource-agents/blob/main/ldirectord/OCF/ldirectord.in
存放到: `/usr/lib/ocf/resource.d/heartbeat/ldirectord` 并添加可执行权限: `chmod +x /usr/lib/ocf/resource.d/heartbeat/ldirectord`

创建完成后，我们可以将 vip 和 lvs 绑定到一个组中，这样 lvs 就会跟着 vip 进行转移了:
``` bash
pcs resource group add balanceGroup virtual_ip my_lvs
```
通过`pcs status`查看就可以看到:
```
Resource Group: balanceGroup
     virtual_ip	(ocf::heartbeat:IPaddr2):	Started node2
     my_lvs	(ocf::heartbeat:ldirectord):	Started node2
```
不断对节点进行关闭测试，可以看到 lvs 和 vip 始终都在同一个节点上

## 增加节点属性

我们这里使用另外一个观察集群状态的命令:`crm_mon`, 比如`crm_mon -A1`
``` bash
[root@node2 rpm]# crm_mon -A1
Stack: corosync
Current DC: node3 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Thu Sep  5 22:09:53 2024
Last change: Wed Sep  4 18:17:25 2024 by root via cibadmin on node3

3 nodes configured
6 resource instances configured

Online: [ node2 node3 node4 ]

Active resources:

 my_ping_fence_device	(stonith:fence_heuristics_ping):	Started node3
 WebService1	(ocf::heartbeat:apache):	Started node4
 WebService2	(ocf::heartbeat:apache):	Started node3
 WebService3	(ocf::heartbeat:apache):	Started node4
 Resource Group: balanceGroup
     virtual_ip	(ocf::heartbeat:IPaddr2):	Started node2
     my_lvs	(ocf::heartbeat:ldirectord):	Started node2

Node Attributes:
* Node node2:
* Node node3:
* Node node4:
.....
```
输出和`pcs status`查看到的效果基本上是差不多的。但是在下面有`Node Attributes`，这里我们看下节点属性怎么设置:
``` bash
pcs node attribute node2 role=master
pcs node attribute node3 role=standby
pcs node attribute node4 role=standby
```
或者
``` bash
crm_attribute --node node2 --name mysql --update master
crm_attribute --node node3 --name mysql --update standby
```
设置完成之后，我们就可以看到节点属性:
```
Node Attributes:
* Node node2:
    + mysql                           	: master
    + role                            	: master
* Node node3:
    + mysql                           	: standby
    + role                            	: standby
* Node node4:
    + role                            	: standby
```
那有人就会好奇这样设置有什么用呢？主要用途是在哪里呢。

这里的指标往往是动态的，可以根据自己喜好结合一些扩展进行变化，比如部署了一套 postgresql 集群，集群中有主有备，有同步节点也有异步节点，有的节点状态可能有问题，那我们怎么能够显示出这个集群的整体情况呢，这样就可以使用 Node Attributes进行设置，关于如果搭建 pcs + postgresql 的集群，大家可以参考这篇文章: [基于Pacemaker的PostgreSQL高可用集群](https://www.cnblogs.com/Alicebat/p/14148933.html)

最终我们看到的效果如下:
```
Node Attributes:
* Node pg01:
    + master-pgsql                    	: 1000      
    + pgsql-data-status               	: LATEST    
    + pgsql-master-baseline           	: 0000000008000098
    + pgsql-status                    	: PRI       
* Node pg02:
    + master-pgsql                    	: -INFINITY 
    + pgsql-data-status               	: STREAMING|ASYNC
    + pgsql-status                    	: HS:async  
* Node pg03:
    + master-pgsql                    	: 100       
    + pgsql-data-status               	: STREAMING|SYNC
    + pgsql-status                    	: HS:sync   
```
当集群发生节点变动，状态异常时，我们就可以根据 attibutes 的一些信息查看定位。