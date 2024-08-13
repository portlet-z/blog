# JDK ThreadLocal

- ThreadLocal可以理解为线程本地变量
- ThreadLocal为变量在每个线程都创建了一个副本，该副本只能被当前线程访问
- ThreadLocal实现原理
  - 在ThreadLocal中维护一个Map,记录线程与实例之间的映射关系，JDK的ThreadLocal是这么实现的吗？
  - 答案是NO, 因为在高并发场景并发修改Map需要加锁，势必会降低性能

[ThreadLocal](../concurrent/22-ThreadLocal.md)

# FastThreadLocal为什么快

```java
public class FastThreadLocalThread extends Thread {
    private InternalThreadLocalMap threadLocalMap;
    //.......
}
public final class InternalThreadLocalMap extends UnpaddedInternalThreadLocalMap {
    private static final int DEFAULT_ARRAY_LIST_INITIAL_CAPACITY = 8;
    private static final int STRING_BUILDER_INITIAL_SIZE;
    private static final int STRING_BUILDER_MAX_SIZE;

    public static final Object UNSET = new Object();

    private BitSet cleanerFlags;
        private InternalThreadLocalMap() {
        super(newIndexedVariableTable());
    }
}
```

