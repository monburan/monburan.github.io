---
layout: post
title: Struts RCE Learning
categories: [Vulnerability-analysis]
tags: [Struts2,]
description: "学习Java代码以及漏洞原理理解"
fullview: false
comments: false
---

官方git：

    https://github.com/apache/struts

官方安全公告：

    https://struts.apache.org/docs/security-bulletins.html

git版本标签等信息：

    https://git-wip-us.apache.org/repos/asf?p=struts.git

历史版本下载：

    http://archive.apache.org/dist/struts/binaries/

首先做代码审计的时候要使用git tag 查看 Struts2的版本
git show 对应版本可以看到更新
git diff 用来对比两个版本的不同