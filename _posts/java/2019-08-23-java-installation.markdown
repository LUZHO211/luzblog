---
layout: post
title: Java开发环境搭建
date: 2019-08-23 10:28:34.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: Java
---

本文主要记录JDK安装以及JDK环境变量配置的问题。

### 一、Windows系统安装JDK

*本文使用Windows 10系统版本来做演示；JDK版本为 1.8.0_221。*

#### 下载并安装JDK
- 到[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载适合自己系统的JDK版本，本文使用`jdk-8u221-windows-x64.exe`版本；
- 双击 `exe` 文件安装，并记录安装的目录，例如：`D:\Program files\Java\jdk1.8.0_221`。

#### 配置Java环境变量


### 二、Linux系统安装JDK

*本文使用Ubuntu 16.04.6 LTS系统版本来做演示；JDK版本为 1.8.0_221。*

#### 下载并解压（安装）JDK

- 到[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载适合自己系统的JDK版本，本文使用`jdk-8u221-linux-x64.tar.gz`版本；
- 将下载的文件放到某个目录下，例如：`/usr/local`

```bash
$ mv jdk-8u221-linux-x64.tar.gz /usr/local/
```

- 进入JDK压缩文件所在目录并解压文件（**解压完毕即安装成功**），将得到`jdk1.8.0_221`文件夹：

```bash
$ tar -zxvf jdk-8u221-linux-x64.tar.gz
```

**即，JDK的安装路径为：`/usr/local/jdk1.8.0_221`，后面环境变量`JAVA_HOME`就是设置成JDK的安装路径。**

#### 配置Java环境变量

- 在用户主目录下编辑`.bashrc`文件（`vim ~/.bashrc`），在文件尾部追加以下内容：

```bash
export JAVA_HOME=/usr/local/jdk1.8.0_221
export PATH=$JAVA_HOME/bin:$PATH
```

- 编辑完毕之后，执行命令：`source ~/.bashrc`来使`.bashrc`文件立即生效。
- 最后，检验一下JDK是否安装以及环境变量是否配置正确：

```bash
$ java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

**出现以上信息说明成功安装JDK并且环境变量配置正确。否则需要检查环境变量的配置是否正确。**

### 三、Mac OS系统安装JDK

*本文使用macOS Mojave 10.14.6系统版本来做演示；JDK版本为 1.8.0_221。*

#### 下载并安装JDK
- 到[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载适合自己系统的JDK版本，本文使用`jdk-8u221-macosx-x64.dmg`版本。
- 下载完毕后，双击`jdk-8u221-macosx-x64.dmg`文件，按照引导一路点击下去即可完成安装。

#### 配置Java环境变量

- 打开终端，执行`/usr/libexec/java_home -V`指令来查看JDK的安装路径：

```bash
$ /usr/libexec/java_home -V
Matching Java Virtual Machines (1):
    1.8.0_221, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home
```

**`/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home`即JDK的安装目录，后面配置`JAVA_HOME`环境变量会使用到。**

- 编辑用户主目录下的`.bash_profile`（`vim ~/.bash_profile`），在文件中加入以下内容：

```bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.

export JAVA_HOME
export PATH
```

- 编辑完毕之后，执行命令：`source ~/.bash_profile`来使`.bash_profile`文件立即生效。
- 最后，检验一下JDK是否安装以及环境变量是否配置正确：

```bash
$ java -verion
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

**出现以上信息说明成功安装JDK并且环境变量配置正确。否则需要检查环境变量的配置是否正确。**

### 四、关于CLASSPATH环境变量

*JDK 1.5之后，不需要再设置`CLASSPATH`环境变量。*

- `CLASSPATH`环境变量作用在于告诉JER在运行Java程序时，该去哪里搜索程序所依赖的JDK类库。JDK 1.5之后已经不需要再设置`CLASSPATH`环境变量了，JRE会自动找到类库路径。
- 但是对于我们自己写的一些简单测试程序，可以通过`-classpath`或者`-cp`来指定依赖的其他第三方库的路径：
有一个`Test.java`类，`javac Test.java`编译之后执行：

```bash
$ java -classpath ~/lib;./fastjson-1.2.58.jar Test
```
