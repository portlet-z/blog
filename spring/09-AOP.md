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

## JDK动态代理Demo

- jdk只能针对接口代理。代理类和被代理类之间是兄弟关系

```java
public class JdkProxyDemo {
    interface Foo {
        void foo();
    }
    static class Target implements Foo {
        @Override
        public void foo() {
            System.out.println("foo");
        }
    }
    public static void main(String[] params) {
        Target target = new Target();
        //用来加载运行期间动态生成的字节码
        ClassLoader loader = JdkProxyDemo.class.getClassLoader();
        Foo foo = (Foo) Proxy.newProxyInstance(loader, new Class[]{Foo.class}, (proxy, method, args) -> {
            System.out.println("before");
            Object result = method.invoke(target, args);
            System.out.println("after");
            return result;
        });
        foo.foo();
    }
}
```

## CGLIB动态代理Demo

- cglib生成的代理类继承了被代理类，和被代理类是父子关系

- cglib的被代理对象不能用final修饰，方法亦不能用final修饰

```java
public class CglibProxyDemo {
    static class Target {
        public void sayHello() {
            System.out.println("hello");
        }
    }
    public static void main(String[] params) {
        Target target = new Target();
        // 创建代理对象
        Target proxy = (Target) Enhancer.create(Target.class, (MethodInterceptor) (p, method, args, methodProxy) -> {
            System.out.println("before");
            //用方法反射调用目标
            //Object result = method.invoke(target, args);
            //methodProxy内部没有用反射，避免了反射调用
            //Object result = methodProxy.invoke(target, args);
            //methodProxy.invokeSuper需要用到代理对象，而methodProxy.invoke需要用到目标对象
            Object result = methodProxy.invokeSuper(p, args);
            System.out.println("after");
            return result;
        });
        proxy.sayHello();
    }
}
```

## JDK动态代理的原理

### 代理demo1

```java
public class A12 {
    interface Foo {
        void foo();
    }
    static class Target implements Foo {
        @Override
        public void foo() {
            System.out.println("foo");
        }
    }
    interface InvocationHandler {
        void invoke();
    }
    static class JdkProxy implements Foo {
        private InvocationHandler h;
        public JdkProxy(A12.InvocationHandler h) {
            this.h = h;
        }
        @Override
        public void foo() {
            h.invoke();
        }
    }
    public static void main(String[] args) {
        Foo foo = new JdkProxy(() -> {
            System.out.println("before");
            new Target().foo();
        });
        foo.foo();
    }
}
```

### 代理demo2

```java
public class A12 {
    interface Foo {
        void foo();
    }
    static class Target implements Foo {
        @Override
        public void foo() {
            System.out.println("foo");
        }
    }
    interface InvocationHandler {
        void invoke(Method method, Object[] args) throws Throwable;
    }
    static class JdkProxy implements Foo {
        private InvocationHandler h;
        public JdkProxy(A12.InvocationHandler h) {
            this.h = h;
        }
        @Override
        public void foo() {
            try {
                Method foo = Foo.class.getMethod("foo");
                h.invoke(foo, new Object[0]);
            } catch (Throwable e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] params) {
        Foo foo = new JdkProxy((method, args) -> {
            System.out.println("before");
            //new Target().foo();
            method.invoke(new Target(), args);
        });
        foo.foo();
    }
}
```

### 代理demo3

```java
public class A12 {
    interface Foo {
        void foo();
        int bar(int i);
    }
    static class Target implements Foo {
        @Override
        public void foo() {
            System.out.println("foo");
        }
        @Override
        public int bar(int i) {
            System.out.println("bar" + i);
            return 100;
        }
    }
    interface InvocationHandler {
        Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
    }
    static class JdkProxy implements Foo {
        static Method foo;
        static Method bar;
        static {
            try {
                foo = Foo.class.getMethod("foo");
                bar = Foo.class.getMethod("bar", int.class);
            } catch (NoSuchMethodException e) {
                throw new NoSuchMethodError(e.getMessage());
            }
        }
        private InvocationHandler h;
        public JdkProxy(A12.InvocationHandler h) {
            this.h = h;
        }
        @Override
        public void foo() {
            try {
                h.invoke(this, foo, new Object[0]);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
        @Override
        public int bar(int i) {
            try {
                return (int)h.invoke(this, bar, new Object[]{i});
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
    }
    public static void main(String[] params) {
        Foo foo = new JdkProxy((proxy, method, args) -> {
            System.out.println("before");
            //new Target().foo();
            return method.invoke(new Target(), args);
        });
        foo.foo();
        foo.bar(4);
    }
}
```

