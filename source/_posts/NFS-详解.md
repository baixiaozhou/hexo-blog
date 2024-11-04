---
title: NFS 详解
author: baixiaozhou
categories:
  - 分布式存储
tags:
  - NFS
  - 存储
description: NFS 简介
date: 2024-10-09 10:51:43
references:
cover:
banner:
---

# 概述

NFS 全拼 Network File System， 由 Sun Microsystems 于 1984 年开发，旨在允许不同机器间通过网络共享文件。它是一个基于客户端-服务器架构的分布式文件系统协议，允许多个客户端系统像访问本地存储一样访问远程服务器上的文件。

NFS 使得用户可以通过网络访问远程计算机上的文件，并对文件进行读取、写入、删除等操作，这些操作像本地文件系统一样透明，用户无需关心文件实际存储的位置。

# 相关概念

NFS 关键概念

1. 服务器和客户端：
	- 服务器：存储文件并将文件系统通过网络共享给客户端。
	- 客户端：通过网络挂载服务器提供的文件系统，并进行文件操作。
2.	挂载（Mounting）：
	- 客户端使用 NFS 挂载（mount）服务器上共享的文件系统。挂载后，远程文件系统可以像本地文件系统一样进行访问。
3.	RPC（Remote Procedure Call）：
	- NFS 基于 RPC（远程过程调用）协议进行通信。通过 RPC，客户端发送请求，服务器响应请求并返回相应的数据。
4.	文件句柄：
	- NFS 使用文件句柄来唯一标识远程文件。客户端在访问文件时，文件句柄将传递给服务器以确定目标文件。
5.	Stateless（无状态）协议：
	- 早期版本的 NFS 是无状态的，这意味着服务器不保存任何有关客户端的信息。每个请求都独立处理，且服务器不跟踪会话状态。
	- 无状态的设计提高了系统的容错性，但也限制了某些功能。现代的 NFS 引入了一些状态管理来增强功能。
6.	版本：
	- NFS 经过多次版本迭代，最常用的版本有 NFSv3 和 NFSv4。
	- NFSv3：最广泛使用的版本，支持较大的文件系统，并增加了多种优化。
	- NFSv4：引入了状态管理、增强的安全性以及简化了防火墙穿越。
7.	文件锁定：
	- NFS 支持文件锁定机制，使得客户端可以在多人同时操作同一文件时避免冲突。NFSv4 将锁定功能内置，而 NFSv3 需要外部锁定协议（如 NLM）。
8.	安全性：
	- 传统的 NFS 依赖于基于 IP 地址的信任机制。现代的 NFSv4 支持通过 Kerberos 进行强身份认证。
9.	性能和缓存：
	- 为了减少网络通信带来的延迟，NFS 客户端通常使用缓存机制。客户端可以缓存文件的部分内容和属性，并在适当的时候将缓存的数据写回服务器。

# 部署安装

