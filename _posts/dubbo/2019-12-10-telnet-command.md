---
layout: post
title: 使用telnet命令来进行Dubbo接口测试
date: 2019-12-10 23:29:00.000000000 +08:00
tags: Dubbo
---

>从 `2.0.5` 版本开始，Dubbo 开始支持通过 telnet 命令来进行服务治理。

使用 telnet 命令来直接连接到 Dubbo 的服务提供者，从而直接在命令行控制台使用 Dubbo 提供的指令来进行服务治理，例如调用 Dubbo 接口、查看服务者状态以及服务者提供的接口等信息。

```bash
$ telnet localhost 20880
Trying ::1...
Connected to localhost.
Escape character is '^]'.

dubbo>
```

telnet连接上 Dubbo 服务提供者之后，可以通过 Dubbo 内建的telnet命令来跟 Dubbo 服务提供者交互。

### 一、`ls` 命令

- `ls` —— 列出 Dubbo 服务提供者提供的接口列表；
- `ls -l` —— 列出 Dubbo 服务提供者提供的接口详细信息列表；
- `ls xxxService` —— 列出某个 Dubbo 接口提供的方法列表；
- `ls -l xxxService` —— 列出某个 Dubbo 接口提供的方法详情列表。

`ls`查看提供了哪些接口

```bash
dubbo>ls
com.luzho211.endurance.dubbo.ComputerService
com.luzho211.endurance.dubbo.CameraService
com.luzho211.endurance.dubbo.PrinterService
```

`ls -l` 查看接口详细信息

```bash
dubbo>ls -l
com.luzho211.endurance.dubbo.ComputerService -> dubbo://192.168.1.101:20880/com.luzho211.endurance.dubbo.ComputerService?anyhost=true&application=dubbo-telnet-service&bind.ip=192.168.1.101&bind.port=20880&dubbo=2.6.2&generic=false&interface=com.luzho211.endurance.dubbo.ComputerService&methods=getComputerType,restart&pid=33024&side=provider&timestamp=1576075027350
com.luzho211.endurance.dubbo.CameraService -> dubbo://192.168.1.101:20880/com.luzho211.endurance.dubbo.CameraService?anyhost=true&application=dubbo-telnet-service&bind.ip=192.168.1.101&bind.port=20880&dubbo=2.6.2&generic=false&interface=com.luzho211.endurance.dubbo.CameraService&methods=getCameraVendorName,getCameraColor&pid=33024&side=provider&timestamp=1576075022280
com.luzho211.endurance.dubbo.PrinterService -> dubbo://192.168.1.101:20880/com.luzho211.endurance.dubbo.PrinterService?anyhost=true&application=dubbo-telnet-service&bind.ip=192.168.1.101&bind.port=20880&dubbo=2.6.2&generic=false&interface=com.luzho211.endurance.dubbo.PrinterService&methods=getPrinterVendorName,printText&pid=33024&side=provider&timestamp=1576075006858
```

`ls xxxService` 查看某个 Dubbo 接口提供的方法

```bash
dubbo>ls com.luzho211.endurance.dubbo.PrinterService
getPrinterVendorName
printText
```

>也可以直接 `ls PrinterService`，省去接口的全限定名。

`ls -l xxxService` 查看提供的方法的详情：方法的参数、返回值类型
 
```bash
dubbo>ls -l PrinterService
java.lang.String getPrinterVendorName()
void printText(java.lang.String)
```

### 二、`ps` 命令

- `ps` —— 查看服务提供者的端口列表；
- `ps -l` —— 查看服务提供者的地址列表；
- `ps port` —— 查看服务提供者的端口上的连接信息（连接的客户端信息）；
- `ps -l port` —— 查看服务提供者的端口上的连接详细信息。

```bash
dubbo>ps
20880

dubbo>ps -l
dubbo://192.168.1.101:20880

dubbo>ps 20880
/0:0:0:0:0:0:0:1:49358

dubbo>ps -l 20880
/0:0:0:0:0:0:0:1:49358 -> /0:0:0:0:0:0:0:1:20880
```

### 三、`cd`、`pwd` 命令

