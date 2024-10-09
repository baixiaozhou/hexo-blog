---
title: ansible详细介绍
author: baixiaozhou
categories:
  - 运维工具
tags:
  - ansible
  - 运维工具
description:  本文将对 ansible 工具的具体使用做一个详细的介绍
repo: ansible/ansible
date: 2024-08-19 19:09:34
banner: /images/ansible-logo.png
cover: /images/ansible-logo.png
references:
  - '[ansible 英文文档](https://docs.ansible.com/ansible/latest/index.html)'
  - '[ansible 中文文档](http://www.ansible.com.cn/docs/intro_adhoc.html)'
---

<!-- Your content starts here -->

{% quot 官方文档 %}

[英文文档](https://docs.ansible.com/ansible/latest/index.html)
[中文文档](http://www.ansible.com.cn/docs/intro_adhoc.html)

# ansible 介绍

我们先看一下 ansible 的相关介绍:
```
Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy and maintain. Automate everything from code deployment to network configuration to cloud management, in a language that approaches plain English, using SSH, with no agents to install on remote systems. https://docs.ansible.com.
```

这里有几点核心:

1. ansible 是一个自动化平台
2. ansible 使用 ssh 协议（通过密码或者密钥等方式进行访问），部署简单，没有客户端，只需在主控端部署 Ansible 环境，无需在远程系统上安装代理程序
3. 模块化：调用特定的模块，完成特定任务
4. 支持自定义扩展

每开发一个工具或者平台的时候，这些工具和平台提供了各种各样的功能，那么我们能用 ansible 来干什么呢？ 下面就是一些 ansible 核心的功能介绍：

| 功能                       | 描述                                                                                           | 常见场景示例                                                                                   |
|----------------------------|------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| 配置管理                   | 自动化管理服务器和设备的配置，确保它们处于期望的状态。                                         | 自动化安装和配置 Web 服务器，确保所有服务器配置一致。                                          |
| 应用部署                   | 自动化应用程序的部署过程，从代码库拉取到在服务器上部署和配置应用。                             | 自动部署多层 Web 应用程序，包括数据库设置、应用服务部署、负载均衡配置等。                      |
| 持续交付与持续集成（CI/CD） | 与 CI/CD 工具集成，实现自动化的构建、测试和部署流程。                                           | 代码提交后自动执行测试、构建容器镜像，并部署到 Kubernetes 集群中。                              |
| 基础设施即代码（IaC）       | 编写和管理基础设施的配置文件，使其像管理代码一样。                                               | 使用 Ansible Playbooks 定义云环境资源配置，实现可重复的基础设施部署。                           |
| 云管理                     | 自动化云服务资源的管理，包括虚拟机、存储、网络等的创建和配置。                                   | 自动化创建和管理 AWS EC2 实例、配置 VPC 和安全组。                                              |
| 网络自动化                 | 管理和配置网络设备，使得大规模的网络设备配置变得简单和一致。                                     | 自动化配置多台交换机的 VLAN 设置和路由协议。                                                    |
| 安全与合规                 | 自动化安全补丁的部署、系统安全配置的强化以及合规性检查。                                         | 自动化应用系统安全补丁，配置防火墙规则，执行安全扫描和合规性检查。                              |
| 多平台环境管理             | 支持多种操作系统，在混合环境中统一管理平台上的配置和应用。                                       | 在混合的 Linux 和 Windows 服务器环境中，统一部署和配置监控软件。                                |
| 灾难恢复                   | 自动化灾难恢复流程，如备份、数据恢复和服务恢复。                                                 | 自动化数据库备份并在需要时恢复和重建数据库服务。                                                |
| 任务调度与批量操作         | 通过 Playbooks 调度定期任务或对大量服务器执行批量操作。                                           | 定期清理服务器上的临时文件或批量更新多个服务器的操作系统。                                      |

## ansible的工作机制

Ansible 在管理节点将 Ansible 模块通过 SSH 协议推送到被管理端执行，执行完之后自动删除，可以使用 SVN 等来管理自定义模块及编排

{% image /images/ansible-diagram.png ansible结构 fancybox:true %}

从这张图中我们可以看到，ansible 由以下模块组成
```
Ansible： ansible的核心模块
Host Inventory：主机清单，也就是被管理的主机列表
Playbooks：ansible的剧本，可想象为将多个任务放置在一起，一块执行
Core Modules：ansible的核心模块
Custom Modules：自定义模块
Connection Plugins：连接插件，用于与被管控主机之间基于SSH建立连接关系
Plugins：其他插件，包括记录日志等
```

## ansible 安装

在 centos 环境下，我们可以通过：
```
yum install ansible -y 进行安装
```
安装完成后我们看下 ansible 提供了哪些命令:
| 命令               | 描述                                                                                     | 示例                                                |
|--------------------|------------------------------------------------------------------------------------------|-----------------------------------------------------|
| `ansible`          | 用于在一个或多个主机上运行单个模块（通常用于临时命令）。                                 | `ansible all -m ping`                               |
| `ansible-playbook` | 用于运行 Ansible playbook 文件，是 Ansible 的核心命令之一。                               | `ansible-playbook site.yml`                         |
| `ansible-galaxy`   | 用于管理 Ansible 角色和集合。可以下载、创建和分享角色。                                   | `ansible-galaxy install geerlingguy.apache`         |
| `ansible-vault`    | 用于加密和解密敏感数据（如密码、密钥）。                                                  | `ansible-vault encrypt secrets.yml`                 |
| `ansible-doc`      | 显示 Ansible 模块的文档和示例用法。                                                       | `ansible-doc -l`                                    |
| `ansible-config`   | 用于查看、验证和管理 Ansible 配置文件。                                                   | `ansible-config list`                               |
| `ansible-inventory`| 管理和检索 Ansible inventory 信息。                                                       | `ansible-inventory --list -i inventory.yml`         |
| `ansible-pull`     | 用于从远程版本控制系统（如 Git）拉取 playbook 并在本地执行，常用于自动化部署。            | `ansible-pull -U https://github.com/username/repo.git` |
| `ansible-console`  | 提供一个交互式命令行接口，可用于动态执行 Ansible 任务和命令。                              | `ansible-console`                                   |

对两个比较常用的命令:`ansible` 和 `ansible-plabook` 做一下具体的介绍:

### ansible 命令

命令格式:
``` bash
ansible 组名 -m 模块名 -a '参数'
```
这里的组名是自定义的一系列组信息，组的定义在后面会讲到。

模块名是ansible提供的一些列支持模块，默认模块是 command，查看 ansible 支持的模块：
``` bash
ansible-doc -l #大概有 3000 多个
```
ansible涉及到的模块非常非常多，按照实际需要使用，初步先掌握一些比较常用的就可以

查看模块描述：
``` bash
ansible-doc -s 模块名称
```
实例:
``` bash
# 查看webserver 组机器的时间信息
ansible webserver -m shell -a "date" # 这里的组就是 webserver，shell 是模块名（比较常用的模块）date 是具体执行的命令
# 将本机的/tmp/test.sh 拷贝到其他机器上的 /etc目录下
ansible webserver -m copy -a "src=/tmp/test.sh dest=/etc/test.sh" # 这里的 copy 是模块名，里面的 src 是源路径，dest 是目标路径
```

### ansible-playbook命令
用于执行 Ansible Playbooks。Playbooks 是一系列任务的集合，用于自动化配置管理、应用部署、任务执行等操作。
基本语法:
``` bash
ansible-playbook [options] playbook.yml
```
命令帮助:
```
positional arguments:
  playbook              Playbook(s)

optional arguments:
  --ask-vault-pass      ask for vault password
  --flush-cache         clear the fact cache for every host in inventory
  --force-handlers      run handlers even if a task fails
  --list-hosts          outputs a list of matching hosts; does not execute
                        anything else
  --list-tags           list all available tags
  --list-tasks          list all tasks that would be executed
  --skip-tags SKIP_TAGS
                        only run plays and tasks whose tags do not match these
                        values
  --start-at-task START_AT_TASK
                        start the playbook at the task matching this name
  --step                one-step-at-a-time: confirm each task before running
  --syntax-check        perform a syntax check on the playbook, but do not
                        execute it
  --vault-id VAULT_IDS  the vault identity to use
  --vault-password-file VAULT_PASSWORD_FILES
                        vault password file
  --version             show program's version number, config file location,
                        configured module search path, module location,
                        executable location and exit
  -C, --check           don't make any changes; instead, try to predict some
                        of the changes that may occur
  -D, --diff            when changing (small) files and templates, show the
                        differences in those files; works great with --check
  -M MODULE_PATH, --module-path MODULE_PATH
                        prepend colon-separated path(s) to module library (def
                        ault=~/.ansible/plugins/modules:/usr/share/ansible/plu
                        gins/modules)
  -e EXTRA_VARS, --extra-vars EXTRA_VARS
                        set additional variables as key=value or YAML/JSON, if
                        filename prepend with @
  -f FORKS, --forks FORKS
                        specify number of parallel processes to use
                        (default=64)
  -h, --help            show this help message and exit
  -i INVENTORY, --inventory INVENTORY, --inventory-file INVENTORY
                        specify inventory host path or comma separated host
                        list. --inventory-file is deprecated
  -l SUBSET, --limit SUBSET
                        further limit selected hosts to an additional pattern
  -t TAGS, --tags TAGS  only run plays and tasks tagged with these values
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)

Connection Options:
  control as whom and how to connect to hosts

  --private-key PRIVATE_KEY_FILE, --key-file PRIVATE_KEY_FILE
                        use this file to authenticate the connection
  --scp-extra-args SCP_EXTRA_ARGS
                        specify extra arguments to pass to scp only (e.g. -l)
  --sftp-extra-args SFTP_EXTRA_ARGS
                        specify extra arguments to pass to sftp only (e.g. -f,
                        -l)
  --ssh-common-args SSH_COMMON_ARGS
                        specify common arguments to pass to sftp/scp/ssh (e.g.
                        ProxyCommand)
  --ssh-extra-args SSH_EXTRA_ARGS
                        specify extra arguments to pass to ssh only (e.g. -R)
  -T TIMEOUT, --timeout TIMEOUT
                        override the connection timeout in seconds
                        (default=10)
  -c CONNECTION, --connection CONNECTION
                        connection type to use (default=smart)
  -k, --ask-pass        ask for connection password
  -u REMOTE_USER, --user REMOTE_USER
                        connect as this user (default=None)

Privilege Escalation Options:
  control how and which user you become as on target hosts

  --become-method BECOME_METHOD
                        privilege escalation method to use (default=sudo), use
                        `ansible-doc -t become -l` to list valid choices.
  --become-user BECOME_USER
                        run operations as this user (default=root)
  -K, --ask-become-pass
                        ask for privilege escalation password
  -b, --become          run operations with become (does not imply password
                        prompting)
```

# ansible 基本使用

我们先看一个最基本的 ansible 组成:

{% image https://docs.ansible.com/ansible/latest/_images/ansible_inv_start.svg ansible基本组成 fancybox:true %}

如上图所示，大部分的 ansible 环境都包含以下三个组件:
1. 控制节点:安装了Ansible的系统。可以在控制节点上运行Ansible命令，例如ansible或ansible-inventory。
2. Inventory: 按逻辑组织的托管节点的列表。可以在控制节点上创建一个清单，以向Ansible描述主机部署。
3. 被管理节点: Ansible控制的远程系统或主机。

## Inventory文件

inventory 主要包括主机和组两个概念, 默认文件 `/etc/ansible/hosts`

### 主机
主机是 Ansible 可以管理的单个设备或虚拟机。主机可以是物理服务器、虚拟机、容器，甚至是网络设备（如路由器和交换机）。每个主机都有一个唯一的标识（通常是主机名或 IP 地址），并且可以通过 Ansible 的 inventory 文件或其他动态方法来定义
```
# 定义单个主机
web1.example.com

# 定义多个主机
web2.example.com
192.168.1.10

# 主机变量
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

### 组
组是主机的集合，可以对一组主机应用相同的配置或操作。组允许用户在多个主机上批量执行任务。一个主机可以属于多个组。组之间还可以嵌套，例如，你可以将所有 Web 服务器放在一个组中，然后将该组嵌套在一个更大的生产环境组中
``` ini
# 定义一下所有节点的信息
[all_hosts]
node1 public_ip=192.168.1.101 ansible_host=192.168.1.101 ansible_user=your_user ansible_ssh_pass=your_password ansible_port=22
node2 public_ip=192.168.1.102 ansible_host=192.168.1.102 ansible_user=your_user ansible_ssh_pass=your_password ansible_port=22
node3 public_ip=192.168.1.103 ansible_host=192.168.1.103 ansible_user=your_user ansible_ssh_pass=your_password ansible_port=22

# 定义一个 mysql 组，假如要在 node1 和 node2上部署 mysql
[mysql]
node1
node2

# 定义一个 nginx 组，假如要在 node1 和 node3上部署 nginx
[nginx]
node1
node3

# 假如我们还想部署一个 redis 组，mysql 部署在那里，redis 就部署在那里，重新写一遍很麻烦，那么我们可以把 redis 当做 mysql 的子集
# 这里有一个疑问，我们既然用一个一模一样的组再了，为啥搞个 children，直接用原来的组不行吗，这里可以是可以，但是从模块划分来看，做一下区分易于后续的管理
[redis:children]
mysql

# 定义组变量
[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

## 简单使用

在我们了解完 inventory 之后，我们开始做一些简单的模拟:

1. 在 node1 上安装 ansible，作为控制节点，在/etc/ansbile/hosts中加入三个节点的信息
2. 如果是通过密码连接的话，需要在 ansible_ssh_pass中输入机器密码，如果是通过密钥链接，这里可不填；配置 ssh 免密登录，可以自行百度，这里只需要配置 node1 到所有节点的免密（包括 node1 到 node1 自己）
3. 免密配置完成后，我们可以做一下简单的操作:
   ``` bash
   # 查看 mysql 组机器的时间信息
   ansilbe mysql -m shell -a "date"

   # 查看 nginx 组机器的启动时间
   ansible nginx -m shell -a "uptime"
   ```

# playbooks
playbook是由一个或多个play组成的列表，play的主要功能在于将事先归并为一组的主机装扮成事先通过ansible中的task定义好的角色。从根本上来讲，所谓的task无非是调用ansible的一个module。将多个play组织在一个playbook中，即可以让它们联合起来按事先编排的机制完成某一任务

## playbook语法:
```
playbook使用yaml语法格式，后缀可以是yaml,也可以是yml。

在单个playbook文件中，可以连续三个连子号(---)区分多个play。还有选择性的连续三个点好(...)用来表示play的结尾，也可省略。

次行开始正常写playbook的内容，一般都会写上描述该playbook的功能。

使用#号注释代码。

缩进必须统一，不能空格和tab混用。

缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行实现的。

YAML文件内容和Linux系统大小写判断方式保持一致，是区分大小写的，k/v的值均需大小写敏感

k/v的值可同行写也可以换行写。同行使用:分隔。

v可以是个字符串，也可以是一个列表

一个完整的代码块功能需要最少元素包括 name: task
```

## playbook核心元素

### 1. Play

Play 是 Playbook 的基本单元，用于定义在一组主机上执行的一系列任务。一个 Playbook 可以包含多个 Play，每个 Play 在不同的主机或组上执行不同的任务。

关键字：
- hosts: 指定要在哪些主机或主机组上执行 Play。
- tasks: 包含一系列任务，这些任务会按顺序执行。
- vars: 定义在 Play 中使用的变量。
- roles: 指定要应用的角色。
- gather_facts: 控制是否收集主机的事实信息（默认 true）。
- become: 是否使用 sudo 或其他特权提升执行任务。

### 2. Tasks

Tasks 是 Play 中的核心部分，定义了要执行的具体操作。每个 Task 通常使用一个 Ansible 模块，并可包含条件、循环、错误处理等。

关键字：
- name: 任务的描述性名称（可选，但推荐使用）。
-	action 或模块名称: 具体执行的操作，如 apt、yum、copy 等。
-	when: 定义条件，满足时才会执行任务。
-	with_items: 用于循环执行任务。
-	register: 保存任务的结果到变量。
-	ignore_errors: 忽略任务执行失败（设为 yes 时）。

### 3. Variables

Variables 是 Playbook 中的动态值，用于提高复用性和灵活性。可以在多个地方定义变量，如 vars、group_vars、host_vars、inventory 文件，或通过命令行传递。

关键字：
-	vars: 在 Play 或 Task 中定义变量。
-	vars_files: 引入外部变量文件。
-	vars_prompt: 运行时提示用户输入变量值。

### 4. Handlers

Handlers 是一种特殊类型的 Task，只会在被触发时执行。通常用于在配置更改后执行动作，如重启服务。比如我要等服务重启完检查端口监听，就可以用handler

关键字：
-	name: Handler 的名称。
-	notify: 在普通 Task 中调用 notify 触发对应的 Handler。

### 5. Roles
Roles 是 Playbook 中组织和复用任务、变量、文件、模板等的一种方式。Roles 使得 Playbook 更加模块化和可维护。举个例子，我现在要在服务器上部署各种各样的组件，webserver、mysql、redis、ng 等等，我们就可以用这个不同的 roles 来管理，我们可以在创建四个文件夹，分别对应起名 webserver、mysql、redis、ng，然后在这些文件夹里面添加服务部署或者更新需要的东西。

角色的目录结构：
```
my_role/
├── tasks/
│   └── main.yml
├── handlers/
│   └── main.yml
├── templates/
├── files/
├── vars/
│   └── main.yml
├── defaults/
│   └── main.yml
└── meta/
    └── main.yml
```

roles内各自目录含义：
- files	用来存放copy模块或script模块调用的文件
- templates	用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件
- tasks	此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件
- handlers	此目录应当包含一个main.yml文件，用于定义此角色中触发条件时执行的动作
- vars	此目录应当包含一个main.yml文件，用于定义此角色用到的变量
- defailts	此目录应当包含一个main.yml文件，用于为当前角色设定默认变量
- meta	此目录应当包含一个main.yml文件，用于定义此角色的特殊设及其依赖关系

### 6. Includes and Imports

Includes 和 Imports 用于在 Playbook 中包含其他任务、变量、文件等。import_tasks 和 include_tasks 的区别在于，import_tasks 在解析 Playbook 时执行，而 include_tasks 在运行时执行。

### 7. Templates

Templates 是使用 Jinja2 模板引擎的文件，用于动态生成配置文件或其他文件。模板通常存放在 templates/ 目录下，并通过 template 模块应用到目标主机

### 8. Tags

标签是用于对 play 进行标注，当你写了一个很长的playbook，其中有很多的任务，这并没有什么问题，不过在实际使用这个剧本时，你可能只是想要执行其中的一部分任务而已，或者，你只想要执行其中一类任务而已，而并非想要执行整个剧本中的全部任务，这时，我们可以借助tags模块为任务进行打标签操作，任务存在标签后，我们可以在执行playbook时利用标签，指定执行哪些任务，或者不执行哪些任务

比如说在实际线上环境中，我们有更新二进制包的操作，那么我们可以在更新二进制的相关 task 中添加名为bin 的 tags，有更新配置文件的操作，那么可以在相关的 tasks 中添加 conf 的 tags

### 完整示例
我们通过一个安装 nginx 的操作来完整演示一下:

设置项目目录结构:
```
.
├── ansible.cfg
├── inventory
├── playbook.yml
└── roles
    └── nginx
        ├── tasks
        │   ├── main.yml
        │   └── install.yml
        ├── handlers
        │   └── main.yml
        ├── templates
        │   └── nginx.conf.j2
        ├── files
        ├── vars
        │   └── main.yml
        └── defaults
            └── main.yml
```

inventory配置文件
``` ini
[webservers]
192.168.1.101 ansible_ssh_user=your_user ansible_ssh_pass=your_password ansible_host=203.0.113.1
```

playbook.yml:
```yaml
---
- name: Update and Install Nginx
  hosts: webservers  #引用组
  become: yes        #开启 sudo
  roles:             
    - role: nginx    #角色是nginx，对应到  roles/nginx 目录
      tags:          #两个 tags
        - bin
        - conf
```

任务文件 roles/nginx/tasks/main.yml:
```yaml
---
# 包含其他任务文件, 这里直接用 install.yml里面的文件内容肯定也是没问题的，但是我们可以通过这样的方式更好的进行管理
- include_tasks: install.yml
```

安装任务 roles/nginx/tasks/install.yml:
``` yaml
---
# 更新安装 Nginx
- name: Update apt cache and install Nginx
  apt:                  #安装nginx 相关包, 不同平台不太一样，比如 centos 可以使用 package
    name: nginx
    state: latest
    update_cache: yes
  tags:
    - bin               #这里添加了 bin 的 tag

# 部署 Nginx 配置文件
- name: Deploy Nginx configuration from template
  template:              #这里是更新 nginx 配置文件
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: '0644'
  notify: Restart Nginx   #这里配合handler 使用，handlers 里面会有一个名称为 “Restart Nginx”的操作
  tags:
    - conf                #这里添加了 conf 的 tag

# 检查 Nginx 是否监听正确端口
- name: Check Nginx is listening on port 80
  command: ss -tuln | grep :80    #通过 command 模块，ss 命令监听 80 端口是否启动
  register: result
  ignore_errors: yes

- name: Print Nginx listening port check result
  debug:
    msg: "{{ result.stdout }}"
  when: result.rc == 0
```

处理程序 roles/nginx/handlers/main.yml：
```yaml
---
# 当配置文件变更时，重启 Nginx, 和上面的 notify 是对应的
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

模板文件 roles/nginx/templates/nginx.conf.j2:
```
server {
    listen {{ nginx_port }};
    server_name localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

默认变量 roles/nginx/defaults/main.yml:
```yaml
---
nginx_port: 80
```

当我们更新安装时,可以通过(在正式执行前，可以在后面加-C -D 做测试使用):
``` bash
ansible-playbook playbook.yml 
```
后续有二进制更新时，可以通过:
``` bash
ansible-playbook playbook.yml -t bin
```
后续有配置文件更新时，可以通过:
``` bash
ansible-playbook playbook.yml -t conf
```

## 逻辑控制语句

### 条件语句when

when条件支持多种判断类型，主要用于根据某些条件决定是否执行某个任务。这些判断类型通常基于Python的语法，因为Ansible的任务是用Python编写的

主要支持的判断类型:
- 比较运算符：==, !=, >, <, >=, <= 用于比较两个值。
- 字符串方法：.startswith(), .endswith(), .find(), .contains() 等字符串方法可以用来检查字符串的特性。
- 逻辑运算符：and, or, not 用于组合多个条件。
- Jinja2模板表达式：由于Ansible使用Jinja2作为模板引擎，因此你也可以在when条件中使用Jinja2的表达式和过滤器。
- Ansible事实（facts）和变量：你可以使用Ansible收集的主机事实（facts）和定义的变量来进行条件判断。
- 函数和内置方法：Python的内置函数和方法也可以在when条件中使用，比如isinstance(), len(), 等等。
- 正则表达式：使用Python的正则表达式模块（如re）进行更复杂的字符串匹配。

其中facts涉及到的判断条件非常多，可以通过如下形式获取
``` yaml
---
- hosts: mysql
  tasks:
    - name: show ansible facts
      debug:
        var: ansible_facts
```
执行以上yml文件之后，会输出一个json串，我们就可以获取到所有的fact信息了.

示例：假如我们想在ip为 192.168.0.102 的机器上创建文件

``` yaml
---
- name: when测试练习
  hosts: webservers
  tasks:
    - name: 文件测试创建
      file: 
        path: /tmp/when.txt
        state: touch
      when: "'192.168.0.102' in ansible_all_ipv4_addresses"
```

### 循环语句loop
用于在任务中循环执行操作

示例:
``` yaml
---
- name: Install multiple packages
  hosts: webservers
  become: yes

  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - curl
```
上面的示例就是循环装包

### 块语句block

在 Ansible 中，block 关键字允许你将多个任务组合成一个逻辑块，并对这个块应用一些条件或错误处理逻辑。这是 Ansible 2.5 版本及以后引入的一个功能，它提供了更高级的任务组织方式。简单来说，block任务块就是一组逻辑的tasks。使用block可以将多个任务合并为一个组。

playbook会定义三种块，三种块的作用分别如下:
- block: block里的tasks,如果运行正确,则不会运行rescue；
- rescue：block里的tasks,如果运行失败,才会运行rescue里的tasks
- always：block和rescue里的tasks无论是否运行成功，都会运行always里的tasks

