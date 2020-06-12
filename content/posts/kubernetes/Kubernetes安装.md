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

   ```
   # 节点1
   # 设置hostname
   sudo hostnamectl set-hostname node1
   # 修改/etc/hosts, 添加以下信息
   192.168.201.120 node1
   192.168.201.121 node2
   192.168.201.122 node3
   
   # 节点2
   # 设置hostname
   sudo hostnamectl set-hostname node2
   # 修改/etc/hosts, 添加以下信息
   192.168.201.120 node1
   192.168.201.121 node2
   192.168.201.122 node3
   
   # 节点3
   # 设置hostname
   sudo hostnamectl set-hostname node3
   # 修改/etc/hosts, 添加以下信息
   192.168.201.120 node1
   192.168.201.121 node2
   192.168.201.122 node3
   ```

3. 安装docker

   ```
   # 安装docker仓库
   yum install -y yum-utils
   yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
   # 安装docker 19.03 版本
   yum install -y docker-ce-19.03.11 docker-ce-cli-19.03.11 containerd.io
   # 安装最新版本
   #yum install -y docker-ce docker-ce-cli containerd.io
   # 启动docker
   systemctl enable docker && systemctl start docker
   ```

4. 设置必需的sysctl参数

   ```
   sysctl net.bridge.bridge-nf-call-iptables=1
   ```

5. 禁用swap

   ```
   sudo swapoff -a
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```

#### Kubernetes 程序安装

##### 在线安装方式

###### kubelet/kubeadm/kubectl

```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
# 开启kubelet
systemctl enable kubelet && systemctl start kubelet
```

###### 镜像

```
# 预先拉去镜像
kubeadm config images pull
```

##### 离线安装方式

###### 下载程序包(翻墙)

```
# 下载程序
yum install -y --downloadonly kubelet kubeadm kubectl --disableexcludes=kubernetes
# 下载镜像
kubeadm config images pull
docker pull quay.io/coreos/flannel:v0.12.0-amd64
# 将镜像保存为tar文件
docker save ...
```

###### kubelet/kubeadm/kubectl

```
# 将程序包拷贝到服务器上
# 进入rpm文件路径
yum -y install *.rpm
# 开启kubelet
systemctl enable kubelet && systemctl start kubelet
```

###### 镜像

```
# 将kubernetes相关镜像拷贝到服务器上
# 进入镜像文件目录, 加载所有镜像
for i in `ls`; do docker load < $i; done
```

离线文件下载地址(1.18版本)

#### 部署

##### 主节点

1. 初始化

   ```
   # flannel
   kubeadm init --pod-network-cidr=10.244.0.0/16
   ```

2. 配置环境变量

   - root

     ```
     # /etc/profile 添加以下内容
     export KUBECONFIG=/etc/kubernetes/admin.conf
     # 立即生效
     source /etc/profile
     ```

   - 常规用户

     ```
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
     ```

3. 设置网络

   ```
   kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   ```

##### 从节点

1. 加入集群

   ```
   # 此命令从主节点init命令输出获得
   kubeadm join 192.168.201.120:6443 --token z8clj6.i8u6s0deqsncftaz --discovery-token-ca-cert-hash sha256:a0cf54fda227bb64d6af37479ed055aad31e55df287566d3793825705355f3d8
   ```

##### 检查状态(Ready)

```
# 主节点执行
kubectl get nodes
# 输出如下
NAME    STATUS     ROLES    AGE   VERSION
node1   Ready      master   21m   v1.18.3
node2   NotReady   <none>   37s   v1.18.3
node3   NotReady   <none>   36s   v1.18.3
# 等待一会, 全部Ready
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    master   22m     v1.18.3
node2   Ready    <none>   2m25s   v1.18.3
node3   Ready    <none>   2m24s   v1.18.3
```

