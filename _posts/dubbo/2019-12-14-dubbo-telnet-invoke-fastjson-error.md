---
layout: post
title: Dubbo telnet：invoke缺少fastjson惹的祸
date: 2019-12-14 14:52:22.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: Dubbo
---

>在使用 Dubbo 的telnet invoke命令测试 Dubbo 接口时，telnet控制台返回了一个错误：`Invalid json argument, cause: com/alibaba/fastjson/JSON`。

从错误信息看，好像跟 fastjson 有点关系。于是就看了一下 Dubbo 处理 invoke 命令的处理器Handler代码来看看：

```java
package org.apache.dubbo.rpc.protocol.dubbo.telnet;

import org.apache.dubbo.remoting.telnet.TelnetHandler;

public class InvokeTelnetHandler implements TelnetHandler {
    
    // 忽略这里的代码 ...
    
    @Override
    @SuppressWarnings("unchecked")
    public String telnet(Channel channel, String message) {
        // 以上代码忽略 ...
        try {
            list = JSON.parseArray("[" + args + "]", Object.class);
        } catch (Throwable t) {
            return "Invalid json argument, cause: " + t.getMessage();
        }
        // 以下代码忽略 ...
    }
    
    // 忽略下面的其他方法 ...
    
}
```

发现里面在解析invoke命令参数的时候，使用了 fastjson 的 JSON.parseArray 方法。由于项目中没有引入 fastjson 的依赖，所以 Dubbo telnet 在解析参数的时候抛出了异常。

为证实这点，启动单点调试。如下图，从详细的异常信息来看，确实是缺少了 fastjson 的依赖导致的异常：

![](/assets/images/2019-12/telnet-invoke-fastjson-error.png)


所以给项目加上 fastjson 依赖就没问题了。

>所以为什么 Dubbo 本身用到了 fastjson 的代码，而自己又不引入 fastjson 这个依赖呢？




