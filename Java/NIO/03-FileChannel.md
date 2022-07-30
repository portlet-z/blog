## FileChannel

FileChannel只能工作在阻塞模式下

### 获取

不能之间打开FileChannel,必须通过FileInputStream, FileOutStream或者RandomAccessFile来获取FileChannel, 它们都有getChannel方法

- 通过FileInputStream获取的channel只能读
- 通过FileOutStream获取的channel只能写
- 通过RandomAccessFile是否能读写根据构造RandomAccessFile时的读写模式决定

### 读取

会从channel读取数据填充ByteBuffer,返回值表示读带了多少字节，-1表示到达了文件的末尾

```java
int readBytes = channel.read(buffer);
```

### 写入

写入的正确姿势如下，

```java
ByteBuffer buffer = ...;
buffer.put(...); //存入数据
buffer.flip(); //切换读模式
while(buffer.hasRemaining()) {
    channel.write(buffer);
}
```

在while中调用channel.write是因为write方法并不能保证一次将buffer中的内容全部写入channel

### 关闭

channel必须关闭，不过调用了FileInputStream, FileOutStream或者RandomAccessFile的close方法会间接的调用channel的close方法

### 位置

获取当前位置

```java
long pos = channel.position();
```

设置当前位置

```java
long newPos = ...;
channel.position(newPos);
```

设置当前位置时，如果设置为文件末尾

- 这时读取会返回-1
- 这时写入，会追加内容，但要注意如果position超过了文件末尾，再写入时在新内容和原末尾之间会有空洞(00)

### 大小

使用size方法获取文件的大小

### 强制写入

操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘。可以调用force(true)方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘

## 两个Channel传输数据

```java
public class TestFileChannelTransferTo {
    public static void main(String[] args) {
        try (
                FileChannel from = new FileInputStream("data.txt").getChannel();
                FileChannel to = new FileOutputStream("to.txt").getChannel();
        ) {
            // 效率高，底层会利用操作系统的零拷贝进行优化，2G数据
            long size = from.size();
            // left变量代表还剩余多少字节
            for (long left = size; left > 0;) {
                left -= from.transferTo((size - left), left, to);
            }
        } catch (IOException e) {
        }
    }
}
```

## Path

jdk7引入了Path和Paths类

- Path用来表示文件路径
- Paths是工具类，用来获取Path实例
- `.`代表了当前路径
- `..`代表了上一级路径

```java
Path source = Paths.get("/data/1.txt");
Path source = Paths.get("1.txt");
Path source = Paths.get("/data", "projects");
Path path = Paths.get("/data/a/../b");
System.out.println(path); // /data/a/../b
System.out.printlb(path.normalize()); //data/b
```

## Files

### 检查文件是否存在

```java
Path path = Paths.get("data.txt");
System.out.println(Files.exists(path));
```

### 创建一级目录

```java
Path path = Paths.get("d1");
Files.createDirectory(path);
```

- 如果目录已存在，会抛出FileAlreadyExistsException
- 不能一次创建多级目录，否则会抛异常NoSuchFileException

### 创建多级目录

```java
Path path = Paths.get("d1/d2");
Files.createDirectories(path);
```

### 拷贝文件

```java
File source = Path.get("data.txt");
Path target = Path.get("target.txt");

Files.copy(source, target);
```

- 如果文件已经存在，会抛出FileAlreadyExistsException
- 如果希望用source覆盖掉target,需要用StandardCopyOption来控制

```java
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```

### 移动文件

```java
Path source = Paths.get("data.txt");
Path target = Paths.get("target.txt");
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

- StandardCopyOption.ATOMIC_MOVE保证文件移动的原子性

### 删除文件

```java
Path target = Paths.get("target.txt");
Files.delete(target);
```

### 遍历目录文件

```java
public class TestFilesWalkTree {
    public static void main(String[] args) throws Exception {
        m1();
    }
    private static void m1() throws Exception {
        AtomicInteger dirCount = new AtomicInteger();
        AtomicInteger fileCount = new AtomicInteger();
        Files.walkFileTree(Paths.get("C:\\Dev"), new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
                System.out.println("===>" + dir);
                dirCount.incrementAndGet();
                return super.preVisitDirectory(dir, attrs);
            }

            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                System.out.println(file);
                fileCount.incrementAndGet();
                return super.visitFile(file, attrs);
            }
        });
        System.out.println("dir count: " + dirCount);
        System.out.println("file count: " + fileCount);
    }
}
```