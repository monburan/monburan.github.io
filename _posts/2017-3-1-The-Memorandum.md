---
layout: post
title: 备忘录
categories: [普通日志]
tags: [memo]
fullview: false
comments: true
---
一个简单的备忘录，记录一些常用的Ubuntu软件安装方法以及一些不需要记住的东西。

# Ubuntu非常用软件

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
