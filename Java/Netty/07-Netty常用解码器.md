# 固定长度解码器FixedLengthFrameDecoder

- 直接通过构造函数设置固定长度大小的frameLength
- 如果累积读取到长度大小为frameLength的消息，那么解码器认为已经获取到了一个完整的消息
- 如果消息长度小于frameLength解码器会一直等后续数据包到达，直至获得完整的消息

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
                            channel.pipeline().addLast(new FixedLengthFrameDecoder(20));
                            channel.pipeline().addLast(new EchoServerHandler());
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
/**
* telnet 127.0.0.1 8088
* 12345678901234567890123
*/
/**控制台打印
*Receive client:[12345678901234567890]
*/
```

# 特殊分隔符解码器DelimiterBasedFrameDecoder

- delimiters: 指定特殊分隔符，通过写入ByteBuf作为参数传入delimiters的类型是ByteBuf数组，所以我们可以同时指定多个分隔符，但是最终会选择长度最短的分隔符进行消息拆分
- maxLength: 是报文长度的最大限制，如果超过maxLength还没有检测到指定分隔符，将会抛出TooLongFramException
- failFast: 可以控制TooLongFrameException的时机
  - 如果failFast=true, 在超出maxLength会立即抛出不再继续进行编码
  - 如果failFast=false,在解码出一个完整的消息后才会抛出TooLongFrameException
- stripDelimiter: 作用是判断解码后得到的消息是否去除分隔符

```java
channel.pipeline().addLast(new DelimiterBasedFrameDecoder(10, true, true,  Unpooled.wrappedBuffer(new byte[]{'#'})));
/**
* 123#321#456
*/
/**
Receive client:[123]
Receive client:[321]
*/
```

# 长度域解码器LengthFieldBasedFrameDecoder

- 是解决TCP拆包/粘包最常用的解码器，它基本上可以覆盖大部分基于长度拆包场景
- 开源消息中间件RocketMQ就是使用LengthFiledBasedFrameDecoder进行解码的

```java
//长度域解码器特有属性
//长度字段的偏移量，也就是存放长度数据的起始位置
private final int lengthFieldOffset;
//长度字段所占用的字节数
private final int lengthFieldLength;
//消息长度的修正值
//在很多较为复杂的一些协议设计中，长度域不仅仅包含消息的长度，而且包含其他的数据，如版本号，数据类型，数据状态等，那么这时候我们需要使用lengthAdustment进行修正
//lengthAdustment = 包体的长度值 - 长度域的值
private final int lengthAdjustment;
//解码后需要跳过的初始字节数，也就是消息内容字段的起始位置
private final int initialBytesToStrip;
//长度字段结束的偏移量，lengthFiledEndOffset = lengthFieldOffset+lengthFieldLength
private final int lengthFieldEndOffset;

//与固定长度解码器和特定分隔符解码器相似的属性
//报文最大限制长度
private final int maxFrameLength;
//是否立即抛出TooLongFrameException?与maxFrameLength搭配使用
private final boolean failFast;
//是否处于丢弃模式
private boolean discardingTooLongFrame;
//需要丢弃的字节数
private long tooLongFrameLength;
//累计丢弃的字节数
private long bytesToDiscard;
```

## 示例1：典型的基于消息长度+消息内容的解码

```java
/*
 * BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
 * +--------+----------------+      +--------+----------------+
 * | Length | Actual Content |----->| Length | Actual Content |
 * | 0x000C | "HELLO, WORLD" |      | 0x000C | "HELLO, WORLD" |
 * +--------+----------------+      +--------+----------------+
 */
```

- lengthFieldOffset=0,因为Length字段就在报文的开始位置
- lengthFieldLength=2,协议设计的固定长度
- lengthAdjustment=0, Length字段只包含消息长度，不需要做任何修正
- initialBytesToStrip=0, 解码后内容依然是Length+Content, 不需要跳过任何初始字节

## 示例2：解码结果需要截断

```java
/*
 * BEFORE DECODE (14 bytes)         AFTER DECODE (12 bytes)
 * +--------+----------------+      +----------------+
 * | Length | Actual Content |----->| Actual Content |
 * | 0x000C | "HELLO, WORLD" |      | "HELLO, WORLD" |
 * +--------+----------------+      +----------------+
 */