- demo3中的JdkProxy类可以类比为用jdk动态代理生成的类

## CGLIB动态代理原理

### 代理demo1

```java
public class A13 {
    static class Target {
        public void save() {
            System.out.println("save()");
        }
        public void save(int i) {
            System.out.println("save(int)");
        }
        public void save(long j) {
            System.out.println("save(long)");
        }
    }
    static class CglibProxy extends Target {
        private MethodInterceptor methodInterceptor;
        public void setMethodInterceptor(MethodInterceptor methodInterceptor) {
            this.methodInterceptor = methodInterceptor;
        }
        static Method save0;
        static Method save1;
        static Method save2;
        static {
            try {
                save0 = Target.class.getMethod("save");
                save1 = Target.class.getMethod("save", int.class);
                save2 = Target.class.getMethod("save", long.class);
            } catch (NoSuchMethodException e) {
                throw new NoSuchMethodError(e.getMessage());
            }
        }
        @Override
        public void save() {
            try {
                methodInterceptor.intercept(this, save0, new Object[0], null);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
        @Override
        public void save(int i) {
            try {
                methodInterceptor.intercept(this, save1, new Object[]{i}, null);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
        @Override
        public void save(long j) {
            try {
                methodInterceptor.intercept(this, save2, new Object[]{j}, null);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
    }
    public static void main(String[] params) {
        CglibProxy proxy = new CglibProxy();
        Target target = new Target();
        proxy.setMethodInterceptor((o, method, args, methodProxy) -> {
            System.out.println("before");
            return method.invoke(target, args);
        });
        proxy.save();
        proxy.save(1);
        proxy.save(2L);
    }
}
```

### 代理demo2

```java
public class A13 {
    static class Target {
        public void save() {
            System.out.println("save()");
        }
        public void save(int i) {
            System.out.println("save(int)");
        }
        public void save(long j) {
            System.out.println("save(long)");
        }
    }
    static class CglibProxy extends Target {
        private MethodInterceptor methodInterceptor;
        public void setMethodInterceptor(MethodInterceptor methodInterceptor) {
            this.methodInterceptor = methodInterceptor;
        }
        static Method save0;
        static Method save1;
        static Method save2;
        static MethodProxy save0Proxy;
        static MethodProxy save1Proxy;
        static MethodProxy save2Proxy;
        static {
            try {
                save0 = Target.class.getMethod("save");
                save1 = Target.class.getMethod("save", int.class);
                save2 = Target.class.getMethod("save", long.class);
                save0Proxy = MethodProxy.create(Target.class, CglibProxy.class, "()V", "save", "saveSuper");
                save1Proxy = MethodProxy.create(Target.class, CglibProxy.class, "(I)V", "save", "saveSuper");
                save2Proxy = MethodProxy.create(Target.class, CglibProxy.class, "(J)V", "save", "saveSuper");
            } catch (NoSuchMethodException e) {
                throw new NoSuchMethodError(e.getMessage());
            }
        }
        public void saveSuper() {
            super.save();
        }
        public void saveSuper(int i) {
            super.save(i);
        }
        public void saveSuper(long j) {
            super.save(j);
        }
        @Override
        public void save() {
            try {
                methodInterceptor.intercept(this, save0, new Object[0], save0Proxy);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
        @Override
        public void save(int i) {
            try {
                methodInterceptor.intercept(this, save1, new Object[]{i}, save1Proxy);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
        @Override
        public void save(long j) {
            try {
                methodInterceptor.intercept(this, save2, new Object[]{j}, save2Proxy);
            } catch (Throwable e) {
                throw new UndeclaredThrowableException(e);
            }
        }
    }
    public static void main(String[] params) {
        CglibProxy proxy = new CglibProxy();
        Target target = new Target();
        proxy.setMethodInterceptor((o, method, args, methodProxy) -> {
            System.out.println("before");
            //return method.invoke(target, args);
            return methodProxy.invoke(target, args);
        });
        proxy.save();
        proxy.save(1);
        proxy.save(2L);
    }
}
```

