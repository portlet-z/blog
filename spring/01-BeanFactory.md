## 什么是BeanFactory

- 它是ApplicationContext的父接口

- 它才是Spring核心容器，主要的ApplicationContext实现都【组合】了它的功能

## BeanFactory能干点啥

- 表面上只有getBean

- 实际上控制反转，基本的依赖注入，直至Bean的生命周期的各种功能，都由它的实现类提供

## ApplicationContext

- 提供国际化的功能

```java
//新建resources/messages.properties, resources/messages_en.properties, resources/messages_zh.properties三个文件
//messages_en.properties hi=hello
//messages_zh.properties hi=你好
System.out.println(context.getMessage("hi", null, Locale.ENGLISH));
System.out.println(context.getMessage("hi", null, Locale.CHINESE));
```

- 获取资源文件

```java
Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
//classpath后面加*是获取jar包里面的资源文件
for(Resource resource : resources) {}
```

- 获取环境变量

```java
//获取系统的环境变量，不区分大小
System.out.println(context.getEnvironment().getProperty("java_home"));
//获取配置文件中的变量
System.out.println(context.getEnvironment().getProperty("server.port"));
```

- 事件发布，用于事件解耦

```java
context.publishEvent(new UserRegisterEvent(context));
```

- BeanFactory与ApplicationContext并不仅仅是简单接口继承的关系，ApplicationConte xt组合并扩展了BeanFactory的功能

## BeanFactory不会做的事情

- 不会主动调用BeanFactory后处理器

- 不会主动添加Bean后处理器

- 不会主动初始化单例

- 不会解析beanFactory还不会解析${}, #{}

```java
public class Main {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //bean的定义(class, scope, 初始化, 销毁)
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton").getBeanDefinition();
        beanFactory.registerBeanDefinition("config", beanDefinition);
        //给BeanFactory添加一些常用的后处理器
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
        //BeanFactory后处理器主要功能，补充了一些bean定义
        beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values().forEach(beanFactoryPostProcessor -> {
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        });
        //Bean后处理器，针对bean的生命周期的各个阶段提供扩展，例如@Autowired, @Resource
        beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(beanFactory::addBeanPostProcessor);
        for (String name : beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //准备好所有单例
        beanFactory.preInstantiateSingletons();
        System.out.println(beanFactory.getBean(Bean2.class).getBean1());
    }

    @Configuration
    static class Config {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2(bean1());
        }
    }

    static class Bean1 {
        public Bean1() {
            System.out.println("Bean1 init");
        }
    }

    static class Bean2 {
        private Bean1 bean1;

        public Bean2(Bean1 bean1) {
            this.bean1 = bean1;
            System.out.println("Bean2 init");
        }
        public Bean1 getBean1() {
            return bean1;
        }
    }
}
```

## ApplicationContext的实现

```java
    //较为经典的容器，基于classpath下的xml格式配置文件来创建
    private static void testClassPathXmlApplicationContext() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:b01.xml");
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        System.out.println(context.getBean(Bean2.class).getBean1());
    }

    //BeanFactory实现读取xml配置
    private static void testBeanFactoryXml() {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        System.out.println("读取之前");
        for (String name : beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        System.out.println("读取之后");
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
        reader.loadBeanDefinitions(new ClassPathResource("b01.xml"));
        for (String name : beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
    }

    //基于磁盘路径下的xml格式配置文件来创建
    private static void testFileSystemXmlApplicationContext() {
        FileSystemXmlApplicationContext context = new FileSystemXmlApplicationContext("//Users/portlet/Developer/code_source/src/main/resources/b01.xml");
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        System.out.println(context.getBean(Bean2.class).getBean1());
    }

    //基于Java配置类来创建
    private static void testAnnotationConfigApplicationContext() {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
    }

    //基于Java配置来创建，用于Web环境
    private static void testAnnotationConfigServletWebServerApplicationContext() {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
    }

    @Configuration
    static class WebConfig {
        @Bean
        public ServletWebServerFactory servletWebServerFactory() {
            return new TomcatServletWebServerFactory();
        }
        @Bean
        public DispatcherServlet dispatcherServlet() {
            return new DispatcherServlet();
        }
        @Bean
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet) {
            return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        }
        @Bean("/hello")
        public Controller controller1() {
            return (request, response) -> {
                response.getWriter().println("hello");
                return null;
            };
        }
    }

    @Configuration
    static class AppConfig {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }
    }

    static class Bean1 {
        public Bean1() {
            System.out.println("Bean1 init");
        }
    }

    static class Bean2 {
        private Bean1 bean1;

        public Bean1 getBean1() {
            return bean1;
        }

        public void setBean1(Bean1 bean1) {
            this.bean1 = bean1;
        }
    }
```
