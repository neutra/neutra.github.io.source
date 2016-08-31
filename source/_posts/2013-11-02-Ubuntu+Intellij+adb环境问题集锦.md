---
title: Ubuntu+Intellij+adb环境问题集锦
date: 2013-11-02
category: android
toc: true
tags:
- android
---

Intellij提示device is not available
-----

Intellij在运行android程序时提示错误 Can't upload file: device is not available.

在终端，使用 adb devices命令，显示设备状态是正常的(device)

使用 adb install xxx 或 adb uninstall yyy 都提示Failure

手机上也是显示已经是调试模式

解决办法：关掉手机调试模式，再打开。最好再重启下adb

环境搭建
-----

1.安装adb

2.安装jdk和intellij

3.把adb的所在目录加到启动PATH

设备接上后，adb无法识别
-----

首先要确定设备的生产商id(idVendor)和产品id(idProduct)
打开终端，连上USB，输入lsusb并回车；断开USB，再次输入lsusb并回车。 对比下两次打印的列表，可以看到区别是类似下面这样一行：

Bus 001 Device 027: ID 2717:1220

记下后面两个数字，前面的数字是生产商id而后面的是产品id

在adb_usb.ini中添加一行，**文件末尾不要加空行**

``` shell
    vi ~/.android/adb_usb.ini
    0x2717
```

修改udev配置并重启udev和adb

``` shell
sudo chmod 666 /etc/udev/rules/51-android.rules
sudo vi /etc/udev/rules/51-android.rules

SUBSYSTEM=="usb", ATTR{idVendor}=="2207", ATTR{idProduct}=="0010", MODE="0666"，OWNER=="<当前用户名>"

sudo service udev restart

sudo chown root:root $ANDROID_SDK/tools/adb
sudo chmod a+s $ANDROID_SDK/tools/adb

adn kill-server
adb devices
```
一些奇葩设备ID
-----

> 原道 N70S 2207:0010

> 小米 红米 2717:1220