- demo中的CglibProxy类可以类比为用cglib动态代理生成的类

## MethodProxy原理

- methodProxy.invoke(target, args); 参数为目标类

```java
public class TargetFastClass {
    static Signature s0 = new Signature("save", "()V");
    static Signature s1 = new Signature("save", "(I)V");
    static Signature s2 = new Signature("save", "(J)V");
    /**
     * 获取目标方法编号
     * Target: save() 0; save(int) 1; save(long) 2
     * signature:包含方法名称，参数，返回值信息
     */
    public int getIndex(Signature signature) {
        if (s0.equals(signature)) {
            return 0;
        } else if (s1.equals(signature)) {
            return 1;
        } else if (s2.equals(signature)) {
            return 2;
        } else {
            return -1;
        }
    }
    /**
     * 根据方法编号，正常调用目标对象方法
     */
    public Object invoke(int index, Object target, Object[] args) {
        if (index == 0) {
            ((A13.Target) target).save();
            return null;
        } else if (index == 1) {
            ((A13.Target) target).save((int)args[0]);
            return null;
        } else if (index == 2) {
            ((A13.Target) target).save((long)args[0]);
            return null;
        } else {
            throw new NoSuchMethodError("无此方法");
        }
    }
    public static void main(String[] args) {
        TargetFastClass fastClass = new TargetFastClass();
        int index = fastClass.getIndex(new Signature("save", "(I)V"));
        System.out.println(index);
        fastClass.invoke(index, new A13.Target(), new Object[]{100});
    }
}
```

- methodProxy.invokeSuper(proxy, args); //参数为代理类

```java
public class ProxyFastClass {
    static Signature s0 = new Signature("saveSuper", "()V");
    static Signature s1 = new Signature("saveSuper", "(I)V");
    static Signature s2 = new Signature("saveSuper", "(J)V");
    /**
     * 获取目标方法编号
     * Proxy: saveSuper() 0; saveSuper(int) 1; saveSuper(long) 2
     * signature:包含方法名称，参数，返回值信息
     */
    public int getIndex(Signature signature) {
        if (s0.equals(signature)) {
            return 0;
        } else if (s1.equals(signature)) {
            return 1;
        } else if (s2.equals(signature)) {
            return 2;
        } else {
            return -1;
        }
    }
    /**
     * 根据方法编号，正常调用目标对象方法
     */
    public Object invoke(int index, Object proxy, Object[] args) {
        if (index == 0) {
            ((A13.CglibProxy) proxy).saveSuper();
            return null;
        } else if (index == 1) {
            ((A13.CglibProxy) proxy).saveSuper((int)args[0]);
            return null;
        } else if (index == 2) {
            ((A13.CglibProxy) proxy).saveSuper((long)args[0]);
            return null;
        } else {
            throw new NoSuchMethodError("无此方法");
        }
    }
    public static void main(String[] args) {
        ProxyFastClass fastClass = new ProxyFastClass();
        int index = fastClass.getIndex(new Signature("saveSuper", "(I)V"));
        System.out.println(index);
        fastClass.invoke(index, new A13.CglibProxy(), new Object[]{100});
    }
}
```

## 切面

- aspect 
  
  - 通知1(advice) + 切点1(pointcut)
  
  - 通知2(advice) + 切点2(pointcut)
  
  - 通知2(advice) + 切点2(pointcut)

