---
layout: post
title: Dubbo调用超时机制
subtitle: 学习Dubbo调用超时机制的底层实现&总结
date: 2020-06-12 21:52:22.000000000 +08:00
header-img: assets/images/tag-bg.jpg
author: PandaQ
tags: Dubbo
---

先上一张官网的Dubbo服务调用过程图：

![dubbo-send-request-process.jpg](/assets/images/2020-06/dubbo-send-request-process.jpg)

- 消费者端引入服务提供者的API包依赖，Dubbo框架会为API接口生成代理类Proxy；
- 消费者调用API实际上是通过代理类Proxy来发起一次远程调用；
- 网络客户端Client将消息编码/序列化后，通过网络发送给服务提供者（Server）；
- 服务提供者（Server）收到请求，解码/反序列化消息，然后通过分发器（Dispatcher）派发到指定的线程池上；
- 最后由线程池调用具体的服务实现。

上面这张图描述得比较简单，实际上整个调用过程还有很多细节。例如消费者在实际发送网络请求之前，会经历如下调用逻辑：

```text
proxy0#sayHello(String)
  —> InvokerInvocationHandler#invoke(Object, Method, Object[])
    —> MockClusterInvoker#invoke(Invocation)
      —> AbstractClusterInvoker#invoke(Invocation)
        —> FailoverClusterInvoker#doInvoke(Invocation, List<Invoker<T>>, LoadBalance)
          —> Filter#invoke(Invoker, Invocation)  // 包含多个Filter调用
            —> ListenerInvokerWrapper#invoke(Invocation) 
              —> AbstractInvoker#invoke(Invocation) 
                —> DubboInvoker#doInvoke(Invocation)
                  —> ReferenceCountExchangeClient#request(Object, int)
                    —> HeaderExchangeClient#request(Object, int)
                      —> HeaderExchangeChannel#request(Object, int)
                        —> AbstractPeer#send(Object)
                          —> AbstractClient#send(Object, boolean)
                            —> NettyChannel#send(Object, boolean)
                              —> NioClientSocketChannel#write(Object)
```

最后`NioClientSocketChannel#write(Object)`这一步才是实际将请求发送到网络上。

### 1. Dubbo超时机制

首先Dubbo的超时逻辑是在消费者客户端实现的，通过Debug调用过程，我们很容易定位到超时逻辑代码，就在DubboInvoker.doInvoke这里：

```java
public class DubboInvoker<T> extends AbstractInvoker<T> {
    @Override
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);

        ExchangeClient currentClient;
        if (clients.length == 1) {
            currentClient = clients[0];
        } else {
            currentClient = clients[index.getAndIncrement() % clients.length];
        }
        try {
            boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
            boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
            int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            if (isOneway) {
                boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
                currentClient.send(inv, isSent);
                RpcContext.getContext().setFuture(null);
                return new RpcResult();
            } else if (isAsync) {
                ResponseFuture future = currentClient.request(inv, timeout);
                RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
                return new RpcResult();
            } else {
                RpcContext.getContext().setFuture(null);
                return (Result) currentClient.request(inv, timeout).get();
            }
        } catch (TimeoutException e) {
            throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        } catch (RemotingException e) {
            throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }
}
```