---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Docker"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-13T17:56:03+08:00
lastmod: 2020-05-13T17:56:03+08:00
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

### 安装(CentOS)

1 更新Yum

```
sudo yum update
```

2 添加docker仓库并安装docker

```
curl -fsSL https://get.docker.com/ | sh
```

3 启动docker

```
sudo systemctl start docker
sudo systemctl enable docker
```



参考：

https://geekflare.com/docker-installation-guide/



#### 删除所有none镜像

```
docker rmi $(docker images | awk '/^<none>/ { print $3 }')
```



### Dockerfile

#### 时区

```
FROM alpine:3.6
RUN apk add --no-cache tzdata
ENV TZ Asia/Shanghai
```

