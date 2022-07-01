```java
@Configuration
@SuppressWarnings("all")
public class A46 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A46.class);
        DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
        ContextAnnotationAutowireCandidateResolver resolver = new ContextAnnotationAutowireCandidateResolver();
        test1(context, resolver, Bean1.class.getDeclaredField("home"));
        test2(context, resolver, Bean1.class.getDeclaredField("age"));
        test3(context, resolver, Bean2.class.getDeclaredField("bean3"));

        DependencyDescriptor dd1 = new DependencyDescriptor(Bean4.class.getDeclaredField("value"), false);
        //获取@Value的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);
        //解析${}
        value = context.getEnvironment().resolvePlaceholders(value);
        System.out.println(value);
        System.out.println(value.getClass());
        //解析#{}
        Object bean4 = context.getBeanFactory().getBeanExpressionResolver().evaluate(value, new BeanExpressionContext(context.getBeanFactory(), null));
        Object result = context.getBeanFactory().getTypeConverter().convertIfNecessary(bean4, dd1.getDependencyType());
        System.out.println(result);
    }
    private static void test3(AnnotationConfigApplicationContext context,
                              ContextAnnotationAutowireCandidateResolver resolver, Field field) throws Exception {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        //获取@Value的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);
        //解析#{}
        Object bean3 = context.getBeanFactory().getBeanExpressionResolver().evaluate(value, new BeanExpressionContext(context.getBeanFactory(), null));
        Object result = context.getBeanFactory().getTypeConverter().convertIfNecessary(bean3, dd1.getDependencyType());
        System.out.println(result);
    }
    private static void test2(AnnotationConfigApplicationContext context,
                              ContextAnnotationAutowireCandidateResolver resolver, Field field) throws Exception {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        //获取@Value的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);
        Object age = context.getBeanFactory().getTypeConverter().convertIfNecessary(value, dd1.getDependencyType());
        System.out.println(age.getClass());
    }
    private static void test1(AnnotationConfigApplicationContext context,
                              ContextAnnotationAutowireCandidateResolver resolver, Field field) throws Exception {
        DependencyDescriptor dd1 = new DependencyDescriptor(field, false);
        //获取@Value的内容
        String value = resolver.getSuggestedValue(dd1).toString();
        System.out.println(value);
        //解析${}
        value = context.getEnvironment().resolvePlaceholders(value);
        System.out.println(value);
        System.out.println(value.getClass());
    }
    public class Bean1{
        @Value("${JAVA_HOME}")
        private String home;
        @Value("18")
        private int age;
    }
    public class Bean2{
        @Value("#{@bean3}")
        private Bean3 bean3;
    }
    @Component("bean3")
    public class Bean3{}
    public class Bean4{
        @Value("#{'hello, ' + '${JAVA_HOME}'}")
        private String value;
    }
}
```
