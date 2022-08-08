在异步处理时，经常用到这两个接口

首先要说明Netty中的Future与JDK中的Future同名，但是是两个接口，Netty的Future继承自jdk的Future, 而Promise又对Netty的Future进行了扩展

- JDK Future只能同步等待任务结束（或成功、或失败）才能得到结果
- Netty Future可以同步等待任务结束得到结果，也可以异步方式得到结果，但都要等待任务结束
- Netty Promise不仅有Netty Future的功能，而且脱离了任务独立存在，只作为两个线程传递结果的容器

| 功能/名称  | JDK Future                     | Netty Future                                                 | Promise      |
| ---------- | ------------------------------ | ------------------------------------------------------------ | ------------ |
| cancel     | 取消任务                       | -                                                            | -            |
| isCanceled | 任务是否取消                   | -                                                            | -            |
| isDone     | 任务是否完成，不能区分成功失败 | -                                                            | -            |
| get        | 获取任务结果，阻塞等待         | -                                                            | -            |
| getNow     | -                              | 获取任务结果，非阻塞，还未产生结果时返回null                 | -            |
| await      | -                              | 等待任务结束，如果任务失败，不会抛异常，而是通过isSuccess判断 | -            |
| sync       | -                              | 等待任务结束，如果任务失败，抛出异常                         | -            |
| isSuccess  | -                              | 判断任务是否成功                                             | -            |
| setSuccess | -                              | -                                                            | 设置成功结果 |
| setFailure | -                              | -                                                            | 设置失败结果 |

## JDK Future

```java
@Slf4j
public class TestJdkFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 1. 线程池
        ExecutorService service = Executors.newFixedThreadPool(2);
        // 2. 提交任务
        Future<Integer> future = service.submit(() -> {
            log.debug("执行计算");
            Thread.sleep(1000);
            return 50;
        });
        // 3. 主线程通过future来获取结果
        log.debug("等待结果");
        log.debug("结果是 {}", future.get());
    }
}
```

打印结果

```
21:24:29.975 [main] DEBUG com.bytebuf.netty.TestJdkFuture - 等待结果
21:24:29.975 [pool-1-thread-1] DEBUG com.bytebuf.netty.TestJdkFuture - 执行计算
21:24:30.978 [main] DEBUG com.bytebuf.netty.TestJdkFuture - 结果是 50
```

## Netty Future

```java
@Slf4j
public class TestNettyFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        EventLoop eventLoop = group.next();
        Future<Integer> future = eventLoop.submit(() -> {
            log.debug("执行计算");
            Thread.sleep(1000);
            return 70;
        });
        //log.debug("等待结果");
        //log.debug("结果是 {}", future.get());
        // 异步接收结果
        future.addListener(f -> {
            log.debug("接收结果: {}", f.getNow());
        });
    }
}
```

## Netty Promise

```java
@Slf4j
public class TestNettyPromise {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 1. 准备EventLoop对象
        EventLoop eventLoop = new NioEventLoopGroup().next();
        // 2. 可以主动创建promise, 结果容器
        DefaultPromise<Integer> promise = new DefaultPromise<>(eventLoop);
        new Thread(() -> {
            // 3. 任意一个线程执行计算，计算完毕后向Promise填充结果
            log.debug("开始计算");
            try {
                int i = 1 / 0;
                Thread.sleep(1000);
                promise.setSuccess(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
                promise.setFailure(e);
            }

        }).start();
        // 4. 接收结果的线程
        log.debug("等待结果");
        log.debug("结果是: {}", promise.get());
    }
}
```