我们以 centos 为例来进行NFS 的部署和安装:
1. 安装 nfs 服务(服务器端和客户端都需要安装)
``` bash
yum install -y nfs-utils
```
安装完毕后，我们可以查看下这个包提供的文件信息:
```
/etc/exports.d
/etc/gssproxy/24-nfs-server.conf
/etc/modprobe.d/lockd.conf
/etc/nfs.conf
/etc/nfsmount.conf
/etc/request-key.d/id_resolver.conf
/etc/sysconfig/nfs
/sbin/mount.nfs
/sbin/mount.nfs4
/sbin/osd_login
/sbin/rpc.statd
/sbin/umount.nfs
/sbin/umount.nfs4
/usr/lib/systemd/scripts/nfs-utils_env.sh
/usr/lib/systemd/system-generators/nfs-server-generator
/usr/lib/systemd/system-generators/rpc-pipefs-generator
/usr/lib/systemd/system/auth-rpcgss-module.service
/usr/lib/systemd/system/nfs-blkmap.service
/usr/lib/systemd/system/nfs-client.target
/usr/lib/systemd/system/nfs-config.service
/usr/lib/systemd/system/nfs-idmap.service
/usr/lib/systemd/system/nfs-idmapd.service
/usr/lib/systemd/system/nfs-lock.service
/usr/lib/systemd/system/nfs-mountd.service
/usr/lib/systemd/system/nfs-secure.service
/usr/lib/systemd/system/nfs-server.service
/usr/lib/systemd/system/nfs-utils.service
/usr/lib/systemd/system/nfs.service
/usr/lib/systemd/system/nfslock.service
/usr/lib/systemd/system/proc-fs-nfsd.mount
/usr/lib/systemd/system/rpc-gssd.service
/usr/lib/systemd/system/rpc-statd-notify.service
/usr/lib/systemd/system/rpc-statd.service
/usr/lib/systemd/system/rpc_pipefs.target
/usr/lib/systemd/system/rpcgssd.service
/usr/lib/systemd/system/rpcidmapd.service
/usr/lib/systemd/system/var-lib-nfs-rpc_pipefs.mount
/usr/sbin/blkmapd
/usr/sbin/exportfs
/usr/sbin/mountstats
/usr/sbin/nfsdcltrack
/usr/sbin/nfsidmap
/usr/sbin/nfsiostat
/usr/sbin/nfsstat
/usr/sbin/rpc.gssd
/usr/sbin/rpc.idmapd
/usr/sbin/rpc.mountd
/usr/sbin/rpc.nfsd
/usr/sbin/rpcdebug
/usr/sbin/showmount
/usr/sbin/sm-notify
/usr/sbin/start-statd
/usr/share/doc/nfs-utils-1.3.0
/usr/share/doc/nfs-utils-1.3.0/ChangeLog
/usr/share/doc/nfs-utils-1.3.0/KNOWNBUGS
/usr/share/doc/nfs-utils-1.3.0/NEW
/usr/share/doc/nfs-utils-1.3.0/README
/usr/share/doc/nfs-utils-1.3.0/THANKS
/usr/share/doc/nfs-utils-1.3.0/TODO
/usr/share/man/man5/exports.5.gz
/usr/share/man/man5/nfs.5.gz
/usr/share/man/man5/nfs.conf.5.gz
/usr/share/man/man5/nfsmount.conf.5.gz
/usr/share/man/man7/nfs.systemd.7.gz
/usr/share/man/man7/nfsd.7.gz
/usr/share/man/man8/blkmapd.8.gz
/usr/share/man/man8/exportfs.8.gz
/usr/share/man/man8/gssd.8.gz
/usr/share/man/man8/idmapd.8.gz
/usr/share/man/man8/mount.nfs.8.gz
/usr/share/man/man8/mountd.8.gz
/usr/share/man/man8/mountstats.8.gz
/usr/share/man/man8/nfsd.8.gz
/usr/share/man/man8/nfsdcltrack.8.gz
/usr/share/man/man8/nfsidmap.8.gz
/usr/share/man/man8/nfsiostat.8.gz
/usr/share/man/man8/nfsstat.8.gz
/usr/share/man/man8/rpc.gssd.8.gz
/usr/share/man/man8/rpc.idmapd.8.gz
/usr/share/man/man8/rpc.mountd.8.gz
/usr/share/man/man8/rpc.nfsd.8.gz
/usr/share/man/man8/rpc.sm-notify.8.gz
/usr/share/man/man8/rpc.statd.8.gz
/usr/share/man/man8/rpcdebug.8.gz
/usr/share/man/man8/showmount.8.gz
/usr/share/man/man8/sm-notify.8.gz
/usr/share/man/man8/statd.8.gz
/usr/share/man/man8/umount.nfs.8.gz
/var/lib/nfs
/var/lib/nfs/etab
/var/lib/nfs/rmtab
/var/lib/nfs/rpc_pipefs
/var/lib/nfs/statd
/var/lib/nfs/statd/sm
/var/lib/nfs/statd/sm.bak
/var/lib/nfs/state
/var/lib/nfs/v4recovery
/var/lib/nfs/xtab
```
其中包含了一些 nfs 相关的服务，服务的详细介绍如下:
| 服务名称               | 功能描述                  | 详细说明																  |
|-----------------------|--------------------------|----------------------------------------------------------------------|
| `nfs-blkmap.service`  | 支持块映射功能             | 用于支持在 NFS 文件系统上的块映射，适合复杂的存储需求，普通 NFS 共享中很少使用。 |
| `nfs-idmapd.service`  | 提供用户和组的 ID 映射功能  | NFSv4 所需，负责将用户名和组名映射为 UID 和 GID，用于身份验证。 |
| `nfs-lock.service`    | 提供文件锁定功能           | 支持 NFSv3 和之前版本的文件锁定，防止多个客户端同时写入导致的数据损坏。 |
| `nfs-mountd.service`  | 处理客户端挂载请求          | NFSv3 所需的挂载守护进程，用于管理客户端挂载请求并验证权限。 |
| `nfs-secure.service`  | 提供安全的 RPC 服务        | 提供基于 Kerberos 认证的安全 RPC 连接，适合需要身份验证的 NFS 环境。 |
| `nfs.service`         | NFS 客户端和服务器主服务    | 核心服务，通过 `nfs-utils` 包提供基本的 NFS 客户端和服务器功能。 |
| `nfs-config.service`  | 加载和应用 NFS 配置        | 启动 NFS 服务时自动加载配置文件，确保系统配置是最新的。 |
| `nfs-idmap.service`   | NFSv4 的用户 ID 映射      | 与 `nfs-idmapd` 类似，NFSv4 环境中用于将用户和组 ID 映射，以匹配权限。 |
| `nfs-rquotad.service` | 远程磁盘配额管理           | 允许设置和查询 NFS 共享中的磁盘配额，适合需要磁盘配额管理的场景。 |
| `nfs-server.service`  | NFS 服务器主服务          | 负责处理服务器端的 NFS 共享请求，是 NFS 服务的核心。 |
| `nfs-utils.service`   | 管理 NFS 的核心工具包      | `nfs-utils` 提供 NFS 的工具和守护进程，包括 `nfsd` 和 `rpcbind`。 |

