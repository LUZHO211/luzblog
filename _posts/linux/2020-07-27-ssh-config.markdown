---
layout: post
title: SSH config
subtitle: ssh config相关配置学习&记录
date: 2020-07-27 16:27:45.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
 - Linux
 - git
 - SSH
---

SSH的配置文件有两个，一个是系统配置，一个是用户配置文件。

- `/etc/ssh/ssh_config`：SSH的系统配置文件；
- `~/.ssh/config`：用户的SSH配置文件

我们可以直接在用户主目录的`.ssh`目录下创建一个名叫`config`的文件来配置自己的一些SSH相关配置，以简化我们日常的一些SSH操作。下面根据场景来说明。

### 1. SSH别名登录远程服务器

SSH登录远程服务器，但是如果每次都需要输入账号、远程服务器域名（或者IP）会很麻烦，例如：

```bash
$ ssh appweb@192.168.100.4
$ ssh root@my.aliyun.ecs
```

使用SSH config我们可以给我们的远程服务器取一个有意义、易识别的别名，例如：`ssh myaliyun`表示登录我自己的阿里云机器；`ssh gongsi`表示登录公司的跳板机。那么，要怎么配置呢？

方法是：在`.ssh`目录下创建一个`config`文件，编辑内容如下：

```bash
Host myaliyun
  HostName 192.168.100.1
  User root
  IdentityFile ~/.ssh/id_rsa
```

以上用到的就是SSH的配置项（这里只用到了比较常用的配置项），说明如下：

```bash
# 配置文件参数
# Host : 远程服务器别名，写一个易识别的名字
# HostName : 远程服务器的域名或者IP地址
# User : SSH登录用户名，例如root
# IdentityFile : SSH登录服务器的私钥文件路径，例如~/.ssh/id_rsa
# Port: SSH端口号，不配置默认22
```

配置完毕之后，就可以直接使用`ssh myaliyun`登录我的阿里云主机，比较方便。

### 2. SSH管理多组密钥对

我们本地电脑可能会存在多组SSH密钥对，例如：

- 公司的GitLab代码仓库：`id_rsa_gitlab/id_rsa_gitlab.pub`
- 个人的GitHub代码仓库：`id_rsa_github/id_rsa_github.pub`

我们在`git clone`或者`git push`的时候都需要手动切换使用的密钥，比较麻烦。通过SSH config配置，我们可以让SSH根据我们的配置帮我们自动选择对应的密钥。配置如下：

```bash
# 公司GitLab仓库SSH配置
Host gongsi.gitlab.com
  HostName gongsi.gitlab.com
  IdentityFile ~/.ssh/id_rsa_gitlab
  User yansong

# 个人GitHub仓库SSH配置
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_github
  User pandaq
```

配置完毕后，就可以直接执行`git clone git@github.com:LUZHO211/luzblog.git`来clone我的GitHub项目，而不用担心我本地存在多组密钥对而导致失败的情况了。SSH会自动选择对应的密钥。

如果本地存在多组SSH密钥对，可能需要通过`ssh-add`命令事先将这些密钥添加到ssh-agent中：

```bash
$ ssh-add ~/.ssh/id_rsa_gitlab
$ ssh-add ~/.ssh/id_rsa_github
```

##### 参考文章

1. <a href="https://deepzz.com/post/how-to-setup-ssh-config.html" target="_blank">https://deepzz.com/post/how-to-setup-ssh-config.html</a>
2. <a href="https://man.linuxde.net/ssh-agent" target="_blank">https://man.linuxde.net/ssh-agent</a>



