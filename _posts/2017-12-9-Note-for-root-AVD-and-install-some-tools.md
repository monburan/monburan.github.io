---
layout: post
title: Note for How to root AVD emulator
categories: [Android]
tags: [AVD, Root, Emulator]
description: "如何将AVD模拟器root的折腾记录"
fullview: false
comments: true
---
# Need

Android Studio，SDK以及AVD必须的工具，这里就不一一列举了。

SuperSU需要su文件以及SuperSu.apk，我是用的是[Recovery V2.79 Flashable.zip](http://www.supersu.com/download)

# Root 教程

## Step 1

创建好虚拟机，然后开机【废话

使用 `adb` 查看当前设备是否在线

    adb devices

解压下载好的 `SuperSU` 安装 `SuperSU.apk`
    
    adb -e install SuperSU.apk

关机【等等...教程没有这么简单。

## Step 2

为了后面写入文件需要，要将设备的系统分区设置成可写。

    emulator -avd <emulator_name> -writable-system

## Step 3

然后重新打开一个 Terminal 向 AVD 写入su的二进制文件。

    adb root
    adb remount

这里要注意！！！请根据你设备的情况选择su文件。如果你是X86架构那就使用X86文件夹下的 su 文件，如果你是 Android 5.1 以上用户请使用 su.pie ，如下：

    adb -e push su.pie /system/xbin/su
    
如果是 Android 5.1 以下用户使用 su ，如下：

    adb -e push su /system/xbin/su

写入su文件的位置其实还可以选用 `/system/bin` 不过我没有测试可行性，有需要的可以试一下。

测试下su文件是否写入，如果没报错就写入成功，如果报错请确保 Step 2 以及 Step 3 的前两个命令是否成功执行。

    adb -e shell su

然后更改su文件权限
    
    adb shell "chmod 06755 /system/xbin/su"

然后执行

     adb shell "su --install"
     adb shell "su --daemon&"
     adb shell "setenforce 0"

这个时候在 AVD 中点开之前安装 SuperSU，提示你更新su的二进制文件，选择Normal模式，更新完之后重启设备就root好了。

具体这个方法参考的是下面的几个链接，也是google了好久才找到的。

1.[Rooting the Android Emulator – on Android Studio 2.3 (Android 4.4+)](https://infosectrek.wordpress.com/2017/03/06/rooting-the-android-emulator/)

2.[Root Android virtual device with Android 7.1.1](https://android.stackexchange.com/questions/171442/root-android-virtual-device-with-android-7-1-1/176400#176400)

目前完美适配Android Studio 3.0.1，用这个方法测试了 Android 4.x 以及 5.x 的虚拟机都可以。

# 后记

这个虚拟机搞了一天也没找到能和测试目标能够适配的环境【苦笑，不是Xposed不能正常安装就是目标应用在模拟器上无法正常运行...嘛...就当是学习记录了。
