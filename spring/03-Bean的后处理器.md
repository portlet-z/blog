## 后处理器作用：为Bean生命周期各个阶段提供扩展

```java

public static void main(String[] args) {
    //GenericApplicationContext是一个【干净】的容器
    GenericApplicationContext context = new GenericApplicationContext();
    //注册bean
    context.registerBean("bean1", Bean1.class);
    context.registerBean("bean2", Bean2.class);
    context.registerBean("bean3", Bean3.class);
    context.registerBean("bean4", Bean4.class);
    context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
    //@Autowired, @Value
    context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
    //@Resource, @PostConstruct, @PreDestroy
    context.registerBean(CommonAnnotationBeanPostProcessor.class);
    //@ConfigurationProperties
    ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());
    //初始化容器,执行beanFactory后处理器，添加bean后处理器，初始化所有单例
    context.refresh();
    System.out.println(context.getBean("bean4"));
    //销毁容器
    context.close();
}
class Bean1 {
    private Bean2 bean2;
    @Autowired
    public void setBean2(Bean2 bean2) {
        System.out.println("Autowired生效" + bean2);
        this.bean2 = bean2;
    }
    private Bean3 bean3;
    @Resource
    public void setBean3(Bean3 bean3) {
        System.out.println("Resource生效" + bean3);
        this.bean3 = bean3;
    }
    private String home;
    @Autowired
    public void setHome(@Value("${JAVA_HOME}") String home) {
        System.out.println("@Value生效" + home);
        this.home = home;
    }
    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct生效");
    }
    @PreDestroy
    public void destroy() {
        System.out.println("@PreDestroy生效");
    }
    @Override
    public String toString() {
        return "Bean1{" +
                "bean2=" + bean2 +
                ", bean3=" + bean3 +
                ", home='" + home + '\'' +
                '}';
    }
}
class Bean2 {

}
class Bean3 {

}
@ConfigurationProperties(prefix = "java")
class Bean4 {
    private String home;
    private String version;
    public String getHome() {
        return home;
    }
    public String getVersion() {
        return version;
    }
    public void setHome(String home) {
        this.home = home;
    }
    public void setVersion(String version) {
        this.version = version;
    }

    @Override
    public String toString() {
        return "Bean4{" +
                "home='" + home + '\'' +
                ", version='" + version + '\'' +
                '}';
    }
}

```

## 常见的后处理器
