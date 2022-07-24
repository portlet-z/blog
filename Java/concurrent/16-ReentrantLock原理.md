## 非公平锁实现原理

### 加锁解锁流程

先从构造器开始看，默认为非公平锁

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
```

NonfairSync继承自AQS, 没有竞争时

![](./images/NonfairSync-1.png)

第一个竞争出现时

![](./images/NonfairSync-2.jpg)

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

Thread-1执行了

1. CAS尝试将state由0改为1，结果失败
2. 进入tryAcquire逻辑，这时state已经时1，结果仍然失败
3. 接下来进入addWaiter逻辑，构造Node队列
   1. 图中黄色三角表示该Node的waitStatus状态，其中0为默认正常状态
   2. Node的创建时懒惰的
   3. 其中第一个Node称为Dummy(哑元)或哨兵，用来占位，并不关联线程

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

![](./images/NonfairSync-3.jpg)

当线程进入acquireQueued逻辑

1. acquireQueued会在一个死循环中不断尝试获得锁，失败后进入park阻塞
2. 如果自己是紧接着head(排第二位)，那么再次tryAcquire尝试获取锁，当然这时state仍为1，失败
3. 进入shouldParkAfterFailedAcquire逻辑，将前驱node,即head的waitStatus改为-1，这次返回false

```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

![](./images/NonfairSync-4.jpg)

4. shouldParkAfterFailedAcquire执行完毕回到acquireQueued,再次tryAcquire尝试获取锁，当然这时state仍为1，失败
5. 当再次进入shouldParkAfterFailedAcquire时，这时因为其前驱node的waitStatus已经是-1，这次返回true
6. 进入parkAndCheckInterrupt,Thread-1 park(灰色表示)

![](./images/NonfairSync-5.jpg)

再次有多个线程经历上述过程竞争失败，变成这个样子

![](./images/NonfairSync-6.jpg)

Thread-0释放锁，进入tryRelease流程，如果成功

- 设置exclusiveOwnerThread为null
- state = 0

![](./images/NonfairSync-7.jpg)

当前队列不为null,并且head的waitStatus = -1,进入unparkSuccessor流程

找到队列中离head最近的一个Node(没有取消的)，unpark恢复其运行，本例中即为Thread-1

回到Thread-1的acquireQueued流程

![](./images/NonfairSync-8.jpg)

如果加锁成功（没有竞争），会设置

- exclusiveOwnerThread为Thread-1, state=1
- head指向刚刚Thread-1所在的Node,该Node清空Thread
- 原本的head因为从链表断开，而可被垃圾回收

如果这时候有其它线程来竞争（非公平的体现），例如这时候有Thread-4来了

![](./images/NonfairSync-9.jpg)

如果不巧被Thread-4占了先

- Thread-4被设置为exclusiveOwnerThread, state = 1
- Thread-1再次进入acquireQueued流程，获取锁失败，重新进入park阻塞