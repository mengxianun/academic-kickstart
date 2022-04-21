---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Mongo"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2022-04-21T11:18:16+08:00
lastmod: 2022-04-21T11:18:16+08:00
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
projects: []z 
---

### 安装

#### docker

```
docker run -d -p 27017:27017 --name metasearch-mongo -e MONGO_INITDB_DATABASE=db_name -e MONGO_INITDB_ROOT_USERNAME=username -e MONGO_INITDB_ROOT_PASSWORD=password -v /data/mongo:/data/db mongo
```

