---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubernetes安装"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-09T18:36:00+08:00
lastmod: 2020-06-09T18:36:00+08:00
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

kubeadm 安装 kubernets 集群

#### 下载程序包(翻墙)

```
# 下载程序
yum install -y --downloadonly kubelet kubeadm kubectl --disableexcludes=kubernetes
# 下载镜像
kubeadm config images pull
docker pull quay.io/coreos/flannel:v0.12.0-amd64
```

注：下载的程序为最新版本

#### 准备工作

1. 开启防火墙端口

   需要开启的端口

   ```
   Master node(s):
   
   TCP     6443*       Kubernetes API Server
   TCP     2379-2380   etcd server client API
   UDP     8472        flannel only
   TCP     10250       Kubelet API
   TCP     10251       kube-scheduler
   TCP     10252       kube-controller-manager
   TCP     10255       Read-Only Kubelet API
   
   Worker nodes (minions):
   
   UDP     8472        flannel only
   TCP     10250       Kubelet API
   TCP     10255       Read-Only Kubelet API
   TCP     30000-32767 NodePort Services
   ```

   ```
   # 主节点
   firewall-cmd --permanent --add-port=6443/tcp
   firewall-cmd --permanent --add-port=2379-2380/tcp
   firewall-cmd --permanent --add-port=10250/tcp
   firewall-cmd --permanent --add-port=10251/tcp
   firewall-cmd --permanent --add-port=10252/tcp
   firewall-cmd --permanent --add-port=10255/tcp
   firewall-cmd --permanent --add-port=8472/udp
   firewall-cmd --add-masquerade --permanent
   # only if you want NodePorts exposed on control plane IP as well
   firewall-cmd --permanent --add-port=30000-32767/tcp
   systemctl restart firewalld
   
   # 从节点
   firewall-cmd --permanent --add-port=10250/tcp
   firewall-cmd --permanent --add-port=10255/tcp
   firewall-cmd --permanent --add-port=8472/udp
   firewall-cmd --permanent --add-port=30000-32767/tcp
   firewall-cmd --add-masquerade --permanent
   systemctl restart firewalld
   ```

   

2. 设置hostname

3. 安装docker

4. 设置必需的sysctl参数

5. 禁用swap

6. 安装kubernetes程序包

7. 开启kubelet

8. 加载kubernetes镜像