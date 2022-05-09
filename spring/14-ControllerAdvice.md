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


