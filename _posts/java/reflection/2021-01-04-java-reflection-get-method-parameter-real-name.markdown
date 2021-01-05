---
layout: post
title: Java通过反射获取方法的参数名称
subtitle: 运行时如何获取到某个方法参数的真实名称？
date: 2021-01-04 16:55:34.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: 
    - Java
    - Java反射
---

`JDK1.8`之前我们通过反射获取一个方法定义的参数名称时，无法获取到真实的参数名称，例如：

```java
public class SomeClass {
    public String someMethod(String nickName, Integer age) {
        return "something to be return";
    }
}
```

在类编译之后，类的方法参数名称会丢失，变成`arg0`、`arg1`这样的。有些场景例如MyBatis这种通过反射来获取Mapper接口上方法参数然后映射到XML的SQL语句上，就没法直接用：

```java
@Mapper
public interface SomeMapper {
    SomePojo selectByNickNameAndMobile(String nickName, String mobile);
}
```

XML文件如下：

```xml
<select id="selectByNickNameAndMobile" resultMap="resultMap">
    SELECT * FROM `some_table` WHERE nick_name = #{nickName} AND mobile = #{mobile};
</select>
```

MyBatis通过反射获取到的参数名称其实是：`nickName -> arg0`、`mobile -> arg1` 这样就没法正确映射到XML的SQL中`#{nickName}`、`#{mobile}`。

所以有时候我们需要使用`@Param`注解来告诉MyBatis我们的参数名称，MyBatis才能正确映射参数：

```java
@Mapper
public interface SomeMapper {
    SomePojo selectByNickNameAndMobile(@Param("nickName") String nickName, @Param("mobile") String mobile);
}
```

除了通过在参数上打注解的方式，自从JDK1.8之后，Java原生的反射机制新增了一个类用于描述方法上的参数：`java.lang.reflect.Parameter`。我们可以直接通过这个反射类获取到方法参数的真实名称。

前提是我们需要在编译的时候加入`-parameter`参数，让编译器在编译类的时候将方法的参数名称也打包到.class文件里面。默认情况下`-parameter`是关闭的，如果需要此功能，需要手动开启：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <compilerArgs>
            <compilerArg>-parameters</compilerArg>
        </compilerArgs>
    </configuration>
</plugin>
```

这样之后，就算我们不在参数上打`@Param`注解，MyBatis也能获取到真实的参数名称`nickName`和`mobile`。

##### 参考文章

1. <a href="https://www.cnblogs.com/kancy/p/10205036.html" target="_blank">https://www.cnblogs.com/kancy/p/10205036.html</a>
2. <a href="https://blog.csdn.net/xiewenfeng520/article/details/102515413" target="_blank">https://blog.csdn.net/xiewenfeng520/article/details/102515413</a>
3. <a href="https://www.jianshu.com/p/4f1f7a9d595f" target="_blank">https://www.jianshu.com/p/4f1f7a9d595f</a>