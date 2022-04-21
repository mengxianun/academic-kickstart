---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Rabbit"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2022-03-29T10:36:47+08:00
lastmod: 2022-03-29T10:36:47+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

#### 安装（Red Hat 7, CentOS 7）

##### 导入签名密钥

```
## primary RabbitMQ signing key
rpm --import https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
## modern Erlang repository
rpm --import https://packagecloud.io/rabbitmq/erlang/gpgkey
## RabbitMQ server repository
rpm --import https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
```

##### 添加 Yum 存储库

```
# In /etc/yum.repos.d/rabbitmq.repo

##
## Zero dependency Erlang
##

[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_erlang-source]
name=rabbitmq_erlang-source
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

##
## RabbitMQ server
##

[rabbitmq_server]
name=rabbitmq_server
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
       https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_server-source]
name=rabbitmq_server-source
baseurl=https://packagecloud.io/rabbitmq/rabbitmq-server/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

##### 安装

```
## Update Yum package metadata
yum update -y

## install the packages
## install these dependencies from standard OS repositories
yum install socat logrotate -y

yum install erlang rabbitmq-server -y
```

##### 启动服务

```
## 所有节点
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service

## 主节点
拷贝 /var/lib/rabbitmq/.erlang.cookie 到副本节点

## 副本节点
rabbitmqctl stop_app # => 停止节点 rabbit@rabbit2 ...完成。
rabbitmqctl reset # => 重置节点 rabbit@rabbit2 ... 
rabbitmqctl join_cluster rabbit@rabbit1 # => 使用 [rabbit@rabbit1] 聚类节点 rabbit@rabbit2 ...完成。
rabbitmqctl start_app # => 开始节点 rabbit@rabbit2 ...完成。
```

#### 操作

##### 创建vhost

```
rabbitmqctl add_vhost qa1
```

##### 创建用户

```
rabbitmqctl add_user "username" "password"
```

##### 用户授权

```
rabbitmqctl set_permissions -p "qa1" "iie" ".*" ".*" ".*"
```

##### 启用UI

```
## Enable
rabbitmq-plugins enable rabbitmq_management

## tag the user with "administrator" for full management UI and HTTP API access
rabbitmqctl set_user_tags full_access administrator

## Access http://{node-hostname}:15672/
```

