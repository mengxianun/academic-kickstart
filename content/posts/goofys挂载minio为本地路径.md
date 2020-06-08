---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Goofys挂载minio为本地路径"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-08T14:59:38+08:00
lastmod: 2020-06-08T14:59:38+08:00
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

goofys挂载minio为本地路径

#### 1 安装fuse

```
yum -y install fuse
```

#### 2 安装goofys

```
wget https://github.com/kahing/goofys/releases/latest/download/goofys -O /bin/goofys && chmod +x /bin/goofys
```

#### 3 配置认证信息

```
$ cat ~/.aws/credentials
[default]
aws_access_key_id = AKID1234567890
aws_secret_access_key = MY-SECRET-KEY
```

#### 4 使用

```
$ $GOPATH/bin/goofys <bucket> <mountpoint>
$ $GOPATH/bin/goofys <bucket:prefix> <mountpoint>
# 例
goofys --endpoint http://localhost:9000 bucket /mnt
# 前端运行
goofys -f --endpoint http://localhost:9000 bucket /mnt
```

#### 5 开机自动挂载

```
# 配置 /etc/fstab, 添加如下信息
goofys#bucket /mnt/mountpoint fuse _netdev,allow_other,--file-mode=0666,--dir-mode=0777,--endpoint=http://localhost:9000 0 0
```

