## DispatcherServlet初始化时机

- DispatcherServlet是在第一次被访问时进行初始化，也可以通过spring.mvc.servlet.load-on-startup=1配置，Tomcat启动时就初始化

- 在初始化DispatcherServlet时会从Spring容器中找一些Web需要的组件，如HandlerMapping, HandlerAdapter等

## DispatcherServlet初始化都做了什么

## RequestMappingHandlerMapping基本用途

- 以@RequestMapping作为映射路径

## RequestMappingHandlerAdapter基本用途

- 调用handler

- 控制器的具体方法会被当做handler
  
  - handler的参数和返回值多种多样
  
  - 需要解析方法参数，由HandlerMethodArgumentResolver来做
  
  - 需要处理方法返回值，由HandlerMethodReturnValueHandler来做

## 自定义参数和返回值处理器

- WebConfig.java

```java
@Configuration
@ComponentScan
@PropertySource("classpath:application.properties")
@EnableConfigurationProperties({WebMvcProperties.class, ServerProperties.class})
public class WebConfig {
    //内嵌Web容器工程
    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    //创建DispatcherServlet
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
    //注册DispatcherServlet, Spring MVC的入口
    @Bean
    public DispatcherServletRegistrationBean dispatcherServletRegistrationBean(
            DispatcherServlet dispatcherServlet, WebMvcProperties webMvcProperties) {
        DispatcherServletRegistrationBean registrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        //load-on-startup > 0启动时初始化DispatcherServlet, 默认为-1，第一次访问是初始化DispatcherServlet
        registrationBean.setLoadOnStartup(webMvcProperties.getServlet().getLoadOnStartup());
        return registrationBean;
    }
    //以下为@EnableMvc导入的bean(注意不加@EnableWebMvc DispatcherServlet初始化还是会添加默认的组件，但并不作为)
    //1. 加入RequestMappingHandlerMapping
    @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
        return new RequestMappingHandlerMapping();
    }
    @Bean
    public MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter() {
        MyRequestMappingHandlerAdapter handlerAdapter = new MyRequestMappingHandlerAdapter();
        TokenArgumentResolver tokenArgumentResolver = new TokenArgumentResolver();
        YmlReturnValueHandler ymlReturnValueHandler = new YmlReturnValueHandler();
        handlerAdapter.setCustomArgumentResolvers(List.of(tokenArgumentResolver));
        handlerAdapter.setCustomReturnValueHandlers(List.of(ymlReturnValueHandler));
        return handlerAdapter;
    }
    //2. 演示RequestMappingHandlerMapping如何获得所有的映射路径
    //3. 继续加入RequestMappingHandlerAdapter,会替换掉DispatcherServlet默认的4个HandlerAdapter
    //4. 演示RequestMappingHandlerAdapter初始化后，有哪些参数，返回值处理器
    //4.1 创建自定义参数处理器
    //4.2 创建自定义返回值处理器
}
```

- Yml.java, Token.java, MyRequestingMappingHandlerAdapter.java

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Yml {
}
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Token {
}
@Component
public class MyRequestMappingHandlerAdapter extends RequestMappingHandlerAdapter {
    @Override
    public ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
        return super.invokeHandlerMethod(request, response, handlerMethod);
    }
}
```

- TokenArgumentResolver.java, YmlReturnValueHandler.java

```java
public class TokenArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        Token token = parameter.getParameterAnnotation(Token.class);
        return token != null;
    }
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return webRequest.getHeader("token");
    }
}
public class YmlReturnValueHandler implements HandlerMethodReturnValueHandler {
    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        Yml yml = returnType.getMethodAnnotation(Yml.class);
        return yml != null;
    }
    @Override
    public void handleReturnValue(Object returnValue, MethodParameter returnType,
                                  ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
        //1.转换返回结果为yaml字符串
        String str = new Yaml().dump(returnValue);
        //2.将yaml字符串写入响应
        HttpServletResponse response = webRequest.getNativeResponse(HttpServletResponse.class);
        response.setContentType("text/plain;charset=utf-8");
        response.getWriter().print(str);
        //3.设置请求已经处理完毕
        mavContainer.setRequestHandled(true);
    }
}
```

- Controller1.java, A20.java

```java
@Controller
public class Controller1 {
    @GetMapping("/test1")
    public ModelAndView test1() {
        System.out.println("test1()");
        return null;
    }
    @GetMapping("/test2")
    public ModelAndView test2(@RequestParam("name") String name) {
        System.out.println("test2 " + name);
        return null;
    }
    @GetMapping("/test3")
    public ModelAndView test3(@Token String token) {
        System.out.println("test3 " + token);
        return null;
    }
    @GetMapping("/test4")
    @Yml
    public User test4() {
        System.out.println("test4");
        return new User("张三", 22);
    }
    @Data
    @AllArgsConstructor
    public static class User {
        private String name;
        private int age;
    }
}

```
