AIO用来解决数据复制阶段的阻塞问题

- 同步意味着，在进行读写操作时，线程需要等待结果，还是相当于闲置
- 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统通过回调方式由另外的线程来获得结果

> 异步模型需要底层操作系统(Kernel)提供支持
>
> Windows系统通过IOCP实现了真正的异步IO
>
> Linux系统异步IO在2.6版本引入，但其实底层还是由多路复用模拟了异步IO,性能没有优势

## 文件AIO

```java
@Slf4j
public class AioFileChannel {
    public static void main(String[] args) throws IOException {
        try (AsynchronousFileChannel channel = AsynchronousFileChannel.open(Paths.get("data.txt"), StandardOpenOption.READ)) {
            ByteBuffer buffer = ByteBuffer.allocate(16);
            log.debug("read begin...");
            channel.read(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    log.debug("read completed...");
                    attachment.flip();
                    debugRead(attachment);
                }
                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    exc.printStackTrace();
                }
            });
            log.debug("read end...");
        } catch (IOException e) {
        }
        System.in.read();
    }
}
```

输出结果

```
22:24:42.521 [main] DEBUG com.bytebuf.nio.AioFileChannel - read begin...
22:24:42.537 [main] DEBUG com.bytebuf.nio.AioFileChannel - read end...
22:24:42.537 [Thread-16] DEBUG com.bytebuf.nio.AioFileChannel - read completed...
22:24:42.537 [Thread-16] DEBUG io.netty.util.internal.logging.InternalLoggerFactory - Using SLF4J as the default logging framework
+--------+-------------------- read -----------------------+----------------+
position: [0], limit: [13]
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 30 31 32 33 34 35 36 37 38 39 61 62 63          |0123456789abc   |
+--------+-------------------------------------------------+----------------+
```

可以看到

- 相应文件读取成功的是另一个线程Thread-16
- 主线程并没有IO阻塞

守护线程

默认文件AIO使用的线程都是守护线程，所以最后要执行`System.in.read();`以避免守护线程意味结束

## 网络AIO

```java
@Slf4j
public class AioServer {
    public static void main(String[] args) throws IOException {
        AsynchronousServerSocketChannel ssc = AsynchronousServerSocketChannel.open();
        ssc.bind(new InetSocketAddress(8080));
        ssc.accept(null, new AcceptHandler(ssc));
        System.in.read();
    }
    private static void closeChannel(AsynchronousSocketChannel sc) {
        try {
            log.debug("{} {} close", Thread.currentThread().getName(), sc.getRemoteAddress());
            sc.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    private static class ReadHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel sc;
        private ReadHandler(AsynchronousSocketChannel sc) {
            this.sc = sc;
        }
        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            try {
                if (result == -1) {
                    closeChannel(sc);
                    return;
                }
                log.debug("{} {} read", Thread.currentThread().getName(), sc.getRemoteAddress());
                attachment.flip();
                log.debug("{}", Charset.defaultCharset().decode(attachment));
                attachment.clear();
                // 处理完第一个read时，需要再次调用read方法来处理下一个read事件
                sc.read(attachment, attachment, this);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
            closeChannel(sc);
            exc.printStackTrace();
        }
    }
    private static class WriteHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel sc;
        private WriteHandler(AsynchronousSocketChannel sc) {
            this.sc = sc;
        }
        @Override
        public void completed(Integer result, ByteBuffer attachment) {
            // 如果作为附件的buffer还有内容，需要再次write写出剩余内容
            if (attachment.hasRemaining()) {
                sc.write(attachment);
            }
        }
        @Override
        public void failed(Throwable exc, ByteBuffer attachment) {
            exc.printStackTrace();
            closeChannel(sc);
        }
    }
    private static class AcceptHandler implements CompletionHandler<AsynchronousSocketChannel, Object> {
        private final AsynchronousServerSocketChannel ssc;
        private AcceptHandler(AsynchronousServerSocketChannel ssc) {
            this.ssc = ssc;
        }
        @SneakyThrows
        @Override
        public void completed(AsynchronousSocketChannel sc, Object attachment) {
            log.debug("{} {} connected", Thread.currentThread().getName(), sc.getRemoteAddress());
            ByteBuffer buffer = ByteBuffer.allocate(16);
            // 读事件由ReadHandler处理
            sc.read(buffer, buffer, new ReadHandler(sc));
            // 写事件由WriteHandler处理
            sc.write(Charset.defaultCharset().encode("server hello!"), ByteBuffer.allocate(16), new WriteHandler(sc));
            // 处理完第一个accept时，需要再次调用accept方法来处理下一个accept事件
            ssc.accept(null, this);
        }
        @Override
        public void failed(Throwable exc, Object attachment) {
            exc.printStackTrace();
        }
    }
}
```

