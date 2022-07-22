## 概念

Fork/Join是JDK1.7加入的新的线程池实现，他体现的是一种分治思想，适用于能够进行拆分CPU密集型运算

所谓的任务拆分，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计算，如归并排序、斐波那契数列、都可以用分治思想进行求解

Fork/Join在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运算效率

Fork/Join默认会创建与cpu核心数大小相同的线程池

## 使用

提交给Fork/Join线程池的任务需要继承RecursiveTask(有返回值)或RecursiveAction(没有返回值)，例如下面定义了一个对1-n之间求和任务

```java
@Slf4j
public class MyTask extends RecursiveTask<Integer> {
    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        log.debug("result {}", pool.invoke(new MyTask(5)));
    }
    private int n;
    public MyTask(int n) {
        this.n = n;
    }
    @Override
    protected Integer compute() {
        if (n == 1) {
            log.debug("join() {}", 1);
            return 1;
        }
        MyTask t1 = new MyTask(n - 1);
        t1.fork();
        log.debug("fork() {} + {}", n, t1);
        int result = n + t1.join();
        log.debug("join() {} + {} = {}", n, t1, result);
        return result;
    }
}
```

打印

```
16:33:39.162 [ForkJoinPool-1-worker-1] DEBUG com.bytebuf.fork.MyTask - fork() 5 + com.bytebuf.fork.MyTask@3bc8364e
16:33:39.162 [ForkJoinPool-1-worker-2] DEBUG com.bytebuf.fork.MyTask - fork() 4 + com.bytebuf.fork.MyTask@5dff2ccb
16:33:39.162 [ForkJoinPool-1-worker-3] DEBUG com.bytebuf.fork.MyTask - fork() 3 + com.bytebuf.fork.MyTask@2fde0476
16:33:39.162 [ForkJoinPool-1-worker-5] DEBUG com.bytebuf.fork.MyTask - join() 1
16:33:39.162 [ForkJoinPool-1-worker-4] DEBUG com.bytebuf.fork.MyTask - fork() 2 + com.bytebuf.fork.MyTask@13be9517
16:33:39.167 [ForkJoinPool-1-worker-4] DEBUG com.bytebuf.fork.MyTask - join() 2 + com.bytebuf.fork.MyTask@13be9517 = 3
16:33:39.168 [ForkJoinPool-1-worker-3] DEBUG com.bytebuf.fork.MyTask - join() 3 + com.bytebuf.fork.MyTask@2fde0476 = 6
16:33:39.168 [ForkJoinPool-1-worker-2] DEBUG com.bytebuf.fork.MyTask - join() 4 + com.bytebuf.fork.MyTask@5dff2ccb = 10
16:33:39.168 [ForkJoinPool-1-worker-1] DEBUG com.bytebuf.fork.MyTask - join() 5 + com.bytebuf.fork.MyTask@3bc8364e = 15
16:33:39.168 [main] DEBUG com.bytebuf.fork.MyTask - result 15
```

改进

```java
@Slf4j
public class MyTask extends RecursiveTask<Integer> {
    public static void main(String[] args) {
        ForkJoinPool pool = new ForkJoinPool();
        log.debug("result {}", pool.invoke(new MyTask(0, 5)));
    }
    private int begin;
    private int end;
    public MyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }
    @Override
    public String toString() {
        return "{" + begin + "," + end + "}";
    }
    @Override
    protected Integer compute() {
        if (begin == end) {
            log.debug("join() {}", begin);
            return begin;
        }
        if (end - begin == 1) {
            log.debug("join() {} + {} = {}", begin, end, end + begin);
            return end + begin;
        }
        int mid = (end + begin) / 2;
        MyTask t1 = new MyTask(begin, mid);
        t1.fork();
        MyTask t2 = new MyTask(mid + 1, end);
        t2.fork();
        log.debug("fork() {} + {} = ?", t1, t2);
        int result = t1.join() + t2.join();
        log.debug("join() {} + {} = {}", t1, t2, result);
        return result;
    }
}
```

打印结果

```
16:39:36.733 [ForkJoinPool-1-worker-7] DEBUG com.bytebuf.fork.MyTask - join() 2
16:39:36.733 [ForkJoinPool-1-worker-4] DEBUG com.bytebuf.fork.MyTask - join() 0 + 1 = 1
16:39:36.733 [ForkJoinPool-1-worker-5] DEBUG com.bytebuf.fork.MyTask - join() 3 + 4 = 7
16:39:36.733 [ForkJoinPool-1-worker-6] DEBUG com.bytebuf.fork.MyTask - join() 5
16:39:36.733 [ForkJoinPool-1-worker-2] DEBUG com.bytebuf.fork.MyTask - fork() {0,1} + {2,2} = ?
16:39:36.733 [ForkJoinPool-1-worker-1] DEBUG com.bytebuf.fork.MyTask - fork() {0,2} + {3,5} = ?
16:39:36.733 [ForkJoinPool-1-worker-3] DEBUG com.bytebuf.fork.MyTask - fork() {3,4} + {5,5} = ?
16:39:36.737 [ForkJoinPool-1-worker-2] DEBUG com.bytebuf.fork.MyTask - join() {0,1} + {2,2} = 3
16:39:36.737 [ForkJoinPool-1-worker-3] DEBUG com.bytebuf.fork.MyTask - join() {3,4} + {5,5} = 12
16:39:36.737 [ForkJoinPool-1-worker-1] DEBUG com.bytebuf.fork.MyTask - join() {0,2} + {3,5} = 15
16:39:36.737 [main] DEBUG com.bytebuf.fork.MyTask - result 15
```

