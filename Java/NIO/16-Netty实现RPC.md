## 消息实体类

```java
@Data
public abstract class Message implements Serializable {
    private int sequenceId;
    private int messageType;
    public abstract int getMessageType();
    public static final int RpcRequestMessage = 101;
    public static final int RpcResponseMessage = 102;
    public static Class<? extends Message> getMessageClass(int messageType) {
        return messageClasses.get(messageType);
    }
    private static final Map<Integer, Class<? extends Message>> messageClasses = new HashMap<>();

    static {
        messageClasses.put(RpcRequestMessage, RpcRequestMessage.class);
        messageClasses.put(RpcResponseMessage, RpcResponseMessage.class);
    }
}
```

### 请求消息体

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class RpcRequestMessage extends Message implements Serializable {
    /**
     * 调用的接口全限定名，服务端根据它找到实现
     */
    private String interfaceName;
    /**
     * 调用接口中的方法
     */
    private String methodName;
    /**
     * 方法返回类型
     */
    private Class<?> returnType;
    /**
     * 方法参数类型数组
     */
    private Class[] parameterTypes;
    /**
     * 方法参数数组
     */
    private Object[] parameterValue;

    public RpcRequestMessage(Integer sequenceId, String interfaceName, String methodName, Class<?> returnType, Class[] parameterTypes, Object[] parameterValue) {
        super.setSequenceId(sequenceId);
        this.interfaceName = interfaceName;
        this.methodName = methodName;
        this.returnType = returnType;
        this.parameterTypes = parameterTypes;
        this.parameterValue = parameterValue;
    }

    @Override
    public int getMessageType() {
        return RpcRequestMessage;
    }
}
```

### 响应消息体

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class RpcResponseMessage extends Message implements Serializable {
    private Object returnValue;
    private Exception exceptionValue;
    @Override
    public int getMessageType() {
        return RpcResponseMessage;
    }
}
```

## Handler

### 编解码Handler

```java
@ChannelHandler.Sharable
@Slf4j
public class MessageCodecSharable extends MessageToMessageCodec<ByteBuf, Message> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Message msg, List outList) throws Exception {
        ByteBuf out = ctx.alloc().buffer();
        // 1. 4字节的魔数
        out.writeInt(0xCAFEBABE);
        // 2. 1字节的版本
        out.writeByte(1);
        // 3. 1字节的序列化方式 jdk 0, json 1, protobuf 2
        out.writeByte(Config.getSerializerAlgorithm().ordinal());
        // 4. 1字节的指令类型
        out.writeByte(msg.getMessageType());
        // 5. 4个字节序列号
        out.writeInt(msg.getSequenceId());
        // 为了凑齐16字节的头，多加一个无效的字节
        out.writeByte(0xff);
        // 6. 获取内容的字节数组
        byte[] bytes = Config.getSerializerAlgorithm().serialize(msg);
        // 7. 内容长度
        out.writeInt(bytes.length);
        // 8. 写入内容
        out.writeBytes(bytes);
        outList.add(out);
    }

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        int magicNumber = in.readInt();
        byte version = in.readByte();
        byte serializedVersion = in.readByte();
        byte messageType = in.readByte();
        int sequenceId = in.readInt();
        in.readByte();
        int contentLength = in.readInt();
        byte[] bytes = new byte[contentLength];
        in.readBytes(bytes, 0, contentLength);
        final Serializer.Algorithm algorithm = Serializer.Algorithm.values()[serializedVersion];
        final Class<?> messageClass = Message.getMessageClass(messageType);
        final Object message = algorithm.deserialize(messageClass, bytes);
        log.debug("magicNumber: {}, version: {}, serializedVersion: {}, messageType: {}, sequenceId: {}, contentLength: {}", magicNumber, version, serializedVersion, messageType, sequenceId, contentLength);
        log.debug("message {}", message);
        out.add(message);
    }
}
```

### 请求处理Handler

```java
public class RpcRequestHandler extends SimpleChannelInboundHandler<RpcRequestMessage> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, RpcRequestMessage message) throws Exception {
        RpcResponseMessage responseMessage = new RpcResponseMessage();
        responseMessage.setSequenceId(message.getSequenceId());
        try {
            final Object service = ServiceFactory.getService(Class.forName(message.getInterfaceName()));
            final Method method = service.getClass().getMethod(message.getMethodName(), message.getParameterTypes());
            final Object invoke = method.invoke(service, message.getParameterValue());
            responseMessage.setMessageType(message.getSequenceId());
            responseMessage.setReturnValue(invoke);
        } catch (Exception e) {
            e.printStackTrace();
            responseMessage.setExceptionValue(e);
        }
        ctx.writeAndFlush(responseMessage);
    }

    public static void main(String[] args) throws Exception {
        RpcRequestMessage message = new RpcRequestMessage(1, "com.bytebuf.netty.rpc.HelloService",
                "sayHello", String.class, new Class[]{String.class}, new Object[] {"aaa"});
        final Object service = ServiceFactory.getService(Class.forName(message.getInterfaceName()));
        final Method method = service.getClass().getMethod(message.getMethodName(), message.getParameterTypes());
        final Object invoke = method.invoke(service, message.getParameterValue());
        System.out.println(invoke);
    }
}
```

