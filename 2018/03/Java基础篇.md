## 基础篇

### 基本功

*   面向对象的特征

    > 面向对象的三个基本特征是：封装、继承、多态。
    > 封装
    > 封装最好理解了。封装是面向对象的特征之一，是对象和类概念的主要特性。
    > 封装，也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。
    > 继承
    > 面向对象编程 (OOP) 语言的一个主要功能就是 “继承”。继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。
    > 多态
    > 多态性（polymorphisn）是允许你将父对象设置成为和一个或更多的他的子对象相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作。简单的说，就是一句话：允许将子类类型的指针赋值给父类类型的指针。
    > 实现多态，有二种方式，覆盖，重载。

*   final, finally, finalize 的区别

    > final 用于声明属性, 方法和类, 分别表示属性不可变, 方法不可覆盖, 类不可继承.
    > finally 是异常处理语句结构的一部分，表示总是执行.
    > finalize 是 Object 类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等. JVM 不保证此方法总被调用.

*   int 和 Integer 有什么区别

    > 链接：[https://www.nowcoder.com/questionTerminal/aad1b52a4d98454da9d1d66d0c243a49](https://www.nowcoder.com/questionTerminal/aad1b52a4d98454da9d1d66d0c243a49)
    > 来源：牛客网
    > int 是 java 提供的 8 种原始数据类型之一。Java 为每个原始类型提供了封装类，Integer 是 java 为 int 提供的封装类。
    > int 的默认值为 0，而 Integer 的默认值为 null，是引用类型，即 Integer 可以区分出未赋值和值为 0 的区别，int 则无法表达出未赋值的情况，
    > Java 中 int 和 Integer 关系是比较微妙的。关系如下：
    > 1、int 是基本的数据类型；
    > 2、Integer 是 int 的封装类；
    > 3、int 和 Integer 都可以表示某一个数值；
    > 4、int 和 Integer 不能够互用，因为他们两种不同的数据类型；

*   重载和重写的区别

    > 重载 Overload 表示同一个类中可以有多个名称相同的方法，但这些方法的参数列表各不相同（即参数个数或类型不同）。
    > 重写 Override 表示子类中的方法可以与父类中的某个方法的名称和参数完全相同，通过子类创建的实例对象调用这个方法时，将调用子类中的定义方法，这相当于把父类中定义的那个完全相同的方法给覆盖了，这也是面向对象编程的多态性的一种表现。子类覆盖父类的方法时，只能比父类抛出更少的异常，或者是抛出父类抛出的异常的子异常，因为子类可以解决父类的一些问题，不能比父类有更多的问题。子类方法的访问权限只能比父类的更大，不能更小。如果父类的方法是 private 类型，那么，子类则不存在覆盖的限制，相当于子类中增加了一个全新的方法。
    > 作者：天天向上
    > 链接：[https://www.zhihu.com/question/35874324/answer/144589616](https://www.zhihu.com/question/35874324/answer/144589616)
    > 来源：知乎
    > 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

*   抽象类和接口有什么区别

    > ### 抽象类和接口的对比

| **参数** | **抽象类** | **接口** |
| --- | --- | --- |
| 默认的方法实现 | 它可以有默认的方法实现 | 接口完全是抽象的。它根本不存在方法的实现 |
| 实现 | 子类使用 **extends** 关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 | 子类使用关键字 **implements** 来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器 | 抽象类可以有构造器 | 接口不能有构造器 |
| 与正常 Java 类的区别 | 除了你不能实例化抽象类之外，它和普通 Java 类没有任何区别 | 接口是完全不同的类型 |
| 访问修饰符 | 抽象方法可以有 **public**、**protected** 和 **default** 这些修饰符 | 接口方法默认修饰符是 **public**。你不可以使用其它修饰符。 |
| main 方法 | 抽象方法可以有 main 方法并且我们可以运行它 | 接口没有 main 方法，因此我们不能运行它。 |
| 多继承 | 抽象方法可以继承一个类和实现多个接口 | 接口只可以继承一个或多个其它接口 |
| 速度 | 它比接口速度要快 | 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。 |
| 添加新方法 | 如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。 | 如果你往接口中添加方法，那么你必须改变实现该接口的类。 |

*   说说反射的用途及实现

    > Java 反射机制是一个非常强大的功能，在很多的项目比如 Spring，Mybatis 都都可以看到反射的身影。通过反射机制，我们可以在运行期间获取对象的类型信息。利用这一点我们可以实现工厂模式和代理模式等设计模式，同时也可以解决 java 泛型擦除等令人苦恼的问题。
    > 获取一个对象对应的反射类，在 Java 中有三种方法可以获取一个对象的反射类，
    > 通过 getClass() 方法
    > 通过 Class.forName() 方法；
    > 使用类. class
    > 通过类加载器实现，getClassLoader()

*   说说自定义注解的场景及实现

    > 登陆、权限拦截、日志处理，以及各种 Java 框架，如 Spring，Hibernate，JUnit 提到注解就不能不说反射，Java 自定义注解是通过运行时靠反射获取注解。实际开发中，例如我们要获取某个方法的调用日志，可以通过 AOP（动态代理机制）给方法添加切面，通过反射来获取方法包含的注解，如果包含日志注解，就进行日志记录。反射的实现在 Java 应用层面上讲，是通过对 Class 对象的操作实现的，Class 对象为我们提供了一系列方法对类进行操作。在 Jvm 这个角度来说，Class 文件是一组以 8 位字节为基础单位的二进制流，各个数据项目按严格的顺序紧凑的排列在 Class 文件中，里面包含了类、方法、字段等等相关数据。通过对 Claas 数据流的处理我们即可得到字段、方法等数据。
    > 作者：LeopPro
    > 链接：[https://juejin.im/post/5a9fad016fb9a028b77a5ce9](https://juejin.im/post/5a9fad016fb9a028b77a5ce9)
    > 来源：掘金
    > 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

*   HTTP 请求的 GET 与 POST 方式的区别

    > 1\. 根据 HTTP 规范，GET 用于信息获取，而且应该是安全的和幂等的。
    > 2\. 根据 HTTP 规范，POST 表示可能修改变服务器上的资源的请求。
    > 3\. 首先是 "GET 方式提交的数据最多只能是 1024 字节"，因为 GET 是通过 URL 提交数据，那么 GET 可提交的数据量就跟 URL 的长度有直接关系了。而实际上，URL 不存在参数上限的问题，HTTP 协议规范没有对 URL 长度进行限制。这个限制是特定的浏览器及服务器对它的限制。IE 对 URL 长度的限制是 2083 字节 (2K+35)。对于其他浏览器，如 Netscape、FireFox 等，理论上没有长度限制，其限制取决于操作系统的支持。注意这是限制是整个 URL 长度，而不仅仅是你的参数值数据长度。

*   POST 是没有大小限制的，HTTP 协议规范也没有进行大小限制

*   session 与 cookie 区别

    > 1、cookie 数据存放在客户的浏览器上，session 数据放在服务器上。
    > 2、cookie 不是很安全，别人可以分析存放在本地的 COOKIE 并进行 COOKIE 欺骗
    > 考虑到安全应当使用 session。
    > 3、session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
    > 考虑到减轻服务器性能方面，应当使用 COOKIE。
    > 4、单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个 cookie。
    > 5、所以个人建议：
    > 将登陆信息等重要信息存放为 SESSION
    > 其他信息如果需要保留，可以放在 COOKIE 中

*   session 分布式处理

    > 1.Session 复制
    > 在支持 Session 复制的 Web 服务器上，通过修改 Web 服务器的配置，可以实现将 Session 同步到其它 Web 服务器上，达到每个 Web 服务器上都保存一致的 Session。
    > 优点：代码上不需要做支持和修改。
    > 缺点：需要依赖支持的 Web 服务器，一旦更换成不支持的 Web 服务器就不能使用了，在数据量很大的情况下不仅占用网络资源，而且会导致延迟。
    > 适用场景：只适用于 Web 服务器比较少且 Session 数据量少的情况。
    > 可用方案：开源方案 tomcat-redis-session-manager，暂不支持 Tomcat8。
    > 2.Session 粘滞
    > 将用户的每次请求都通过某种方法强制分发到某一个 Web 服务器上，只要这个 Web 服务器上存储了对应 Session 数据，就可以实现会话跟踪。
    > 优点：使用简单，没有额外开销。
    > 缺点：一旦某个 Web 服务器重启或宕机，相对应的 Session 数据将会丢失，而且需要依赖负载均衡机制。
    > 适用场景：对稳定性要求不是很高的业务情景。
    > 3.Session 集中管理
    > 在单独的服务器或服务器集群上使用缓存技术，如 Redis 存储 Session 数据，集中管理所有的 Session，所有的 Web 服务器都从这个存储介质中存取对应的 Session，实现 Session 共享。
    > 优点：可靠性高，减少 Web 服务器的资源开销。
    > 缺点：实现上有些复杂，配置较多。
    > 适用场景：Web 服务器较多、要求高可用性的情况。
    > 可用方案：开源方案 Spring Session，也可以自己实现，主要是重写 HttpServletRequestWrapper 中的 getSession 方法，博主也动手写了一个，github 搜索 joincat 用户，然后自取。
    > 4\. 基于 Cookie 管理
    > 这种方式每次发起请求的时候都需要将 Session 数据放到 Cookie 中传递给服务端。
    > 优点：不需要依赖额外外部存储，不需要额外配置。
    > 缺点：不安全，易被盗取或篡改；Cookie 数量和长度有限制，需要消耗更多网络带宽。
    > 适用场景：数据不重要、不敏感且数据量小的情况。
    > 总结
    > 这四种方式，相对来说，Session 集中管理更加可靠，使用也是最多的。
    > 作者：JavaQ
    > 链接：[https://www.jianshu.com/p/3dd4e06bdfa4](https://www.jianshu.com/p/3dd4e06bdfa4)
    > 來源：简书
    > 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

*   JDBC 流程

    > （1）向 DriverManager 类注册驱动数据库驱动程序
    > （2）调用 DriverManager.getConnection 方法， 通过 JDBC URL，用户名，密码取得数据库连接的 Connection 对象。
    > （3）获取 Connection 后， 便可以通过 createStatement 创建 Statement 用以执行 SQL 语句。
    > （4） 有时候会得到查询结果，比如 select，得到查询结果，查询（SELECT）的结果存放于结果集（ResultSet）中。
    > （5）关闭数据库语句，关闭数据库连接。

*   MVC 设计思想

    > MVC 是三个单词的首字母缩写，它们是 Model（模型）、View（视图）和 Controller（控制）。
    > 这个模式认为，程序不论简单或复杂，从结构上看，都可以分成三层。
    > 1）最上面的一层，是直接面向最终用户的 "视图层"（View）。它是提供给用户的操作界面，是程序的外壳。
    > 2）最底下的一层，是核心的 "数据层"（Model），也就是程序需要操作的数据或信息。
    > 3）中间的一层，就是 "控制层"（Controller），它负责根据用户从 "视图层" 输入的指令，选取 "数据层" 中的数据，然后对其进行相应的操作，产生最终结果。

*   equals 与 == 的区别

    > == 与 equals 的主要区别是：== 常用于比较原生类型，而 equals() 方法用于检查对象的相等性。另一个不同的点是：如果 == 和 equals() 用于比较对象，当两个引用地址相同，== 返回 true。而 equals() 可以返回 true 或者 false 主要取决于重写实现。最常见的一个例子，字符串的比较，不同情况 == 和 equals() 返回不同的结果。
    > 使用 == 比较原生类型如：boolean、int、char 等等，使用 equals() 比较对象。
    > == 返回 true 如果两个引用指向相同的对象，equals() 的返回结果依赖于具体业务实现
    > 字符串的对比使用 equals() 代替 == 操作符
    > 使用 == 比较原生类型如：boolean、int、char 等等，使用 equals() 比较对象。
    > == 返回 true 如果两个引用指向相同的对象，equals() 的返回结果依赖于具体业务实现
    > 字符串的对比使用 equals() 代替 == 操作符

### 集合

*   List 和 Set 区别

    > 1、List,Set 都是继承自 Collection 接口
    > 2、List 特点：元素有放入顺序，元素可重复 ，Set 特点：元素无放入顺序，元素不可重复（注意：元素虽然无放入顺序，但是元素在 set 中的位置是有该元素的 HashCode 决定的，其位置其实是固定的）
    > 3、List 接口有三个实现类：LinkedList，ArrayList，Vector ，Set 接口有两个实现类：HashSet(底层由 HashMap 实现)，LinkedHashSet

*   List 和 Map 区别

    > List 特点：元素有放入顺序，元素可重复;
    > Map 特点：元素按键值对存储，无放入顺序 ;
    > List 接口有三个实现类：LinkedList，ArrayList，Vector;
    > LinkedList：底层基于链表实现，链表内存是散乱的，每一个元素存储本身内存地址的同时还存储下一个元素的地址。链表增删快，查找慢;
    > Map 接口有三个实现类：HashMap，HashTable，LinkeHashMap
    > Map 相当于和 Collection 一个级别的；Map 该集合存储键值对，且要求保持键的唯一性；

*   Arraylist 与 LinkedList 区别

    > 1) 因为 Array 是基于索引 (index) 的数据结构，它使用索引在数组中搜索和读取数据是很快的。Array 获取数据的时间复杂度是 O(1), 但是要删除数据却是开销很大的，因为这需要重排数组中的所有数据。
    > 2) 相对于 ArrayList，LinkedList 插入是更快的。因为 LinkedList 不像 ArrayList 一样，不需要改变数组的大小，也不需要在数组装满的时候要将所有的数据重新装入一个新的数组，这是 ArrayList 最坏的一种情况，时间复杂度是 O(n)，而 LinkedList 中插入或删除的时间复杂度仅为 O(1)。ArrayList 在插入数据时还需要更新索引（除了插入数组的尾部）。
    > 3) 类似于插入数据，删除数据时，LinkedList 也优于 ArrayList。
    > 4) LinkedList 需要更多的内存，因为 ArrayList 的每个索引的位置是实际的数据，而 LinkedList 中的每个节点中存储的是实际的数据和前后节点的位置。
    > 5) 你的应用不会随机访问数据。因为如果你需要 LinkedList 中的第 n 个元素的时候，你需要从第一个元素顺序数到第 n 个数据，然后读取数据。
    > 6) 你的应用更多的插入和删除元素，更少的读取数据。因为插入和删除元素不涉及重排数据，所以它要比 ArrayList 要快。

