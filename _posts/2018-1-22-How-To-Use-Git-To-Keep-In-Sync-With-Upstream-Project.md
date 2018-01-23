---
layout: post
title: How to use Git to keep in sync with upstream project
categories: [Git]
tags: [git,]
description: "如何使用Git保持当前项目与上游项目同步"
fullview: false
comments: true
---
手里fork的项目的上游更新了，需要将本地的代码与上游同步，不知如何操作，Google一下之后做个笔记。

# Step 1:

检查本地仓库使用的远程仓库。

    git remote -v

    # origin  git@github.com:monburan/vulhub.git (fetch)
    # origin  git@github.com:monburan/vulhub.git (push)

# Step 2：

为本地仓库添加一个名为 `upsteam` 的上游仓库。

    git remote add upsteam https://github.com/vulhub/vulhub.git

这时再检查检查本地仓库使用的远程仓库就会发现多了两行。

    git remote -v

    # origin  git@github.com:monburan/vulhub.git (fetch)
    # origin  git@github.com:monburan/vulhub.git (push)
    # upstream        https://github.com/vulhub/vulhub.git (fetch)
    # upstream        https://github.com/vulhub/vulhub.git (push)

# Step 3:

将 `upsteam` 的远程仓库取回到本地。

    git frech upsteam

# Step 4:

切换到主分支，然后合并分支。

    git checkout master
    git merge upstream/master

# Step 5:

然后将本地仓库的代码推送到自己的远程仓库就可以了。

    git push origin master