### 响应处理Handler

```java
@Slf4j
public class RpcResponseHandler extends SimpleChannelInboundHandler<RpcResponseMessage>  {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, RpcResponseMessage msg) throws Exception {
        log.debug("{}", msg);
    }
}
```

## 业务Service

### 接口

```java
public interface HelloService {
    String sayHello(String name);
}
```

### 实现类

```java
public class HelloServiceImpl implements HelloService {
    @Override
    public String sayHello(String name) {
        return "hello " + name;
    }
}
```

### Service工厂

```java
public class ServiceFactory {
    static Properties properties;
    static Map<Class<?>, Object> map = new ConcurrentHashMap<>();
    static {
        try(InputStream in = Config.class.getResourceAsStream("/application.properties")) {
            properties = new Properties();
            properties.load(in);
            final Set<String> names = properties.stringPropertyNames();
            for (String name : names) {
                if (name.endsWith("Service")) {
                    final Class<?> interfaceClass = Class.forName(name);
                    final Class<?> instanceClass = Class.forName(properties.getProperty(name));
                    map.put(interfaceClass, instanceClass.newInstance());
                }
            }
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public static <T> T getService(Class<T> interfaceClass) {
        return (T) map.get(interfaceClass);
    }
}
```

### 配置类

```java
public abstract class Config {
    static Properties properties;
    static {
        try(InputStream in = Config.class.getResourceAsStream("/application.properties")) {
            properties = new Properties();
            properties.load(in);
        } catch (IOException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    public static int getServerPort() {
        String value = properties.getProperty("server.port");
        if (value == null) {
            return 8080;
        } else {
            return Integer.parseInt(value);
        }
    }
    public static Serializer.Algorithm getSerializerAlgorithm() {
        String value = properties.getProperty("serializer.algorithm");
        if (value == null) {
            return Serializer.Algorithm.Java;
        } else {
            return Serializer.Algorithm.valueOf(value);
        }
    }
}
```

### 配置文件application.properties

```xml
serializer.algorithm=Json
com.bytebuf.netty.rpc.HelloService=com.bytebuf.netty.rpc.HelloServiceImpl
```

### 序列化工具类

```java
/**
 * 用于扩展序列化、反序列化算法
 */
public interface Serializer {
    /**
     * 反序列化算法
     * @param clazz
     * @param bytes
     * @param <T>
     * @return
     */
    <T> T deserialize(Class<T> clazz, byte[] bytes);

    /**
     * 序列化算法
     * @param object
     * @param <T>
     * @return
     */
    <T> byte[] serialize(T object);

    enum Algorithm implements Serializer {
        Java {
            @Override
            @SneakyThrows
            public <T> T deserialize(Class<T> clazz, byte[] bytes) {
                ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bytes));
                return (T) ois.readObject();
            }

            @Override
            @SneakyThrows
            public <T> byte[] serialize(T object) {
                ByteArrayOutputStream bos = new ByteArrayOutputStream();
                ObjectOutputStream oos = new ObjectOutputStream(bos);
                oos.writeObject(object);
                return bos.toByteArray();
            }
        },
        Json {
            @Override
            public <T> T deserialize(Class<T> clazz, byte[] bytes) {
                String json = new String(bytes, StandardCharsets.UTF_8);
                return new GsonBuilder().registerTypeAdapter(Class.class, new ClassCodec()).create().fromJson(json, clazz);
            }
            @Override
            public <T> byte[] serialize(T object) {
                String json = new GsonBuilder().registerTypeAdapter(Class.class, new ClassCodec()).create().toJson(object);
                return json.getBytes(StandardCharsets.UTF_8);
            }
        }
    }
    class ClassCodec implements JsonSerializer<Class<?>>, JsonDeserializer<Class<?>> {
        @Override
        public JsonElement serialize(Class<?> aClass, Type type, JsonSerializationContext jsonSerializationContext) {
            return new JsonPrimitive(aClass.getName());
        }

        @Override
        @SneakyThrows
        public Class<?> deserialize(JsonElement jsonElement, Type type, JsonDeserializationContext jsonDeserializationContext) throws JsonParseException {
            return Class.forName(jsonElement.getAsString());
        }
    }
}
```

## 服务器

```java
@Slf4j
public class RpcServer {
    public static void main(String[] args) {
        NioEventLoopGroup boss = new NioEventLoopGroup(1);
        NioEventLoopGroup worker = new NioEventLoopGroup();
        try {
            final ChannelFuture channelFuture = new ServerBootstrap()
                    .group(boss, worker)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<NioSocketChannel>() {
                        @Override
                        protected void initChannel(NioSocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 12, 4, 0, 0));
                            ch.pipeline().addLast(new LoggingHandler());
                            ch.pipeline().addLast(new MessageCodecSharable());
                            ch.pipeline().addLast(new RpcRequestHandler());
                        }
                    }).bind(8080);
            final Channel channel = channelFuture.sync().channel();
            channel.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
        }
    }
}
```

