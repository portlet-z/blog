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
