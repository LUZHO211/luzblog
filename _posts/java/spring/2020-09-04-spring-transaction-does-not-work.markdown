---
layout: post
title: Spring事务失效的原因
subtitle: Spring事务使用不当导致事务失效的问题分析
date: 2020-09-04 11:36:26.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: Spring
---

最近在Review业务方的代码时发现使用了Spring事务：createDefaultSob方法中调用了save方法

![spring-transaction-save-invoke.png](/assets/images/2020-09/spring-transaction-save-invoke.png)

save方法是一个事务方法，打了`@Transactional`注解：期望发生异常时自动回滚数据库

![spring-transaction-save.png](/assets/images/2020-09/spring-transaction-save.png)

先给出结论：在createDefaultSob方法中直接通过调用save方法的方式，会导致save方法的事务不生效，发生异常时也不会回滚。这种事务方法的调用方式是错误的。

##### 为什么事务会失效？

我们知道，Spring事务本质就是通过动态代理给我们的事务方法织入异常处理的逻辑，并在发生异常时执行ROLLBACK回滚数据库状态。例如我们有个OrderService接口：

```java
public interface OrderService {
    void createOrder();
}

@Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    @Override
    public void createOrder() {
        // 创建订单逻辑：涉及到好几个DB操作
    }
}
```

通常，Spring在启动的时候扫描到`@Service`注解，会为类创建实例对象并加入到IoC容器中。但是由于Spring扫描到OrderServiceImpl的createOrder方法上有`@Transactional`注解，于是Spring框架知道这是一个事务方法，所以会为OrderServiceImpl生成一个代理类来拦截OrderServiceImpl中的所有方法并将该代理类实例添加到IoC容器中。

所以我们在`@Autowired`这个OrderService的时候，实际上拿到的是OrderServiceImpl的代理类的实例，而不是OrderServiceImpl类的实例：

![proxy-instance.png](/assets/images/2020-09/proxy-instance.png)

因为createOrder上有`@Transactional`注解，所以Spring在代理类中对这个方法进行了增强：在反射调用invoke进行try...catch，并在catch到异常的时候进行数据库ROLLBACK操作。

所以Spring事务的是否生效，取决于我们是否是通过「代理类的对象实例」来进行方法的调用。例如最上面提到的例子，直接在createDefaultSob方法中调用了save这个事务方法，这种调用方式是不会走代理调用的，所以事务也根本不会生效。我们应该通过`xxxService.method`这种调用方式，才能使事务生效：

```java
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired
    private OrderService orderService;

    public void create() {
        orderService.createOrder();
    }

    @Transactional
    @Override
    public void createOrder() {
        // 创建订单逻辑：涉及到好几个DB操作
    }
}
```

因为`@Autowired private OrderService orderService;`注入的是代理类对象，所以在使用`orderService.createOrder()`调用时能正确走到事务的逻辑。

除了`xxxService.method`这种调用方式，我们也可以通过`@EnableAspectJAutoProxy`注解，将代理类暴露到ThreadLocal中，然后通过`AopContext.currentProxy()`来获取当前类的代理对象：

![enableAspectJAutoProxy.png](/assets/images/2020-09/enableAspectJAutoProxy.png)
![aop-context.png](/assets/images/2020-09/aop-context.png)

**但是注意，`@EnableAspectJAutoProxy(exposeProxy = true)`不能保证一定能够正确工作：**

![EnableAspectJAutoProxy-exposeProxy.png](/assets/images/2020-09/EnableAspectJAutoProxy-exposeProxy.png)

如果直接调用`createOrder()`就没有走代理，直接走的普通方法调用：

```java
@Service
public class OrderServiceImpl implements OrderService {
    @Autowired
    private OrderService orderService;

    public void create() {
        createOrder();
    }

    @Transactional
    @Override
    public void createOrder() {
        // 创建订单逻辑：涉及到好几个DB操作
    }
}
```

这样一来，Spring的事务就失效了。以上是Spring事务失效的其中一种情况的分析。我们在使用Spring事务的时候，想要使Spring事务正常工作，可能还需要注意以下几点：

- 包含`@Transactional`注解的类必须要被Spring IoC容器管理，否则Spring没法扫描到该Bean为其生成代理类；
- 要为数据源配置「事务管理器」：PlatformTransactionManager；
- 要确保我们的数据库操作的表是支持事务的，例如InnoDB支持事务，而MyISAM的数据表就不支持事务；
- 方法的异常不能自己try...catch消化掉，否则Spring事务没法感知到你的方法抛了异常，也就不会回滚；
- 事务的rollbackFor异常类型配置错误，例如配置rollbackFor=SQLException.class，但是你在方法中却抛出BuzzException.class异常，异常类型不匹配也无法让Spring事务感知到；
- Spring事务传播机制要配置正确，例如：

```java
@Service
public class Test {
    @Autowired
    private Test test;

    @Transactional(propagation = Propagation.REQUIRED)
    public void method1() {
        /** 业务操作 ... */
        test.method2();
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void method2() {
        /** 数据库操作 ... */
    }
}
```

method2()的事务传播机制是`Propagation.NOT_SUPPORTED`，即不支持事务。如果当前存在事务，它会挂起当前事务，并以非事务的方式执行method2()。执行完method2再恢复method1的事务。在这个例子中，method2()就不会以事务方式执行，发生异常也不会回滚method2()中涉及到的数据库操作。
