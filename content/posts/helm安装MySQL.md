---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Helm安装MySQL"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-15T17:37:27+08:00
lastmod: 2020-05-15T17:37:27+08:00
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

helm 安装MySQL，使用本地存储的方式。

本文介绍如何使用 helm 安装 MySQL，使用本地存储的方式。

#### 1 创建StorageClass

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage-mysql
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

这里绑定模式选择 WaitForFirstConsumer

默认情况下，Immediate 模式指示在创建 PersistentVolumeClaim 之后发生卷绑定和动态配置。 对于拓扑受限且不能从集群中的所有节点全局访问的存储后端，persistentvolume 将在不知道 Pod 的调度需求的情况下进行绑定或提供。 这可能会导致不可调度的Pod。

集群管理员可以通过指定 WaitForFirstConsumer 模式来解决这个问题，该模式将延迟 PersistentVolume 的绑定和配置，直到创建使用 PersistentVolumeClaim 的 Pod。 Persistentvolumes 将根据 Pod 的调度约束所指定的拓扑结构进行选择或配给。 这些因素包括但不限于资源需求、节点选择器、Pod关联性和反关联性、污染和耐受性。

Local 通过预先创建的 PersistentVolume 绑定支持 WaitForFirstConsumer。

参考：

https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode

https://kubernetes.io/docs/concepts/storage/volumes/#local

#### 2 创建PersistentVolume

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-mysql
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage-mysql
  local:
    path: /db/mysql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
```

使用本地卷时需要 PersistentVolume nodeAffinity。 它使 Kubernetes 调度程序能够将使用本地卷的 Pods 正确调度到正确的节点。

#### 3 在指定节点上创建存储目录

这里需要在节点 node3 上手动创建存储目录 /db/mysql，Kubernetes不会自动创建它。

#### 4 安装MySQL

```
helm install my-mysql --set persistence.storageClass=local-storage-mysql stable/mysql
```

这里设置 storageClassName 属性为上面创建的 StorageClass，其他可设置属性参看官方说明 https://github.com/helm/charts/tree/master/stable/mysql#configuration