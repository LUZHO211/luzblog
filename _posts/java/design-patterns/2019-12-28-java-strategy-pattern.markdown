---
layout: post
title: 设计模式系列「1」：策略模式
subtitle: 策略模式在Java中的应用
date: 2019-12-28 00:00:00.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
  - Java
  - 策略模式
---

策略模式就是定义一系列算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。策略模式属于对象行为模式，它通过对算法进行封装，把使用算法的责任和算法的实现分割开来，并委派给不同的对象对这些算法进行管理。

策略模式可以优化代码中出现很多`if...else`，提升代码的封装性以及可维护性；而且恰当地使用继承可以把算法族的公共代码转移到父类里面，从而避免重复的代码。

登录场景的例子：用户登录可能会使用短信验证码、企业微信验证码或者将军令动态码进行登录，这时候可以将每一种验证码的认证逻辑都封装一种策略类，在登录的时候动态选择对应的认证策略。

策略类接口：`LoginCodeAuthenticator` <br />
```java
public class LoginCodeAuthenticator {
    void checkLoginCode(Credential credential);
}
```

验证码认证策略类：`SmsCodeAuthenticator`、`EwechatCodeAuthenticator` <br />
```java
@Service
public class SmsCodeAuthenticator implements LoginCodeAuthenticator {
    @Override
    public void checkLoginCode(Credential credential) {
        // 验证短信验证码是否正确的逻辑...
    }
}

@Service
public class EwechatCodeAuthenticator implements LoginCodeAuthenticator {
    @Override
    public void checkLoginCode(Credential credential) {
        // 验证企业微信验证码是否正确的逻辑...
    }
}
```

登录参数类：`Credential` <br />
```java
public class Credential {
    private String username;
    private String password;
    private CodeTypeEnum codeType;  // 验证码类型
    private String code;  // 验证码
}

/* 验证码类型枚举类 */
public enum CodeTypeEnum {
    SMS, EWECHAT, OTP
}
```

策略环境类（Context）：`LoginCodeAuthenticatorHolder` <br />
```java
@Service
public class LoginCodeAuthenticatorHolder {

    @Autowired
    private Map<String, LoginCodeAuthenticator> loginCodeAuthenticatorMap; // 存放所有的策略类

    public LoginCodeAuthenticator get(CodeTypeEnum codeType) {
        return loginCodeAuthenticatorMap.get(getBeanName(codeType));
    }

    private String getBeanName(CodeTypeEnum codeType) {
        return codeType.name().toLowerCase() + "LoginCodeAuthenticator";
    }
}
```

登录验证码认证逻辑：<br />
```java
@Service
public class LoginCodeService {
    @Autowired
    private LoginCodeAuthenticatorHolder loginCodeAuthenticatorHolder;

    public void checkLoginCode(Credential credential) {
        // 如果不使用策略模式，将会是一连串的if...else
        if (codeType == CodeTypeEnum.SMS) {
            // 短信验证码认证逻辑 ...
        } else if (codeType == CodeTypeEnum.EWECHAT) {
            // 企业微信验证码认证逻辑 ...
        } else if (codeType == CodeTypeEnum.OTP) {
            // 将军令动态码认证逻辑 ...
        } else if (....) {

        } else {

        }
        ... ...
        // 下面使用策略模式
        CodeTypeEnum codeType = credential.getCodeType();
        loginCodeAuthenticatorHolder.get(codeType).checkLoginCode(credential);
    }
}
```

如上面例子所示，如果策略很多，`LoginCodeService.checkLoginCode`方法里面将会是很长的一串`if...else`，可维护性和封装性都比较差，而且不是很面向对象的编程方式。

使用策略模式：可以优化代码中的一大串`if...else`，提升代码的可维护性和封装性，将不同的处理逻辑交给对应的策略类来完成，更加面向对象。后续在新增其他认证方式，这个方法也不需要改动；新增的认证方式封装在新的策略类里面，也不容易影响到现有的策略类的行为。




