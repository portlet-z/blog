## 阶段一：SpringApplication容器

- 记录BeanDefinition源

- 推断应用类型

- 记录ApplicationContext初始化容器

- 记录监听器

- 推断主启动类

```java
@Configuration
public class A39_1 {
    public static void main(String[] args) throws Exception {
        System.out.println("1. 演示获取Bean Definition源");
        SpringApplication spring = new SpringApplication(A39_1.class);
        spring.setSources(Set.of("classpath:b01.xml"));
        System.out.println("2. 演示推断应用类型");
        Method deduceFromClasspath = WebApplicationType.class.getDeclaredMethod("deduceFromClasspath");
        deduceFromClasspath.setAccessible(true);
        System.out.println("\t应用类型为：" + deduceFromClasspath.invoke(null));
        System.out.println("3. 演示ApplicationContext初始化器");
        spring.addInitializers(applicationContext -> {
            if (applicationContext instanceof GenericApplicationContext context) {
                context.registerBean("bean3", Bean3.class);
            }
        });
        System.out.println("4. 演示监听器与事件");
        spring.addListeners(event -> System.out.println("\t事件为:" + event.getClass()));
        System.out.println("5. 演示主类推断");
        Method deduceMainApplicationClass = SpringApplication.class.getDeclaredMethod("deduceMainApplicationClass");
        deduceMainApplicationClass.setAccessible(true);
        System.out.println("\t主类是：" + deduceMainApplicationClass.invoke(spring));

        ConfigurableApplicationContext context = spring.run(args);
        //创建ApplicationContext
        //调用初始化器对ApplicationContext做扩展
        //ApplicationContext.refresh
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println("name: " + name + " 来源：" + context.getBeanFactory().getBeanDefinition(name).getResourceDescription());
        }
        context.close();
    }
    static class Bean1 {}
    static class Bean2 {}
    static class Bean3 {}
    @Bean
    public Bean2 bean2() {
        return new Bean2();
    }
    @Bean
    public TomcatServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
}

```

## 阶段二：执行run方法

1. 得到SpringApplicationRunListeners, 名字取的不好，实际是事件发布器
  
   - 发布application starting事件
   
   ```java
   public class A39_2 {
       public static void main(String[] args) throws Exception {
           SpringApplication app = new SpringApplication();
           app.addListeners(e -> System.out.println(">>>>>>>>" + e.getClass()));
           //获取事件发送器实现类名
           List<String> names = SpringFactoriesLoader.loadFactoryNames(SpringApplicationRunListener.class, A39_2.class.getClassLoader());
           for (String name : names) {
               //System.out.println(name);
               Class<?> clazz = Class.forName(name);
               Constructor<?> constructor = clazz.getConstructor(SpringApplication.class, String[].class);
               SpringApplicationRunListener publisher = (SpringApplicationRunListener) constructor.newInstance(app, args);
               //发布事件
               DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
               publisher.starting(bootstrapContext);//spring boot开始启动
               publisher.environmentPrepared(bootstrapContext, new StandardEnvironment()); //环境信息准备完毕
               GenericApplicationContext context = new GenericApplicationContext();
               publisher.contextPrepared(context); //在Spring容器创建，并调用初始化器之后，发送此事件
               publisher.contextLoaded(context); //所有bean definition记载完毕
               context.refresh();
               publisher.started(context); //spring 容器初始化完成（refresh 方法调用完毕）
               publisher.running(context); //spring boot启动完毕
               publisher.failed(context, new Exception("failed"));
           }
       }
   }
   ```

2. 封装启动args
  
   ```java
   public class A39_3 {
       public static void main(String[] args) throws Exception {
           SpringApplication app = new SpringApplication();
           app.addInitializers(applicationContext -> System.out.println("执行初始化容器增强........."));
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>2. 封装启动args");
           DefaultApplicationArguments arguments = new DefaultApplicationArguments(args);
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>8. 创建容器");
           GenericApplicationContext context = createApplicationContext(WebApplicationType.SERVLET);
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>9. 准备容器");
           for (ApplicationContextInitializer initializer : app.getInitializers()) {
               initializer.initialize(context);
           }
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>10. 加载bean定义");
           DefaultListableBeanFactory beanFactory = context.getDefaultListableBeanFactory();
           AnnotatedBeanDefinitionReader reader1 = new AnnotatedBeanDefinitionReader(beanFactory);
           reader1.register(Config.class);
           XmlBeanDefinitionReader reader2 = new XmlBeanDefinitionReader(beanFactory);
           reader2.loadBeanDefinitions(new ClassPathResource("b03.xml"));
           ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(beanFactory);
           scanner.scan("com.bytebuf.a39.sub");
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>11. refresh容器");
           context.refresh();
           for (String name : context.getBeanDefinitionNames()) {
               System.out.println("name:" + name + " 来源：" + beanFactory.getBeanDefinition(name).getResourceDescription());
           }
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>>>12. 执行runner");
           for (CommandLineRunner runner : context.getBeansOfType(CommandLineRunner.class).values()) {
               runner.run(args);
           }
           for (ApplicationRunner runner : context.getBeansOfType(ApplicationRunner.class).values()) {
               runner.run(arguments);
           }
       }
       private static GenericApplicationContext createApplicationContext(WebApplicationType type) {
           GenericApplicationContext context = null;
           switch (type) {
               case SERVLET -> context = new AnnotationConfigServletWebServerApplicationContext();
               case REACTIVE -> context = new AnnotationConfigReactiveWebServerApplicationContext();
               case NONE -> context = new AnnotationConfigApplicationContext();
           }
           return context;
       }
       static class Bean4{}
       static class Bean5{}
       @Configuration
       static class Config {
           @Bean
           public Bean5 bean5() {return new Bean5();}
           @Bean
           public ServletWebServerFactory servletWebServerFactory() {
               return new TomcatServletWebServerFactory();
           }
           @Bean
           public CommandLineRunner commandLineRunner() {
               return args -> System.out.println("commandLineRunner()..." + Arrays.toString(args));
           }
           @Bean
           public ApplicationRunner applicationRunner() {
               return args -> {
                   System.out.println("applicationRunner()..." + Arrays.toString(args.getSourceArgs()));
                   System.out.println(args.getOptionNames());
                   System.out.println(args.getOptionValues("server.port"));
                   System.out.println(args.getNonOptionArgs());
               };
           }
       }
   }
   ```