- advisor = 更细粒度的切面，包含一个通知和切点

## Spring选择代理方式

- proxyTargetClass = false, 目标实现了接口，用JDK代理实现

- proxyTargetClass = false, 目标没有实现接口，用CGLIB代理实现

- proxyTargetClass = true, 总是用CGLIB实现

```java
public class A14 {
    public static void main(String[] args) {
        //1. 备好切点
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* foo())");
        //2. 备好通知
        MethodInterceptor advice = invocation -> {
            System.out.println("before");
            Object result = invocation.proceed();
            System.out.println("after");
            return result;
        };
        //3. 备好切面
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);
        //4. 创建代理
        Target1 target = new Target1();
        ProxyFactory factory = new ProxyFactory();
        factory.setTarget(target);
        factory.addAdvisor(advisor);
        factory.setInterfaces(target.getClass().getInterfaces());
        I1 proxy = (I1) factory.getProxy();
        System.out.println(proxy.getClass());
        proxy.foo();
        proxy.bar();
    }
    interface I1{
        void foo();
        void bar();
    }
    static class Target1 implements I1 {
        @Override
        public void foo() {
            System.out.println("target1 foo");
        }
        @Override
        public void bar() {
            System.out.println("target1 bar");
        }
    }
    static class Target2 {
        public void foo() {
            System.out.println("target2 foo");
        }
        public void bar() {
            System.out.println("target2 bar");
        }
    }
}
```

- ProxyFactory是用来创建代理的核心实现，用AopProxyFactory选择具体代理实现
  
  - JdkDynamicAopProxy
  
  - ObjenesisCglibAopProxy

## 切点匹配

```java
public class A15 {
    public static void main(String[] args) throws NoSuchMethodException {
        AspectJExpressionPointcut pt1 = new AspectJExpressionPointcut();
        pt1.setExpression("execution(* bar())");
        System.out.println(pt1.matches(T1.class.getMethod("foo"), T1.class));
        System.out.println(pt1.matches(T1.class.getMethod("bar"), T1.class));

        AspectJExpressionPointcut pt2 = new AspectJExpressionPointcut();
        pt2.setExpression("@annotation(org.springframework.transaction.annotation.Transactional)");
        System.out.println(pt2.matches(T1.class.getMethod("foo"), T1.class));
        System.out.println(pt2.matches(T1.class.getMethod("bar"), T1.class));

        StaticMethodMatcherPointcut pt3 = new StaticMethodMatcherPointcut() {
            @Override
            public boolean matches(Method method, Class<?> targetClass) {
                //检查方法上是否加了Transactional注解
                MergedAnnotations annotations = MergedAnnotations.from(method);
                if (annotations.isPresent(Transactional.class)) {
                    return true;
                }
                //查看类上是否加了Transactional注解
                //annotations = MergedAnnotations.from(targetClass);
                //查看类以及继承体系结构中是否加了Transactional注解
                annotations = MergedAnnotations.from(targetClass, MergedAnnotations.SearchStrategy.TYPE_HIERARCHY);
                if (annotations.isPresent(Transactional.class)) {
                    return true;
                }
                return false;
            }
        };
        System.out.println(pt3.matches(T1.class.getMethod("foo"), T1.class));
        System.out.println(pt3.matches(T1.class.getMethod("bar"), T1.class));
        System.out.println(pt3.matches(T2.class.getMethod("foo"), T2.class));
        System.out.println(pt3.matches(T3.class.getMethod("foo"), T3.class));
    }
    static class T1 {
        @Transactional
        public void foo() {}
        public void bar() {}
    }
    @Transactional
    static class T2{
        public void foo() {}
        public void bar() {}
    }
    @Transactional
    interface I3{
        void foo();
    }
    static class T3 implements I3{
        @Override
        public void foo() {

        }
    }
}
```

- 底层切点实现是如何匹配的：调用了aspectj的匹配方法

- 比较关键的是实现了MethodMatcher接口，用来执行方法匹配