*   ArrayList 与 Vector 区别

    > 链接：[https://www.nowcoder.com/questionTerminal/0953369f92054cbfbf1024a1e723e04f](https://www.nowcoder.com/questionTerminal/0953369f92054cbfbf1024a1e723e04f)
    > 来源：牛客网
    > 1） 同步性: Vector 是线程安全的，也就是说是同步的 ，而 ArrayList 是线程序不安全的，不是同步的 数 2。
    > 2）数据增长: 当需要增长时, Vector 默认增长为原来一倍 ，而 ArrayList 却是原来的 50% ，这样, ArrayList 就有利于节约内存空间。
    > 
    > ```
    > 如果涉及到堆栈，队列等操作，应该考虑用Vector，如果需要快速随机访问元素，应该使用ArrayList 。
    > 
    > ```

*   HashMap 和 Hashtable 的区别

    > 1)HashMap 几乎可以等价于 Hashtable，除了 HashMap 是非 synchronized 的，并可以接受 null(HashMap 可以接受为 null 的键值 (key) 和值(value)，而 Hashtable 则不行)。
    > 2) HashMap 是非 synchronized，而 Hashtable 是 synchronized，这意味着 Hashtable 是线程安全的，多个线程可以共享一个 Hashtable；而如果没有正确的同步的话，多个线程是不能共享 HashMap 的。Java 5 提供了 ConcurrentHashMap，它是 HashTable 的替代，比 HashTable 的扩展性更好。
    > 3) 另一个区别是 HashMap 的迭代器 (Iterator) 是 fail-fast 迭代器，而 Hashtable 的 enumerator 迭代器不是 fail-fast 的。所以当有其它线程改变了 HashMap 的结构（增加或者移除元素），将会抛出 ConcurrentModificationException，但迭代器本身的 remove()方法移除元素则不会抛出 ConcurrentModificationException 异常。但这并不是一个一定发生的行为，要看 JVM。这条同样也是 Enumeration 和 Iterator 的区别。
    > 4) 由于 Hashtable 是线程安全的也是 synchronized，所以在单线程环境下它比 HashMap 要慢。如果你不需要同步，只需要单一线程，那么使用 HashMap 性能要好过 Hashtable。
    > 5) HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