3. 准备Environment添加命令行参数（*）
  
   ```java
   public class Step3 {
       public static void main(String[] args) throws IOException  {
           ApplicationEnvironment env = new ApplicationEnvironment();
           env.getPropertySources().addLast(new ResourcePropertySource(new ClassPathResource("application.properties")));
           env.getPropertySources().addFirst(new SimpleCommandLinePropertySource(args));
           for (PropertySource<?> ps : env.getPropertySources()) {
               System.out.println(ps);
           }
           System.out.println(env.getProperty("JAVA_HOME"));
           System.out.println(env.getProperty("server.port"));
       }
   }
   ```

4. ConfigurationPropertySource处理（*）
  
   - 发布application environment已准备事件
   
   ```java
   public class Step4 {
       public static void main(String[] args) throws Exception {
           ApplicationEnvironment env = new ApplicationEnvironment();
           env.getPropertySources().addLast(new ResourcePropertySource("step4", new ClassPathResource("step4.properties")));
           ConfigurationPropertySources.attach(env);
           for (PropertySource<?> ps : env.getPropertySources()) {
               System.out.println(ps);
           }
           System.out.println(env.getProperty("user.first-name"));
           System.out.println(env.getProperty("user.middle-name"));
           System.out.println(env.getProperty("user.last-name"));
       }
   }
   ```

5. 通过EnvironmentPostProcessorApplicationContext进行env后处理（*）
  
   - application.properties,由StandardConfigDataLocationResovler解析
   
   - spring.application.json
   
   ```java
   public class Step5 {
       public static void main(String[] args) {
           SpringApplication app = new SpringApplication();
           ApplicationEnvironment env = new ApplicationEnvironment();
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>增强前");
           for (PropertySource<?> ps : env.getPropertySources()) {
               System.out.println(ps);
           }
           ConfigDataEnvironmentPostProcessor postProcessor1 = new ConfigDataEnvironmentPostProcessor(new DeferredLogs(), new DefaultBootstrapContext());
           postProcessor1.postProcessEnvironment(env, app);
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>增强后");
           for (PropertySource<?> ps : env.getPropertySources()) {
               System.out.println(ps);
           }
           RandomValuePropertySourceEnvironmentPostProcessor postProcessor2 = new RandomValuePropertySourceEnvironmentPostProcessor(new DeferredLog());
           postProcessor2.postProcessEnvironment(env, app);
           System.out.println(">>>>>>>>>>>>>>>>>>>>>>>>>>增强后");
           for (PropertySource<?> ps : env.getPropertySources()) {
               System.out.println(ps);
           }
           System.out.println(env.getProperty("server.port"));
           System.out.println(env.getProperty("random.int"));
           System.out.println(env.getProperty("random.int"));
       }
   }
   ```

6. 绑定spring.main到SpringApplication对象（*）
  
   ```java
   public class Step6 {
       public static void main(String[] args) throws Exception {
           SpringApplication application = new SpringApplication();
           ApplicationEnvironment env = new ApplicationEnvironment();
           env.getPropertySources().addLast(new ResourcePropertySource("step4", new ClassPathResource("step4.properties")));
           User user = Binder.get(env).bind("user", User.class).get();
           System.out.println(user);
           User user1 = new User();
           Binder.get(env).bind("user", Bindable.ofInstance(user1));
           System.out.println(user1);
           System.out.println(application);
           Binder.get(env).bind("spring.main", Bindable.ofInstance(application));
           System.out.println(application);
       }
       @Data
       static class User {
           private String firstName;
           private String middleName;
           private String lastName;
       }
   }
   ```

7. 打印banner（*）
  
   ```java
   
   ```

8. 创建容器

9. 准备容器
  
   - 发布application context已初始化事件

10. 加载bean定义
  
    - 发布application prepared事件

11. refresh容器
  
    - 发布application started事件

12. 执行runner
  
    - 发布application ready事件
    
    - 这其中有异常，发布application failed事件

## 收获

1. SpringApplication构造方法中所做的操作
  
   - 可以有多种源来加载bean定义
   
   - 应用类型推断
   
   - 添加容器初始化器
   
   - 添加监听器
   
   - 演示主类推断

2. 如何读取spring.factories中配置

3. 从配置中获取重要的事件发布器：SpringApplicationRunListeners

4. 容器的创建，初始化器增强，加载bean定义等

5. CommandLinRunner, ApplicationRunner作用

6. 环境对象
  
   - 命令行PropertySource
   
   - ConfigurationPropertySources规范环境键名称
   
   - EnvironmentPostProcessor后处理器：由EventPublishingRunListener通过监听事件来调用
   
   - 绑定spring.main前缀的key value至SpringApplication

```java

```
