---
title: "如何在 GitHub Pages 使用 hugo 建立 blog"
date: 2018-01-30T18:37:50+08:00
categories: ['blog']
tags: ['hugo', 'blog', 'git', 'github']

---

# 使用 user site

目標：使用 `<your-github-account>.github.io` 當作域名建立 blog，並且上傳原始檔和靜態網頁。
這邊參考[官網](https://gohugo.io/hosting-and-deployment/hosting-on-github/#github-user-or-organization-pages)操作，並作一點修改。

<!--more-->

## Step 1: 建立 GitHub repo

先在 GitHub 新增一個 repo, 名稱設定為 `<your-github-account>.github.io`, 用來存放靜態網頁並顯示。

再新增一個 repo, 名稱設定為 `<blog-project-name>`, 用來存放原始檔。
接下來回到 local 操作

## Step 2: 建立 hugo site 

```bash
$ hugo new site <blog-project-name> /home/bin
```

```
$ cd <blog-project-name>
$ git init
$ echo .DS_Store >> .gitignore

# add git remote repo
$ git remote add origin git@github.com:<your-github-account>/<blog-project-name>.git
```

## Step 3: 安裝 theme
先挑選自己喜歡的 [theme](https://themes.gohugo.io/)，我選擇 [Cactus Plus](https://themes.gohugo.io/hugo-theme-cactus-plus/)。

在這邊有個坑，如果只用 git clone 如下：

```
$ git clone https://github.com/nodejh/hugo-theme-cactus-plus themes/cactus-plus
```

在 push 到 GitHub 之後會報錯，必須改用 git submodule：

```
$ git submodule add https://github.com/nodejh/hugo-theme-cactus-plus themes/cactus-plus
```

接下來參考 exampleSite 設定 config.toml。

## Step 4: 開啟 hugo server 預覽

```
# -D 預覽時包含草稿內容
#
# 如果要在遠端架設 hugo server 來預覽，則需要:
# -b <your IP address> --bind="0.0.0.0"
$ hugo server -w
```

## Step 5: 新增 deploy.sh

```
$ cat > deploy.sh
```

這邊要注意，按照官網的操作並不會把原始檔上傳到 `<blog-project>` 的 repo，所以加入

```
$ git add -A
...
$ git commit ...
```

```
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# 
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi

git commit -m "$msg"
git push origin master

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```

最後記得改變 deploy.sh 權限為可執行

```
$ chmod +x deploy.sh
```
