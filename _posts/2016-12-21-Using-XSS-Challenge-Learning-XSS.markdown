---
layout: post
title: 利用XSS平台学习XSS之XSS Challenge
categories: [学姿势]
tags: [Xss, XSS Challenge]
fullview: true
comments: true
---
我觉得我自己对XSS的理解还是有些不够，毕竟挖这块的东西经验太少，资料倒是看了不少。但是啊。。。并么有什么卵用。。。总结起来还是要练习啊，在不搞站的情况下一些挑战平台给我提供了良好的练习环境，就让我先从XSS Challenge开始吧。

## 第一关
既然是第一关，先看看是什么套路，输入<>看看什么效果，看到一对双引号包围了<>。
尝试输入A""B闭合双引号，查看页面输出。 

再在上面的代码中间尝试插入
{% highlight javascript %}
<script>alert(1)<\script>
{% endhighlight %}
弹窗了。这个时候先不急着过关，看看还有没有什么姿势达到相同的效果。
尝试<code>A"<img src=xss onerror=alert(1)></code>

尝试1"<img scr xss onerror=location="javascript:alert(1)">"B
