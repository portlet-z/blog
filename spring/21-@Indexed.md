- 代码编译时根据@Indexed生成META-INF/spring.components文件

- 扫描时：
  
  - 如果发现META-INF/spring.components存在，以它为准加载bean definition
  
  - 否则，会遍历包下所有class资源（包括jar内的）

- pom.xml添加以下依赖

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-indexer</artifactId>
            <optional>true</optional>
        </dependency>
```

```java

```
