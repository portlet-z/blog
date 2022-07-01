- FactoryBean在Spring发展过程中重要，但目前已经很鸡肋的接口

- 他的作用是制造创建过程较为复杂的产品，如SqlSessionFactory,但@Bean已具备等价功能

- 使用上较为古怪，一不留神就会出错
  
  - 被FactoryBean创建的产品
    
    - 会认为创建，依赖注入，Aware接口回调，前期初始化这些都是FactoryBean的职责，这些流程都不会走
    
    - 唯有后初始化的流程会走，也就是产品可以被代理增强
    
    - 单例的产品不会存储于BeanFactory的singletonObjects成员中，而是另一个factoryBeanObject
  
  - 按名字去获取时，拿到的是产品对象，名字前加&获取的是工厂对象

- 但目前此接口被大量使用，想被废弃很难

```java
@ComponentScan
public class A43 {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(A43.class);
        Bean1 bean1 = (Bean1) context.getBean("bean1");
        Bean1 bean2 = (Bean1) context.getBean("bean1");
        Bean1 bean3 = (Bean1) context.getBean("bean1");
        System.out.println(bean1);
        System.out.println(bean2);
        System.out.println(bean3);
        System.out.println(context.getBean(Bean1.class));
        System.out.println(context.getBean(Bean1FactoryBean.class));
        System.out.println(context.getBean("&bean1"));
        context.close();
    }
}
@Slf4j
public class Bean1 implements BeanFactoryAware {
    private Bean2 bean2;
    @Autowired
    public void setBean2(Bean2 bean2) {
        log.debug("setBean2({})", bean2);
        this.bean2 = bean2;
    }
    public Bean2 getBean2() {
        return bean2;
    }
    @PostConstruct
    public void init() {
        log.debug("init()");
    }
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        log.debug("setBeanFactory({})", beanFactory);
    }
}
@Slf4j
@Component("bean1")
public class Bean1FactoryBean implements FactoryBean<Bean1> {
    //决定了根据【类型】获取或依赖注入能否成功
    @Override
    public Class<?> getObjectType() {
        return Bean1.class;
    }
    //决定了getObject()方法被调用一次还是多次
    @Override
    public boolean isSingleton() {
        return true;
    }
    @Override
    public Bean1 getObject() throws Exception {
        Bean1 bean1 = new Bean1();
        log.debug("create bean: {}", bean1);
        return bean1;
    }
}
@Slf4j
public class Bean1PostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("bean1") && bean instanceof Bean1) {
            log.debug("before [{}] init", beanName);
        }
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("bean1") && bean instanceof Bean1) {
            log.debug("after [{}] init", beanName);
        }
        return bean;
    }
}
public class Bean2 {
}
```
