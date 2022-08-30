---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Frp"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2022-04-24T10:02:10+08:00
lastmod: 2022-04-24T10:02:10+08:00
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

### 服务端配置

/etc/frp/frps.ini

```
[common]
bind_port = 7000
token = ***
```

### 客户端配置

/etc/frp/frpc.ini

```
[common]
server_addr = 服务端IP(公网)
server_port = 7000
token = ***

[gitlab]
type = tcp
local_ip = 内网服务IP
local_port = 内网服务端口
remote_port = 公网服务端口
```

### 管理

#### 重启

```
systemctl restart frpc.service
```

