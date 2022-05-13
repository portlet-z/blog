## BeanNameUrlHandlerMapping与SimpleControllerHandlerAdapter

- BeanNameUrlHandlerMapping,以/开头的bean的名字会被当做映射路径

- 这些bean本身当做handler,要求实现Controller接口

- SimpleControllerHandlerAdapter，调用handler

- 对比
  
  - RequestMappingHandlerAdapter以@RequestMapping作为映射路径
  
  - 控制器方法会被当做handler
  
  - RequestMappingHandlerAdapter，调用hander

```java
public class A33 {
    public static void main(String[] args) {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
    }
}
@Configuration
public class WebConfig {
    @Bean
    public TomcatServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
    @Bean
    public DispatcherServletRegistrationBean servletRegistrationBean(DispatcherServlet dispatcherServlet) {
        DispatcherServletRegistrationBean registrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        registrationBean.setLoadOnStartup(1);
        return registrationBean;
    }
    @Bean
    public BeanNameUrlHandlerMapping beanNameUrlHandlerMapping() {
        return new BeanNameUrlHandlerMapping();
    }
    @Bean
    public SimpleControllerHandlerAdapter simpleControllerHandlerAdapter() {
        return new SimpleControllerHandlerAdapter();
    }
    @Component("/c1")
    public static class Controller1 implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            response.getWriter().print("this is c1");
            return null;
        }
    }
    @Component("/c2")
    public static class Controller2 implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            response.getWriter().print("this is c2");
            return null;
        }
    }
    @Bean("/c3")
    public Controller controller3() {
        return (request, response) -> {
            response.getWriter().print("this is c3");
            return null;
        };
    }
}
public class A33_1 {
    public static void main(String[] args) {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig_1.class);
    }
}
@Configuration
public class WebConfig_1 {
    @Bean
    public TomcatServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
    @Bean
    public DispatcherServletRegistrationBean servletRegistrationBean(DispatcherServlet dispatcherServlet) {
        DispatcherServletRegistrationBean registrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        registrationBean.setLoadOnStartup(1);
        return registrationBean;
    }
    @Component
    static class MyHandlerMapping implements HandlerMapping {
        @Override
        public HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
            String key = request.getRequestURI();
            Controller controller = collect.get(key);
            if (controller == null) {
                return null;
            }
            return new HandlerExecutionChain(controller);
        }
        @Autowired
        private ApplicationContext context;
        private Map<String, Controller> collect;
        @PostConstruct
        public void init() {
            collect = context.getBeansOfType(Controller.class).entrySet()
                    .stream().filter(e -> e.getKey().startsWith("/"))
                    .collect(Collectors.toMap(e -> e.getKey(), e -> e.getValue()));
            System.out.println(collect);
        }
    }
    @Component
    static class MyHandlerAdapter implements HandlerAdapter {
        @Override
        public boolean supports(Object handler) {
            return handler instanceof Controller;
        }
        @Override
        public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            if (handler instanceof Controller controller) {
                controller.handleRequest(request, response);
            }
            return null;
        }
        @Override
        public long getLastModified(HttpServletRequest request, Object handler) {
            return -1;
        }
    }
    @Component("/c1")
    public static class Controller1 implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            response.getWriter().print("this is c1");
            return null;
        }
    }
    @Component("/c2")
    public static class Controller2 implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
            response.getWriter().print("this is c2");
            return null;
        }
    }
    @Bean("/c3")
    public Controller controller3() {
        return (request, response) -> {
            response.getWriter().print("this is c3");
            return null;
        };
    }
}
```

## RouterFunctionMapping与HandlerFunctionAdapter

- RouterFunctionMapping，收集所有RouterFunction,它包括两部分：RequestPredicate设置映射条件，HandlerFunction包含处理逻辑

- 请求到达，根据映射条件找到HandlerFunction,即handler

- HandlerFunctionAdapter,调用handler

```java
@Configuration
public class WebConfig {
    @Bean
    public TomcatServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
    @Bean
    public DispatcherServletRegistrationBean servletRegistrationBean(DispatcherServlet dispatcherServlet) {
        DispatcherServletRegistrationBean registrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        registrationBean.setLoadOnStartup(1);
        return registrationBean;
    }
    @Bean
    public RouterFunctionMapping routerFunctionMapping() {
        return new RouterFunctionMapping();
    }
    @Bean
    public HandlerFunctionAdapter handlerFunctionAdapter() {
        return new HandlerFunctionAdapter();
    }
    @Bean
    public RouterFunction<ServerResponse> r1() {
        return RouterFunctions.route(RequestPredicates.GET("/r1"), request -> ServerResponse.ok().body("this is r1"));
    }
    @Bean
    public RouterFunction<ServerResponse> r2() {
        return RouterFunctions.route(RequestPredicates.GET("/r2"), request -> ServerResponse.ok().body("this is r2"));
    }
}
public class A34 {
    public static void main(String[] args) {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
    }
}
```