- `cd xxxService` —— 改变缺省服务。设置缺省服务可以不用每次都输入全限定名来使用服务，方便操作；
- `cd /` —— 取消缺省服务，即不设置缺省服务；
- `pwd` —— 使用 `pwd` 可以查看当前设置的缺省服务。

```bash
dubbo>cd com.luzho211.endurance.dubbo.PrinterService
Used the com.luzho211.endurance.dubbo.PrinterService as default.
You can cancel default service by command: cd /

dubbo>pwd
com.luzho211.endurance.dubbo.PrinterService

dubbo>cd /
Cancelled default service com.luzho211.endurance.dubbo.PrinterService.

dubbo>pwd
/
```

### 四、`invoke` 命令

>invoke命令用来调用 Dubbo 方法进行接口测试。可以省去接口全限定名而直接调用方法名，但是如果不同的接口服务包含了同一个方法名，就需要全限定名或者 xxxService.method 方式来调用。当 invoke 时匹配到了多个方法，也可以使用 `select` 命令来选择要掉用的方法，例如：select 1

```bash
dubbo>invoke printText("Hello, World!")
null
elapsed: 0 ms.
```

### 五、`count` 命令：统计服务调用情况

- `count xxxService 5` —— 统计某接口服务下的所有方法的调用情况：统计5次
- `count xxxService xxxMethod 5` —— 统计某接口服务下的某个方法的调用情况：统计5次

```bash
dubbo>count com.luzho211.endurance.dubbo.PrinterService getPrinterVendorName 3
+----------------------+-------+--------+--------+---------+-----+
| method               | total | failed | active | average | max |
+----------------------+-------+--------+--------+---------+-----+
| getPrinterVendorName | 0     | 0      | 0      | 0ms     | 0ms |
+----------------------+-------+--------+--------+---------+-----+

+----------------------+-------+--------+--------+---------+-----+
| method               | total | failed | active | average | max |
+----------------------+-------+--------+--------+---------+-----+
| getPrinterVendorName | 0     | 0      | 0      | 0ms     | 0ms |
+----------------------+-------+--------+--------+---------+-----+

+----------------------+-------+--------+--------+---------+-----+
| method               | total | failed | active | average | max |
+----------------------+-------+--------+--------+---------+-----+
| getPrinterVendorName | 0     | 0      | 0      | 0ms     | 0ms |
+----------------------+-------+--------+--------+---------+-----+
```

### 六、`status` 命令：查看服务的汇总情况

```bash
dubbo>status -l
+------------+--------+--------------------------------------------------------+
| resource   | status | message                                                |
+------------+--------+--------------------------------------------------------+
| threadpool | OK     | Pool status:OK, max:200, core:200, largest:24, active:1, task:24, service port: 20880 |
| load       | OK     | load:3.22021484375,cpu:4                               |
| memory     | OK     | max:1820M,total:163M,used:60M,free:103M                |
| registry   | OK     | 47.111.96.193:2281(connected)                          |
| server     | OK     | /192.168.1.101:20880(clients:1)                        |
| spring     | OK     |                                                        |
| summary    | OK     |                                                        |
+------------+--------+--------------------------------------------------------+
```

>以上所有状态均OK

```bash
dubbo>status -l
+------------+--------+--------------------------------------------------------+
| resource   | status | message                                                |
+------------+--------+--------------------------------------------------------+
| threadpool | OK     | Pool status:OK, max:200, core:200, largest:28, active:1, task:28, service port: 20880 |
| load       | WARN   | load:7.51611328125,cpu:4                               |
| memory     | OK     | max:1820M,total:163M,used:62M,free:101M                |
| registry   | OK     | 47.111.96.193:2281(connected)                          |
| server     | OK     | /192.168.1.101:20880(clients:1)                        |
| spring     | OK     |                                                        |
| summary    | WARN   | load                                                   |
+------------+--------+--------------------------------------------------------+
```

>以上有些状态为警告WARN，例如上面的WARN显示的是cpu负载警告，可能是cpu负载有点高的原因。

<br /><br />

*参考文章*

*1. [Telnet 命令参考手册](http://dubbo.apache.org/zh-cn/docs/user/references/telnet.html)* <br />