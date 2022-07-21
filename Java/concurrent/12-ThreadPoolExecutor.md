## 线程池状态

ThreadPoolExecutor使用int的高3位来表示线程池状态，低29位表示线程数量

| 状态名     | 高3位 | 接收新任务 | 处理阻塞队列任务 | 说明                                     |
| ---------- | ----- | ---------- | ---------------- | ---------------------------------------- |
| RUNNING    | 111   | Y          | Y                | -                                        |
| SHUTDOWN   | 000   | N          | Y                | 不会接收新任务，但会处理阻塞队列剩余任务 |
| STOP       | 001   | N          | N                | 会中断正在执行的任务，并抛弃阻塞队列任务 |
| TIDYING    | 010   | -          | -                | 任务全执行完毕，活动线程为0即将进入终结  |
| TERMINATED | 011   | -          | -                | 终结状态                                 |

从数字上比较，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING

这些信息存储在一个原子变量ctl中，目的是将线程池状态与线程个数合二为一，这样就可以用一次cas原子操作进行赋值

```java
// c为旧值，ctlOf返回结果为新值
ctl.compareAndSet(c, ctlOf(targetState, workerCount(c)));
// rs为高3位代表线程池状态，wc为低29位代表线程个数，ctl是合并它们
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

## 构造方法

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHadler hadler)
```

- corePoolSize核心线程数目（最多保留的线程数）
- maximumPoolSize最大线程数
- keepAliveTime生存时间-针对救急线程
- unit时间单位-针对救急线程
- workQueue阻塞队列
- threadFactory线程工厂-可以为线程创建时起个名字
- handler拒绝策略

## 工作过程

- 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务
- 当线程数达到corePoolSize并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue队列排队，直到有空闲的线程
- 如果队列选择了有界队列，那么任务超过了队列大小是，会创建maximumPoolSize - corePoolSize数目的线程来救急
- 如果线程达到了maximumPoolSize仍然有新任务这时会执行拒绝策略。拒绝策略jdk提供了4种实现，其他著名框架也提供了实现
  - AbortPolicy让调用者抛出RejectedExecutionExection异常，这是默认策略
  - CallerRunsPolicy让调用者运行任务
  - DiscardPolicy放弃本次任务
  - DiscardOldestPolicy放弃队列中最早的任务，本任务取而代之
  - Dubbo的实现，在抛出RejectedExecutionExection异常之前会记录日志，并dump线程信息，方便定位问题
  - Netty的实现，是创建一个新线程来执行任务
  - ActiveMQ的实现，带超时等待(60s)尝试放入队列，类似我们自定义的拒绝策略
  - PinPoint的实现，它使用了一个拒绝策略，会逐一尝试策略链中每种拒绝策略
- 当高峰过去后，超过corePoolSize的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由keepAliveTime和unit来控制

## newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int threads) {
    return new ThreadPoolExecutor(threads, threads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

特点

- 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间
- 阻塞队列是无界的，可以放任意数量的任务

> 适用于任务量已知，相对耗时的任务

## newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}
```

特点

- 核心线程数是0，最大线程数是Integer.MAX_VALUE,救急线程的空闲生存时间是60s, 意味着
  - 全部都是救急线程（60s后可以回收）
  - 救急线程可以无限创建
- 队列采用了SynchronousQueue实现特点是，它没有容量，没有线程来取是放不进去的（一手交钱，一手交货）

```java
```

