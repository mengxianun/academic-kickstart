---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Linux后台守护进程运行脚本"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-25T09:08:48+08:00
lastmod: 2020-06-25T09:08:48+08:00
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

linux 后台守护进程运行脚本

```
setsid myscript.sh >/dev/null 2>&1 < /dev/null &
# 输出到日志文件
setsid myscript.sh >/path/to/logfile 2>&1 < /dev/null &
```

参考：https://stackoverflow.com/questions/19233529/run-bash-script-as-daemon