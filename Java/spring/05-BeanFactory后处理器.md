- BeanFactory后处理器作用：为BeanFactory提供扩展

```java
@Configuration
@ComponentScan("com.bytebuf.a05.component")
public class Config {
    @Bean
    public Bean1 bean1() {
        return new Bean1();
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }

    @Bean(initMethod = "init")
    public DruidDataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("");
        return dataSource;
    }

//    @Bean
//    public MapperFactoryBean<Mapper1> mapper1(SqlSessionFactory sqlSessionFactory) {
//        MapperFactoryBean<Mapper1> mapperFactoryBean = new MapperFactoryBean<>(Mapper1.class);
//        mapperFactoryBean.setSqlSessionFactory(sqlSessionFactory);
//        return mapperFactoryBean;
//    }
//
//    @Bean
//    public MapperFactoryBean<Mapper2> mapper2(SqlSessionFactory sqlSessionFactory) {
//        MapperFactoryBean<Mapper2> mapperFactoryBean = new MapperFactoryBean<>(Mapper2.class);
//        mapperFactoryBean.setSqlSessionFactory(sqlSessionFactory);
//        return mapperFactoryBean;
//    }
}

public class A05Application {
    public static void main(String[] args) throws IOException {
        //GenericApplicationContext是一个【干净】的容器
        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean("config", Config.class);
        //@ComponentScan, @Bean, @Import, @ImportResource注解的解析器
        //context.registerBean(ConfigurationClassPostProcessor.class);
        context.registerBean(ComponentScanPostProcessor.class);
        context.registerBean(AtBeanPostProcessor.class);
        //@MapperScan注解的解析器
        //context.registerBean(MapperScannerConfigurer.class, bd -> bd.getPropertyValues().add("basePackage", "com.bytebuf.a05.mapper"));
        context.registerBean(MapperPostProcessor.class);
        //初始化容器
        context.refresh();
        for (String name : context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //销毁容器
        context.close();
    }
}

```

- ComponentScan后处理器实现

```java
public class ComponentScanPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanFactory) throws BeansException {
        try {
            ComponentScan componentScan = AnnotationUtils.findAnnotation(Config.class, ComponentScan.class);
            if (componentScan != null) {
                for (String p : componentScan.basePackages()) {
                    //System.out.println(p);
                    String path = "classpath*:" + p.replace(".", "/") + "/**/*.class";
                    //System.out.println(path);
                    CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
                    Resource[] resources = new PathMatchingResourcePatternResolver().getResources(path);
                    AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();
                    for (Resource resource : resources) {
                        //System.out.println(resource);
                        MetadataReader reader = factory.getMetadataReader(resource);
                        //System.out.println("类名：" + reader.getClassMetadata().getClassName());
                        //System.out.println("是否加了@Component注解：" + reader.getAnnotationMetadata().hasAnnotation(Component.class.getName()));
                        //System.out.println("是否加了@Component派生注解：" + reader.getAnnotationMetadata().hasMetaAnnotation(Component.class.getName()));
                        AnnotationMetadata annotationMetadata = reader.getAnnotationMetadata();
                        if (annotationMetadata.hasAnnotation(Component.class.getName()) || annotationMetadata.hasMetaAnnotation(Component.class.getName())) {
                            AbstractBeanDefinition bd = BeanDefinitionBuilder.genericBeanDefinition(reader.getClassMetadata().getClassName()).getRawBeanDefinition();
                            String name = generator.generateBeanName(bd, beanFactory);
                            beanFactory.registerBeanDefinition(name, bd);
                        }
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {

    }
}

```

- Bean后处理器实现

```java
public class AtBeanPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        try {
            CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
            MetadataReader reader = factory.getMetadataReader(new ClassPathResource("com/bytebuf/a05/Config.class"));
            Set<MethodMetadata> methods = reader.getAnnotationMetadata().getAnnotatedMethods(Bean.class.getName());
            for (MethodMetadata method : methods) {
                System.out.println("method name is : " + method.getMethodName());
                String initMethod = method.getAnnotationAttributes(Bean.class.getName()).get("initMethod").toString();
                System.out.println("initMethod is : " + initMethod);
                BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
                builder.setFactoryMethodOnBean(method.getMethodName(), "config");
                builder.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
                if (initMethod.length() > 0) {
                    builder.setInitMethodName(initMethod);
                }
                AbstractBeanDefinition bd = builder.getBeanDefinition();
                if (configurableListableBeanFactory instanceof DefaultListableBeanFactory beanFactory) {
                    beanFactory.registerBeanDefinition(method.getMethodName(), bd);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

- Mapper后处理器实现

```java
public class MapperPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanFactory) throws BeansException {
        try {
            PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            Resource[] resources = resolver.getResources("com/bytebuf/a05/mapper/**/*.class");
            AnnotationBeanNameGenerator generator = new AnnotationBeanNameGenerator();
            CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
            for (Resource resource : resources) {
                MetadataReader reader = factory.getMetadataReader(resource);
                ClassMetadata classMetadata = reader.getClassMetadata();
                if (classMetadata.isInterface()) {
                    AbstractBeanDefinition bd = BeanDefinitionBuilder.genericBeanDefinition(MapperFactoryBean.class)
                            .addConstructorArgValue(classMetadata.getClassName())
                            .setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE)
                            .getBeanDefinition();
                    AbstractBeanDefinition bd2 = BeanDefinitionBuilder.genericBeanDefinition(classMetadata.getClassName()).getBeanDefinition();
                    String name = generator.generateBeanName(bd2, beanFactory);
                    beanFactory.registerBeanDefinition(name, bd);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {

    }
}

```
