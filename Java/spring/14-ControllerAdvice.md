## InitBinder

- InitBinder的来源有两个
  
  - @ControllerAdvice中的@InitBinder标注的方法，由RequestMappingHandlerAdapter在初始化时解析并记录
  
  - @Controller 中的@InitBinder 标注的方法，由RequestMappingHandlerAdapter 会在控制器方法首次执行时解析并记录

- Method对象的获取利用了缓存来进行加速

- 绑定器工厂的扩展点(advice之一)，通过@InitBinder扩展类型转换器

```java
@Slf4j
public class A24 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
        RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
        handlerAdapter.setApplicationContext(context);
        handlerAdapter.afterPropertiesSet();
        log.debug("1.刚开始");
        showBindMethods(handlerAdapter);
        Method getDataBinderFactory = RequestMappingHandlerAdapter.class.getDeclaredMethod("getDataBinderFactory", HandlerMethod.class);
        getDataBinderFactory.setAccessible(true);
        log.debug("2.模拟调用Controller1的foo方法");
        getDataBinderFactory.invoke(handlerAdapter, new HandlerMethod(new WebConfig.Controller1(), WebConfig.Controller1.class.getMethod("foo")));
        showBindMethods(handlerAdapter);
        log.debug("3. 模拟调用Controller2的bar方法");
        getDataBinderFactory.invoke(handlerAdapter, new HandlerMethod(new WebConfig.Controller2(), WebConfig.Controller2.class.getMethod("bar")));
        showBindMethods(handlerAdapter);
        context.close();
    }
    private static void showBindMethods(RequestMappingHandlerAdapter handlerAdapter) throws Exception {
        Field initBinderAdviceCache = RequestMappingHandlerAdapter.class.getDeclaredField("initBinderAdviceCache");
        initBinderAdviceCache.setAccessible(true);
        Map<ControllerAdviceBean, Set<Method>> globalMap = (Map<ControllerAdviceBean, Set<Method>>) initBinderAdviceCache.get(handlerAdapter);
        log.debug("全局的@InitBinder方法{}", globalMap.values().stream().flatMap(ms -> ms.stream().map(m -> m.getName())).collect(Collectors.toList()));
        Field initBinderCache = RequestMappingHandlerAdapter.class.getDeclaredField("initBinderCache");
        initBinderCache.setAccessible(true);
        Map<Class<?>, Set<Method>> controllerMap = (Map<Class<?>, Set<Method>>) initBinderCache.get(handlerAdapter);
        log.debug("控制器的@InitBinder方法{}", controllerMap.entrySet().stream()
                .flatMap(e -> e.getValue().stream().map(v -> e.getKey().getSimpleName() + "." + v.getName())).collect(Collectors.toList()));

    }
}
@Configuration
public class WebConfig {
    @ControllerAdvice
    static class MyControllerAdvice {
        @InitBinder
        public void binder3(WebDataBinder webDataBinder) {
            webDataBinder.addCustomFormatter(new MyDateFormatter("binder3 转换器"));
        }
    }
    @Controller
    static class Controller1 {
        @InitBinder
        public void binder1(WebDataBinder webDataBinder) {
            webDataBinder.addCustomFormatter(new MyDateFormatter("binder1 转换器"));
        }
        public void foo() {}
    }
    @Controller
    static class Controller2 {
        @InitBinder
        public void binder21(WebDataBinder webDataBinder) {
            webDataBinder.addCustomFormatter(new MyDateFormatter("binder21 转换器"));
        }
        @InitBinder
        public void binder22(WebDataBinder webDataBinder) {
            webDataBinder.addCustomFormatter(new MyDateFormatter("binder22 转换器"));
        }
        public void bar() {}
    }
}
```

## 控制器方法执行流程

- HandlerMethod需要
  
  - bean即是哪个Controller
  
  - method即是Controller中的哪个方法

- ServletInvocableHandlerMethod需要
  
  - WebDataBinderFactory: 负责对象绑定，类型转换
  
  - ParameterNameDiscover: 负责参数名解析
  
  - HandlerMethodArgumentResolverComposite: 负责解析参数
  
  - HandlerMethodReturnValueHandlerComposite: 负责处理返回值

## @ModelAttribute

