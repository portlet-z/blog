ChannelHandler用来处理Channel上的各种事件，分为入站、出战两种。所有ChannelHandler被连成一串，就是Pipeline

- 入站处理器通常是ChannelInboundHandlerAdapter的子类，主要用来读取客户端数据，写回结果
- 出战处理器通常是ChannelOutboundHandlerAdapter的子类，主要对写回结果进行加工

打个比喻，每个Channel是一个产品的加工车间，Pipeline是车间中的流水线，ChannelHandler就是流水线上的各道工序，而后面要讲的ByteBuf是原材料，经过很多工序的加工：先经过一道道入站工序，再经过一道道出战工序最终变成产品

```java
@Slf4j
public class TestPipeline {
    public static void main(String[] args) {
        new ServerBootstrap()
                .group(new NioEventLoopGroup())
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel channel) throws Exception {
                        ChannelPipeline pipeline = channel.pipeline();
                        // head -> h1 -> h2 -> h3 -> h4 -> h5 -> h6 -> tail
                        pipeline.addLast("handler1", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                log.debug("1");
                                ByteBuf buf = (ByteBuf) msg;
                                String name = buf.toString(Charset.defaultCharset());
                                super.channelRead(ctx, name);
                            }
                        });
                        pipeline.addLast("handler2", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                log.debug("2");
                                Student student = new Student(msg.toString());
                                super.channelRead(ctx, student);
                            }
                        });
                        pipeline.addLast("handler3", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                log.debug("3, 结果: {}, class: {}", msg, msg.getClass());
                                super.channelRead(ctx, msg);
                                channel.writeAndFlush(Charset.defaultCharset().encode("server..."));
                            }
                        });
                        pipeline.addLast("handler4", new ChannelOutboundHandlerAdapter() {
                            @Override
                            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                log.debug("4");
                                super.write(ctx, msg, promise);
                            }
                        });
                        pipeline.addLast("handler5", new ChannelOutboundHandlerAdapter() {
                            @Override
                            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                log.debug("5");
                                super.write(ctx, msg, promise);
                            }
                        });
                        pipeline.addLast("handler6", new ChannelOutboundHandlerAdapter() {
                            @Override
                            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                                log.debug("6");
                                super.write(ctx, msg, promise);
                            }
                        });
                    }
                })
                .bind(8080);
    }
}
@Data
@AllArgsConstructor
class Student {
    private String name;
}
```

打印结果

```
21:47:28.055 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 1
21:47:28.055 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 2
21:47:28.055 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 3, 结果: Student(name=1), class: class com.bytebuf.netty.Student
21:47:28.070 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 6
21:47:28.070 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 5
21:47:28.070 [nioEventLoopGroup-2-2] DEBUG com.bytebuf.netty.TestPipeline - 4
```

- channel.writeAndFlush是从tail节点开始往前找出站handler
- ChannelHandlerContext.writeAndFlush是从当前节点往前找出站handler

EmbededChannel用来测试入站出站handler的类

```java
@Slf4j
public class TestEmbeddedChannel {
    public static void main(String[] args) {
        ChannelInboundHandlerAdapter h1 = new ChannelInboundHandlerAdapter() {
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                log.debug("1");
                super.channelRead(ctx, msg);
            }
        };
        ChannelInboundHandlerAdapter h2 = new ChannelInboundHandlerAdapter() {
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                log.debug("2");
                super.channelRead(ctx, msg);
            }
        };
        ChannelInboundHandlerAdapter h3 = new ChannelInboundHandlerAdapter() {
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                log.debug("3");
                super.channelRead(ctx, msg);
            }
        };
        ChannelOutboundHandlerAdapter h4 = new ChannelOutboundHandlerAdapter() {
            @Override
            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                log.debug("4");
                super.write(ctx, msg, promise);
            }
        };
        ChannelOutboundHandlerAdapter h5 = new ChannelOutboundHandlerAdapter() {
            @Override
            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                log.debug("5");
                super.write(ctx, msg, promise);
            }
        };
        ChannelOutboundHandlerAdapter h6 = new ChannelOutboundHandlerAdapter() {
            @Override
            public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                log.debug("6");
                super.write(ctx, msg, promise);
            }
        };
        EmbeddedChannel channel = new EmbeddedChannel(h1, h2, h3, h4, h5, h6);
        // 模拟入站操作
        channel.writeInbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("hello".getBytes()));
        // 模拟出站操作
        channel.writeOutbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("world".getBytes()));
    }
}
```

