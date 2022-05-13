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

2. 封装启动args

3. 准备Environment添加命令行参数（*）

4. ConfigurationPropertySource处理（*）
   
   - 发布application environment已准备事件

5. 通过EnvironmentPostProcessorApplicationContext进行env后处理（*）
   
   - application.properties,由StandardCOnfigDataLocationResovler解析
   
   - spring.application.json

6. 绑定spring.main到SpringApplication对象（*）

7. 打印banner（*）

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
