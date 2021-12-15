---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "MinIO安装"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-09-25T11:08:58+08:00
lastmod: 2020-09-25T11:08:58+08:00
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

MinIO

#### 安装

```
wget https://dl.min.io/server/minio/release/linux-amd64/minio -O /usr/local/bin/minio
chmod +x /usr/local/bin/minio
```

#### 环境变量

```
export MINIO_ACCESS_KEY=<ACCESS_KEY>
export MINIO_SECRET_KEY=<SECRET_KEY>
```

#### 单节点

```
minio server /data
```

#### 分布式

##### 单节点多磁盘

```
minio server /data{1...4}
or
minio server /data1 /data2 /data3 /data4

# 后台运行
setsid minio server /home/minio/data >/home/minio/data/minio.log 2>&1 < /dev/null &
```

##### 多节点多磁盘

```
minio server http://host{1...n}/export{1...m}
or
minio server http://host1/export1 http://host1/export2 http://host2/export1 http://host2/export2 ...
```

注：

- 所有运行分布式 MinIO 的节点需要具有相同的访问密钥和秘密密钥才能连接。为了实现这一点，建议在执行 MINIO 服务器命令之前，将所有节点上的访问密钥和秘密密钥作为环境变量 MINIO _ access _ key 和 MINIO _ secret _ key 导出。
- 分布式 Minio 使用的磁盘里必须是干净的，里面没有数据。
- MinIO 创建每组4到16个驱动器的擦除编码集。您提供的驱动器总数必须是这些数字之一的倍数。
- _ MinIO 选择最大的 EC 设置大小，将其划分为驱动器的总数或给定的节点的总数——确保保持均匀分布，即每个节点参与每个设置的驱动器数相等。
- 建议所有运行分布式 MinIO 设置的节点都是同构的，即相同的操作系统、相同数量的磁盘和相同的网络互连。
- 运行分布式 MinIO 实例的服务器时间差不应超过15分钟。

#### 服务

##### 单节点

1. 创建用户

   ```
   useradd minio
   passwd minio
   ```

2. 创建默认配置文件

   ```
   $ cat <<EOT >> /etc/default/minio
   # Volume to be used for MinIO server.
   MINIO_VOLUMES="/tmp/minio/"
   # Use if you want to run MinIO on a custom port.
   MINIO_OPTS="--address :9199"
   # Access Key of the server.
   MINIO_ACCESS_KEY=Server-Access-Key
   # Secret key of the server.
   MINIO_SECRET_KEY=Server-Secret-Key
   
   EOT
   ```

3. 下载 service 文件 https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service 到 /etc/systemd/system/，**并替换里面的用户和组**

   ```
   [Unit]
   Description=MinIO
   Documentation=https://docs.min.io
   Wants=network-online.target
   After=network-online.target
   AssertFileIsExecutable=/usr/local/bin/minio
   
   [Service]
   WorkingDirectory=/usr/local/
   
   User=minio-user
   Group=minio-user
   
   EnvironmentFile=/etc/default/minio
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
   
   ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
   
   # Let systemd restart this service always
   Restart=always
   
   # Specifies the maximum file descriptor number that can be opened by this process
   LimitNOFILE=65536
   
   # Disable timeout logic and wait until process is stopped
   TimeoutStopSec=infinity
   SendSIGKILL=no
   
   [Install]
   WantedBy=multi-user.target
   
   # Built for ${project.name}-${project.version} (${project.name})
   ```

4. 启动

   ```
   systemctl daemon-reload
   systemctl start minio
   systemctl enable minio
   ```

##### 分布式

1. 创建用户

   ```
   useradd minio
   passwd minio
   ```

2. 创建存储目录

   ```
   mkdir /export
   sudo chown -R minio /export && sudo chmod u+rxw /export
   ```

3. 创建默认配置文件

   ```
   $ cat <<EOT >> /etc/default/minio
   # Remote volumes to be used for MinIO server.
   MINIO_VOLUMES=http://node{1...6}/export{1...32}
   # Use if you want to run MinIO on a custom port.
   MINIO_OPTS="--address :9199"
   # Access Key of the server.
   MINIO_ACCESS_KEY=Server-Access-Key
   # Secret key of the server.
   MINIO_SECRET_KEY=Server-Secret-Key
   
   EOT
   ```

4. 下载 service 文件 https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/distributed/minio.service 到 /etc/systemd/system/，**并替换里面的用户和组**

   ```
   [Unit]
   Description=MinIO
   Documentation=https://docs.min.io
   Wants=network-online.target
   After=network-online.target
   AssertFileIsExecutable=/usr/local/bin/minio
   
   [Service]
   WorkingDirectory=/usr/local
   
   User=minio-user
   Group=minio-user
   
   EnvironmentFile=-/etc/default/minio
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_ACCESS_KEY}\" ]; then echo \"Variable MINIO_ACCESS_KEY not set in /etc/default/minio\"; exit 1; fi"
   ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_SECRET_KEY}\" ]; then echo \"Variable MINIO_SECRET_KEY not set in /etc/default/minio\"; exit 1; fi"
   
   ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
   
   # Let systemd restart this service always
   Restart=always
   
   # Specifies the maximum file descriptor number that can be opened by this process
   LimitNOFILE=65536
   
   # Disable timeout logic and wait until process is stopped
   TimeoutStopSec=infinity
   SendSIGKILL=no
   
   [Install]
   WantedBy=multi-user.target
   
   # Built for ${project.name}-${project.version} (${project.name})
   ```

5. 启动

   ```
   systemctl daemon-reload
   systemctl start minio
   systemctl enable minio
   ```

#### 客户端

##### 安装

```
wget https://dl.min.io/client/mc/release/linux-amd64/mc -O /usr/local/bin/mc
chmod +x /usr/local/bin/mc
```

##### 常用命令

| 命令 | 说明                     |
| ---- | ------------------------ |
| ls   | list buckets and objects |
|      |                          |
|      |                          |

#### 挂载minio为本地路径

1. 安装goofys

   ```
   yum install -y fuse
   wget https://github.com/kahing/goofys/releases/latest/download/goofys -P /usr/local/bin/
   chmod +x /usr/local/bin/goofys
   ```

2. 配置认证信息

   ```
   $ mkdir ~/.aws
   $ cat <<EOT > ~/.aws/credentials
   [default]
   aws_access_key_id=Server-Access-Key
   aws_secret_access_key=Server-Secret-Key
   EOT
   ```

3. 挂载

   ```
   goofys <bucket> <mountpoint>
   goofys <bucket:prefix> <mountpoint> # if you only want to mount objects under a prefix
   
   # 示例
   goofys --endpoint http://10.111.54.230:9000 test /minio-local-data
   ```

4. 开机自动挂载

   ```
   # 修改 /etc/fstab, 添加如下内容
   goofys#bucket /mnt/mountpoint fuse _netdev,allow_other,--file-mode=0666,--dir-mode=0777 0 0
   ```

##### 卸载

```
fusermount -u /path/to/mountpoint
```

#### 定时删除文件

```
## 每天2点，删除30天前的文件
0 2 * * * /root/minio-binaries/mc rm --recursive --dangerous --force --older-than 30d local
```

