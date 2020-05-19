---


# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Helm安装Kfaka"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-19T11:15:58+08:00
lastmod: 2020-05-19T11:15:58+08:00
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

本文介绍如何使用 helm 安装 Kafka，使用本地存储的方式。

#### 1 配置 chart 仓库

```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
```

#### 2 创建本地存储StorageClass

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```
kubectl apply -f local-storage-class.yaml
```

这里绑定模式选择 WaitForFirstConsumer

默认情况下，Immediate 模式指示在创建 PersistentVolumeClaim 之后发生卷绑定和动态配置。 对于拓扑受限且不能从集群中的所有节点全局访问的存储后端，persistentvolume 将在不知道 Pod 的调度需求的情况下进行绑定或提供。 这可能会导致不可调度的Pod。

集群管理员可以通过指定 WaitForFirstConsumer 模式来解决这个问题，该模式将延迟 PersistentVolume 的绑定和配置，直到创建使用 PersistentVolumeClaim 的 Pod。 Persistentvolumes 将根据 Pod 的调度约束所指定的拓扑结构进行选择或配给。 这些因素包括但不限于资源需求、节点选择器、Pod关联性和反关联性、污染和耐受性。

Local 通过预先创建的 PersistentVolume 绑定支持 WaitForFirstConsumer。

参考：

https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode

https://kubernetes.io/docs/concepts/storage/volumes/#local

#### 3 创建Kafka和Zookeeper的PersistentVolume

- 创建Kafka的PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-kafka-0
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/kafka/data-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-kafka-1
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/kafka/data-1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-kafka-2
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/kafka/data-2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
```

```
mkdir -p /localdata/kafka/data-0
mkdir -p /localdata/kafka/data-1
mkdir -p /localdata/kafka/data-2
```

- 创建Zookeeper的PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-zookeeper-0
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/zookeeper/data-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-zookeeper-1
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/zookeeper/data-1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage-zookeeper-2
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /localdata/zookeeper/data-2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3
```

```
mkdir -p /localdata/zookeeper/data-0
mkdir -p /localdata/zookeeper/data-1
mkdir -p /localdata/zookeeper/data-2
```

这里需要在节点上手动创建下存储目录 ，Kubernetes不会自动创建它。

使用本地卷时需要 PersistentVolume nodeAffinity。 它使 Kubernetes 调度程序能够将使用本地卷的 Pods 正确调度到正确的节点。

#### 4 部署Kafka

- 创建value文件kafka-values.yaml

  ```
  external:
    enabled: true
  configurationOverrides:
    "advertised.listeners": |-
      EXTERNAL://192.168.1.1:$((31090 + ${KAFKA_BROKER_ID}))
    "listener.security.protocol.map": |-
      PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
  persistence:
    storageClass: local-storage
  ```
  
这里开启外部访问，配置IP地址（第5行），集群任意节点均可，同时配置storageClass。其他更多配置看官方文档 https://github.com/helm/charts/tree/master/incubator/kafka
  
- 安装

  ```
  helm install kafka-dev -f kafka-values.yaml incubator/kafka
  ```

#### 5 查看Kafka版本

```
kubectl exec kafka-dev-0 -- ls /usr/share/java/kafka | grep kafka
```

