---
title: Nginx详解
author: baixiaozhou
categories:
  - 反向代理
tags:
  - Nginx
  - 反向代理
  - 负载均衡
description: Nginx 使用介绍
repo: nginx/nginx
date: 2024-09-12 10:47:54
references:
  - '[nginx github](https://github.com/nginx/nginx)'
  - '[MIME 类型（IANA 媒体类型）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)'
cover: https://nginx.org/nginx.png
banner: https://nginx.org/nginx.png
---

<!-- Your content starts here -->
# Nginx 简介

Nginx 是一个高性能的 HTTP 和反向代理服务器，同时也可以作为 IMAP/POP3 邮件代理服务器。Nginx 以其轻量级、速度快、并发处理能力强等特点而闻名，广泛用于网站架构中作为反向代理、负载均衡器和静态内容服务器。

## Nginx 的主要特点

1. **高并发处理能力**：
   - Nginx 采用异步、事件驱动的架构，与传统的 Apache 等基于进程或线程的模型不同。它可以同时处理大量并发连接，具有很高的性能和较低的资源消耗。

2. **反向代理**：
   - Nginx 作为反向代理服务器时，可以将客户端的请求转发给后端服务器（如 Apache、Tomcat、Node.js 等），然后将后端的响应返回给客户端。常用于负载均衡、缓存加速和安全增强。

3. **负载均衡**：
   - Nginx 支持多种负载均衡算法，如轮询（Round Robin）、最少连接（Least Connections）、IP 哈希等，能够将请求分发到多个后端服务器以提高应用的可靠性和处理能力。

4. **静态内容服务**：
   - Nginx 在处理静态文件（如 HTML、CSS、JS、图片等）方面非常高效，常用于静态文件托管。它可以作为内容分发网络（CDN）的一部分来加速网页加载。

5. **模块化设计**：
   - Nginx 支持模块化设计，允许根据需要启用或禁用功能模块。可以通过模块扩展 Nginx 的功能，如 Lua、SSL、缓存等。

6. **内存消耗低**：
   - Nginx 的内存占用相对较低，适合高并发环境下的稳定运行。其设计目标是处理大量的请求而不占用过多的资源。

7. **高可扩展性**：
   - 通过编写自定义模块，Nginx 可以与多种语言（如 Lua、Python、Go）集成，极大地增强了其功能和灵活性。OpenResty 就是基于 Nginx 的一个扩展框架，集成了 Lua 来支持动态处理。

8. **HTTPS/SSL 支持**：
   - Nginx 具备完整的 HTTPS 支持，可以通过简单的配置启用 SSL/TLS 加密，从而提高网站的安全性。

## Nginx 的常见使用场景

1. **静态内容服务**：
   - Nginx 非常适合用于提供网站的静态资源，如 HTML 文件、图片、CSS、JavaScript 文件等。

2. **反向代理和负载均衡**：
   - Nginx 常用于前端反向代理，将客户端的请求转发给后端服务器集群。通过负载均衡算法，Nginx 能有效分配请求并提高后端服务器的利用率和可靠性。

3. **API 网关**：
   - 在微服务架构中，Nginx 常作为 API 网关来路由和管理 API 请求，提供身份验证、速率限制等功能。

4. **缓存服务器**：
   - Nginx 可以作为缓存代理，缓存静态和动态内容，提高性能并减少后端服务器的负担。

5. **邮件代理服务器**：
   - 除了 HTTP 服务器，Nginx 还可以用作 IMAP/POP3/SMTP 代理服务器，处理邮件请求并进行负载均衡。

## Nginx vs. Apache

- **架构**：
  - Nginx 是基于事件驱动的异步架构，适合高并发；Apache 使用的是基于进程/线程的架构，每个请求占用一个进程或线程，容易在高并发场景下消耗大量资源。

- **性能**：
  - 在处理大量静态内容和高并发请求时，Nginx 的表现优于 Apache，且内存消耗更少。

