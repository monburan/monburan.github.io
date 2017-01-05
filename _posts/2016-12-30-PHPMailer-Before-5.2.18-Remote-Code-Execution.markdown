---
layout: post
title: PHPMailer远程代码执行
categories: [漏洞分析]
tags: [PHPMailer,Remote Code Execution]
fullview: false
comments: true
---
前两天在微博看到了这个，国内一票大神都已近分析过了，以学习为目的自己尝试总结一下。

## 首先介绍一下主角PHPMailer ##

PHPMailer continues to be the world's most popular transport class, with an
estimated 9 million users worldwide. Downloads continue at a significant
pace daily.

>PHPMailer以全球约900万的用户量成为最受欢迎的邮件类。下载量每天还在上升。

Probably the world's most popular code for sending email from PHP!
Used by many open-source projects: WordPress, Drupal, 1CRM, SugarCRM, [..],
Joomla! and many more

>PHPMailer可能是全球PHP发送邮件最流行的代码。亦被诸多开源项目所采用，包括WordPress、Drupal、1CRM、SugarCRM、Joomla!等。

用户量这么大的一个邮件类出现问题也算是一个重磅炸弹了。

## 出现漏洞的位置 ##

这里看到的代码是5.2.16版本的。PHPMailer使用的是PHP自带的<code>mail()</code>函数。这次出出现问题的源头是<code>mail()</code>函数的第五个参数<code>$params</code>未经过滤。

可以看到这里有一个条件，当开启<code>safe_mode</code>的时候就不能执行这个函数。

![pic1-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-1.png)

根据<code>mailPassthru</code>这个函数名就可以追到这个<code>$params</code>赋值的地方。

![pic1-2](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-2.png)

<code>$this->Sender</code>将值赋给了<code>$params</code>。

![pic1-3](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-3.png)

上面看到在<code>$address</code>赋值给<code>$this->Sender</code>之前经过一系列的参数过滤。
最关键的是这里，要仔细看一下。

![pic1-4](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-4.png)

首先<code>$address</code>经过<code>trim()</code>的过滤去掉了可能存在的<code> </code>，<code>\t</code>，<code>\n</code>，<code>\r</code>，<code>\0</code>，<code>\x0B</code>。

下面的一组判断是：

<code>($pos = strrpos($address, '@')) === false or
(!$this->has8bitChars(substr($address, ++$pos)) or 
!$this->idnSupported()) and
!$this->validateAddress($address)</code>

首先判断了<code>$address</code>中是否含有<code>@</code>。

然后再判断<code>@</code>后面是否有任意8位字符（其实就是看有没有汉字和汉字标点）

![pic1-5](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-5.png)

重点来了，<code>validateAddress()</code>函数是用来检查输入的邮件地址是不是合法的Email地址的。这次漏洞利用主要是突破这个函数中的限制。

![pic1-6](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmBailer-1-6.png)

![pic1-7](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-7.png)

首先说一下条件最苛刻但利用起来比较简单的方法，根据上面的代码可以看到，如果在<code>$patternselect</code>为<code>'noregex'</code>的时候，直接跳过了正则表达式的检测，条件：没有安装pcre，且PHP版本小于5.2.0。

![pic1-8](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-8.png)

![pic1-9](http://o8lgx56x1.bkt.clouddn.com/blog/img/phpmailer-1-9.png)

从这里来看，这个漏洞利用起来比较鸡肋，但是正则表达式能绕过，这个漏洞就没有那么鸡肋了。

这里根据[phithon大神的总结](https://www.leavesongs.com/PENETRATION/how-to-analyze-long-regex.html)自己又复现了一遍：

![pic1-10]()

![pic1-11]()

首先看最容易理解的第二组：<code>(?>(?>(?>\x0D\x0A)?[\t ])+|(?>[\t ]*\x0D\x0A)?[\t ]+)</code>

这里主要是针对<code> </code>，<code>\t</code>以及回车<code>\x0D</code>换行<code>\x0A</code>的匹配。

接下来是第三组：<code>(\((?>(?2)(?>[\x01-\x08\x0B\x0C\x0E-\'*-\[\]-\x7F]|\\\[\x00-\x7F]|(?3)))*(?2)\))</code>

第三组中套用了第二组，并对第三组进行递归，也就是说在第三组过滤特殊字符同时也引入了问题