*   HashSet 和 HashMap 区别

    > | _HashMap_ | _HashSet_ |
    > | ------------------------------------------- | ------------------------------------------------------------ |
    > | HashMap 实现了 Map 接口 | HashSet 实现了 Set 接口 |
    > | HashMap 储存键值对 | HashSet 仅仅存储对象 |
    > | 使用 put() 方法将元素放入 map 中 | 使用 add() 方法将元素放入 set 中 |
    > | HashMap 中使用键对象来计算 hashcode 值 | HashSet 使用成员对象来计算 hashcode 值，对于两个对象来说 hashcode 可能相同，所以 equals() 方法用来判断对象的相等性，如果两个对象不同的话，那么返回 false |
    > | HashMap 比较快，因为是使用唯一的键来获取对象 | HashSet 较 HashMap 来说比较慢 |

*   HashMap 和 ConcurrentHashMap 的区别

    > 1）放入 HashMap 的元素是 key-value 对。
    > （2）底层说白了就是以前数据结构课程讲过的散列结构。
    > （3）要将元素放入到 hashmap 中，那么 key 的类型必须要实现实现 hashcode 方法，默认这个方法是根据对象的地址来计算的，具体我也记不太清楚了，接着还必须覆盖对象的 equal 方法。
    > （4）ConcurrentHashMap 对整个桶数组进行了分段，而 HashMap 则没有
    > （5）ConcurrentHashMap 在每一个分段上都用锁进行保护，从而让锁的粒度更精细一些，并发性能更好，而 HashMap 没有锁机制，不是线程安全的。。。

