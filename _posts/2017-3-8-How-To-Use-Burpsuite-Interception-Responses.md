---
layout: post
title: 使用burpsuitel拦截响应
categories: [Configuration]
tags: [Burpsuite]
description: "以前用Burpsuite都是用来拦截请求包，昨天面试，面试官问我是否知道Burpsuite如何拦截响应，瞬间懵逼，今天总结下如何拦截响应。"
fullview: false
comments: true
---
首先打开<code>Burpsuite -> Proxy -> Options</code>来到代理的设置界面。

![pic-1](http://o8lgx56x1.bkt.clouddn.com//blog/img/burp-options.png)

找到<code>Intercept Server Responses</code>,勾选<code>Intercept responses based on the following rules:</code>。

![pic-2](http://o8lgx56x1.bkt.clouddn.com//blog/img/burp-options-responses.png)

其他几个规则视实际情况需要勾选，下面看下效果：

请求：

![pic-3](http://o8lgx56x1.bkt.clouddn.com//blog/img/burp-request.png)

响应：

![pic-4](http://o8lgx56x1.bkt.clouddn.com//blog/img/burp-response.png)