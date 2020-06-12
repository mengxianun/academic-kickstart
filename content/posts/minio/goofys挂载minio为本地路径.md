---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Goofys挂载minio为本地路径"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-12T14:38:16+08:00
lastmod: 2020-06-12T14:38:16+08:00
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

goofys 挂载 minio 为本地路径

#### goofys

1. 安装

   ```
   wget https://github.com/kahing/goofys/releases/latest/download/goofys -P /usr/local/bin/
   chmod +x /usr/local/bin/goofys
   ```

2. 配置认证信息

   ```
   $ cat ~/.aws/credentials
   [default]
   aws_access_key_id = AKID1234567890
   aws_secret_access_key = MY-SECRET-KEY
   ```

#### 挂载minio

1. 挂载

   ```
   goofys <bucket> <mountpoint>
   goofys <bucket:prefix> <mountpoint> # if you only want to mount objects under a prefix
   
   # 示例
   goofys --endpoint http://10.111.54.230:9000 test /minio-local-data
   ```

2. 开机自动挂载

   ```
   # 修改 /etc/fstab, 添加如下内容
   goofys#bucket /mnt/mountpoint fuse _netdev,allow_other,--file-mode=0666,--dir-mode=0777 0 0
   ```