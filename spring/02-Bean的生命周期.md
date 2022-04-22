## Bean生命周期的各个阶段

```java
@Component
public class LifeCycleBean {
    public LifeCycleBean() {
        System.out.println("构造方法()");
    }
    @Autowired
    public void autowired(@Value("${JAVA_HOME}") String home){
        System.out.println("依赖注入" + home);
    }
    @PostConstruct
    public void init(){
        System.out.println("init");
    }
    @PreDestroy
    public void destroy(){
        System.out.println("destroy");
    }
}
```

```java
@Component
public class MyBeanPostProcessor implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("销毁前执行");
        }
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("实例化前执行，这里返回的对象会替换掉原本的bean");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("实例化之后执行，这里如果返回false会跳过依赖注入阶段");
        }
        return true;
    }

    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("依赖注入阶段执行，如@Autowired, @Resource");
        }
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("初始化之前执行，这里返回的对象会替换掉原本的bean,如@PostConstruct, @ConfigurationProperties");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("lifeCycleBean")) {
            System.out.println("初始化之后执行，这里返回的对象会替换掉原本的bean,如代理增强");
        }
        return bean;
    }
}
```

## 模板设计模式

```java
public static void main(String[] args) {
        MyBeanFactory beanFactory = new MyBeanFactory();
        beanFactory.addBeanProcessor(bean -> System.out.println("解析Autowired注解"));
        beanFactory.addBeanProcessor(bean -> System.out.println("解析Resource注解"));
        beanFactory.getBean();
    }
    //模板方法 Template Method Pattern
    static class MyBeanFactory {
        List<BeanPostProcessor> processors = new ArrayList<>();
        public void addBeanProcessor(BeanPostProcessor processor) {
            processors.add(processor);
        }
        public Object getBean() {
            Object bean = new Object();
            System.out.println("构造 " + bean);
            System.out.println("依赖注入 " + bean); //Autowired, Resource
            for (BeanPostProcessor processor : processors) {
                processor.inject(bean);
            }
            return bean;
        }
    }

    static interface BeanPostProcessor {
        void inject(Object bean); //对依赖注入阶段扩展
    }
```