```java
public class A26 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
        RequestMappingHandlerAdapter adapter = new RequestMappingHandlerAdapter();
        adapter.setApplicationContext(context);
        adapter.afterPropertiesSet();
        MockHttpServletRequest request = new MockHttpServletRequest();
        request.setParameter("name", "zhang");
        //现在可以通过ServletInvocableHandlerMethod把这些整合在一起，并完成控制器方法的调用，如下
        ServletInvocableHandlerMethod handlerMethod = new ServletInvocableHandlerMethod(
                new WebConfig.Controller1(), WebConfig.Controller1.class.getMethod("foo", WebConfig.User.class));
        ServletRequestDataBinderFactory factory = new ServletRequestDataBinderFactory(null, null);
        handlerMethod.setDataBinderFactory(factory);
        handlerMethod.setParameterNameDiscoverer(new DefaultParameterNameDiscoverer());
        handlerMethod.setHandlerMethodArgumentResolvers(argumentResolver(context));
        ModelAndViewContainer container = new ModelAndViewContainer();
        //获取模型工厂方法
        Method getModelFactory = RequestMappingHandlerAdapter.class.getDeclaredMethod("getModelFactory", HandlerMethod.class, WebDataBinderFactory.class);
        getModelFactory.setAccessible(true);
        ModelFactory modelFactory = (ModelFactory) getModelFactory.invoke(adapter, handlerMethod, factory);
        //初始化模型数据
        modelFactory.initModel(new ServletWebRequest(request), container, handlerMethod);
        handlerMethod.invokeAndHandle(new ServletWebRequest(request), container);
        System.out.println(container.getModel());
        handlerMethod.invokeAndHandle(new ServletWebRequest(request), container);
        context.close();
    }
    public static HandlerMethodArgumentResolverComposite argumentResolver(AnnotationConfigApplicationContext context) {
        HandlerMethodArgumentResolverComposite composite = new HandlerMethodArgumentResolverComposite();
        composite.addResolvers(
                new RequestParamMethodArgumentResolver(context.getDefaultListableBeanFactory(), false),
                new PathVariableMethodArgumentResolver(),
                new RequestHeaderMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ServletCookieValueMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ExpressionValueMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ServletRequestMethodArgumentResolver(),
                new ServletModelAttributeMethodProcessor(false),
                new RequestResponseBodyMethodProcessor(List.of(new MappingJackson2HttpMessageConverter())),
                new ServletModelAttributeMethodProcessor(true),
                new RequestParamMethodArgumentResolver(context.getDefaultListableBeanFactory(), true)
        );
        return composite;
    }
}
@Configuration
public class WebConfig {
    @ControllerAdvice
    static class MyControllerAdvice {
        @ModelAttribute("a")
        public String aa () {
            return "aa";
        }
    }
    @Controller
    static class Controller1 {
        @ModelAttribute("b")
        public String bb () {
            return "bb";
        }
        @ResponseStatus(HttpStatus.OK)
        public ModelAndView foo(User user) {
            System.out.println("foo");
            return null;
        }
    }
    @Data
    static class User {
        private String name;
    }
}
```

## 返回值处理器

- 常见的返回值处理器
  
  - ModelAndView, 分别获取其模型和视图名放入ModelAndViewContainer
  
  - 返回值类型为String时，把它当做视图名，放入ModelAndViewContainer
  
  - 返回值添加了@ModelAttribute注解时，将返回值作为模型，放入ModelAndViewContainer:此时需要找到默认视图名
  
  - 返回值类型为ResponseEntity时，此时走MessageConvert,并设置ModelAndViewContainer.requestHandled为true
  
  - 返回值类型为HttpHeaders时，会设置ModelAndViewContainer.requestHandled为true
  
  - 返回值添加了@ResponseBody注解时，此时走MessageConvert,并设置ModelAndViewContainer.requestHandled为true

## MessageConverter

- 作用：@ResponseBody是返回值处理器解析的，但具体转换工作是MessageConvert做的

- 如何选择MediaType: 首先看@RequestMapping上有没有指定；其次看request的Accept头有没有指定；最后按MessageConverter的顺序，谁能谁先转换