```

- lengthFieldOffset=0,因为Length字段就在报文的开始位置
- lengthFieldLength=2,协议设计的固定长度
- lengthAdjustment=0, Length字段只包含消息长度，不需要做任何修正
- initialBytesToStrip=2, 跳过Length字段的字节长度，解码后ByteBuf中只包含Content字段

## 示例3：长度字段包含消息长度和消息内容所占的字节

```java
/*
 * BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
 * +--------+----------------+      +--------+----------------+
 * | Length | Actual Content |----->| Length | Actual Content |
 * | 0x000E | "HELLO, WORLD" |      | 0x000E | "HELLO, WORLD" |
 * +--------+----------------+      +--------+----------------+
 */
```

- lengthFieldOffset=0,因为Length字段就在报文的开始位置
- lengthFieldLength=2,协议设计的固定长度
- lengthAdjustment=-2, 长度字段为14字节，需要减2才是拆包所需要的长度
- initialBytesToStrip=0, 解码后内容依然是Length+Content, 不需要跳过任何初始字节

## 示例4：基于长度字段偏移的解码

```java
/*
 * BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
 * +----------+----------+----------------+      +----------+----------+----------------+
 * | Header 1 |  Length  | Actual Content |----->| Header 1 |  Length  | Actual Content |
 * |  0xCAFE  | 0x00000C | "HELLO, WORLD" |      |  0xCAFE  | 0x00000C | "HELLO, WORLD" |
 * +----------+----------+----------------+      +----------+----------+----------------+
```

- lengthFieldOffset=2,需要跳过Header1所占用的2字节，才是Length的起始位置
- lengthFieldLength=3,协议设计的固定长度
- lengthAdjustment=0, Length字段只包含消息长度，不需要做任何修改
- initialBytesToStrip=0, 解码后内容依然是Length+Content, 不需要跳过任何初始字节

## 示例5：长度字段与内容字段不再相邻

```java
/*
 * BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
 * +----------+----------+----------------+      +----------+----------+----------------+
 * |  Length  | Header 1 | Actual Content |----->|  Length  | Header 1 | Actual Content |
 * | 0x00000C |  0xCAFE  | "HELLO, WORLD" |      | 0x00000C |  0xCAFE  | "HELLO, WORLD" |
 * +----------+----------+----------------+      +----------+----------+----------------+
 */
```

- lengthFieldOffset=0,因为Length字段就在报文的开始位置
- lengthFieldLength=3,协议设计的固定长度
- lengthAdjustment=2, 由于Header+Content一共占用2+12=14字节，所以Length字段值(12字节)加上lengthAdustment(2字节)才能得到Header+Content的内容(14字节)
- initialBytesToStrip=0, 解码后内容依然是Length+Content, 不需要跳过任何初始字节

## 示例6：基于长度偏移和长度修正的解码

```java
/*
 * BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
 * +------+--------+------+----------------+      +------+----------------+
 * | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
 * | 0xCA | 0x000C | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
 * +------+--------+------+----------------+      +------+----------------+
 */
```

- lengthFieldOffset=1,需要跳过HDR1所占用的1字节，才是Length的起始位置
- lengthFieldLength=2,协议设计的固定长度
- lengthAdjustment=1, 由于HDR2+Content一共占用1+12=13字节，所以Length字段值(12字节)加上lengthAdustment(1)才能得到HDR2+Content的内容(13字节)
- initialBytesToStrip=3, 解码后跳过HDR1和Length字段，共占用3字节

## 示例7：长度字段包含除Content外的多个其他字段

```java
/*
 * BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
 * +------+--------+------+----------------+      +------+----------------+
 * | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
 * | 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
 * +------+--------+------+----------------+      +------+----------------+
 */
```

- lengthFieldOffset=1,需要跳过HDR1所占用的1字节，才是Length的起始位置
- lengthFieldLength=2,协议设计的固定长度
- lengthAdjustment=-3, Length字段值(16字节)需要减去HDR1(1字节)和Length自身所占用字节长度(2字节)才能得到HDR2和Content的内容(1+12=13字节)
- initialBytesToStrip=3, 解码后跳过HDR1和Length字段，共占用3字节