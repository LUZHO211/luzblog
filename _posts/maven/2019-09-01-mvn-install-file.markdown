---
layout: post
title: mvn install:install-file
subtitle: 通过mvn install命令将jar包安装到本地Maven仓库
date: 2019-09-01 20:14:31.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: Maven
---

有时候拿到一个第三方jar包，但是在中央仓库中并没有这个Maven依赖，为了顺利进行本地开发，可以先将该jar包安装到本地Maven仓库。例如有个`otpserver.jar`包，安装命令如下：


`mvn install:install-file -DgroupId=otpserver -DartifactId=otpserver -Dversion=1.0 -Dpackaging=jar -Dfile=./otpserver.jar`


![mvn-install.png](/assets/images/2019-09/mvn-install.png)

安装完毕，可以正常在项目的pom.xml文件中引入该依赖：

```text
<dependency>
    <groupId>otpserver</groupId>
    <artifactId>otpserver</artifactId>
    <version>1.0</version>
</dependency>
```

##### 参考文章

1. <a href="https://maven.apache.org/plugins/maven-install-plugin/usage.html" target="_blank">https://maven.apache.org/plugins/maven-install-plugin/usage.html</a>
