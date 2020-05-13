---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Hugo Github Pages"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-13T15:45:43+08:00
lastmod: 2020-05-13T15:45:43+08:00
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





### 安装

1 在github上创建仓库\<USERNAME\>\.github.io

2 [**fork**](https://github.com/sourcethemes/academic-kickstart#fork-destination-box) *Academic Kickstart* 仓库，并clone到本地（替换你的用户名）。

```
git clone https://github.com/<USERNAME>/academic-kickstart.git My_Website
cd My_Website
git submodule update --init --recursive
git submodule add -f -b master https://github.com/<USERNAME>/<USERNAME>.github.io.git public
```

3 将所有内容添加到本地 git 存储库，并将其推送到 GitHub 上的远程存储库:

```
git add .
git commit -m "Initial commit"
git push -u origin master
```

4 通过运行 Hugo 重新生成网站的 HTML 代码，并将 public 子模块上传到 GitHub:

```
hugo
cd public
git add .
git commit -m "Build website"
git push origin master
cd ..
```

5 通过浏览器访问 https://<USERNAME>.github.io

### 添加内容

1 创建文件

```
hugo new posts/my-first-post.md
```

2 部署脚本 deploy.sh

```
#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master
```

3 更新网站

```
./deploy.sh "Your optional commit message"
```

参考：

https://georgecushen.com/create-your-website-with-hugo/

https://sourcethemes.com/academic/docs/deployment/