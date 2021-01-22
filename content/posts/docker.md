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

### Docker(CentOS)

#### 安装(开发环境)

```
## 1 更新Yum
sudo yum update
## 2 添加docker仓库并安装docker
curl -fsSL https://get.docker.com/ | sh
## 3 启动docker
sudo systemctl start docker
sudo systemctl enable docker
```

参考：

https://geekflare.com/docker-installation-guide/

#### 卸载

```
sudo yum remove -y docker-ce docker-ce-cli
```

#### 修改存储位置

```
## 1 创建文件 /etc/docker/daemon.json, 内容如下
{
  "data-root": "/mnt/newlocation"
}
## 2 重启docker
sudo systemctl restart docker
```

#### 删除冗余数据

```
docker system prune -a
```

#### 删除所有none镜像

```
docker rmi $(docker images | awk '/^<none>/ { print $3 }')
```

#### 删除所有容器

```
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
```



### Dockerfile

#### 时区

```
FROM alpine:3.6
RUN apk add --no-cache tzdata
ENV TZ Asia/Shanghai
```

#### examples

##### Spring Boot

```
FROM openjdk:8-jre-alpine
RUN apk add --no-cache tzdata
ENV TZ Asia/Shanghai
WORKDIR /app
COPY springboot.jar /app
ENTRYPOINT ["java", "-jar", "springboot.jar"]
```

### Docker Compose

#### Web 应用