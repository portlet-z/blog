- @Autowired 本质上是根据成员变量或方法参数的类型进行装配
- 如果待装配类型是 Optional，需要根据 Optional 泛型找到 bean，再封装为 Optional 对象装配
- 如果待装配的类型是 ObjectFactory，需要根据 ObjectFactory 泛型创建 ObjectFactory 对象装配
  - 此方法可以延迟 bean 的获取
- 如果待装配的成员变量或方法参数上用 @Lazy 标注，会创建代理对象装配
  - 此方法可以延迟真实 bean 的获取
  - 被装配的代理不作为 bean

```java
@Configuration
public class A47_1 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_1.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        //1. 根据成员变量的类型注入
        DependencyDescriptor dd1 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean2"), false);
        System.out.println(beanFactory.doResolveDependency(dd1, "bean1", null, null));
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        //2. 根据参数的类型注入
        Method setBean2 = Bean1.class.getDeclaredMethod("setBean2", Bean2.class);
        DependencyDescriptor dd2 = new DependencyDescriptor(new MethodParameter(setBean2, 0), false);
        System.out.println(beanFactory.doResolveDependency(dd2, "bean1", null, null));
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        //3. 结果包装为Optional<Bean2>
        DependencyDescriptor dd3 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean3"), false);
        if (dd3.getDependencyType() == Optional.class) {
            dd3.increaseNestingLevel();
            Object result = beanFactory.doResolveDependency(dd3, "bean1", null, null);
            System.out.println(Optional.ofNullable(result));
        }
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        //4. 结果包装为ObjectFactory<Bean2>
        DependencyDescriptor dd4 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean4"), false);
        if (dd4.getDependencyType() == ObjectFactory.class) {
            dd4.increaseNestingLevel();
            ObjectFactory objectFactory = () -> {
                Object result = beanFactory.doResolveDependency(dd4, "bean1", null, null);
                return result;
            };
            System.out.println(objectFactory.getObject());
        }
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        //5. 对@Lazy的处理
        DependencyDescriptor dd5 = new DependencyDescriptor(Bean1.class.getDeclaredField("bean2"),false);
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        Object proxy = resolver.getLazyResolutionProxyIfNecessary(dd5, "bean1");
        System.out.println(proxy);
        System.out.println(proxy.getClass());
    }
    static class Bean1 {
        @Autowired @Lazy
        private Bean2 bean2;
        @Autowired
        public void setBean2(Bean2 bean2) {
            this.bean2 = bean2;
        }
        @Autowired
        private Optional<Bean2> bean3;
        @Autowired
        private ObjectFactory<Bean2> bean4;
    }
    @Component("bean2")
    static class Bean2 {}
}

```

- 如果待装配类型是数组，需要获取数组元素类型，根据此类型找到多个 bean 进行装配
- 如果待装配类型是 Collection 或其子接口，需要获取 Collection 泛型，根据此类型找到多个 bean
- 如果待装配类型是 ApplicationContext 等特殊类型
  - 会在 BeanFactory 的 resolvableDependencies 成员按类型查找装配
  - resolvableDependencies 是 map 集合，key 是特殊类型，value 是其对应对象
  - 不能直接根据 key 进行查找，而是用 isAssignableFrom 逐一尝试右边类型是否可以被赋值给左边的 key 类型
- 如果待装配类型有泛型参数
  - 需要利用 ContextAnnotationAutowireCandidateResolver 按泛型参数类型筛选
- 如果待装配类型有 @Qualifier
  - 需要利用 ContextAnnotationAutowireCandidateResolver 按注解提供的 bean 名称筛选

