## Java-基础篇

### 基本功

#### 面向对象的特征

* 抽象：忽略一个主题中与当前目标无关的东西，专注与当前目标有关的方面
  - 数据抽象：表示世界中一类事物的特征，就是对象的属性。比如鸟有翅膀羽毛(类的属性)
  - 过程抽象：表示世界中一类事物的行为，就是对象的行为。比如鸟会飞，会叫(类的方法)
* 封装：把过程和数据包围起来，对数据的访问只能通过特定的界面。如私有变量，用set,get方法获取
* 继承：一种联结类的层次模型，并且允许和鼓励类的重用，提供一种明确表达共性的方法。对象的一个新类可以从现有的类中派生，这个过程成为继承。
* 多态：允许不同类的对象对同一消息做出响应。



#### final,finally,finalize的区别

* final用于生命属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承
* finally是异常处理语句结构的一部分，表示总是执行
* finalize是Object类中的一个方法，在垃圾收集器执行的时候会调用被回收对象的方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等。JVM不保证此方法总被调用



#### int和Integer的区别

* int是基本类型，直接存储数值。Integer是对象，用一个引用指向这个地址
* int默认初始值为0 Integer默认为null
* Integer是int的封装类，是int的扩展



####重载和重写的区别

* 重载：
  - 必须具有不同的参数列表
  - 可以有不同的返回类型，只要参数列表不同就可以了
  - 可以有不同的访问修饰符
  - 可以抛出不同的异常
* 重写：
  - 参数列表必须完全与被重写的方法相同
  - 返回类型必须与被重写的方法相同
  - 访问修饰符的限制一定要大于被重写方法的访问修饰符(public>protected>default>private)
  - 重写方法抛出的异常不能比被重写方法的异常更加宽泛



#### 抽象类和接口的区别

* 抽象类：
  - abstract修饰
  - 可以有默认的方法实现
  - 子类继承抽象类使用extends关键字
  - 可以有构造器
  - 可以有public,protected和default修饰符
  - 单继承
* 接口：
  - interface修饰
  - 不能有默认的方法实现
  - 子类实现接口使用interface关键字
  - 不可以有构造器
  - 默认只有public关键字修饰符
  - 可以有多个接口实现



#### 反射的用途及实现

* 用途：可以获取在运行期间对象的类型信息。利用这一点可以实现工厂模式和代理模式，同时也解决Java泛型擦除等问题
* 实现：
  - getClass()
  - Class.forName()
  - 类.class
  - 类加载器getClassLoader()



#### 自定义注解的场景及实现

* 场景：跟踪代码的依赖性，实现代替配置文件的功能。比较常见的是Spring框架中的基于注解的配置，还可以生成文档常见的@See@Param
* 实现：使用@interface自定义注解时，自动继承 了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节，在定义注解时，不能继承其他注解或接口



#### session和cookie区别

* session存在服务器端，cookie存在客户端(浏览器)
* session默认被存在在服务器的一个文件里(不是内存)
* session的运行依赖session id, 而session id是存在cookie中的，也就是说，如果浏览器禁用了cookie，同时session也会失效(但是可以通过其他方式实现，比如在url中传递session id)
* session可以存放在文件，数据库，或内存中
* 用户验证一般会用session



#### session分布式处理

* session复制
  - 在支持session复制的服务器上，通过修改web服务器的配置，可以实现将session同步到其他web服务器上，达到每个web服务器上都保存一致的session
  - 优点：代码上不需要做支持和修改
  - 缺点：需要依赖支持的web服务器，在数据量很大的情况下占用大量网络资源，而且会导致延迟
  - 适用场景：web服务器少，session数据量少
  - 可用方案：tomcat-redis-session-manager
* session粘带
  - 将用户的请求通过某种方法强制分发到某一个web服务器上，只要这个web服务器存储了对应的session数据，就可以实现会话跟踪
  - 优点：使用简单，没有额外开销
  - 缺点：一旦某个web服务器重启或宕机，session数据就会丢失，而且需要负载均衡
  - 适用场景：对稳定性要求不是很高的业务场景
* session集中管理
  - 在单独的服务器或服务器集群上使用的缓存技术，如Redis存储Session数据，集中管理所有的Session,所有的Web服务器都会从这个存储介质中获取对应的session,实现session共享
  - 优点：可靠性高，减少Web服务器的资源开销
  - 缺点：实现上有些负责，配置较多
  - 适用场景：Web服务器较多，要求高可用性的情况
  - 可用方案：Spring Session(主要是重写HttpServletRequestWrapper中的getSession方法)
* 基于Cookie管理
  - 每次发起请求的时候都需要将Session数据放到Cookie中传递给服务器
  - 优点：不需要依赖外部存储，不需要额外配置
  - 缺点：不安全，易被盗取或篡改；Cookie数量和长度有限制
  - 适用场景：数据不重要，不敏感且数据量小的情况



#### JDBC流程

* 向DriverManager中注册驱动：Class.forName
* 调用DriverManager.getConnection()方法，通过url,用户名，密码获得数据库链接的Connection对象
* 获取Connection后，通过createStatement创建Statment用以执行sql语句
* 查询返回ResultSet中
* 关闭数据库链接



#### MVC设计思想

* 最上面一层视图层(View),提供用户的操作界面，是程序的外壳
* 最底下的一层数据层(Model),程序需要操作的数据或信息
* 中间的一层控制层(Controller),根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行操作处理，产生结果
* SpringMVC
  - 用户发起请求被DispatcherServlet捕获，对URL解析，得到URI。
  - 根据URI调用HandlerMapping获得该Handler配置的所有相关对象，返回HandlerExecutionChain对象
  - DispatcherServlet根据获得的Handler,选择一个合适的HandlerAdapter
  - 提取Request中的模型数据，填充Handler,执行Handler
  - HttpMessageConverter:将请求消息(json, xml)转换成一个对象，将对象转换为指定的响应消息
  - Handler执行完成后返回ModelAndView对象，选择一个合适的ViewResolver



#### equals和==的区别

* ==常用于原生类型比较，equals用于检查对象的相等性
* 当两个引用地址相同时 == 返回true.equals根据重写来判断