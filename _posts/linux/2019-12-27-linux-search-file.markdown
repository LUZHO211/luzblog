---
layout: post
title: Linux查找文件的命令
date: 2019-12-27 17:28:34.000000000 +08:00
tags: Linux
---

日常开发中经常会用到Linux查找某个文件的命令，这里记录一下自己常用的几个。参考鸟哥的Linux学习网站：[鳥哥的 Linux 私房菜](http://linux.vbird.org)。

### 一、find

>Linux下最常见、最强大的文件查找命令算是find命令了。find命令直接搜索磁盘目录，虽然遍历磁盘目录效率低，但是却可以找到你想找的任何文件。

```
find [PATH] [option] [action]

PATH   - 指定搜索的目录，可选。如果不指定搜索目录，则默认搜索当前目录及其子目录。
option - 指定搜索的选项和参数，可选。
action - 额外执行的动作，可选。
```

重点看搜索的选项和参数：[option]。

>与文件权限和文件名称搜索相关的参数（只列出部分参数）

```bash
-name filename：使用-name参数来指定搜索名称为 filename 的文件。
-size [+-]SIZE：搜索比SIZE还要大(+)或小(-)的文件。
                SIZE的单位可以为：c：表示byte；k表示2014B；M表示2014kB；G表示1024MB
                例如要搜索比1GB还要大的文件，就是：-size +1G  
```
             
                
*范例1：查找文件名为 hosts 的文件*

```
$ find / -name hosts
/var/cache/nscd/hosts
/var/lib/docker/containers/7fef2acae8e8dc2156f8387cbf3b98aab1b582c981af4ef536a53277d248acf4/hosts
/etc/hosts
```

*范例2：结合通配符`*`进行文件名的模糊搜索，结果看出文件名包含hosts关键字的都搜出来了*

```
$ find / -name "*hosts*"
/etc/hosts
/etc/hosts.allow
/etc/hosts.deny
/lib/x86_64-linux-gnu/security/pam_rhosts.so
/usr/share/vim/vim74/ftplugin/hostsaccess.vim
```

通配符`*`可以匹配任意的字符。如果要找文件名以hosts开头的文件，则应该为`"hosts*"`；如果要找文件名以hosts结尾的文件，应该为`"*hosts"`。

需要注意的是，find命令是直接搜索硬盘的，所以搜索时会比较消耗磁盘。

### 二、locate

locate命令与find不同，它不会直接搜索硬盘，而是查询一个文件存档数据库：`var/lib/mlocate/mlocate.db`，所以locate的搜索效率会比find高很多。`var/lib/mlocate/mlocate.db`里面维护了硬盘里面所有文件的创建，但是一般情况下Linux只会每天通过扫描硬盘来更新这个数据库，这时候新建或者删除的文件才能感知到。也就是说，如果你现在新建一个文件，立即用locate来搜索，是搜索不到的。

locate命令直接就是模糊搜索，还是挺好用的：

```bash
$ locate password
/var/lib/pam/password
/usr/share/man/man1/systemd-tty-ask-password-agent.1.gz
/usr/share/man/man8/systemd-ask-password-console.path.8.gz
/usr/share/man/man8/systemd-ask-password-console.service.8.gz
/usr/share/man/man8/systemd-ask-password-wall.path.8.gz
/usr/share/man/man8/systemd-ask-password-wall.service.8.gz
```

如果我现在新建一个文件，然后搜索该文件：

```bash
$ touch test-file
$ locate test-file
```

将会搜索不到test-file这个文件，因为数据库还没有被更新。我们可以通过手动更新`var/lib/mlocate/mlocate.db`数据库，使用`updatedb`命令。然后再搜索test-file：

```bash
$ updatedb
$ locate test-file
/root/test-file
```

这个时候就能搜索到test-file文件了。同样的，如果某个文件被删除了，在没有更新`var/lib/mlocate/mlocate.db`数据库之前，通过locate还是能搜索到该文件的。

>updatedb命令会根据/etc/updatedb.conf配置文件中的配置去搜索对应的目录文件，然后更新/var/lib/mlocate目录中的文件档案数据库。

### 三、which

which命令用于搜索可执行的命令文件，例如查找`vim`、`ls`、`touch`等命令，想知道某个Linux命令位于哪个目录下，用which就对了。which会根据PATH环境变量中设定的目录去查找命令文件。

```bash
范例1：搜索which命令所在目录
$ which which
/sbin/ifconfig

范例2：搜索ls命令所在目录
$ which ls
/bin/ls

范例3：搜索history命令所在目录
$ which history

搜索不到history，因为history是bash内建的命令，并不在PATH所指定的目录中。
```

### 四、whereis

whereis命令跟find命令一样也是直接搜索硬盘，只不过whereis搜索的是特定的一些目录。使用`whereis -l`来查看whereis命令搜索的目录：

```bash
$ whereis -l
bin: /usr/bin
bin: /usr/sbin
... ...
man: /usr/share/man/pt
man: /usr/share/info
src: /usr/src/linux-headers-4.4.0-148-generic
src: /usr/src/linux-headers-4.4.0-159-generic
src: /usr/src/linux-headers-4.4.0-148
src: /usr/src/linux-headers-4.4.0-159
```


<br /><br />
*参考文章*

*1. [6.5 指令與檔案的搜尋](http://linux.vbird.org/linux_basic/0220filemanager.php#file_find)* <br />