```java
@Configuration
public class A47_2 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_2.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        testArray(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>");
        testList(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>");
        testApplicationContext(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>");
        testGeneric(beanFactory);
        System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>");
        testQualifier(beanFactory);
    }
    private static void testQualifier(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd5 = new DependencyDescriptor(Target.class.getDeclaredField("service"), true);
        Class<?> type = dd5.getDependencyType();
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type);
        for (String name : names) {
            BeanDefinition bd = beanFactory.getMergedBeanDefinition(name);
            if (resolver.isAutowireCandidate(new BeanDefinitionHolder(bd, name), dd5)) {
                System.out.println(name);
                System.out.println(dd5.resolveCandidate(name, type, beanFactory));
            }
        }
    }
    private static void testGeneric(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd4 = new DependencyDescriptor(Target.class.getDeclaredField("dao"), true);
        Class<?> type = dd4.getDependencyType();
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        resolver.setBeanFactory(beanFactory);
        String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type);
        for (String name : names) {
            BeanDefinition bd = beanFactory.getMergedBeanDefinition(name);
            //对比BeanDefinition与DependencyDescriptor的泛型是否匹配
            if (resolver.isAutowireCandidate(new BeanDefinitionHolder(bd, name), dd4)) {
                System.out.println(name);
                System.out.println(dd4.resolveCandidate(name, type, beanFactory));
            }
        }
    }
    private static void testApplicationContext(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd3 = new DependencyDescriptor(Target.class.getDeclaredField("applicationContext"), true);
        Field resolvableDependencies = DefaultListableBeanFactory.class.getDeclaredField("resolvableDependencies");
        resolvableDependencies.setAccessible(true);
        Map<Class<?>, Object> dependencies = (Map<Class<?>, Object>) resolvableDependencies.get(beanFactory);
        dependencies.forEach((k, v) -> {
            System.out.println("key:" + k + " value:" + v);
        });
        for (Map.Entry<Class<?>, Object> entry : dependencies.entrySet()) {
            if (entry.getKey().isAssignableFrom(dd3.getDependencyType())) {
                System.out.println(entry.getValue());
                break;
            }
        }
    }
    private static void testList(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd2 = new DependencyDescriptor(Target.class.getDeclaredField("serviceList"), true);
        if (dd2.getDependencyType() == List.class) {
            Class<?> resolve = dd2.getResolvableType().getGeneric().resolve();
            System.out.println(resolve);
            String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, resolve);
            List<Object> list = new ArrayList<>();
            for (String name : names) {
                Object bean = dd2.resolveCandidate(name, resolve, beanFactory);
                list.add(bean);
            }
            System.out.println(list);
        }
    }
    private static void testArray(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd1 = new DependencyDescriptor(Target.class.getDeclaredField("serviceArray"), true);
        if (dd1.getDependencyType().isArray()) {
            Class<?> componentType = dd1.getDependencyType().getComponentType();
            System.out.println(componentType);
            String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, componentType);
            List<Object> beans = new ArrayList<>();
            for (String name : names) {
                System.out.println(name);
                Object bean = dd1.resolveCandidate(name, componentType, beanFactory);
                beans.add(bean);
            }
            Object array = beanFactory.getTypeConverter().convertIfNecessary(beans, dd1.getDependencyType());
            System.out.println(array);
        }
    }
    static class Target {
        @Autowired private Service[] serviceArray;
        @Autowired private List<Service> serviceList;
        @Autowired private ConfigurableApplicationContext applicationContext;
        @Autowired private Dao<Teacher> dao;
        @Autowired @Qualifier("service2") private Service service;
    }
    interface Service {}
    @Component("service1")
    static class Service1 implements Service {}
    @Component("service2")
    static class Service2 implements Service {}
    @Component("service3")
    static class Service3 implements Service {}
    interface Dao<T> {}
    static class Student {}
    static class Teacher {}
    @Component("dao1")
    static class Dao1 implements Dao<Student> {}
    @Component("dao2")
    static class Dao2 implements Dao<Teacher> {}
}
```

- 有 @Primary 标注的 @Component 或 @Bean 的处理
- 与成员变量名或方法参数名同名 bean 的处理

```java
@Configuration
public class A47_3 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A47_3.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        testPrimary(beanFactory);
        testDefault(beanFactory);
    }
    private static void testDefault(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd = new DependencyDescriptor(Target2.class.getDeclaredField("service3"), false);
        Class<?> type = dd.getDependencyType();
        String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type);
        for (String name : names) {
            if (name.equals(dd.getDependencyName())) {
                System.out.println(name);
            }
        }
    }
    private static void testPrimary(DefaultListableBeanFactory beanFactory) throws Exception {
        DependencyDescriptor dd = new DependencyDescriptor(Target1.class.getDeclaredField("service"), false);
        Class<?> type = dd.getDependencyType();
        String[] names = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(beanFactory, type);
        for (String name : names) {
            if (beanFactory.getMergedBeanDefinition(name).isPrimary()) {
                System.out.println(name);
            }
        }
    }
    static class Target1 {
        @Autowired private Service service;
    }
    static class Target2 {
        @Autowired private Service service3;
    }
    interface Service {}
    @Component("service1")
    static class Service1 implements Service {}
    @Component("service2") @Primary
    static class Service2 implements Service {}
    @Component("service3")
    static class Service3 implements Service {}
}
```
