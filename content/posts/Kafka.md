---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kafka"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-02-09T13:32:56+08:00
lastmod: 2021-02-09T13:32:56+08:00
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

Kafka

### 维护

#### 查看所有consumer

```
kafka-consumer-groups.sh  --list --bootstrap-server localhost:9092
```

#### 查看offset

```
kafka-consumer-offset-checker.sh --group SampleConsumer --topic topicName  --zookeeper localhost:2181
```

#### 重置offset

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group SampleConsumer --reset-offsets --to-earliest --topic topicName --execute
```

