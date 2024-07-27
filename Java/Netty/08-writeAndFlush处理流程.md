# Pipeline事件传播回顾

- writeAndFlush是如何触发事件传播的？数据是怎样写到Socket底层的？
- 为什么会由write和flush两个动作？执行flush之前数据是如何存储的？
- writeAndFlush是同步还是异步？它是线程安全的吗？

# writeAndFlush事件传播分析

```java
public class EchoServer {
    public void startEchoServer(int port) throws Exception{
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            new ServerBootstrap()
                    .group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<NioSocketChannel>() {
                        @Override
                        protected void initChannel(NioSocketChannel channel) throws Exception {
                            channel.pipeline().addLast(new FixedLengthFrameDecoder(10));
                            channel.pipeline().addLast(new ResponseSampleEncoder());
                            channel.pipeline().addLast(new RequestSampleHandler());
                            //channel.pipeline().addLast(new EchoServerHandler());
                        }
                    })
                    .bind(port)
                    .sync()
                    .channel()
                    .closeFuture()
                    .sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception{
        new EchoServer().startEchoServer(8088);
    }
}
public class RequestSampleHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String data = ((ByteBuf)msg).toString(CharsetUtil.UTF_8);
        ResponseSample response = new ResponseSample("OK", data, System.currentTimeMillis());
        ctx.channel().writeAndFlush(response);
    }
}
public class ResponseSampleEncoder extends MessageToByteEncoder<ResponseSample> {
    @Override
    protected void encode(ChannelHandlerContext ctx, ResponseSample msg, ByteBuf out) throws Exception {
        if (msg != null) {
            out.writeBytes(msg.getCode().getBytes(StandardCharsets.UTF_8));
            out.writeBytes(msg.getData().getBytes(StandardCharsets.UTF_8));
            out.writeLong(msg.getTimestamp());
        }
    }
}
@Data
@AllArgsConstructor
public class ResponseSample {
    private String code;
    private String data;
    private Long timestamp;
}
```

![](./images/writeAndFlush示例.png)

# writeAndFlush核心逻辑

```java
private void write(Object msg, boolean flush, ChannelPromise promise) {
  // 省略部分非核心代码
  // 找到Pipeline链表中下一个Outbound类型的ChannelHandler节点
  final AbstractChannelHandlerContext next = findContextOutbound(flush ?
                (MASK_WRITE | MASK_FLUSH) : MASK_WRITE);
  final Object m = pipeline.touch(msg, next);
  EventExecutor executor = next.executor();
  //判断当前线程是否是NioEventLoop中的线程
  if (executor.inEventLoop()) {
    if (flush) {
      //因为flush=true，所以流程走到这里
      next.invokeWriteAndFlush(m, promise);
    } else {
      next.invokeWrite(m, promise);
    }
  } else {
    final WriteTask task = WriteTask.newInstance(next, m, promise, flush);
    if (!safeExecute(executor, task, promise, m, !flush)) {
      task.cancel();
    }
  }
}

void invokeWriteAndFlush(Object msg, ChannelPromise promise) {
  if (invokeHandler()) {
    invokeWrite0(msg, promise);
    invokeFlush0();
  } else {
    writeAndFlush(msg, promise);
  }
}

private void invokeWrite0(Object msg, ChannelPromise promise) {
  try {
    ((ChannelOutboundHandler) handler()).write(this, msg, promise);
  } catch (Throwable t) {
    notifyOutboundHandlerException(t, promise);
  }
}

// HeadContext # write
@Override
public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
  unsafe.write(msg, promise);
}

// AbstractChannel # AbstractUnsafe # write
@Override
public final void write(Object msg, ChannelPromise promise) {
  assertEventLoop();
  ChannelOutboundBuffer outboundBuffer = this.outboundBuffer;
  if (outboundBuffer == null) {
    try {
      ReferenceCountUtil.release(msg);
    } finally {
      safeSetFailure(promise,
                     newClosedChannelException(initialCloseCause, "write(Object, ChannelPromise)"));
    }
    return;
  }
  int size;
  try {
    // 过滤消息
    msg = filterOutboundMessage(msg);
    size = pipeline.estimatorHandle().size(msg);
    if (size < 0) {
      size = 0;
    }
  } catch (Throwable t) {
    try {
      ReferenceCountUtil.release(msg);
    } finally {
      safeSetFailure(promise, t);
    }
    return;
  }
  // 向Buffer中添加数据
  outboundBuffer.addMessage(msg, size, promise);
}

public void addMessage(Object msg, int size, ChannelPromise promise) {
  Entry entry = Entry.newInstance(msg, size, total(msg), promise);
  if (tailEntry == null) {
    flushedEntry = null;
  } else {
    Entry tail = tailEntry;
    tail.next = entry;
  }
  tailEntry = entry;
  if (unflushedEntry == null) {
    unflushedEntry = entry;
  }
  incrementPendingOutboundBytes(entry.pendingSize, false);
}
```