## SimpleUrlHandlerMapping与HttpRequestHandlerAdapter

#### 静态资源处理

- SimpleUrlHandlerMapping做映射

- ResourceHttpRequestHandler作为处理器处理静态资源

- HttpRequestHandlerApapter调用处理器

- SimpleUrlHandlerMapping不会在初始化时手机映射信息，需要手动收集

- SimpleUrlHandlerMapping映射路径

- ResourceHttpReqeustHandler作为静态资源Handler

#### 欢迎页处理

- 欢迎页支持静态欢迎页和动态欢迎页

- WelcomePageHandlerMapping映射欢迎页（即只映射'/'）
  
  - 它内置的handler ParameterizableViewController作用是不执行逻辑，仅根据视图名找到视图
  
  - 视图名固定为forward:index.html

- SimpleControllerHandlerAdapter,调用handler
  
  - 转发至/index.html
  
  - 处理/index.html又会走上面的静态资源处理流程

#### HandlerMapping,Handler,HandlerAdapter功能总结

- HandlerMapping负责建立请求与控制器之间的映射关系
  
  - RequestMappingHandlerMapping（与@RequestMapping匹配）
  
  - WelcomePageHandlerMapping(/)
  
  - BeanNameUrlHandlerMapping(与bean的名字匹配，以/开头)
  
  - RouterFunctionMappinig(函数式RequestPredicate, HandlerFunction)
  
  - SimpleUrlHandlerMaping(静态资源通配符 /** /img/**)
  
  - 之间也会有顺序问题,boot中默认顺序如上

- HandlerAdapter负责实现对各种各样的handler的适配调用
  
  - RequestMappingHandlerAdapter处理：@RequestMapping方法
    
    - 参数解析器，返回值处理器体现了组合模式
  
  - SimpleControllerHandlerAdapter处理：Controller接口
  
  - HandlerFunctionAdapter处理：HandlerFunction函数式接口
  
  - HttpRequestHandlerAdapter处理：HttpRequestHandler接口（静态资源处理）
  
  - 这也是典型适配器模式体现

```java
@Configuration
public class WebConfig {
    @Bean
    public TomcatServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
    @Bean
    public DispatcherServletRegistrationBean servletRegistrationBean(DispatcherServlet dispatcherServlet) {
        DispatcherServletRegistrationBean registrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        registrationBean.setLoadOnStartup(1);
        return registrationBean;
    }
    @Bean
    public SimpleUrlHandlerMapping simpleUrlHandlerMapping(ApplicationContext context) {
        SimpleUrlHandlerMapping handlerMapping = new SimpleUrlHandlerMapping();
        Map<String, ResourceHttpRequestHandler> map
                = context.getBeansOfType(ResourceHttpRequestHandler.class);
        handlerMapping.setUrlMap(map);
        return handlerMapping;
    }
    @Bean
    public HttpRequestHandlerAdapter httpRequestHandlerAdapter() {
        return new HttpRequestHandlerAdapter();
    }
    @Bean("/**")
    public ResourceHttpRequestHandler handler1() {
        ResourceHttpRequestHandler handler = new ResourceHttpRequestHandler();
        handler.setLocations(List.of(new ClassPathResource("static/")));
        handler.setResourceResolvers(List.of(
                new CachingResourceResolver(new ConcurrentMapCache("cache1")),
                new EncodedResourceResolver(),
                new PathResourceResolver()
        ));
        return handler;
    }
    @Bean("/img/**")
    public ResourceHttpRequestHandler handler2() {
        ResourceHttpRequestHandler handler = new ResourceHttpRequestHandler();
        handler.setLocations(List.of(new ClassPathResource("images/")));
        return handler;
    }
    @PostConstruct
    public void initGzip() throws IOException {
        Resource resource = new ClassPathResource("static");
        File dir = resource.getFile();
        for (File file : dir.listFiles(pathname -> pathname.getName().endsWith(".html"))) {
            System.out.println(file);
            try(FileInputStream fis = new FileInputStream(file);
                GZIPOutputStream fos = new GZIPOutputStream(new FileOutputStream(file.getAbsoluteFile() + ".gz"))) {
                byte[] bytes = new byte[8 * 1024];
                int len;
                while ((len = fis.read(bytes)) != -1) {
                    fos.write(bytes, 0, len);
                }
            }
        }
    }
}

```