```java
public class A28 {
    public static void main(String[] args) throws Exception {
        test1();
        test2();
        test3();
        test4();
    }
    public User user() {
        return null;
    }
    private static void test4() throws Exception {
        MockHttpServletRequest request = new MockHttpServletRequest();
        MockHttpServletResponse response = new MockHttpServletResponse();
        ServletWebRequest webRequest = new ServletWebRequest(request, response);
        //request.addHeader("Accept", "application/xml");
        //response.setContentType("application/json");
        RequestResponseBodyMethodProcessor processor = new RequestResponseBodyMethodProcessor(List.of(
                new MappingJackson2HttpMessageConverter(), new MappingJackson2XmlHttpMessageConverter()
        ));
        processor.handleReturnValue(
                new User("纸箱子", 22),
                new MethodParameter(A28.class.getMethod("user"), -1),
                new ModelAndViewContainer(),
                webRequest
        );
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
    }
    private static void test3() throws IOException {
        MockHttpInputMessage message = new MockHttpInputMessage("""
                {
                    "name": "纸箱子",
                    "age": 30
                }
                """.getBytes(StandardCharsets.UTF_8));
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        if (converter.canWrite(User.class, MediaType.APPLICATION_JSON)) {
            Object read = converter.read(User.class, message);
            System.out.println(read);
        }
    }
    private static void test2() throws IOException {
        MockHttpOutputMessage message = new MockHttpOutputMessage();
        MappingJackson2XmlHttpMessageConverter converter = new MappingJackson2XmlHttpMessageConverter();
        if (converter.canWrite(User.class, MediaType.APPLICATION_XML)) {
            converter.write(new User("纸箱子", 22), MediaType.APPLICATION_XML, message);
            System.out.println(message.getBodyAsString());
        }
    }
    private static void test1() throws IOException {
        MockHttpOutputMessage message = new MockHttpOutputMessage();
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        if (converter.canWrite(User.class, MediaType.APPLICATION_JSON)) {
            converter.write(new User("纸箱子", 22), MediaType.APPLICATION_JSON, message);
            System.out.println(message.getBodyAsString());
        }
    }
    @Data
    public static class User {
        private String name;
        private int age;
        @JsonCreator
        public User(@JsonProperty("name") String name, @JsonProperty("age") int age) {
            this.name = name;
            this.age = age;
        }
    }
}
```

## @ResponseBodyAdvice

- 返回响应体前包装

```java
public class A29 {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
        ServletInvocableHandlerMethod handlerMethod = new ServletInvocableHandlerMethod(
                context.getBean(WebConfig.MyController.class),
                WebConfig.MyController.class.getMethod("user")
        );
        handlerMethod.setDataBinderFactory(new ServletRequestDataBinderFactory(Collections.emptyList(), null));
        handlerMethod.setParameterNameDiscoverer(new DefaultParameterNameDiscoverer());
        handlerMethod.setHandlerMethodArgumentResolvers(argumentResolver(context));
        handlerMethod.setHandlerMethodReturnValueHandlers(returnValueHandler(context));
        MockHttpServletRequest request = new MockHttpServletRequest();
        MockHttpServletResponse response = new MockHttpServletResponse();
        handlerMethod.invokeAndHandle(new ServletWebRequest(request, response), new ModelAndViewContainer());
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
        context.close();
    }
    public static HandlerMethodReturnValueHandlerComposite returnValueHandler(AnnotationConfigApplicationContext context) {
        List<ControllerAdviceBean> annotatedBeans = ControllerAdviceBean.findAnnotatedBeans(context);
        List<Object> collect = annotatedBeans.stream().filter(b -> ResponseBodyAdvice.class.isAssignableFrom(b.getBeanType()))
                .collect(Collectors.toList());
        HandlerMethodReturnValueHandlerComposite composite = new HandlerMethodReturnValueHandlerComposite();
        composite.addHandlers(List.of(
                new ModelAndViewMethodReturnValueHandler(),
                new ViewNameMethodReturnValueHandler(),
                new ServletModelAttributeMethodProcessor(false),
                new HttpEntityMethodProcessor(List.of(new MappingJackson2HttpMessageConverter())),
                new HttpHeadersReturnValueHandler(),
                new RequestResponseBodyMethodProcessor(List.of(new MappingJackson2HttpMessageConverter()), collect),
                new ServletModelAttributeMethodProcessor(true)
        ));
        return composite;
    }
    public static HandlerMethodArgumentResolverComposite argumentResolver(AnnotationConfigApplicationContext context) {
        HandlerMethodArgumentResolverComposite composite = new HandlerMethodArgumentResolverComposite();
        composite.addResolvers(
                new RequestParamMethodArgumentResolver(context.getDefaultListableBeanFactory(), false),
                new PathVariableMethodArgumentResolver(),
                new RequestHeaderMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ServletCookieValueMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ExpressionValueMethodArgumentResolver(context.getDefaultListableBeanFactory()),
                new ServletRequestMethodArgumentResolver(),
                new ServletModelAttributeMethodProcessor(false),
                new RequestResponseBodyMethodProcessor(List.of(new MappingJackson2HttpMessageConverter())),
                new ServletModelAttributeMethodProcessor(true),
                new RequestParamMethodArgumentResolver(context.getDefaultListableBeanFactory(), true)
        );
        return composite;
    }
}
@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Result {
    private int code;
    private String msg;
    private Object data;
    @JsonCreator
    private Result(@JsonProperty("code") int code, @JsonProperty("data") Object data) {
        this.code = code;
        this.data = data;
    }
    private Result(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }
    public static Result ok() {
        return new Result(200, null);
    }
    public static Result ok(Object data) {
        return new Result(200, data);
    }
    public static Result error(String msg) {
        return new Result(500, "服务器内部错误" + msg);
    }
}
@Configuration
public class WebConfig {
    @ControllerAdvice
    static class MyControllerAdvice implements ResponseBodyAdvice<Object> {
        @Override
        public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
            if (returnType.getMethodAnnotation(ResponseBody.class) != null ||
                    AnnotationUtils.findAnnotation(returnType.getContainingClass(), ResponseBody.class) != null) {
                return true;
            }
            return true;
        }
        @Override
        public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
            if (body instanceof Result) {
                return body;
            }
            return Result.ok(body);
        }
    }
    @Controller
    public static class MyController {
        @ResponseBody
        public User user() {
            return new User("纸箱子", 22);
        }
    }
    @Data
    @AllArgsConstructor
    static class User {
        private String name;
        private int age;
    }
}
```