*   HashMap 的工作原理及代码实现

    > HashMap 基于 hashing 原理，我们通过 put() 和 get() 方法储存和获取对象。当我们将键值对传递给 put() 方法时，它调用键对象的 hashCode() 方法来计算 hashcode，让后找到 bucket 位置来储存值对象。当获取对象时，通过键对象的 equals() 方法找到正确的键值对，然后返回值对象。HashMap 使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap 在每个链表节点中储存键值对对象。

*   ConcurrentHashMap 的工作原理及代码实现

    > ConcurrentHashMap 采用了非常精妙的 "分段锁" 策略，ConcurrentHashMap 的主干是个 Segment 数组。Segment 继承了 ReentrantLock，所以它就是一种可重入锁（ReentrantLock)。在 ConcurrentHashMap，一个 Segment 就是一个子哈希表，Segment 里维护了一个 HashEntry 数组，并发环境下，对于不同 Segment 的数据进行操作是不用考虑锁竞争的。

### 线程

*   创建线程的方式及实现

    > 1.  继承 Thread 类创建线程类
    >     （1）定义 Thread 类的子类，并重写该类的 run 方法，该 run 方法的方法体就代表了线程要完成的任务。因此把 run() 方法称为执行体。
    >     （2）创建 Thread 子类的实例，即创建了线程对象。
    >     （3）调用线程对象的 start() 方法来启动该线程。
    > 2.  通过 Runnable 接口创建线程类
    >     （1）定义 runnable 接口的实现类，并重写该接口的 run() 方法，该 run() 方法的方法体同样是该线程的线程执行体。
    >     （2）创建 Runnable 实现类的实例，并依此实例作为 Thread 的 target 来创建 Thread 对象，该 Thread 对象才是真正的线程对象。
    >     （3）调用线程对象的 start() 方法来启动该线程。
    > 3.  通过 Callable 和 Future 创建线程
    >     （1）创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。
    >     （2）创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对象封装了该 Callable 对象的 call() 方法的返回值。
    >     （3）使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。
    >     （4）调用 FutureTask 对象的 get() 方法来获得子线程执行结束后的返回值
    >     采用实现 Runnable、Callable 接口的方式创见多线程时，优势是：
    >     线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。
    >     在这种方式下，多个线程可以共享同一个 target 对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将 CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。
    >     劣势是：
    >     编程稍微复杂，如果要访问当前线程，则必须使用 Thread.currentThread() 方法。
    >     使用继承 Thread 类的方式创建多线程时优势是：
    >     编写简单，如果需要访问当前线程，则无需使用 Thread.currentThread() 方法，直接使用 this 即可获得当前线程。
    >     劣势是：
    >     线程类已经继承了 Thread 类，所以不能再继承其他父类。