2. 我们这里只模拟下简单的挂载，所以启动 nfs-server 服务并设置开机自启动（服务器端操作）
``` bash
systemctl start nfs-server
systemctl enable nfs-server
```

3. 配置共享目录 （服务器端操作）
  - 创建共享目录,例如目录为/mnt/nfs_share:
``` bash
mkdir /mnt/nfs_share
```
  - 设置权限，允许其他用户访问:
``` bash
chmod -R 777 /mnt/nfs_share
```
  - 编辑 /etc/exports 文件以配置共享路径及其权限。在文件末尾添加以下内容：
``` bash
/mnt/nfs_share *(rw,sync,no_root_squash)
```
``` bash
这里的配置选项含义如下：
	•	* 表示允许所有 IP 访问。如果需要限制特定 IP，可以替换为特定的 IP 地址或子网。
	•	rw 表示读写权限。
	•	sync 确保数据同步写入。
	•	no_root_squash 允许客户端以 root 身份访问。
```

4. 应用 NFS 配置 （服务器端操作）
``` bash
exportfs -r
```
应用完成后，可以查看:
``` bash
[root@node2 mnt]# exportfs -s
/mnt/nfs_share  *(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

5. 配置防火墙,确保防火墙允许 NFS 服务的流量通过。可以使用以下命令打开相应的端口(服务器端操作，测试环境可以直接关闭)：
``` shell
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

6. 客户端进行挂载
```
mkdir /mnt/test
mount -t nfs serverip:/mnt/nfs_share /mnt/test
```
挂载完成后，通过 df 查看:
``` bash
[root@node1 ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/sda3                     1.1T  257G  787G  25% /
tmpfs                         126G  2.3M  126G   1% /tmp
node2:/mnt/nfs_share          1.1T  103G  941G  10% /mnt/test
```
至此，nfs 部署和挂载就完成了

# 测试

我们可以在客户端的挂载目录上，进行文件读写,比如创建文件，写入以及读取等
``` bash 
[root@node1 ~]# cd /mnt/test/
[root@node1 test]# ls
[root@node1 test]# touch test.txt
[root@node1 test]# echo 123 > test.txt
[root@node1 test]# cat test.txt
123
```
在服务器端的 /mnt/nfs_share目录下可以看到同样的文件信息

