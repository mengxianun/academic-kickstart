---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Minikube安装"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-08T17:52:45+08:00
lastmod: 2020-06-08T17:52:45+08:00
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

Linux 安装minikube

#### 1 安装docker

```
$ curl -fsSL https://get.docker.com/ | sh
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

#### 2 创建用户

minikube 需要非root用户运行

```
adduser developer
passwd developer
usermod -aG sudo developer
# 如果上面命令出现 usermod: group 'sudo' does not exist, 运行以下命令
usermod -aG wheel developer
su - developer
sudo usermod -aG docker $USER
- 重新登录
```

#### 3 安装kubectl

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
$ kubectl version --client
```

#### 4 安装minikube

```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
$ sudo mkdir -p /usr/local/bin/
$ sudo install minikube /usr/local/bin/
```

#### 5 启动minikube

```
# 需要翻墙
minikube start --driver=docker
```

#### 6 验证安装

```
minikube status
```

显示如下

```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

