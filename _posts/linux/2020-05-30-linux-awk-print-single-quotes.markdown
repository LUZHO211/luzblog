---
layout: post
title: awk指令中单引号的处理
subtitle: awk中如何输出单引号？
date: 2020-05-30 11:44:51.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: 
 - Linux
 - awk
---

最近使用awk指令来处理一个文本文件，获取文本内容中的某些信息并转换成INSERT SQL语句，便于插入数据库。文本内容如下所示：

```text
zhangsan	张三	11981111360
lisi	李四	11020000138
wangwu	王五	11132222962
```

>第一列是昵称，第二列是中文名，第三列是手机号，这里都是随便写的。

最终要转换成INSERT SQL如下：

```sql
INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES ('zhangsan', '张三', '11981111360');
INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES ('lisi', '李四', '11020000138');
INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES ('wangwu', '王五', '11132222962');
```

这里使用awk指令来处理，INSERT SQL中的值需要单引号包裹起来，例如 `'zhangsan'`。但是由于`awk '{print}'`语法中已经包含单引号`'`，所以我们在awk中不能直接写单引号`'`，否则awk会无法正常执行。

awk中可以使用其他方式来变相输出单引号，这里列出三种方式。

- 使用单引号`'`的16进制`\x27`替代

```bash
$ awk '{print "INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES (\x27"$1"\x27, \x27"$2"\x27, \x27"$3"\x27);"}' users.txt
```

- 使用单引号`'`的8进制`\047`替代

```bash
$ awk '{print "INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES (\047"$1"\047, \047"$2"\047, \047"$3"\047);"}' users.txt
```

- 使用awk自带选项`-v`定义变量

```bash
$ awk -v sq="'" '{print "INSERT INTO `st_user` (`nickname`, `nickname_cn`, `mobile`) VALUES ("sq$1sq", "sq$2sq", "sq$3sq");"}' users.txt
```

`-v sq="'"`定义了一个变量sq，变量的值为`"'"`，两个双引号将一个单引号包裹，表示sq的值为单引号`'`。然后就可以在print表达式中使用变量sq来替换单引号`'`的输出。