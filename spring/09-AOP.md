## 使用aspectjrt编译器实现AOP功能

- pom引入依赖

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
</dependency>
```

- 添加aspectj-maven-plugin插件

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.14.0</version>
    <configuration>
        <complianceLevel>1.8</complianceLevel>
        <source>8</source>
        <target>8</target>
        <showWeaveInfo>true</showWeaveInfo>
        <verbose>true</verbose>
        <Xlint>ignore</Xlint>
        <encoding>UTF-8</encoding>
    </configuration>
    <executions>
        <execution>
            <goals>
                <!-- use this goal to weave all your main classes -->
                <goal>compile</goal>
                <!-- use this goal to weave all your test classes -->
                <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

```java
@Aspect //切面并未被Spring管理
public class MyAspect {
    @Before("execution(* com.bytebuf.a10.MyService.foo())")
    public void before() {
        System.out.println("before()");
    }
}
@Service
public class MyService {
    public void foo() {
        System.out.println("foo()");
    }
}
@SpringBootApplication
public class A10Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(A10Application.class, args);
        MyService service = context.getBean(MyService.class);
        System.out.println(service.getClass());
        service.foo();
        context.close();
    }
}
```

- 编译后MyService生成的代码,多了一行MyAspect.aspectOf().before();

```java
@Service
public class MyService {
    public MyService() {
    }

    public void foo() {
        MyAspect.aspectOf().before();
        System.out.println("foo()");
    }
}
```

## 使用agent类加载来实现AOP

- 去掉上面的aspectj-maven-plugin插件

- 新增resources/META-INF/aop.xml文件

```xml
<aspectj>
    <aspects>
        <aspect name="com.bytebuf.a10.MyAspect"/>
        <weaver options="-verbose -showWeaveInfo">
            <include within="com.bytebuf.a10.MyService"/>
            <include within="com.bytebuf.a10.MyAspect"/>
        </weaver>
    </aspects>
</aspectj>
```

- 启动时VM:options设置为 -javaagent:/Users/portlet/.m2/repository/org/aspectj/aspectjweaver/1.9.7/aspectjweaver-1.9.7.jar

- 查看运行时的class文件
  
  - 启动arthas : java -jar arthas-boot.jar
  
  - 选择要监听的process
  
  - 执行jad com.bytebuf.a10.MyService即可看到反编译后的字节码文件
