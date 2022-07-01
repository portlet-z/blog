当浏览器发送一个请求http://localhost:8080/hello后，请求到达服务器，器处理流程是：

1. 服务器提供了DispatcherServlet,它使用的是标准的Servlet技术
   
   - 路径：默认映射路径为`/`,即会匹配到所有请求URL,可作为请求的统一入口，也被称之为前控制器
     
     - jsp不会匹配到DispatcherServlet
     
     - 其他有路径的Servlet匹配优先级也高于DispatcherServlet
   
   - 创建：在Boot中，由DispatcherServletAutoConfiguation这个自动配置类提供DispatcherServlet的bean
   
   - 初始化：DispatcherServlet初始化时会优先到容器里寻找各种组件，作为它的成员变量
     
     - HandlerMapping,初始时记录映射关系
     
     - HandlerAdapter,初始化时准备参数解析器，返回值处理器，消息转换器
     
     - HandlerExceptionResolver,初始化时准备参数解析器，返回值处理器，消息转换器
     
     - ViewResolver

2. DispatcherServlet会利用RequestMappingHandlerMapping查找控制器方法
   
   - 例如根据/hello路径找到@RequestMapping("/hello")对应的控制器方法
   
   - 控制器方法会被封装为HandlerMethod对象，并结合匹配到的拦截器一起返回给DispatcherServlet
   
   - HandlerMethod和拦截器合在一起被称为HandlerExecutionChain(调用链)对象

3. DispatcherServlet接下来会：
   
   - 调用拦截器的preHandle方法
   
   - RequestMappingHandlerAdapter调用handle方法，准备数据绑定工厂，模型工厂，ModelAndViewContainer, 将HandlerMethod完善为ServletInvocableHandlerMethod
     
     - @ControllerAdvice全局增强点：补充数据模型，补充自定义类型转换器，RequestBody增强，ResponseBody增强
     
     - 使用HandlerMethodArgumentResovler准备参数
     
     - 使用HandlerMethodReturnValueHandler处理返回值
     
     - 根据ModelAndViewContainer获取ModelAndView.如果返回的ModelAndView为null,不走视图解析以及渲染流程。例如：有的返回值处理器调用了HttpMessageConverter来将结果转换为JSON,这时ModelAndView为null.如果返回的ModelAndView不为null,就会走视图解析及渲染流程
   
   - 调用拦截器的postHandle方法
   
   - 处理异常或视图渲染
     
     - 如果上述3个步骤出现异常，走ExecutionHandlerExceptionResolver处理异常流程。@ControllerAdvice全局增强点：@ExceptionHandler异常处理
     
     - 正常，走试图解析及渲染流程
   
   - 调用拦截器的afterCompletion方法
