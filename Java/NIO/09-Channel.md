## 作用

- close()可以用来关闭channel
- closeFuture()用来处理channel的关闭
  - sync方法作用的是同步等待channel关闭
  - 而addListener方法是异步等待channel关闭
- pipeline()方法添加处理器
- write()方法将数据写入
- writeAndFlush()方法将数据写入并刷出

```java
@Slf4j
public class EventLoopClient {
    public static void main(String[] args) {
        ChannelFuture channelFuture = new Bootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                // 1. 连接到服务器
                // 异步非阻塞，main发起了调用，真正执行connect的是nio线程
                .connect(new InetSocketAddress("localhost", 8080));
        // 使用sync方法同步处理结果
        // channelFuture.sync();
        // 无阻塞向下执行获取channel
        // Channel channel = channelFuture.channel();
        // log.debug("{}", channel);
        // 向服务器发送数据
        // channel.writeAndFlush("hello");

        // 使用addListener（回调对象）方法异步处理结果
        channelFuture.addListener(new ChannelFutureListener() {
            @Override
            // 在nio线程连接建立好之后，会调用operationComplete
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                Channel channel = channelFuture.channel();
                log.debug("{}", channel);
                channel.writeAndFlush("hello");
            }
        });
    }
}
```

## 优雅的处理关闭

```java
@Slf4j
public class CloseFutureClient {
    public static void main(String[] args) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        ChannelFuture channelFuture = new Bootstrap()
                .group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new LoggingHandler());
                        ch.pipeline().addLast(new StringEncoder());
                    }
                })
                .connect(new InetSocketAddress("localhost", 8080));
        Channel channel = channelFuture.sync().channel();
        new Thread(() -> {
            Scanner scanner = new Scanner(System.in);
            while (true) {
                String line = scanner.nextLine();
                if ("q".equals(line)) {
                    channel.close();
                    break;
                }
                channel.writeAndFlush(line);
            }
        }, "input").start();
        // 获取CloseFuture对象，1同步处理结果 2异步处理结果
        ChannelFuture closeFuture = channel.closeFuture();
//        log.debug("waiting close...");
//        closeFuture.sync();
//        log.debug("处理关闭之后的操作");
//        group.shutdownGracefully();
        // 异步处理
        closeFuture.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture channelFuture) throws Exception {
                log.debug("处理关闭之后的操作");
                group.shutdownGracefully();
            }
        });
    }
}
```

