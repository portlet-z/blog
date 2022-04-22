- Aware接口提供了一种【内置】的注入手段，可以注入BeanFactory, ApplicationContext

- InitializingBean接口提供了一种【内置】的初始化手段

- 内置的注入和初始化不受扩展功能的影响，总会被执行，因此Spring框架内部的类常用他们

- @Autowired失效分析

- Aware接口用于注入一些与容器相关信息，例如
  
  - a. BeanNameAware注入bean的名字
  
  - b. BeanFactoryAware注入BeanFactory容器
  
  - c. ApplicationContextAware注入ApplicationContext容器
  
  - d. EmbeddedValueResolverAware ${}

```java
public class MyBean implements BeanNameAware, InitializingBean, ApplicationContextAware {
    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean: " + this + " 名称为: " + name);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("当前bean: " + this + " 初始化完成");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("当前bean: " + this + " 容器是：" + applicationContext);
    }
}
```

- 上述b,c,d功能用@Autowired就能实现，为啥还用Aware接口
  - @Autowired的解析需要用到bean后处理器，属于扩展功能
  - 而Aware接口属于内置功能，不加任何扩展，Spring就能识别。某些情况下，扩展功能会失效，而内置功能不会失效
