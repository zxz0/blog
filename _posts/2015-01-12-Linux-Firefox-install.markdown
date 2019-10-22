---
layout: post
title:  "Linux下firefox安装"
date:   2015-01-12 15:01:48 +0800
categories: Linux
tags: Linux Firefox Install
description: 在Linux下安装Firefox，出现的一系列问题以及解决方法
---
转为root，
```sh
yum install firefox
```
但是随即出现问题：
(firefox:4871): GnomeUI-WARNING \*\*: While connecting to session manager:
None of the authentication protocols specified are supported.

/usr/lib/firefox/firefox: symbol lookup error: /usr/lib/libgdk-x11-2.0.so.0: undefined symbol: \_XGetRequest

yum真不是个好东西。。。

google被墙，百度找不到，辗转到yahoo日本去找。

找到：
[Problem With VNC refreshing (Centos 6.2) - nixCraft Linux / UNIX ...](https://www.nixcraft.com/showthread.php/19976-Problem-With-VNC-refreshing-(Centos-6-2))

/usr/bin/gnome-session: symbol lookup error: /usr/lib64/libgdk-x11-2.0.so.0: undefined symbol: \_XGetRequest (nautilus:17444): EggSMClient-WARNING \*\*: Failed to connect to the session manager: IO error occured opening ...
地址：http://nixcraft.com/showthread.php/19976-Problem-With-VNC-refreshing-(Centos-6-2)



正文太长懒得看，反正有一样的错误，直接看解决方案：

> I figrued out what was going wrong. After looking into the log file and finding that error message, I was able to deduce that I need to upade xlibs. Once this was done everything worked fine.

> For anyone that runs into this issue run this command:
> ```sh
	yum install libX11
  ```

解决。