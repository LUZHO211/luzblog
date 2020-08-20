---
layout: post
title: 设计模式系列(3)：模板模式
subtitle: 模板模式在Java中的应用
date: 2019-12-28 04:00:00.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags:
  - Java
  - 设计模式
---

一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。模板模式的意图就是定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

打个比方，公司有一套固定的办事流程，公司（抽象父类）规定了该流程的执行规则、执行顺序，但是该流程中的具体实施步骤需要具体某些员工（子类）去完成。公司只管控整个流程的执行规范、顺序，具体的各个步骤就留给底下的员工去实施。这样做的好处很明显，大家都按照这个公司规范的骨架来办事，就算以后换了一个新员工也不会影响公司的整个办事流程，一样有章可循。

从代码实现角度来说，模板模式就是一个抽象父类定义了一套执行流程，这些流程由父类控制如何调用，但是流程中涉及到某些方法可能需要子类提供具体的实现，那么就把这些方法暴露给子类，让子类去覆写提供自己的实现。这种模式在很多通用的框架中都有使用，因为要做到通用就必须提供一个骨架来规范框架的行为，但是也会暴露方法让使用框架的开发者去提供自己的实现。

去银行办理业务的时候，银行都有自己的一个流程：排队取号、办理业务、评价打分等流程。

```java
public abstract class BankService {
    /** 排队取号 */
    protected abstract void queuing();
    /** 办理具体业务 */
    protected abstract void handleTask();
    /** 给业务员的服务评价打分 */
    protected abstract void evaluate();

    /** 银行业务处理流程：模板方法 */
    public final void process() {
        queuing();
        handleTask();
        evaluate();
    }
}
```

>模板方法`process()`使用了`final`关键字修饰，这个很重要。因为模板方法是规范流程的，不能被子类覆写，否则整个流程有可能被修改，导致银行业务处理混乱。

银行系统只要在模板方法中规范这些流程的执行顺序即可，剩下的步骤由具体业务部门去处理。例如，现在来了一个客户办理取钱业务：

```java
public class WithdrawMoneyService extends BankService {
    @Override
    protected void queuing() {
        System.out.println("排队取「综合业务」的服务号 ...");
    }
    @Override
    protected void handleTask() {
        System.out.println("正在办理取钱业务 ...");
    }
    @Override
    protected void evaluate() {
        System.out.println("给取钱业务服务评价打分 ...");
    }
}
```

现在，又来了一个客户要办理信用卡业务：

```java
public class CreditCardService extends BankService {
    @Override
    protected void queuing() {
        System.out.println("排队取「VIP业务」的服务号 ...");
    }
    @Override
    protected void handleTask() {
        System.out.println("正在办理信用卡业务 ...");
    }
    @Override
    protected void evaluate() {
        System.out.println("给信用卡业务服务评价打分 ...");
    }
}
```

最后来看看如何使用：

```java
public class Test {
    public static void main(String[] args) {
        BankService bankService = new WithdrawMoneyService();
        bankService.process();  // 办理取钱
        bankService = new CreditCardService();
        bankService.process();  // 办理信用卡
    }
}
```

模板模式的优点：

- 父类控制算法结构，子类无法影响算法结构，只负责提供实现；
- 符合「开闭原则」：对修改关闭，对扩展开放；
- 通用的代码可以放在父类，更好地封装、重用代码。

模板模式的缺点：

- 不同的实现都要对应一个子类，实现过多的时候可能会导致代码臃肿。

##### Dubbo框架中模板模式的应用

Dubbo作为一个通用的框架，也提供了很多扩展接口，例如负载均衡策略的实现，Dubbo顶层提供了一个抽象类AbstractLoadBalance来规范负载均衡的调用逻辑，并且暴露`doSelect`方法让各个子类去提供自己的算法实现。并且在AbstractLoadBalance类中定义了一些通用的方法例如`getWeight`，这样子类也都能用到，体现了代码封装性、重用性。

![abstract-loadbalance.png](/assets/images/2019-12/abstract-loadbalance.png)

然后Dubbo提供了四种负载均衡策略：加权随机算法、加权最小活跃调用数算法、一致性Hash均衡算法、以及加权轮询策略。下面随便贴一个具体的实现：

![random-loadbalance.png](/assets/images/2019-12/random-loadbalance.png)
