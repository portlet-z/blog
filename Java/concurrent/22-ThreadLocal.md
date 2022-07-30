- ThreadLocal可以实现【资源对象】的线程隔离，让每个线程各用各的【资源对象】，避免争用引发的线程安全问题
- ThreadLocal同时实现了线程内的资源共享
- 其原理是，每个线程内有一个ThreadLocalMap类型的成员遍历，用来存储资源对象
  1. 调用set方法，就是以ThreadLocal自己作为key,资源对象作为value,放入当前线程的ThreadLocalMap集合中
  2. 调用get方法，就是以ThreadLocal自己作为key,当当前线程中查找关联的资源值
  3. 调用remove方法，就是以ThreadLocal自己作为key,移除当前线程关联的资源值
- 为什么ThreadLocalMap中的key(即ThreadLocal)要设计为弱引用
  1. Thread可能需要长时间运行（如线程池中的线程），如果key不再使用，需要在内存不足(GC)时释放其占有的内存
  2. 但GC仅是让key的内存释放，后续还要根据key是否为null来进一步释放值的内存，释放时机有
     1. 获取key发现null key
     2. set key时，会使用启发式扫描，清除临近的null key, 启发次数与元素个数，是否发现null key有关
     3. remove时（推荐），因为一般使用ThreadLocal时都把它作为静态变量，因此GC无法回收