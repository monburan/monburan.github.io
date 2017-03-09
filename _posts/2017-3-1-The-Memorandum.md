---
layout: post
title: 一些软件安装的备忘录
categories: [Configuration]
tags: [Memo, Ubuntu]
description: "一个简单的备忘录，记录一些常用的Ubuntu软件安装方法以及一些不需要记住的东西。"
fullview: false
comments: true
---
# Ubuntu非常用软件

## Ubuntu install WhatWeb(fix bug)

在安装whatweb之后发现报错:

    /usr/bin/whatweb: /usr/lib/ruby/vendor_ruby/rchardet/universaldetector.rb:39: invalid multibyte escape: /[\x80-\xFF]/ (SyntaxError)

运行环境:
    
    Ubuntu 16.04
    ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]

问题解决:

    sudo vi /usr/lib/ruby/vendor_ruby/rchardet/universaldetector.rb

在文件第一行加入

    # encoding: US-ASCII

Bug fix 启动正常，但是仍然报错

    /usr/share/whatweb/lib/tld.rb:85: warning: key "2nd_level_registration" is duplicated and overwritten on line 85
    /usr/share/whatweb/lib/tld.rb:93: warning: key "2nd_level_registration" is duplicated and overwritten on line 93
    /usr/share/whatweb/lib/tld.rb:95: warning: key "2nd_level_registration" is duplicated and overwritten on line 95


## Ubuntu install Metasploit

<pre><code>curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
chmod 755 msfinstall && \
./msfinstall</code></pre>

## Ubuntu install exploit-db

Frist step: get exploit-db in github

<pre><code>git clone https://github.com/offensive-security/exploit-database.git /opt/exploit-database
</code></pre>

Next:link 

    ln -sf /opt/exploit-database/searchsploit /usr/local/bin/searchsploit

## Ubuntu 显示天气

    indicator-china-weather

## Ubuntu 屏幕防蓝光

    redshift

## Ubuntu wps 中文字体解决

下载字体包解压

<code>cp</code>所有字体文件到<code>/usr/share/fonts</code>路径下

生成字体索引信息

    sudo mkfontscale 
    sudo mkfontdir

更新字体缓存

    sudo fc-cache
