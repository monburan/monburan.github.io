---
layout: post
title: 一些软件安装的备忘录
categories: [Configuration]
tags: [Memo, Ubuntu]
description: "一个简单的备忘录，记录一些常用的Ubuntu软件安装方法以及一些不需要记住的东西。"
fullview: false
comments: true
---
# Ubuntu 常用软件配置

## Ubuntu shutter 截图快捷键设置

    shutter -s

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

## Ubuntu Terminator 图标修复

默认Ubuntu 使用 Unity 桌面，Terminator使用Gonme驱动所以修改后才能使图标固定在启动栏

    gsettings set org.gnome.desktop.default-applications.terminal exec 'terminator'

## Git 代理设置

    git config --global http.proxy http://127.0.0.1:1080
    git config --global https.proxy http://127.0.0.1:1080

## 设置系统目录为英文

默认的中文目录非常不方便，可以只设置系统目录为英文

    export LANG=en_US
    xdg-user-dirs-gtk-update
    export LANG=zh-CN

## 安装 Google Chrome

    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add - 
    sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list'
    sudo apt update
    sudo apt install google-chrome-xxxxx

# Ubuntu 非常用软件配置

## Ubuntu 安装 WhatWeb(fix bug)

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

## Ubuntu 安装 Metasploit

    curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
    chmod 755 msfinstall && \
    ./msfinstall

## Ubuntu 安装 exploit-db

Frist step: get exploit-db in github

    git clone https://github.com/offensive-security/exploit-database.git /opt/exploit-database

Next:link

    ln -sf /opt/exploit-database/searchsploit /usr/local/bin/searchsploit