- **配置和模块**：
  - Nginx 的模块在编译时确定，而 Apache 支持运行时动态加载模块，因此 Apache 在模块化灵活性方面稍胜一筹。

## 整体结构

{% image https://pic3.zhimg.com/v2-1cd1dab80fc4b50113439120ed54ff22_r.jpg Nginx 整体框架 fancybox:true %}


# 部署安装

## 安装
1. centos:
``` shell
yum install -y nginx
systemctl start nginx
systemctl enable nginx
```
2. docker
``` shell
docker pull nginx
docker run -d --name nginx -p 8080:80 nginx
```

## 组件

我们先看一下 nginx 包中提供的相关组件:
``` bash
## 配置
/etc/logrotate.d/nginx
/etc/nginx/fastcgi.conf
/etc/nginx/fastcgi.conf.default
/etc/nginx/fastcgi_params
/etc/nginx/fastcgi_params.default
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/mime.types
/etc/nginx/mime.types.default
/etc/nginx/nginx.conf
/etc/nginx/nginx.conf.default
/etc/nginx/scgi_params
/etc/nginx/scgi_params.default
/etc/nginx/uwsgi_params
/etc/nginx/uwsgi_params.default
/etc/nginx/win-utf
## 服务及程序
/usr/bin/nginx-upgrade
/usr/lib/systemd/system/nginx.service
/usr/lib64/nginx/modules
/usr/sbin/nginx
## 文档
/usr/share/doc/nginx-1.20.1
/usr/share/doc/nginx-1.20.1/CHANGES
/usr/share/doc/nginx-1.20.1/README
/usr/share/doc/nginx-1.20.1/README.dynamic
/usr/share/doc/nginx-1.20.1/UPGRADE-NOTES-1.6-to-1.10
/usr/share/licenses/nginx-1.20.1
/usr/share/licenses/nginx-1.20.1/LICENSE
/usr/share/man/man3/nginx.3pm.gz
/usr/share/man/man8/nginx-upgrade.8.gz
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx/html/404.html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/en-US
/usr/share/nginx/html/icons
/usr/share/nginx/html/icons/poweredby.png
/usr/share/nginx/html/img
/usr/share/nginx/html/index.html
/usr/share/nginx/html/nginx-logo.png
/usr/share/nginx/html/poweredby.png
/usr/share/nginx/modules
/usr/share/vim/vimfiles/ftdetect/nginx.vim
/usr/share/vim/vimfiles/ftplugin/nginx.vim
/usr/share/vim/vimfiles/indent/nginx.vim
/usr/share/vim/vimfiles/syntax/nginx.vim
/var/lib/nginx
/var/lib/nginx/tmp
## 日志
/var/log/nginx
/var/log/nginx/access.log
/var/log/nginx/error.log
```

## 命令操作

我们可以看下 nginx 的命令操作和服务配置：
``` bash
[root@node1 ~]# /usr/sbin/nginx -h
nginx version: nginx/1.20.1
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/share/nginx/)
  -e filename   : set error log file (default: /var/log/nginx/error.log)
  -c filename   : set configuration file (default: /etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file

[root@node1 ~]# systemctl cat nginx.service
# /usr/lib/systemd/system/nginx.service
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
nginx 命令提供的参数非常简单，包括检查配置文件、发送信号、配置文件路径、error 日志路径等。
``` bash
nginx -t              # 检查配置文件是否有语法错误
nginx -s reload       # 热加载，重新加载配置文件
nginx -s stop         # 快速关闭
nginx -s quit         # 等待工作进程处理完成后关闭
```

# 配置详解

我们先看一下 nginx 默认的配置文件内容:
``` nginx
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```
我们可以看到配置文件基本上被分为了三部分，我们可以划分为全局块，event 块和 http 块

## 全局块

配置文件中是一些全局的基本配置, 除此之外还包括了一些其他的配置，主要参数如下:
| 参数名                  | 含义    |示例                              |
|-------------------------|---------------------------------------------------|-----------------------------------|
| `user`                  | 指定运行 Nginx 的用户和用户组                         | `user www-data;`                 |
| `worker_processes`       | 设置 Nginx 启动的工作进程数，通常根据 CPU 核心数设置    | `worker_processes auto;`         |
| `error_log`             | 定义错误日志文件路径和日志级别                         | `error_log /var/log/nginx/error.log warn;` |
| `pid`                   | 定义存储 Nginx 主进程 PID 的文件路径                  | `pid /var/run/nginx.pid;`         |
| `worker_rlimit_nofile`   | 设置工作进程可打开的最大文件描述符数                   | `worker_rlimit_nofile 1024;`      |
| `worker_priority`        | 设置工作进程的优先级，负值表示更高优先级                | `worker_priority -5;`            |
| `include`               | 包含其他配置文件，用于模块化管理配置                    | `include /etc/nginx/conf.d/*.conf;` |
| `daemon`                | 控制 Nginx 是否以守护进程方式运行，`on` 为默认值        | `daemon on;`                     |
| `worker_cpu_affinity`    | 绑定 Nginx 工作进程到特定 CPU 核心，提升并行处理性能    | `worker_cpu_affinity auto;`       |
| `worker_shutdown_timeout`| 设置工作进程终止时的最大等待时间                       | `worker_shutdown_timeout 10s;`    |
| `lock_file`             | 定义锁文件路径，用于管理 `accept_mutex` 下的进程锁      | `lock_file /var/run/nginx.lock;`  |
| `env`                   | 设置环境变量                                        | `env OPENSSL_CONF=/etc/ssl/openssl.cnf;` |

### Nginx进程的工作模式

我们先通过`ps`看一下 nginx 进程的详细情况:
```
[root@node1 nginx]# ps -auxf | grep nginx | grep -v grep
root     20588  0.0  0.0  39308  1064 ?        Ss   15:11   0:00 nginx: master process /usr/sbin/nginx
nginx    20589  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20590  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20591  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20592  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20593  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20594  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20595  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20596  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20597  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20598  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20599  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20600  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
nginx    20601  0.0  0.0  39696  1904 ?        S<   15:11   0:00  \_ nginx: worker process
.....
```
这里我们可以看到 nginx 有一个 master 进程和多个 worker 进程，这里worker 进程的数量取决于配置文件中 `worker_processes`的配置。主进程的主要目的是读取和评估配置，以及维护工作进程。工作进程实际处理请求。nginx采用基于事件的模型和依赖于操作系统的机制来有效地在工作进程之间分发请求。

一旦主进程收到重新加载配置的信号，它会检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果成功，主进程将启动新的工作进程，并向旧的工作进程发送消息，请求它们关闭。否则，主进程回滚更改并继续使用旧配置。旧的工作进程，收到命令关闭，停止接受新的连接，并继续服务当前的请求，直到所有这些请求都得到服务。之后，旧的worker进程退出。

## event 块

Nginx events 模块的配置参数用于控制 Nginx 如何处理并发连接以及选择合适的事件模型。主要涉及 Nginx 的网络事件处理机制，帮助优化高并发性能。

以下是 Nginx events 模块的参数及其含义：
| 参数名                  | 含义             |示例                              |
|-------------------------|--------------------------------------------------------------|-----------------------------------|
| `worker_connections`     | 每个工作进程能够处理的最大连接数，直接影响并发连接数量         | `worker_connections 1024;`       |
| `use`                   | 指定使用的事件驱动模型。通常根据操作系统选择适合的模型         | `use epoll;`（Linux）             |
| `multi_accept`           | 是否启用一次接受尽可能多的连接，`on` 表示接受所有可用连接       | `multi_accept on;`               |
| `accept_mutex`           | 是否启用接受连接的互斥锁，避免多个工作进程同时抢占新连接       | `accept_mutex on;`               |
| `accept_mutex_delay`     | 在 `accept_mutex` 启用时，设置工作进程获取新连接前的等待时间    | `accept_mutex_delay 500ms;`      |
| `debug_connection`       | 启用特定 IP 地址的连接调试日志，便于诊断连接问题              | `debug_connection 192.168.1.1;`  |

参数请参考: https://github.com/nginx/nginx/blob/00637cce366f17b78fe1ed5c1ef0e534143045f6/src/event/ngx_event.c

## http 块

用于配置 HTTP 协议下的相关行为。该块通常位于 nginx.conf 配置文件中，负责处理 HTTP 请求的行为、代理、缓存、日志等。http块中可以包含多个server块，server块也可以包含多个location块。

### 1.全局配置参数

| 参数                      | 说明                                                                   |
|---------------------------|-----------------------------------------------------------------------|
| `include`                 | 引入外部配置文件，便于模块化管理配置。                                      |
| `default_type`            | 为那些在 mime.types 文件中没有特定映射的文件设置默认 MIME 类型               ｜
| `sendfile`                | 启用 `sendfile` 以提高文件传输效率，适用于静态资源的传输。                    |
| `tcp_nopush`              | 提高 TCP 数据包发送效率，常与 `sendfile` 一起使用，用于减少网络延迟。          |
| `tcp_nodelay`             | 在保持长连接时，尽快将数据发送到客户端，减少数据延迟。                         |
| `keepalive_timeout`       | 设置保持连接的超时时间，控制空闲连接的保持时间。                              |
| `server_tokens`           | 控制是否显示 NGINX 版本信息。关闭可以增加安全性，避免暴露版本细节。             |
| `client_max_body_size`    | 限制客户端请求的最大内容大小，通常用于限制上传文件的大小。                      |
| `client_body_timeout`     | 设置读取客户端请求体的超时时间。如果在指定时间内无法读取完成，请求将被关闭。       |
| `reset_timedout_connection`| 启用后，连接超时会主动发送 RST 包关闭连接，而不是等待超时。                   |

关于 mime types部分：
``` bash
include             /etc/nginx/mime.types;      # 这里引用了一个文件，文件里面指定了一些内容映射
default_type        application/octet-stream;   # 默认类型

## 我们可以看下 mime.types的一些映射
types {
    text/html                                        html htm shtml;
    text/css                                         css;
    text/xml                                         xml;
    image/gif                                        gif;
    image/jpeg                                       jpeg jpg;
    application/javascript                           js;
    application/atom+xml                             atom;
    application/rss+xml                              rss;
    ......
}
```
大家可以参考这篇文章:[MIME 类型（IANA 媒体类型）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

### 2.日志管理
| 参数             | 说明                                                   |
|------------------|-------------------------------------------------------|
| `access_log`    | 设置访问日志的路径和格式，用于记录所有客户端请求的详细信息。     |
| `log_format`    | 定义日志格式，可以自定义哪些信息会写入访问日志。               |
| `error_log`     | 指定错误日志的路径及日志级别，记录服务器运行过程中发生的错误。   |

``` bash
# 在全局 可以定义不同的日志格式类型，比如
log_format  maina  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
log_format  mainb  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "xxx"' ;
# 在使用的时候就可以指定类型，比如有两个服务， servicea,serviceb,分别使用两个不同格式的日志
access_log  /var/log/nginx/servicea.log  maina;
access_log  /var/log/nginx/serviceb.log  mainb;
```

### 3.代理配置

| 参数                     | 说明                     |作用域              |
|--------------------------|----------------------------------------------------------------------|---------------------|
| `proxy_buffers`           | 设置代理响应的缓冲区数量和大小。                                      | `http`, `server`, `location` |
| `proxy_buffer_size`       | 设置代理服务器用于读取响应头的缓冲区大小。                            | `http`, `server`, `location` |
| `proxy_connect_timeout`   | 设置与上游服务器建立连接的超时时间。                                  | `http`, `server`, `location` |
| `proxy_hide_header`       | 隐藏来自上游服务器的指定 HTTP 响应头。                                | `http`, `server`, `location` |
| `proxy_http_version`      | 指定代理使用的 HTTP 版本（如 1.0 或 1.1）。                           | `http`, `server`, `location` |
| `proxy_ignore_client_abort`| 当客户端中止请求时，是否让 NGINX 继续代理到上游服务器。               | `http`, `server`, `location` |
| `proxy_max_temp_file_size` | 设置用于临时文件存储的最大大小。                                      | `http`, `server`, `location` |
| `proxy_next_upstream`     | 定义上游服务器发生错误时是否将请求转发到下一个上游服务器。             | `http`, `server`, `location` |
| `proxy_next_upstream_tries`| 设置最大尝试数，当上游服务器出错时可转发到下一个服务器。              | `http`, `server`, `location` |
| `proxy_pass`              | 定义代理请求的上游服务器地址。                                        | `location`           |
| `proxy_read_timeout`      | 设置读取上游服务器响应的超时时间。                                    | `http`, `server`, `location` |
| `proxy_redirect`          | 重写上游服务器返回的 `Location` 或 `Refresh` 头中的地址。             | `http`, `server`, `location` |
| `proxy_request_buffering` | 设置是否将客户端请求的主体缓存在代理服务器中。                        | `http`, `server`, `location` |
| `proxy_send_timeout`      | 设置发送请求到上游服务器时的超时时间。                                | `http`, `server`, `location` |
| `proxy_set_header`        | 设置在代理请求中发送的 HTTP 请求头。                                  | `http`, `server`, `location` |
| `proxy_temp_path`         | 设置存储代理临时文件的目录。                                          | `http`, `server`, `location`   |

### 4. 连接和请求控制   

| 参数                | 作用                            | 作用域                 |
|---------------------|--------------------------------|------------------------|
| `limit_conn_zone`    | 定义共享区域以限制连接数           | `http`                 |
| `limit_rate`         | 限制响应传输速率                  | `http`, `server`, `location` |
| `limit_req_zone`     | 定义共享区域以限制请求速率         | `http`                 |
| `limit_upload_rate`  | 限制上传传输速率                  | `http`, `server`, `location` |
| `limit_conn`        | 限制客户端的连接数                 | `http`, `server`, `location` |
| `limit_req`        | 限制请求速率                       | `http`, `server`, `location` |
| `limit_rate`       | 限制客户端的下载或上传速率            | `http`, `server`, `location` |
| `keepalive_requests`| 设置一个连接在关闭之前允许的最大请求数 | `http`, `server`, `location` |
| `keepalive_timeout`| 设置 Keep-Alive 连接的超时时间      | `http`, `server`, `location` |

### 5.压缩和请求

| 参数                    | 说明                                              |作用域                   |
|-------------------------|----------------------------------------------------------------------------------------------|--------------------------|
| `gzip`                  | 启用 Gzip 压缩。                                   |`http`, `server`, `location` |
| `gzip_min_length`       | 设置启用压缩的响应体最小长度，低于此长度的响应不会被压缩。 |`http`, `server`, `location` |
| `gzip_buffers`          | 设置用于存储压缩数据的缓冲区数量和大小。                |`http`, `server`, `location` |
| `gzip_http_version`     | 设置支持的最小 HTTP 版本，低于该版本的请求不会使用 Gzip 压缩。                                        | `http`, `server`, `location` |
| `gzip_comp_level`       | 设置 Gzip 压缩级别，范围从 1 到 9，数字越大压缩率越高，资源消耗也越大。                                 | `http`, `server`, `location` |
| `gzip_vary`             | 启用 `Vary: Accept-Encoding` 头，指示代理和浏览器缓存应根据 Accept-Encoding 头处理不同的响应。         | `http`, `server`, `location` |
| `gzip_types`            | 指定需要压缩的 MIME 类型。                            |`http`, `server`, `location` |
| `proxy_cache`           | 启用代理缓存，缓存上游服务器的响应。                                 | `http`, `server`, `location` |
| `proxy_cache_path`      | 定义缓存路径和缓存相关的控制参数，如缓存大小和策略。                    | `http`                   |
| `proxy_cache_valid`     | 设置缓存的有效期，不同状态码可设置不同的缓存时间。                      | `http`, `server`, `location` |

### Server块

作用: 
  - server 块定义了一个虚拟主机配置。每个 server 块处理指定的域名或 IP 地址的请求，并根据配置响应客户端请求。可以在一个 http 块中定义多个 server 块。server 块中可以包含多个 location 块来处理不同的请求路径。
  
用途:
 -	配置特定域名或 IP 的虚拟主机，包括监听端口、域名、根目录等。
 -	设置特定于虚拟主机的配置，例如访问控制、SSL/TLS 设置、错误页面等。
 -	定义处理请求的 location 块，用于细化请求的路由和处理

 支持参数:
| 参数                     | 描述                                    | 示例                                         |
|--------------------------|----------------------------------------|----------------------------------------------|
| `listen`                 | 定义 `nginx` 监听的 IP 地址和端口号        |`listen 80;`                                 |
| `server_name`            | 指定虚拟主机的域名或 IP 地址               |`server_name example.com;`                   |
| `root`                   | 设置服务器的根目录                        |`root /var/www/html;`                        |
| `index`                  | 指定默认的首页文件                        |`index index.html;`                          |
| `location`               | 定义匹配 URI 的块                        |`location / { try_files $uri $uri/ =404; }`  |
| `error_page`             | 自定义错误页面                           |`error_page 404 /404.html;`                  |
| `return`                 | 返回指定的状态码或重定向                   |`return 301 https://$host$request_uri;`      |
| `rewrite`                | 重写 URL                                |`rewrite ^/old-path/(.*)$ /new-path/$1;`     |
| `proxy_pass`             | 代理请求到另一个服务器                     |`proxy_pass http://backend_server;`          |
| `ssl`                    | 启用 HTTPS 支持                         |`listen 443 ssl;`                            |
| `ssl_certificate`        | SSL 证书文件路径                         |`ssl_certificate /etc/nginx/ssl/example.crt;`|
| `ssl_certificate_key`    | SSL 证书私钥文件路径                      |`ssl_certificate_key /etc/nginx/ssl/example.key;`|
| `allow`                  | 允许指定 IP 地址访问                      |`allow 192.168.1.0/24;`                      |
| `deny`                   | 拒绝指定 IP 地址访问                      |`deny 192.168.1.100;`                        |
| `client_max_body_size`    | 限制客户端请求体的最大大小                 |`client_max_body_size 10M;`                  |
| `client_body_timeout`     | 请求体接收超时时间                       |`client_body_timeout 60s;`                   |
| `keepalive_timeout`       | 设置 `keep-alive` 连接超时时间           |`keepalive_timeout 65;`                      |
| `send_timeout`            | 发送响应超时时间                         |`send_timeout 30s;`                          |
| `expires`                | 设置缓存过期时间                          |`expires 30d;`                               |
| `proxy_cache`            | 启用代理缓存                              |`proxy_cache my_cache;`                      |
| `add_header`             | 添加自定义 HTTP 响应头                    |`add_header X-Frame-Options SAMEORIGIN;`     |

 示例:
``` json
server {
    listen       80;
    server_name  testa.com;

    charset utf-8;
    #include mime.types;

    access_log  /var/logs/nginx/servivea.log maina;
    large_client_header_buffers 4 8k;

    location / {
        proxy_ignore_client_abort on;
        proxy_connect_timeout 5s;
        proxy_read_timeout 60s;
        proxy_send_timeout 10s;
        client_body_timeout 60;
        client_max_body_size 20m;
        proxy_request_buffering off;
        proxy_buffers 64 256k;
        proxy_next_upstream error timeout http_502 http_504;
        proxy_set_header X-Forward-For $remote_addr;
        proxy_pass http://servicea;
        proxy_set_header Host $http_host;
    }
}
```

# 实战配置


