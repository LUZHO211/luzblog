---
layout: post
title: Linux时间/时区设置
subtitle: /etc/localtime & /etc/timezone
date: 2020-07-27 14:08:45.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
 - Linux
---

Linux中设置时间为东八区时间（北京时间）：

```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ echo 'Asia/Shanghai' > /etc/timezone
$ date -R
Mon, 27 Jul 2020 14:12:04 +0800
```

/etc/localtime文件描述的是本机时间；/etc/timezone文件描述的是本机所属的时区。/usr/share/zoneinfo目录下面存放的是全球各个时区/时间的文件：

```bash
$ ls /usr/share/zoneinfo
Africa      Atlantic   Chile    Eire     Factory  GMT-0      Iceland      Jamaica            localtime  Navajo   Portugal    right      Turkey     WET
America     Australia  CST6CDT  EST      GB       GMT+0      Indian       Japan              MET        NZ       posix       ROC        UCT        W-SU
Antarctica  Brazil     Cuba     EST5EDT  GB-Eire  Greenwich  Iran         Kwajalein          Mexico     NZ-CHAT  posixrules  ROK        Universal  zone1970.tab
Arctic      Canada     EET      Etc      GMT      Hongkong   iso3166.tab  leap-seconds.list  MST        Pacific  PRC         Singapore  US         zone.tab
Asia        CET        Egypt    Europe   GMT0     HST        Israel       Libya              MST7MDT    Poland   PST8PDT     SystemV    UTC        Zulu
```

举个例子，有时候在构建Docker镜像的时候，需要设置好时区，否则运行起来会发现时间是UTC时间。只需要在Dockerfile文件中加入以下内容：

```bash
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' > /etc/timezone
```

如果是Java应用，通过`echo 'Asia/Shanghai' > /etc/timezone`已经正确设置了时区为东八区，那么代码里面就不需要对时区进行设置：

```java
TimeZone timeZone = TimeZone.getTimeZone("Asia/Shanghai");
TimeZone.setDefault(timeZone);
```

##### 参考文章

1. [https://blog.csdn.net/alinyua/article/details/80944543](https://blog.csdn.net/alinyua/article/details/80944543)