*   sleep() 、join（）、yield（）有什么区别

    > sleep()
    > 　　sleep() 方法需要指定等待的时间，它可以让当前正在执行的线程在指定的时间内暂停执行，进入阻塞状态，该方法既可以让其他同优先级或者高优先级的线程得到执行的机会，也可以让低优先级的线程得到执行机会。但是 sleep() 方法不会释放 “锁标志”，也就是说如果有 synchronized 同步块，其他线程仍然不能访问共享数据。

> wait()
> 　　wait() 方法需要和 notify() 及 notifyAll() 两个方法一起介绍，这三个方法用于协调多个线程对共享数据的存取，所以必须在 synchronized 语句块内使用，也就是说，调用 wait()，notify() 和 notifyAll() 的任务在调用这些方法前必须拥有对象的锁。注意，它们都是 Object 类的方法，而不是 Thread 类的方法。
> 　　wait()方法与 sleep()方法的不同之处在于，wait()方法会释放对象的 “锁标志”。当调用某一对象的 wait() 方法后，会使当前线程暂停执行，并将当前线程放入对象等待池中，直到调用了 notify()方法后，将从对象等待池中移出任意一个线程并放入锁标志等待池中，只有锁标志等待池中的线程可以获取锁标志，它们随时准备争夺锁的拥有权。当调用了某个对象的 notifyAll()方法，会将对象等待池中的所有线程都移动到该对象的锁标志等待池。
> 　　除了使用 notify() 和 notifyAll() 方法，还可以使用带毫秒参数的 wait(long timeout) 方法，效果是在延迟 timeout 毫秒后，被暂停的线程将被恢复到锁标志等待池。
> 　　此外，wait()，notify() 及 notifyAll() 只能在 synchronized 语句中使用，但是如果使用的是 ReenTrantLock 实现同步，该如何达到这三个方法的效果呢？解决方法是使用 ReenTrantLock.newCondition() 获取一个 Condition 类对象，然后 Condition 的 await()，signal() 以及 signalAll() 分别对应上面的三个方法。
> yield()
> 　　yield()方法和 sleep()方法类似，也不会释放 “锁标志”，区别在于，它没有参数，即 yield() 方法只是使当前线程重新回到可执行状态，所以执行 yield()的线程有可能在进入到可执行状态后马上又被执行，另外 yield()方法只能使同优先级或者高优先级的线程得到执行机会，这也和 sleep()方法不同。
> join()
> 　　join() 方法会使当前线程等待调用 join() 方法的线程结束后才能继续执行

