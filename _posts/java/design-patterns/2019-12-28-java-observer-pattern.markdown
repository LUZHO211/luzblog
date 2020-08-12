---
layout: post
title: 设计模式系列「2」：观察者模式
subtitle: 观察者模式在Java中的应用
date: 2019-12-28 02:00:00.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
  - Java
  - 设计模式
---

观察者模式是一种对象行为模式，定义对象间一种一对多的依赖关系，使得当每一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

观察者模式包含「观察目标」和「观察者」两类对象，一个目标可以有任意数目的与之相依赖的观察者，一旦观察目标的状态发生改变，所有的观察者都将得到通知。

观察者模式也叫发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。细究的话，发布-订阅和观察者有些不同，可以理解成发布-订阅模式属于广义上的观察者模式。

观察者模式中的「角色」：

- Subject（目标）：Subject是被观察者，可以是接口、抽象类或者具体的类；
- ConcreteSubject（具体目标）：ConcreteSubject是目标的子类，当它的状态发生变化，它会向它的各个观察者发出通知；
- Observer（观察者）：Observer一般定义为接口，接口声明了更新数据的方法`update()`；
- ConcreteObserver（具体观察者）：实现了观察者接口，目标对象发生变化，会通知到具体观察者。

### 1. JDK对观察者模式的支持

JDK提供了`java.util.Observable`类以及`java.util.Observer`接口，分别对应了「Subject目标」和「Observer观察者」。下面来看看如何使用JDK提供的观察者模式。

先定义具体的被观察者：订单服务类OrderService.java 继承 `java.util.Observable`类 <br />

```java
public class OrderService extends Observable {
    public void submitOrder() {
        setChanged();  // 设置状态变化
        System.out.println("「被观察者」状态发生变化，通知各个观察者 ...");
        this.notifyObservers();
    }
}
```

然后再定义两个具体的观察者类：物流服务CarService.java和短信服务SmsService.java类，这两个观察者类都实现`java.util.Observer`接口 <br />

```java
public class CarService implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("「物流观察者」收到通知，装车发货 ...");
    }
}

public class SmsService implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("「短信观察者」收到通知，给客户发送短信 ...");
    }
}
```

最后来使用「客户端」：<br />

```java
public class Test {
    public static void main(String[] args) {
        OrderService orderService = new OrderService();
        orderService.addObserver(new SmsService());  // 向被观察者注册短信观察者对象
        orderService.addObserver(new CarService());  // 向被观察者注册物流观察者对象
        // 用户提交订单
        orderService.submitOrder();
    }
}
```

![observable-pattern-1.png](/assets/images/2019-12/observable-pattern-1.png)
![observable-pattern-2.png](/assets/images/2019-12/observable-pattern-2.png)

### 2. Spring中的观察者模式

Spring框架中的「事件监听机制」就是观察者模式的一种应用。它由四个角色组成：

- 事件（ApplicationEvent）：继承自`java.util.EventObject`，表示通知的事件类型，是通知的上下文载体。Spring内置了很多事件：ContextRefreshedEvent、ContextStartedEvent、ContextStoppedEvent、ContextClosedEvent等。
- 监听器（ApplicationListener）：即观察者，继承自`java.util.EventListener`。该接口有一个方法`onApplicationEvent`，当监听的事件发布后，该方法会被回调。
- 事件源（ApplicationContext）：ApplicationContext是Spring的核心容器，在事件监听机制中作为事件发布者的角色，继承自`ApplicationEventPublisher`接口（定义了`publishEvent`方法用于发布事件通知）。
- 事件管理（ApplicationEventMulticaster）：用于注册事件监听器和事件的广播。

下面来看看Spring的观察者模式（事件监听机制）如何使用：

定义一个下单成功的事件：OrderSuccessEvent.java

```java
public class OrderSuccessEvent extends ApplicationEvent {
    public OrderSuccessEvent(Object source) {
        super(source);
    }
}
```

定义两个事件监听器（即观察者）：SmsService.java和CarService.java

```java
@Service
public class SmsService implements ApplicationListener<OrderSuccessEvent> {
    @Override
    public void onApplicationEvent(OrderSuccessEvent event) {
        System.out.println("「短信监听器（观察者）」收到通知，给客户发送短信 ...");
    }
}

@Service
public class CarService implements ApplicationListener<OrderSuccessEvent> {
    @Override
    public void onApplicationEvent(OrderSuccessEvent event) {
        System.out.println("「物流监听器（观察者）」收到通知，装车发货 ...");
    }
}
```

然后是订单服务类（被观察者）：OrderService.java

```java
@Service
public class OrderService {

    @Autowired
    private ApplicationContext applicationContext;

    public void submitOrder() {
        System.out.println("「被观察者」订单创建成功，发布下单成功事件通知 ...");
        applicationContext.publishEvent(new OrderSuccessEvent(this));
    }
}
```

### 3. 「观察者模式」总结

观察者模式的优点：

- 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系；
- 目标与观察者之间建立了一套触发机制；
- 符合「开闭原则」的要求，对扩展开放，对修改是封闭的。

观察者模式的缺点：

- 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用，使用时应避免循环引用的情况；
- 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

##### 参考文章

1. <a href="https://juejin.im/post/6844904100459446285" target="_blank">观察者模式，这一篇就搞定</a>
2. <a href="https://www.runoob.com/design-pattern/observer-pattern.html" target="_blank">观察者模式</a>
3. <a href="https://www.cnblogs.com/az4215/p/11489712.html" target="_blank">设计模式六大原则：开闭原则</a>