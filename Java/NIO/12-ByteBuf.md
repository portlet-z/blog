ByteBuf是对字节数据的封装，扩展了NIO中ByteBuffer的功能

## 创建

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(10);
log(buffer);
```

上面代码创建了一个默认的ByteBuf(池化基于直接内存的ByteBuf),初始容量10

输出

```
read index:0 write index:0 capacity:10
```

```java
public class TestByteBuf {
    public static void main(String[] args) {
        ByteBuf buf = ByteBufAllocator.DEFAULT.buffer(10);
        log(buf);
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 20; i++) {
            sb.append("a");
        }
        buf.writeBytes(sb.toString().getBytes());
        log(buf);
    }
    private static void log(ByteBuf buffer) {
        int length = buffer.readableBytes();
        int rows = length / 16 + (length % 15 == 0 ? 0 : 1) + 4;
        StringBuilder buf = new StringBuilder(rows * 80 * 2)
                .append("read index:").append(buffer.readerIndex())
                .append(" write index:").append(buffer.writerIndex())
                .append(" capacity:").append(buffer.capacity())
                .append(NEWLINE);
        appendPrettyHexDump(buf, buffer);
        System.out.println(buf);
    }
}
```

输出

```
read index:0 write index:0 capacity:10

read index:0 write index:20 capacity:64
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 |aaaaaaaaaaaaaaaa|
|00000010| 61 61 61 61                                     |aaaa            |
+--------+-------------------------------------------------+----------------+
```

## 直接内存 VS 堆内存

可以使用下面的代码来创建池化基于堆的ByteBuf]

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.heapBuffer(10);
```

也可以使用下面的代码来创建池化基于直接内存的ByteBuf

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.directBuffer(10);
```

- 直接内存创建和销毁的代价昂贵，但读写性能高（少一次内存复制），适合配合池化功能一起用
- 直接内存堆GC压力小，因为这部分内存不受JVM垃圾回收的管理，但也要注意及时主动释放

## 池化 vs 非池化

池化的最大意义在于可以重用ByteBuf,优点有

- 没有池化，则每次都得创建新的ByteBuf实例，这个操作对直接内存代价昂贵，就算是堆内存，也会增加GC压力
- 有了池化，则可以重用池中ByteBuf实例，并且采用了与jemalloc类似的内存分配算法提升分配效率
- 高并发时，池化功能更节约内存，减少内存溢出的可能

池化功能是否开启，可以通过下面的系统环境变量来设置

```
-Dio.netty.allocator.type={pooled|unpooled}
```

- 4.1以后，非Android平台默认启用池化实现，Android平台启用非池化实现
- 4.1之前，池化功能还不成熟，默认是非池化实现

## 组成

ByteBuf有四部分组成

- capacity
- max capacity
- readerIndex
- writerIndex

最开始读写指针都在0的位置

## 写入

| 方法签名                                                     | 含义                | 备注                                    |
| ------------------------------------------------------------ | ------------------- | --------------------------------------- |
| writeBoolean(boolean value)                                  | 写入boolean值       | 用一字节01\|00代表true\|false           |
| writeByte(int value)                                         | 写入byte值          |                                         |
| writeShort(int value)                                        | 写入short值         |                                         |
| writeInt(int value)                                          | 写入int值           | Big Endian,即0x250,写入后 00 00 02 50   |
| writeIntLE(int value)                                        | 写入int值           | Little Endian,即0x250,写入后50 02 00 00 |
| writeLong(long value)                                        | 写入long值          |                                         |
| writeChar(int value)                                         | 写入char值          |                                         |
| writeFloat(float value)                                      | 写入float值         |                                         |
| writeDouble(double value)                                    | 写入double值        |                                         |
| writeBytes(ByteBuf src)                                      | 写入netty的ByteBuf  |                                         |
| writeBytes(byte[] src)                                       | 写入byte[]          |                                         |
| writeBytes(ByteBuffer src)                                   | 写入NIO的ByteBuffer |                                         |
| int writeCharSequence(CharSequence sequence, <br />Carset charset) | 写入字符串          |                                         |

- 这些方法未指明返回值的，其返回值都是ByteBuf,意味着可以链式调用
- 网络传输，默认习惯是Big Endian

先写入4个字节

```java
ByteBuf buf = ByteBufAllocator.DEFAULT.buffer(10);
buf.writeBytes(new byte[] {1, 2, 3, 4});
log(buf);
```

结果是

```
read index:0 write index:4 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 01 02 03 04                                     |....            |
+--------+-------------------------------------------------+----------------+
```

再写入一个int整数，也是4个字节

```java
buf.writeInt(5);
log(buf);
```

结果是

```
read index:0 write index:8 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 01 02 03 04 00 00 00 05                         |........        |
+--------+-------------------------------------------------+----------------+
```

还有一类方法是set开头的一系列方法，也可以写入数据，但不会改变写指针位置

## 扩容

再写入一个int整数时，容量不够了（初始容量为10），这时会引发扩容

```
buf.writeInt(6);
log(buf);
```

结果是

```
read index:0 write index:12 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 01 02 03 04 00 00 00 05 00 00 00 06             |............    |
+--------+-------------------------------------------------+----------------+
```

扩容规则是

- 如果写入后数据大小未超过512，则选择下一个16的整数倍，例如写入后大小为12，则扩容后capacity是16
- 如果写入后数据大小超过512，则下一个2^n,例如写入后大小为513，则扩容后capacity是1024
- 扩容不能超过max capacity