## 异常处理

- 能够重用参数解析器，返回值处理器，实现组件重用

- 能够支持嵌套异常

```java
public class A30 {
    public static void main(String[] args) throws Exception {
        ExceptionHandlerExceptionResolver resolver = new ExceptionHandlerExceptionResolver();
        resolver.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
        resolver.afterPropertiesSet();
        //测试json
        MockHttpServletRequest request = new MockHttpServletRequest();
        MockHttpServletResponse response = new MockHttpServletResponse();
        HandlerMethod handlerMethod = new HandlerMethod(new Controller1(), Controller1.class.getMethod("foo"));
        Exception e = new ArithmeticException("被0除");
        resolver.resolveException(request, response, handlerMethod, e);
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
        //测试mav
        HandlerMethod handlerMethod1 = new HandlerMethod(new Controller2(), Controller2.class.getMethod("foo"));
        Exception e1 = new ArithmeticException("被0除");
        ModelAndView mav = resolver.resolveException(request, response, handlerMethod1, e1);
        System.out.println(mav.getModel());
        System.out.println(mav.getViewName());
        //测试嵌套异常
        HandlerMethod handlerMethod2 = new HandlerMethod(new Controller3(), Controller3.class.getMethod("foo"));
        Exception e2 = new Exception("e1", new RuntimeException("e2", new IOException("e3")));
        resolver.resolveException(request, response, handlerMethod2, e2);
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
        //测试异常处理方法参数解析
        HandlerMethod handlerMethod3 = new HandlerMethod(new Controller4(), Controller4.class.getMethod("foo"));
        Exception e3 = new Exception("e1");
        resolver.resolveException(request, response, handlerMethod3, e3);
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
    }
    static class Controller1 {
        public void foo() {}
        @ExceptionHandler
        @ResponseBody
        public Map<String, Object> handle(ArithmeticException e) {
            return Map.of("error", e.getMessage());
        }
    }
    static class Controller2 {
        public void foo() {}
        @ExceptionHandler
        public ModelAndView handle(ArithmeticException e) {
            return new ModelAndView("test2", Map.of("error", e.getMessage()));
        }
    }
    static class Controller3 {
        public void foo() {}
        @ExceptionHandler
        @ResponseBody
        public Map<String, Object> handle(IOException e) {
            return Map.of("error1", e.getMessage());
        }
    }
    static class Controller4 {
        public void foo() {}
        @ExceptionHandler
        @ResponseBody
        public Map<String, Object> handle(Exception e, HttpServletRequest request) {
            System.out.println(request);
            return Map.of("error2", e.getMessage());
        }
    }
}

```

## @ExceptionHandler

- ExceptionHandlerExceptionResolver初始化会解析@ControllerAdvice中的@ExceptionHandler方法

- ExceptionHandlerExceptionResolver会以类为单位，在该类首次处理异常时，解析此类的@ExceptionHandler方法

- 以上两种@ExceptionHandler的解析结果都会缓存来避免重复解析

