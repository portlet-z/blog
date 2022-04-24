## Scope类型

- singleton
  
  - Spring容器存在单例实例

- prototype
  
  - Spring容器会创建多个实例

- request
  
  - 每次请求Spring容器会创建一个实例

- session
  
  - 每次会话Spring容器会创建一个实例
  
  - spring设置session失效时间server.servlet.session.timeout=10s

- application

## 在singletion中使用其他几种scope

```java
@Scope("prototype")
@Component
public class F1 {
}
@Component
public class E {
    @Autowired
    private F1 f1;
    public F1 getF1() {
        return f1;
    }
}
public static void main(String[] args) {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A09Application.class);
    E e = context.getBean(E.class);
    System.out.println(e.getF1());//com.bytebuf.a09.F1@59fd97a8
    System.out.println(e.getF1());//com.bytebuf.a09.F1@59fd97a8
    context.close();
}
```

- 对于单例对象来讲，依赖注入仅发生了一次，后续再没有用到多例的F，因此E用的始终是第一次依赖注入的F

- 解决方案
  
  - ①使用@Lazy生成代理，代理对象虽然是同一个，但每次使用代理对象的任意方法时，由代理对象创建新的f对象
  
  - ②@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
  
  - ③注入ObjectFactory<F3> f3;
  
  ```java
  @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
  @Component
  public class F2 {
  }
  @Scope(value = "prototype")
  @Component
  public class F3 {
  }
  @Autowired
  private ObjectFactory<F3> f3;
  ```
