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

helm 安装MySQL

1 创建storageClass

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mysql-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

2 创建PersistentVolume，使用本地存储

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: mysql-storage
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

3 在指定节点上（node3）创建存储目录，/db/mysql

4 安装MySQL，设置storageClassName属性

```
helm install my-mysql --set persistence.storageClass=mysql-storage stable/mysql
```

