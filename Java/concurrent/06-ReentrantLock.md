相较于synchronized它具有如下特点

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

与synchronized一样，都支持可重入

基本语法

```java
//获取锁
reentrantLock.lock();
try {
  //临界区
} finally {
  //释放锁
  reentrantLock.unLock();
}
```

## 可重入

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权再次获取这把锁

如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住

```java
@Slf4j
public class Test1 {
    static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        method1();
    }
    public static void method1() {
        lock.lock();
        try {
            log.debug("execute method1");
            method2();
        } finally {
            lock.unlock();
        }
    }
    public static void method2() {
        lock.lock();
        try {
            log.debug("execute method2");
        } finally {
            lock.unlock();
        }
    }
}
```

## 可打断

```java
@Slf4j
public class Test2 {
    static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        new Thread(() -> {
            try {
                // 如果没有竞争那么此方法获取lock对象锁
                // 如果有竞争就进入阻塞队列，可以被其它线程用interrupt方法打断
                log.debug("尝试获取锁");
                lock.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("没有获得锁，返回");
                return;
            }
            try {
                log.debug("获取锁");
            } finally {
                lock.unlock();
            }
        }).start();
    }
}
```

## 锁超时

立刻失败

```java
@Slf4j
public class Test3 {
    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        Thread t1 = new Thread(() -> {
            log.debug("启动...");
            if (!lock.tryLock()) {
                log.debug("获取失败，返回");
                return;
            }
            try {
                log.debug("获得了锁");
            } finally {
                lock.unlock();
            }
        });
        lock.lock();
        log.debug("获得了锁");
        t1.start();
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {

        } finally {
            lock.unlock();
        }
    }
}
```

输出

```
13:50:02.357 [main] DEBUG com.bytebuf.reentrant.Test3 - 获得了锁
13:50:02.360 [Thread-0] DEBUG com.bytebuf.reentrant.Test3 - 启动...
13:50:02.360 [Thread-0] DEBUG com.bytebuf.reentrant.Test3 - 获取失败，返回
```

使用tryLock解决哲学家就餐问题

```java
@AllArgsConstructor
@ToString
public class Chopstick extends ReentrantLock {
    private String name;
}
@Slf4j
public class Philosopher extends Thread {
    private Chopstick left;
    private Chopstick right;
    public Philosopher(String name, Chopstick left, Chopstick right) {
        super(name);
        this.left = left;
        this.right = right;
    }
    @Override
    public void run() {
        while (true) {
            if (left.tryLock()) {
                try {
                    if (right.tryLock()) {
                        try {
                            eat();
                        } finally {
                            right.unlock();
                        }
                    }
                } finally {
                    left.unlock();
                }
            }
        }
    }
    private void eat() {
        log.debug("eating...");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## 公平锁

ReentrantLock默认是不公平的

`ReentrantLock lock = new ReentrantLock(true); //公平锁`

公平锁一般没有必要，会降低并发度

## 条件变量

synchronized中也有条件变量，就是我们讲原理时那个waitSet休息室，当条件不满足时进入waitSet等待

ReentrantLock的条件变量比synchronized强大之处在于，他是支持多个条件变量的

- synchronized是那些不满足条件的线程都在一间休息室等消息
- 而ReentrantLock支持多间休息室，有专门的等烟的休息室，专门等早餐的休息室，唤醒时也是按休息室来唤醒

使用流程

- await前需要获得锁
- await执行后，会释放锁，进入conditionObject等待
- await的线程被唤醒（或打断，超时）去重新竞争lock锁
- 竞争lock锁成功后，从await后继续执行

```java
```