## 客户端

```java
@Slf4j
public class RpcClient {
    public static void main(String[] args) {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            final Channel channel = new Bootstrap()
                    .group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<NioSocketChannel>() {
                        @Override
                        protected void initChannel(NioSocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 12, 4, 0, 0));
                            ch.pipeline().addLast(new LoggingHandler());
                            ch.pipeline().addLast(new MessageCodecSharable());
                            ch.pipeline().addLast(new RpcResponseHandler());
                        }
                    }).connect("localhost", 8080).sync().channel();
            channel.writeAndFlush(new RpcRequestMessage(1, "com.bytebuf.netty.rpc.HelloService",
                    "sayHello", String.class, new Class[]{String.class}, new Object[] {"aaa"}))
                    .addListener(future -> {
                        if (!future.isSuccess()) {
                            final Throwable cause = future.cause();
                            log.error("error", cause);
                        }
                    });
            channel.closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

## 改造&优化

### 客户端

```java
@Slf4j
public class RpcClient {
    public static void main(String[] args) {
        HelloService service = getProxyService(HelloService.class);
        System.out.println(service.sayHello("zhangsan"));
//        System.out.println(service.sayHello("lisi"));
//        System.out.println(service.sayHello("wangwu"));
    }
    public static <T> T getProxyService(Class<T> serviceClass) {
        ClassLoader loader = serviceClass.getClassLoader();
        Class<?>[] interfaces = new Class[]{serviceClass};
        final Object o = Proxy.newProxyInstance(loader, interfaces, (proxy, method, args) -> {
            // 1. 将方法调用转为消息对象
            final RpcRequestMessage message = new RpcRequestMessage(SequenceIdGenerator.nextId(), serviceClass.getName(), method.getName(), method.getReturnType(), method.getParameterTypes(), args);
            // 2. 将消息对象发送出去
            getChannel().writeAndFlush(message);
            // 3. 返回结果
            final DefaultPromise<Object> promise = new DefaultPromise<>(getChannel().eventLoop());
            RpcResponseHandler.PROMISE_MAP.put(message.getSequenceId(), promise);
            // 4. 等待promise结果
            promise.await();
            if (promise.isSuccess()) {
                return promise.getNow();
            } else {
                throw new RuntimeException(promise.cause());
            }
        });
        return (T) o;
    }
    private static Channel channel = null;
    private static final Object LOCK = new Object();
    public static Channel getChannel() {
        if (channel != null) {
            return channel;
        }
        synchronized (LOCK) {
            if (channel != null) {
                return channel;
            }
            initChannel();
            return channel;
        }
    }
    public static void initChannel() {
        NioEventLoopGroup group = new NioEventLoopGroup();
        final Bootstrap bootstrap = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(1024, 12, 4, 0, 0));
                        ch.pipeline().addLast(new LoggingHandler());
                        ch.pipeline().addLast(new MessageCodecSharable());
                        ch.pipeline().addLast(new RpcResponseHandler());
                    }
                });
        try {
            channel = bootstrap.connect("localhost", 8080).sync().channel();
            channel.closeFuture().addListener(future -> group.shutdownGracefully());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### id生成器

```java
public class SequenceIdGenerator {
    private static final AtomicInteger id = new AtomicInteger();
    public static int nextId() {
        return id.incrementAndGet();
    }
}
```

### 响应处理器

```java
@Slf4j
public class RpcResponseHandler extends SimpleChannelInboundHandler<RpcResponseMessage>  {
    public static final Map<Integer, Promise<Object>> PROMISE_MAP = new ConcurrentHashMap<>();
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, RpcResponseMessage msg) throws Exception {
        log.debug("{}", msg);
        final Promise<Object> promise = PROMISE_MAP.get(msg.getSequenceId());
        if (promise != null) {
            final Object returnValue = msg.getReturnValue();
            final Exception exceptionValue = msg.getExceptionValue();
            if (exceptionValue != null) {
                promise.setFailure(exceptionValue);
            } else {
                promise.setSuccess(returnValue);
            }
        }
    }
}
```

### 请求处理器

```java
public class RpcRequestHandler extends SimpleChannelInboundHandler<RpcRequestMessage> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, RpcRequestMessage message) throws Exception {
        RpcResponseMessage responseMessage = new RpcResponseMessage();
        responseMessage.setSequenceId(message.getSequenceId());
        try {
            final Object service = ServiceFactory.getService(Class.forName(message.getInterfaceName()));
            final Method method = service.getClass().getMethod(message.getMethodName(), message.getParameterTypes());
            final Object invoke = method.invoke(service, message.getParameterValue());
            responseMessage.setMessageType(message.getSequenceId());
            responseMessage.setReturnValue(invoke);
        } catch (Exception e) {
            e.printStackTrace();
            String msg = e.getCause().getMessage();
            responseMessage.setExceptionValue(new Exception("远程调用出错: " + msg));
        }
        ctx.writeAndFlush(responseMessage);
    }
}
```