# 写Buffer队列

![](./images/写Buffer队列.png)

# 刷新Buffer队列

## doWrite方法的处理流程主要分为三步

- 第一，根据配置获取自旋锁的次数writeSpinCount.这个自旋锁的次数主要是用来干什么的呢？
  - 自旋锁的次数相当于控制一次写入数据的最大循环执行次数，如果超过所设置的自旋锁次数，那么写操作将会被暂时中断
- 第二，根据自旋锁次数重复调用doWriteInternal方法发送数据，每成功发送一次数据，自旋锁的次数writeSpinCount减1，当writeSpinCount耗尽，那么doWrite操作将会被暂时中断
  - 在于删除缓存中的链表节点以及调用底层API发送数据
- 第三，调用incompleteWrite方法确保数据能够全部发送出去，因为自旋锁次数的限制，数据没有写完，需要继续OP_WRITE事件
  - 如果数据已经写完，清楚OP_WRITE事件即可

```mermaid
sequenceDiagram
participant NioSocketChannel
participant DefaultChannelPipeline
participant TailContext
participant DefaultChannelHandlerContext
participant NioEventLoop
participant WriteAndFlush
participant StringEncoder
participant HeadContext
participant NioSocketChannelUnsafe 
NioSocketChannel -->> NioSocketChannel : writeAndFlush
NioSocketChannel ->> DefaultChannelPipeline : writeAndFlush
DefaultChannelPipeline ->> TailContext : writeAndFlush
TailContext ->> TailContext : write
TailContext ->> TailContext : findContextOutbound
TailContext ->> TailContext : safeExecute
TailContext ->> NioEventLoop : execute
NioEventLoop ->> NioEventLoop : runAllTasks
NioEventLoop ->> NioEventLoop : safeExecute
NioEventLoop ->> WriteAndFlush : run
WriteAndFlush ->> WriteAndFlush : write
WriteAndFlush ->> DefaultChannelHandlerContext : invokeWrite
DefaultChannelHandlerContext ->> DefaultChannelHandlerContext : invokeWrite0
DefaultChannelHandlerContext ->> StringEncoder : write
StringEncoder ->> DefaultChannelHandlerContext : write
DefaultChannelHandlerContext ->> DefaultChannelHandlerContext : findContextOutbound
DefaultChannelHandlerContext ->> HeadContext : invokeWrite
HeadContext ->> HeadContext : invokeWrite0
HeadContext ->> HeadContext : write
HeadContext ->> NioSocketChannelUnsafe : write
WriteAndFlush ->> HeadContext : invokeFlush
HeadContext ->> HeadContext : invokeFlush0
HeadContext ->> HeadContext : flush
HeadContext ->> NioSocketChannelUnsafe : flush
NioSocketChannelUnsafe ->> NioSocketChannelUnsafe : flush0
NioSocketChannelUnsafe ->> NioSocketChannelUnsafe : doWrite
```

# writeAndFlush的处理流程

- writeAndFlush属于出站操作，从Pipeline的Tail节点开始进行事件传播，一直向前传播到Head节点
- write方法并没有将数据写入Socket缓冲区，只是将数据写入到ChannelOutboundBuffer缓存中
- flush方法才最终将数据写入到Socket缓冲区

# 总结

- Channel和ChannelHandlerContext都有writeAndFlush方法，他们之间的区别是什么？
