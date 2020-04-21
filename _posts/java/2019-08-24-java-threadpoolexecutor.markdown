---
layout: post
title: Java线程池的使用
date: 2019-08-24 12:41:45.000000000 +08:00
tags: Java
---

```java
// 创建线程池的核心构造函数
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
    
        if (corePoolSize < 0 || maximumPoolSize <= 0 || maximumPoolSize < corePoolSize || keepAliveTime < 0)
            throw new IllegalArgumentException();
        
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        
        this.acc = System.getSecurityManager() == null ? null : AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

|参数名称|参数说明|
|:---|:---|
|corePoolSize|线程池的核心线程数，一旦创建就会一直保留在线程池中（除非调用allowCoreThreadTimeOut(true)方法将allowCoreThreadTimeOut参数设置成true）|
|maximumPoolSize|线程池中允许存活的最大线程数|
|keepAliveTime|当创建的线程数量**超过了核心线程数**，允许线程池中处于空闲状态的**非核心线程**的存活时间（若设置了allowCoreThreadTimeOut参数为true，核心线程也会被回收）|
|unit|keepAliveTime参数的时间单位|
|workQueue|工作队列，用于存放将被执行的线程任务（Runnable tasks）：阻塞队列|
|threadFactory|创建线程的工厂，可以用于标记区分不同线程池所创建出来的线程|
|handler|当线程池和工作队列的容量**均达到上限**，再向线程池提交任务时所触发的拒绝策略逻辑handler|

### 一、线程池大小与线程存活时间

线程池有两个关于线程数量配置的参数：corePoolSize和maximumPoolSize：

- corePoolSize —— 设置线程池的核心线程数
- maximumPoolSize —— 设置线程池中允许存活的最大线程数

keepAliveTime参数用于设置**非核心线程**的存活时间，使用参数unit指定时间单位。即如果**非核心线程**空闲的时间超过了所设置的keepAliveTime，线程就会被回收。默认情况下，核心线程一旦被创建就不会被回收，但是如果若设置了allowCoreThreadTimeOut参数为true，核心线程也会被回收。

```java
// 允许回收空闲的核心线程
threadPoolExecutor.allowCoreThreadTimeOut(true);
```

线程池创建后，默认情况下线程池中不会有任何线程，只有当有线程任务提交到线程池中，线程池才会创建线程来处理任务。但是如果显式调用了prestartAllCoreThreads()或者prestartCoreThread()方法，会立即创建核心线程。

```java
// 调用prestartAllCoreThreads()来立即创建所有核心线程
threadPoolExecutor.prestartAllCoreThreads();
```

### 二、线程池工作队列

Java使用阻塞队列（BlockingQueue）作为线程池的工作队列，可以直接应用在多线程并发的环境下缓存线程任务。

>如果阻塞队列为空（empty），则尝试从队列中获取（读取）任务的线程会被阻塞；如果阻塞队列满了（full），则尝试往队列中插入任务的线程会被阻塞。

阻塞队列java.util.concurrent.BlockingQueue是一个接口，这个接口类中定义的四种类型的方法，分别会在无法完成写入、读取元素等队列操作的情况下作出不同的反应：

|操作类型|直接抛异常|返回特定值|一直阻塞|超时退出|
|:---|:---|:---|:---|:---|
|插入元素（Insert）|add(e)|offer(e)|put(e)|offer(e, time, unit)|
|删除元素（Remove）|remove()|poll()|take()`|poll(time, unit)|
|检查（Examine）|element()|peek()|not applicable|not applicable|


阻塞队列在JDK实现类中的常用几个如下说明：

|阻塞队列实现类|阻塞队列说明|
|:---|:---|
|ArrayBlockingQueue|一个基于数组结构的**有界**阻塞队列。|
|LinkedBlockingQueue|一个基于链表结构的**有界**阻塞队列。|
|SynchronousQueue|一个不存储任何元素的阻塞队列。一个线程的插入操作必须等待另一个线程来读取才能完成，才会允许下一个插入操作。CachedThreadPool用的就是这个队列，所以向线程池中提交任务就会新建一个或分配一个空闲的线程来处理任务，因为SynchronousQueue不会存储任何元素。|
|PriorityBlockingQueue|一个支持优先级排序的**无界**阻塞队列。|
|DelayQueue|一个使用优先级队列实现的**无界**阻塞队列。用于处理延迟任务。|

