## Tomcat基本结构

```
Server
└───Service
    ├───Connector (协议, 端口)
    └───Engine
        └───Host(虚拟主机 localhost)
            ├───Context1 (应用1, 可以设置虚拟路径, / 即 url 起始路径; 项目磁盘路径, 即 docBase )
            │   │   index.html
            │   └───WEB-INF
            │       │   web.xml (servlet, filter, listener) 3.0
            │       ├───classes (servlet, controller, service ...)
            │       ├───jsp
            │       └───lib (第三方 jar 包)
            └───Context2 (应用2)
                │   index.html
                └───WEB-INF
                        web.xml
```

```java
public class TestTomcat {
    public static void main(String[] args) throws Exception {
        //1. 创建Tomcat对象
        Tomcat tomcat = new Tomcat();
        tomcat.setBaseDir("tomcat");
        //2. 创建项目文件夹，即docBase
        File docBase = Files.createTempDirectory("boot.").toFile();
        docBase.deleteOnExit();
        //3. 创建Tomcat目录，在Tomcat中称为Context
        Context context = tomcat.addContext("", docBase.getAbsolutePath());
        WebApplicationContext applicationContext = getApplicationContext();
        //4.编程添加Servlet
        context.addServletContainerInitializer((set, servletContext) -> {
            HelloServlet helloServlet = new HelloServlet();
            servletContext.addServlet("aaa", helloServlet).addMapping("/hello");
            //DispatcherServlet dispatcherServlet = applicationContext.getBean(DispatcherServlet.class);
            //servletContext.addServlet("dispatcherServlet", dispatcherServlet).addMapping("/");
            for (ServletRegistrationBean registrationBean : applicationContext.getBeansOfType(ServletRegistrationBean.class).values()) {
                registrationBean.onStartup(servletContext);
            }
        }, Collections.emptySet());
        //5.启动Tomcat
        tomcat.start();
        //6. 创建连接器，设置监听端口号
        Connector connector = new Connector(new Http11Nio2Protocol());
        connector.setPort(8080);
        tomcat.setConnector(connector);
    }
    public static WebApplicationContext getApplicationContext() {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(Config.class);
        context.refresh();
        return context;
    }
    @Configuration
    static class Config {
        @Bean
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet) {
            return new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        }
        @Bean
        public DispatcherServlet dispatcherServlet() {
            return new DispatcherServlet();
        }
        @Bean
        public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
            RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
            handlerAdapter.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
            return handlerAdapter;
        }
        @RestController
        static class MyController {
            @GetMapping("/hello2")
            public Map<String, Object> hello2() {
                return Map.of("hello2", "hello2, spring");
            }
        }
    }
}

```
