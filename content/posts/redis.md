---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Redis安装"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-09-01T17:11:11+08:00
lastmod: 2020-09-01T17:11:11+08:00
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

#### Redis Cluster

**注：建议6个节点。在没有6个节点的情况下，每个节点可以创建2个实例**

1. 下载安装包

   ```
   wget http://download.redis.io/releases/redis-5.0.5.tar.gz
   ```

2. 解压

   ```
   tar zxf redis-5.0.5.tar.gz -C /opt/
   ```

3. 安装

   ```
   yum -y install gcc-c++
   cd /opt/redis-5.0.5/ && make MALLOC=libc
   ./utils/install_server.sh
   # 1 Please select the redis port for this instance: [6379] -- 
   # 2 Please select the redis config file name [/etc/redis/6379.conf] --
   # 3 Please select the redis log file name [/var/log/redis_6379.log] --
   # 4 Please select the data directory for this instance [/var/lib/redis/6379] --
   # 5 Please select the redis executable path [] -- /opt/redis-5.0.5/src/redis-server
   ```

4. 修改配置 /etc/redis/6379.conf

   ```
   # 绑定地址
   bind 0.0.0.0
   # 端口
   port 6379
   # 安全密码
   requirepass foobared
   ## =========集群相关=========
   cluster-enabled yes
   cluster-config-file nodes-6379.conf
   cluster-node-timeout 15000
   appendonly yes
   ```

5. 启动

   ```
   systemctl restart redis_6379
   ```

6. 创建集群

   ```
   # 3个主节点, 3个副本节点
   redis-cli --cluster create 192.168.201.101:6379 192.168.201.101:6380 192.168.201.102:6379 192.168.201.102:6380 192.168.201.103:6379 192.168.201.103:6380 -a foobared --cluster-replicas 1
   ```

7. 创建别名

   ```
   # 修改 ~/.bashrc，添加如下内容
   alias redis-cli=/opt/redis-5.0.5/src/redis-cli
   alias redis-server=/opt/redis-5.0.5/src/redis-server
   # 生效
   source ~/.bashrc
   ```
