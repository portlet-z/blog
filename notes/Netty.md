# NIO基础

non-blocking io 非阻塞io

## 1.三大组件

### 1.1 Channle & Buffer

 channel有一点类似于stream，它就是读写数据的双向通道，可以从channel将数据读入buffer，也可以将buffer的数据写入channel，而之前的stream要么是输入，要么是输出，channel比stream更为底层

常见的Channel有：FileChannel, DatagramChannel, SocketChannel, ServerSocketChannel

buffer则用来缓冲读写数据，常见的buffer有：

- ByteBuffer: MappedByteBuffer, DirectByteBuffer, HeapByteBuffer
- ShortBuffer, IntBuffer, LongBuffer, FloatBuffer, DoubleBuffer, CharBuffer

### 1.2 Selector



## 2. ByteBuffer

### 2.1 ByteBuffer正确使用姿势

- 向buffer写入数据，如调用channel.read(buffer)
- 调用flip()切换至读模式
- 从buffer读取数据，如调用buffer.get()
- 调用clear()或compact()切换至写模式
- 重复1-4步骤