```java
@Configuration
public class WebConfig {
    @ControllerAdvice
    static class MyControllerAdvice {
        @ExceptionHandler
        @ResponseBody
        public Map<String, Object> handle(Exception e) {
            return Map.of("error", e.getMessage());
        }
    }
    @Bean
    public ExceptionHandlerExceptionResolver resolver() {
        ExceptionHandlerExceptionResolver resolver = new ExceptionHandlerExceptionResolver();
        resolver.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
        return resolver;
    }
}
public class A31 {
    public static void main(String[] args) throws Exception {
        MockHttpServletRequest request = new MockHttpServletRequest();
        MockHttpServletResponse response = new MockHttpServletResponse();
//        ExceptionHandlerExceptionResolver resolver = new ExceptionHandlerExceptionResolver();
//        resolver.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
//        resolver.afterPropertiesSet();
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
        ExceptionHandlerExceptionResolver resolver = context.getBean(ExceptionHandlerExceptionResolver.class);
        HandlerMethod handlerMethod = new HandlerMethod(new Controller5(), Controller5.class.getMethod("foo"));
        Exception e = new Exception("e1");
        resolver.resolveException(request, response, handlerMethod, e);
        System.out.println(new String(response.getContentAsByteArray(), StandardCharsets.UTF_8));
    }
    static class Controller5 {
        public void foo() {}
    }
}
```

## Tomcat异常处理

- 我们知道@ExceptionHandler只能处理发生在mvc流程中的异常，例如控制器内，拦截器内，那么如果是Filter出现了异常如何进行处理呢

- 在SpringBoot中的实现

- 因为内嵌了Tomcat容器，因此可以配置Tomcat的错误页面，Filter与错误页面之间是通过请求转发跳转的，可以在这里做手脚

- 先通过ErrorPageRegistrarBeanPostProcessor这个后处理器配置错误页面地址，默认为`/error`也可以通过`${server.error.path}`进行配置

- 当Filter发生异常时，不会走Spring流程，但会走Tomcat的错误处理，于是就希望转发至`/error`这个地址，当然，如果没有@ExceptionHandler,那么最终也会走到Tomcat的错误处理

- SpringBoot又提供了一个BasicErrorController, 它就是一个标准的@Controller,@RequestMapping配置为`/error`,所以处理异常的职责就又回到了Spring

- 异常信息由于会被Tomcat放入request作用域，因此BasicErrorController里也能获取到

- BasicErrorController通过Accept头判断需要生成哪种MediaType的响应
  
  - 如果要的不是text/html, 走MessageConverter流程
  
  - 如果需要text/html, 走mvc流程，此时又分两种情况：配置了ErrorViewResolver,根据状态码去找View；没有配置或没找到，用BeanNameViewResovler根据一个固定为error的名字找到View, 即所谓的WhitelabelErrorView

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
        return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
    }
    @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
        return new RequestMappingHandlerMapping();
    }
    @Bean//注意默认的RequestMappingHandlerAdapter不会带jackson转换器
    public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
        RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
        handlerAdapter.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
        return handlerAdapter;
    }
    @Bean//修改了Tomcat服务器默认错误地址
    public ErrorPageRegistrar errorPageRegistrar() {
        return webServerFactory -> webServerFactory.addErrorPages(new ErrorPage("/error"));
    }
    @Bean//TomcatServletWebServerFactory 初始化前用它增强, 注册所有 ErrorPageRegistrar
    public ErrorPageRegistrarBeanPostProcessor errorPageRegistrarBeanPostProcessor() {
        return new ErrorPageRegistrarBeanPostProcessor();
    }
    @Bean
    public BasicErrorController basicErrorController() {
        ErrorProperties errorProperties = new ErrorProperties();
        errorProperties.setIncludeException(true);
        return new BasicErrorController(new DefaultErrorAttributes(), errorProperties);
    }
    @Bean
    public View error() {
        return new View() {
            @Override
            public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
                System.out.println(model);
                response.setContentType("text/html;charset=UTF-8");
                response.getWriter().print("<h3>服务器内部错误</h3>");
            }
        };
    }
    @Bean
    public ViewResolver viewResolver() {
        return new BeanNameViewResolver();
    }
    @Controller
    public static class MyController {
        @RequestMapping("test")
        public ModelAndView test() {
            int i = 1/0;
            return null;
        }
//        @RequestMapping("/error")
//        @ResponseBody
//        public Map<String, Object> error(HttpServletRequest request) {
//            Throwable e = (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
//            return Map.of("error", e.getMessage());
//        }
    }
}

```