*   说说 CountDownLatch 原理

    > CountDownLatch 内部维护了一个整数 n，n（要大于等于 0）在 == 当前线程 == 初始化 CountDownLatch 方法指定。当前线程调用 CountDownLatch 的 await() 方法阻塞当前线程，等待其他调用 CountDownLatch 对象的 CountDown() 方法的线程执行完毕。 其他线程调用该 CountDownLatch 的 CountDown() 方法，该方法会把 n-1，直到所有线程执行完成，n 等于 0，== 当前线程 == 就恢复执行。

*   说说 CyclicBarrier 原理

    > CyclicBarrier 简介 CyclicBarrier 是一个同步辅助类, 允许一组线程互相等待, 直到到达某个公共屏障点 (commonbarrierpoint)。因为该 barrier 在释放等待线程后可以重用, 所以称它为循环的 barrier。

*   说说 Semaphore 原理

    > Semaphore 直译为信号。实际上 Semaphore 可以看做是一个信号的集合。不同的线程能够从 Semaphore 中获取若干个信号量。当 Semaphore 对象持有的信号量不足时，尝试从 Semaphore 中获取信号的线程将会阻塞。直到其他线程将信号量释放以后，阻塞的线程会被唤醒，重新尝试获取信号量。

*   说说 Exchanger 原理

    > 当一个线程到达 exchange 调用点时，如果它的伙伴线程此前已经调用了此方法，那么它的伙伴会被调度唤醒并与之进行对象交换，然后各自返回。如果它的伙伴还没到达交换点，那么当前线程将会被挂起，直至伙伴线程到达——完成交换正常返回；或者当前线程被中断——抛出中断异常；又或者是等候超时——抛出超时异常。

