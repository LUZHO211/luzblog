---
layout: post
title: 记一次Spring RedisTemplate序列化乱码问题的排查过程
date: 2020-03-14 16:00:00.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: SpringBoot
---

最近在SpringBoot项目中使用Redis + Lua脚本来做一个功能需求，使用的是[Spring Data Redis](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-nosql.html#boot-features-redis)中的RedisTemplate来进行Redis的操作。Lua脚本内容如下：

```lua
local key = KEYS[1]
local limit = ARGV[1]
if (redis.call('INCR', key) > limit) then
    redis.call('DEL', key)
    return 0
else
    return 1
end
```

>脚本说明：<br />
>对某个Key进行`+1`操作，若`+1`之后大于`ARGV[1]`参数指定的值，则删除该Key并返回`0`；否则返回`1`。

执行脚本的Java代码如下：

```java
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;
import java.util.Collections;

@Service
public class IncrHandler {

    private final RedisTemplate redisTemplate;
    private final DefaultRedisScript<Long> redisScript;

    public IncrHandler(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
        // Lua脚本路径设置
        redisScript = new DefaultRedisScript<>();
        redisScript.setResultType(Long.class);
        redisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("lua/incr.lua")));
    }

    public boolean incrOK() {
        String key = "incr_key";
        Long OK = 1L;
        int limit = 10;
        Object result = redisTemplate.execute(redisScript, Collections.singletonList(key), limit);
        return OK.equals(result);
    }

}
```

在调用执行`incrHandler.incrOK()`的时候抛出了异常：

>RR Error running script (call to f_263e9cf28f8033489b846caf5394c82459740761): <br />
>@user_script:3: user_script:3: attempt to compare string with number ; <br />
>nested exception is redis.clients.jedis.exceptions.JedisDataException: <br />
>ERR Error running script (call to f_263e9cf28f8033489b846caf5394c82459740761): <br />
>@user_script:3: user_script:3: attempt to compare string with number

意思是脚本在执行第3行代码的时候，发现两个比较的参数不是同一个数据类型，一个是string而另一个是number。

`redis.call('INCR', key)`这个代码返回的是INCR操作的结果，肯定是number；那么意思就是limit参数是string类型？我明明调用的时候传入的limit是整数类型：

```java
int limit = 10;
Object result = redisTemplate.execute(redisScript, Collections.singletonList(key), limit);
```

于是想到在Lua脚本中增加日志输出，观察一下我传入的limit参数在进入Lua脚本之后是什么东西：

```lua
local key = KEYS[1]
local limit = ARGV[1]
redis.log(redis.LOG_NOTICE, type(limit))
redis.log(redis.LOG_NOTICE, limit)
if (redis.call('INCR', key) > limit) then
    redis.call('DEL', key)
    return 0
else
    return 1
end
```

>Lua脚本中可以使用redis.log()来打印日志，这个日志会输出在Redis服务器的日志中。所以调用redis.log()时注意Redis服务器的配置文件redis.conf中设置的日志级别要匹配，不然有可能看不到日志输出。

Redis服务器输出的日志如下：

```bash
1:M 14 Mar 2020 08:50:59.797 * Ready to accept connections
1:M 14 Mar 2020 08:51:21.071 * string
1:M 14 Mar 2020 08:51:21.071 * ��
```

可以看到我的参数传到Lua脚本之后，类型变成了string，而且参数的值变成了乱码。

对代码进行单步调试，发现在将Key和args进行序列化的时候都变成了乱码，所以Lua脚本在获取ARGV[1]参数的时候，发现无法转换成number。

![redistemplate-jdk-serialize.png](/assets/images/2020-03/redistemplate-jdk-serialize.png)

这是因为RedisTemplate默认使用的是JDK序列化，反序列化也只能使用JDK反序列化。所以Redis服务器并不能正确地反序列化Key和args参数，导致反序列化的结果为乱码。

这有点坑啊，为什么默认使用JDK序列化，明知JDK序列化之后Redis服务器是不能正确反序列化的😓。

所以我们需要为RedisTemplate设置其他通用的序列化方式。

或者直接使用StringRedisTemplate来替代RedisTemplate，完美避开JDK序列化。

```java
public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        // StringRedisTemplate使用StringRedisSerializer来作为默认的序列化方式
        RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        setKeySerializer(stringSerializer);
        setValueSerializer(stringSerializer);
        setHashKeySerializer(stringSerializer);
        setHashValueSerializer(stringSerializer);
    }
}
```