### 三、线程池拒绝策略

当线程池的工作队列满了，并且线程池中线程数量已达到最大值；再往线程池中提交任务时，就会触发拒绝策略。ThreadPoolExecutor内置了四种拒绝策略：

- AbortPolicy：取消策略，丢弃任务并抛出RejectedExecutionException，默认的拒绝策略。
- DiscardPolicy：丢弃策略，丢弃任务但是不会抛出任何异常。
- DiscardOldestPolicy：丢弃策略，丢弃队列中最老的任务并尝试重新执行所提交的任务。
- CallerRunsPolicy：调用者执行策略，将任务直接给提交该任务的线程来执行。

>以上四种拒绝策略是 ThreadPoolExecutor 内置的，对于被拒绝的任务处理比较简单。当然，我们也可以根据我们的需求场景来继承这些策略类或者直接实现 RejectedExecutionHandler 接口来自定义拒绝策略。

### 四、线程池内部处理逻辑（重点理解）

1. 当线程池中的线程数量**小于**核心线程数：新提交一个任务时，无论是否存在空闲的线程，线程池都将新建一个新的线程来执行新任务；
2. 当线程池中的线程数量**等于**核心线程数（核心线程已满）：新提交的任务会被存储到工作队列中，等待空闲线程来执行，**而不会创建新线程**；
3. 当工作队列已满，并且池中的线程数量**小于**最大线程数（maximumPoolSize）：如果继续提交新的任务，线程池会创建新线程来处理任务；
4. 当工作队列已满，并且池中线程数量已达到最大值：继续提交新任务时，线程池会触发拒绝策略处理逻辑；
5. 如果线程池中存在空闲的线程并且其空闲时间达到了 keepAliveTime 参数的限定值，线程池会回收这些空闲线程，但是线程池不会回收空闲的核心线程（若设置了 allowCoreThreadTimeOut 参数为true，核心线程也会被回收）。

### 五、向线程池提交任务

向线程池中提交线程任务有两种方式：调用execute方法或者调用submit方法。

- **execute**：定义在`java.util.concurrent.Executor`接口中

```java
void execute(Runnable command);
```

- **submit**：定义在`java.util.concurrent.ExecutorService`接口中

```java
// submit方法定义有三个，参数和返回值不一样而已：这里只贴出其中一个方法的实现
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

- submit方法只是将线程任务封装成一个FutureTask，最终还是调用execute方法来执行任务。
- submit方法会返回一个Future，通过调用Future.get()方法可以获取到任务的执行结果；execute方法返回void，无法获取任务执行结果。
- 若提交的线程任务里面抛出异常：execute会直接将异常抛出并退出线程；submit会**吞掉异常**，并在Future.get()的时候将异常抛出。


### 六、Executors工具类
JDK提供了`java.util.concurrent.Executors`这个工具类来帮助开发者快速创建线程池：

- `Executors.newFixedThreadPool(int nThreads)`：创建一个固定数量线程的线程池，池中的线程数量**达到最大值后会始终保持不变**。
- `Executors.newSingleThreadExecutor()`：创建一个只包含单个线程的线程池，可以保证所有任务按提交的顺序被单一的一个线程串行地执行。
- `Executors.newCachedThreadPool()`：创建一个会根据任务按需地创建、回收线程的线程池。这种类型线程池适合执行数量多、耗时少的任务。
- `Executors.newScheduledThreadPool(int corePoolSize)`：创建一个具有定时功能的线程池，适用于执行定时任务。

>这些线程池本质上也是对 ThreadPoolExecutor 进行封装，所以我们还是必须要了解 ThreadPoolExecutor 。

```java
// newFixedThreadPool
new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());

// newSingleThreadExecutor
new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());

// newCachedThreadPool
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());

// newScheduledThreadPool
new ScheduledThreadPoolExecutor(corePoolSize);

public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
}
```

以上四种定义好的线程池在某些场景确实很方便我们的使用，但是我们需要了解它们的隐患之处：

- **FixedThreadPool**和**SingleThreadPool**所允许的工作队列最大容量为`Integer.MAX_VALUE`，这有可能会随着工作队列中的任务堆积而导致`OOM`；
- **CachedThreadPool**和**ScheduledThreadPool**所允许创建的线程数量为`Integer.MAX_VALUE`，这也有可能因为创建大量线程导致`OOM`或者线程切换开销巨大。