*   说说 CountDownLatch 与 CyclicBarrier 区别

    > (01) CountDownLatch 的作用是允许 1 或 N 个线程等待其他线程完成执行; 而 CyclicBarrier 则是允许 N 个线程相互等待。
    > (02) CountDownLatch 的计数器无法被重置; CyclicBarrier 的计数器可以被重置后使用, 因此它被称为是循环的 barrier。

*   ThreadLocal 原理分析

    > ThreadLocal 提供了线程本地变量，它可以保证访问到的变量属于当前线程，每个线程都保存有一个变量副本，每个线程的变量都不同。ThreadLocal 相当于提供了一种线程隔离，将变量与线程相绑定。

*   讲讲线程池的实现原理

    > 当提交一个新任务到线程池时，线程池的处理流程如下。
    > 1）线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作
    > 线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
    > 2）线程池判断工作队列是否已经满。如果工作队列没有满，则将新提交的任务存储在这
    > 个工作队列里。如果工作队列满了，则进入下个流程。
    > 3）线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程
    > 来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

*   线程池的几种方式

    > 在 Executors 类里面提供了一些静态工厂，生成一些常用的线程池。
    > 1、newFixedThreadPool：创建固定大小的线程池。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
    > 2、newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60 秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说 JVM）能够创建的最大线程大小。
    > 3、newSingleThreadExecutor：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
    > 4、newScheduledThreadPool：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
    > 5、newSingleThreadScheduledExecutor：创建一个单线程的线程池。此线程池支持定时以及周期性执行任务的需求。

*   线程的生命周期

    > 新建 (New)、就绪（Runnable）、运行（Running）、阻塞(Blocked) 和死亡(Dead)5 种状态

### 锁机制

