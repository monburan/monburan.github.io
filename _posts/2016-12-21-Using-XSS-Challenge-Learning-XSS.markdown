---
layout: post
title: 利用XSS Challenge学习XSS
categories: [学姿势]
tags: [Xss, XSS Challenge]
fullview: false
comments: true
---
我觉得我自己对XSS的理解还是有些不够，毕竟挖这块的东西经验太少，资料倒是看了不少。但是啊。。。并么有什么卵用。。。总结起来还是要练习啊，在不搞站的情况下一些挑战平台给我提供了良好的练习环境，就让我先从XSS Challenge开始吧。


## 第一关
既然是第一关，先看看是什么套路，输入<code><></code>看看什么效果，看到一对双引号包围了<code><></code>。

![pic1-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-1.png)

尝试输入<code>A""B</code>闭合双引号，查看页面输出。 

![pic1-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-2.png)

再在上面的代码中间尝试插入代码就弹窗了。<pre><code>&lt;script&gt;alert(1)&lt;/script&gt;</code></pre>

这个时候先不急着过关，看看还有没有什么姿势达到相同的效果。

尝试<pre><code>A"<img src=xss onerror=alert(1)></code></pre>

![pic1-3](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-3.png)

![pic1-4](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-4.png)

尝试<code>1"<img scr xss onerror=location="javascript:alert(1)">"B</code>

![pic1-5](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-5.png)

![pic1-6](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-1-6.png)


## 第二关

和前面类似，不过这次闭合的是双引号和尖括号，输入<code>"><"</code>

![pic2-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-2-1.png)

![pic2-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-2-2.png)

可以看到前面的<code>"></code>闭合<code>input</code>标签

![pic2-3](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-2-3.png)

![pic2-4](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-2-4.png)


## 第三关

这关卡了好久，一直以为是在输入框哪里被转义了，一直尝试这在输入框这里下手。后来想起来抓包，发现第二个参数并没有进行过滤。

![pic3-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-3-1.png)

![pic3-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-3-2.png)

![pic3-3](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-3-3.png)

![pic3-4](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-3-4.png)


## 第四关

和上面的很像，只不过这里的第三个参数并不会直接显示在页面中，而是存在一个<code>value</code>中。

![pic4-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-4-1.png)

这里没有过滤，直接将测试代码存到了value里。于是需要做的就是闭合引号和标签，在后面加入xss代码就可以了。

！[pic4-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-4-2.png)


## 第五关

提示：<code>length limited text box</code>，试了下在输入框上限制了输入长度，可以使用浏览器自带的工具突破限制，不过觉得这样挺无聊的。这次依旧利用抓包，不过要用点其他的姿势，使用事件触发xss。

首先还是要闭合<code>"</code>然后插入onfocus="alert(document.domain)" 

![pic5-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-5-1.png)

这样只要点击输入框就可以弹窗了！

![pic5-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/xss-challenges-5-2.png)