*   说说线程安全问题

    > 线程安全是多线程领域的问题，线程安全可以简单理解为一个方法或者一个实例可以在多线程环境中使用而不会出现问题。
    > 在 Java 多线程编程当中，提供了多种实现 Java 线程安全的方式：
    > 
    > *   最简单的方式，使用 Synchronization 关键字: Java Synchronization 介绍
    > *   使用 java.util.concurrent.atomic 包中的原子类，例如 AtomicInteger
    > *   使用 java.util.concurrent.locks 包中的锁
    > *   使用线程安全的集合 ConcurrentHashMap
    > *   使用 volatile 关键字，保证变量可见性（直接从内存读，而不是从线程 cache 读）

*   volatile 实现原理

    > *   在 JVM 底层 volatile 是采用 “内存屏障” 来实现的。
    > *   缓存一致性协议（MESI 协议）它确保每个缓存中使用的共享变量的副本是一致的。其核心思想如下：当某个 CPU 在写数据时，如果发现操作的变量是共享变量，则会通知其他 CPU 告知该变量的缓存行是无效的，因此其他 CPU 在读取该变量时，发现其无效会重新从主存中加载数据。

*   synchronize 实现原理

    > 同步代码块是使用 monitorenter 和 monitorexit 指令实现的，同步方法（在这看不出来需要看 JVM 底层实现）依靠的是方法修饰符上的 ACC_SYNCHRONIZED 实现。

*   synchronized 与 lock 的区别

    > 一、synchronized 和 lock 的用法区别
    > （1）synchronized(隐式锁)：在需要同步的对象中加入此控制，synchronized 可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。
    > （2）lock（显示锁）：需要显示指定起始位置和终止位置。一般使用 ReentrantLock 类做为锁，多个线程中必须要使用一个 ReentrantLock 类做为对 象才能保证锁的生效。且在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以一般会在 finally 块中写 unlock() 以防死锁。
    > 二、synchronized 和 lock 性能区别
    > synchronized 是托管给 JVM 执行的，而 lock 是 java 写的控制锁的代码。在 Java1.5 中，synchronize 是性能低效的。因为 这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用 Java 提供的 Lock 对象，性能更高一些。但 是到了 Java1.6，发生了变化。synchronize 在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致 在 Java1.6 上 synchronize 的性能并不比 Lock 差。
    > 三、synchronized 和 lock 机制区别
    > （1）synchronized 原始采用的是 CPU 悲观锁机制，即线程获得的是独占锁。独占锁意味着其 他线程只能依靠阻塞来等待线程释放锁。
    > （2）Lock 用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就 是 CAS 操作（Compare and Swap）。

*   CAS 乐观锁

    > CAS 是项乐观锁技术，当多个线程尝试使用 CAS 同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。
    > CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值 (B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。” 这其实和乐观锁的冲突检查 + 数据更新的原理是一样的。

*   ABA 问题

    > CAS 会导致 “ABA 问题”。
    > CAS 算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。
    > 比如说一个线程 one 从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且 two 进行了一些操作变成了 B，然后 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操作发现内存中仍然是 A，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但是不代表这个过程就是没有问题的。
    > 部分乐观锁的实现是通过版本号（version）的方式来解决 ABA 问题，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行 + 1 操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现 ABA 问题，因为版本号只会增加不会减少。

*   乐观锁的业务场景及实现方式

    > 乐观锁（Optimistic Lock）：
    > 每次获取数据的时候，都不会担心数据被修改，所以每次获取数据的时候都不会进行加锁，但是在更新数据的时候需要判断该数据是否被别人修改过。如果数据被其他线程修改，则不进行数据更新，如果数据没有被其他线程修改，则进行数据更新。由于数据没有进行加锁，期间该数据可以被其他线程进行读写操作。
    > 比较适合读取操作比较频繁的场景，如果出现大量的写入操作，数据发生冲突的可能性就会增大，为了保证数据的一致性，应用层需要不断的重新获取数据，这样会增加大量的查询操作，降低了系统的吞吐量。