Netty
NIO
AIO
Reactor
Proactor
reactor-siemens
Scalable IO in Java

## Buffer

java.nio.Buffer提供了如下实现：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer
- MappedByteBuffer

### 缓冲区原理

- position : 指定下一个将被要写入或者读取的元素索引，它的值由get()/put()方法字段更新，在新创建一个Buffer对象时，position被初始化为0。
- capacity : 指定了可以存储在缓冲区中的最大数据容量，实际上，它指定了底层数组的大小，或者只是是指定了准许我们使用的底层数组的容量。
- limit：





![](C:/Users/linpeng.xiong/Desktop/document/images/NIO-buffer.png)



### 缓冲区分配

使用缓冲区之前，首先需要对其进行分配。

非直接缓冲区

```java
//分配一个可以存储1024个字节大小的非直接缓冲区
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
//分配一个可以存储1024个字符大小的非直接缓冲区
CharBuffer charBuffer = CharBuffer.allocate(1024);
```

直接缓冲区

```java
//分配一个可以存储1024个字符大小的直接缓冲区
ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
```

将字节数组封装到缓冲区中。

```java
String val = "Hello Buffer";
ByteBuffer buffer = ByteBuffer.wrap(val.getBytes());
ByteBuffer buffer = ByteBuffer.wrap(val.getBytes(), 0, val.length());
```

> 对于直接缓冲区，只有ByteBuffer和MappedByteBuffer才能分配。

### 缓冲区数据读写

这里所介绍的缓冲区数据读写仅仅是指对缓存区的操作，而不是通过通道对缓冲区进行读写。

Buffer的继承类实现了一些列put和get方法，通过这些方法，可以想缓冲区添加和获取数据。需要注意的是，put和get方法会使position指针移动。



```java
ByteBuffer buffer = ByteBuffer.allocate(1024);
for (int i = 0; i < 100; i++) {
    buffer.putInt(i);
}
buffer.flip(); // 翻转Buffer
for (int i = 0; i < 100; i++) {
    int val = buffer.getInt();
    System.out.printf("%d, ", val);
}
```







### 翻转缓冲区

在向缓冲区写完数据之后，需要再次进行读取，必须先要进行翻转，即将position指针使其指向底层数组索引0，limit指针指向最大可读取数据的索引。

通过flip()方法可以完成这一点，flip()函数源码：

```java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

示例：

```java
buffer.flip();
```

### 倒带缓冲区

随着对缓冲区的读取或写入，position指针会不停地向前移动，通过倒带缓冲区，可以重新从头开始读取或写入。

rewind()方法源码：

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

rewind()方法与flip()的唯一区别就是没有变更limit指针。

### 清理缓冲区

一旦缓冲区中的数据不再被使用，那么就需要准备好再次被写入。可以通过clear()和compact()方法完成。

**清理全部缓冲区**

清理全部缓冲区是通过clear()方法。需要注意的是，此处所说的清理不是常规意义上的清理（断开数据引用，使其被GC回收）。在Buffer中调用clear()方法，仅仅是将position设为0，使limit与capacity，mark标记为-1。简而言之，仅仅只是重置了相关标记，告诉我们可以从哪里开始写入数据，而旧数据被‘遗忘’了，实际上旧数据并没有被删除。

clear()源码：

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

**清理已读缓冲区**

如果Buffer中仍有未读数据，且后续还需要这些数据，但此时想要先写一些数据，那么可以通过compact()方法清理已读部分的缓冲区。

compact()方法将所有未读的数据拷贝到Buffer的开头，然后将position设置为最后一个未读元素的正后面，而limit仍然与clear()一样设置为capacity。如此，就可以开始写数据了。

compact()源码：

```java
public abstract ByteBuffer compact();
```

示例：

```java
//创建一个缓冲区，并写入数据0~9
ByteBuffer buffer = ByteBuffer.allocate(10);
for (int i = 0; i < 10; i++) {
    buffer.put((byte) i);
}
buffer.flip();
//读取一部分数据
for (int i = 0; i < 5; i++) {
    byte val = buffer.get();
    System.out.printf("%d ", val);
}
//清理已读缓冲区
buffer.compact();
//重新写入数据到已清理缓冲区
for (int i = 0; i < 5; i++) {
    buffer.put((byte) (i * 5));
}
//输出所有数据
System.out.printf("\n%s\n", Arrays.toString(buffer.array()));

//输出结果：
//0 1 2 3 4 
//[5, 6, 7, 8, 9, 0, 5, 10, 15, 20]
```

### 标记缓冲区

通过调用buffer.mark()方法可以标记缓冲区中一个特定的position他，然后再次调用buffer.reset()方法可以恢复到这个标记。

示例

```java
buffer.mark();
// call other operation ... 
buffer.reset();
```

buffer.mark()方法源码：

```java
public final Buffer mark() {
    mark = position;
    return this;
}
```

buffer.reset()方法源码：

```java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

### 只读缓冲区

只读缓冲区，故名思义，就是只能读不能写的缓冲区。`asReadOnlyBuffer()`方法可以返回一个只读缓冲区。

只读缓冲区特性：

1. 只读缓冲区与原缓冲区是完全一致的，它们共享缓冲区，唯一不同的是，只读缓冲区是只读的。
2. 如果原缓冲区中的数据发生改变，由于共享缓冲区，所以只读缓冲区也会跟着改变。
3. 如果尝试修改只读缓冲区中的数据，那么将会报`java.nio.ReadOnlyBufferException`异常。
4. 只可以将常规缓冲区转换为只读缓冲区，而不能将只读缓冲区转换为常规缓冲区。

只读缓冲区的作用：

只读缓冲区对于数据的保护是很有用的。例如将缓冲区传递给一个方法，但不能保证该方法不对缓冲区进行修改，那么就可以传递一个只读缓冲区保证缓冲区中的数据不会被修改。

示例：

```java
ByteBuffer readOnlyBuffer = buffer.asReadOnlyBuffer();
```

### 比较缓冲区

可以使用equals和compareTo两个方法比较两个Buffer是否相等。

**boolean equals(Object)**

当满足一下条件时，两个Buffer相等：

1. 它们具有相同的元素类型。
2. 它们的剩余元素数量相同。
3. 它们的剩余元素的内容相同。

**int compareTo(Buffer)**

两个字节缓冲区的比较方法是按字典序比较其剩余元素的序列，而不考虑每个序列在相应缓冲区中的起始位置。

### 缓冲区类型转换

ByteBuffer提供了as{XXX}Buffer()系列方法可以将ByteBuffer转换为其他类型的Buffer：

- `public abstract CharBuffer asCharBuffer();`
- `public abstract DoubleBuffer asDoubleBuffer();`
- `public abstract FloatBuffer asFloatBuffer();`
- `public abstract IntBuffer asIntBuffer();`
- `public abstract LongBuffer asLongBuffer();`
- `public abstract ShortBuffer asShortBuffer();`

示例：

```java
ByteBuffer buffer = ByteBuffer.allocate(1024);
CharBuffer charBuffer = buffer.asCharBuffer();
DoubleBuffer doubleBuffer = buffer.asDoubleBuffer();
FloatBuffer floatBuffer = buffer.asFloatBuffer();
IntBuffer intBuffer = buffer.asIntBuffer();
LongBuffer longBuffer = buffer.asLongBuffer();
ShortBuffer shortBuffer = buffer.asShortBuffer();
```

> as{XXX}Buffer()系列方法只有ByteBuffer有。

### 其他API

- `hasRemaining()`：返回当前position和limit之间是否有任何元素。

- `remaining()`：返回当前position与limit之间的元素个数。

- `duplicate()`：创建一个新的缓冲区，它们完全一致，例如旧缓冲区是直接的，那么新缓冲区也是直接的；旧缓冲区是只读的，那么新缓冲区也是只读的。新缓冲区与旧缓冲区共享缓冲区空间，即意味着对旧缓冲区的更改对新缓冲区可见，反之亦然。不过它们的capacity、limit、position和mark是独立的。

- `slice()`：创建一个新的缓冲区，该新缓冲区是旧缓冲区内容的共享子序列。它们之间特性完全一致，例如旧缓冲区是直接的，那么新缓冲区也是直接的；旧缓冲区是只读的，那么新缓冲区也是只读的。新缓冲区与旧缓冲区共享缓冲区子序列空间，即意味着对旧缓冲区的更改对新缓冲区可见，反之亦然。不过它们的capacity、limit、position和mark是独立的。子区间位于旧缓冲区position~limit之间。

  > **duplicate()与slice()的区别**
  >
  > duplicate()与slice()唯一的区别就是duplicate()是全序列，slice()是子序列，其他特性完全一致。

- `hasArray()`：测试当前缓冲区是否由一个可访问的数组支持。如果是（由数组支持，并且非只读），则返回true。如果返回true，则可以安全的调用array和arrayOffset方法。

- `isDirect()`：测试当前缓冲区是否是直接缓冲区，如果是，则返回true。

- `isReadOnly()`：测试当前缓冲区是否是只读的，如果是，再返回true。

- `array()`：返回当前缓冲区的数组。注意，返回的数组与缓冲区内的数组仍是同一个对象实例，即对缓冲区内容的修改将会导致返回的数据发生变化，反之亦然。在调用这个方法之前调用hasArray()方法，以确保这个缓冲区有一个可访问的后备数组。

- `arrayOffset()`：返回当前缓冲区数组中第一个元素的偏移量。常规情况下，`arrayOffset()`始终返回0。要获取0以外的值的方法之一是创建一个Buffer，然后调用`slice()`，slice会产生一个偏移量`offset`，对应于旧缓冲区的相对位置。在调用这个方法之前调用`hasArray()`方法，以确保这个缓冲区有一个可访问的后备数组。

- `capacity()`：返回当前缓冲区的capacity。

- `position()`：返回当前缓冲区的position。

- `limit()`：返回当前缓冲区的limit。

- `order()`：检索当前缓冲区的字节顺序。字节顺序在读写多字节值以及创建作为该字节缓冲区视图的缓冲区时使用。新创建的字节缓冲区的顺序总是BIG_ENDIAN。

- `ByteBuffer order(ByteOrder bo)`：修改当前缓冲区的字节顺序，新的字节顺序可以是BIG_ENDIAN或者LITTLE_ENDIAN。

- `Buffer limit(int newLimit)`：设置当前缓冲区的limit，如果position大于新limit，则将其设置为新limit，如果mark被定义并且大于新limit，则将被丢弃。

- `Buffer position(int newPosition)`：设置当前缓冲区的position，如果mark被定义并且大于新position，则将被丢弃。新的position值必须为非负数并且不大于当前limit。

### 非直接缓冲区

非直接缓冲区是指在用户空间创建的缓冲区。对应到JVM就是堆内内存空间。

如下图所示，应用程序要从磁盘中读取数据，操作系统首先需要将数据从磁盘拷贝到内核空间的内核缓冲区，然后再拷贝到用户空间的用户缓冲区，经过两次拷贝，才能读取到数据。同理，也需要经过两次拷贝才能将数据写入到磁盘。因此使用非直接缓冲区进行I/O操作有四次数据拷贝的性能损耗。

![](images/IO-Non-Direct-Memory.png)

### 直接缓冲区

直接缓冲区是指在直接内核空间创建的缓冲区。对应到JVM就是堆外内存空间。

如下图所示，建立了内核空间内核缓冲区与用户空间用户缓冲区的物理内存映射文件，使应用程序申请的缓冲区空间可以直接申请在内核空间，同时也使应用程序对用户缓冲区的读写操作等于直接对内核缓冲区的读写操作。如此，就绕过了用户缓冲区与磁盘进行的I/O读写操作，减少了内核缓冲区与用户缓冲区的两次CPU拷贝。相对非直接缓冲区提高了性能。

![](images/IO-Direct-Memory.png)

直接缓冲区通过减少两次CPU拷贝从而提高性能，但这并不是没有代价。直接缓冲区的创建要比非直接缓冲区花费更高的成本。直接缓冲区的创建是通过调用本地操作系统的接口创建，绕过了JVM堆栈。因此直接缓冲区的建立和销毁明显比基于堆栈的缓冲区消耗的代价更大，这取决于操作系统和JVM的实现。基于上述原因，直接缓冲区不适用于频繁的创建和少量的读写，适用于大量数据的I/O操作。

#### MappedByteBuffer

MappedByteBuffer是直接字节缓冲区的实现，其内容是文件的内存映射区域。

MappedByteBuffer是通过`FileChannel.map`方法创建的。该类使用特定于内存映射文件区域的操作扩展了`ByteBuffer`类。

一个映射的字节缓冲区和它所代表的文件映射在缓冲区本身被垃圾回收之前仍然有效。

MappedByteBuffer的内容可以在任何时候更改，例如，如果映射文件的相应区域的内容被此程序或其他程序更改。这些更改是否发生以及何时发生取决于操作系统，因此不确定。

MappedByteBuffer的全部或部分在任何时候都可能变得不可访问，例如，如果映射文件被截断。试图访问映射字节缓冲区的不可访问区域不会改变缓冲区的内容，并且会在访问时或稍后的某个时间引发未指定的异常。因此，强烈建议采取适当的预防措施，以避免该程序或同时运行的程序操作映射文件，除了读写文件的内容。

MappedByteBuffer的行为与普通直接字节缓冲区没有什么不同。

MappedByteBuffer继承`ByteBuffer`，因此，`ByteBuffer`具有的功能，MappedByteBuffer也有。

相比于`ByteBuffer`，MappedByteBuffer增加了三个新的方法：

- `MappedByteBuffer force()`：

  ​		强制将对缓冲区内容所做的任何更改写入包含映射文件的存储设备。

  ​		如果映射到该缓冲区的文件驻留在本地存储设备上，那么当此方法返回时，它保证自创建该缓冲区或自上次调用该方法以来对该缓冲区所做的所有更改都已写入该设备。

  ​		如果文件不在本地设备上，则不会做出这样的保证。

  ​		如果这个缓冲区没有以读/写模式映射(java.nio.channels.FileChannel.MapMode.READ_WRITE)，那么调用这个方法是无效的。

- `boolean isLoaded()`：

  测试这个缓冲区的内容是否驻留在物理内存中。

  返回值为true意味着很有可能这个缓冲区中的所有数据都驻留在物理内存中，因此可以在不引起任何虚拟内存页错误或I/O操作的情况下进行访问。返回值为false并不一定意味着缓冲区的内容不在物理内存中。

  返回值是一个提示，而不是保证，因为在调用此方法返回时，底层操作系统可能已经调出了缓冲区的一些数据。

- `MappedByteBuffer load()`：

  ​		将缓冲区的内容加载到物理内存中。

  ​		这个方法尽最大努力确保，当它返回时，这个缓冲区的内容驻留在物理内存中。调用此方法可能会导致一些页面错误和I/O操作的发生。

> 注意，不能通过`MappedByteBuffer.allocate`，`MappedByteBuffer.allocateDirect`以及`MappedByteBuffer.wrap`创建`MappedByteBuffer`，因为其实际调用的是`ByteBuffer`的对应方法，返回的也是`ByteBuffer`类型。

## Channel

### FileChannel



```java
public abstract class FileChannel
    extends AbstractInterruptibleChannel
    implements SeekableByteChannel, GatheringByteChannel, ScatteringByteChannel
{

    /**
     * @since   1.7
     */
    public static FileChannel open(Path path,
                                   Set<? extends OpenOption> options,
                                   FileAttribute<?>... attrs)
        throws IOException
    {
        FileSystemProvider provider = path.getFileSystem().provider();
        return provider.newFileChannel(path, options, attrs);
    }

    @SuppressWarnings({"unchecked", "rawtypes"}) // generic array construction
    private static final FileAttribute<?>[] NO_ATTRIBUTES = new FileAttribute[0];

    /**
     * @since   1.7
     */
    public static FileChannel open(Path path, OpenOption... options)
        throws IOException
    {
        Set<OpenOption> set = new HashSet<OpenOption>(options.length);
        Collections.addAll(set, options);
        return open(path, set, NO_ATTRIBUTES);
    }

    // -- Channel operations --

    /**
     * Reads a sequence of bytes from this channel into the given buffer.
     *
     * <p> Bytes are read starting at this channel's current file position, and
     * then the file position is updated with the number of bytes actually
     * read.  Otherwise this method behaves exactly as specified in the {@link
     * ReadableByteChannel} interface. </p>
     */
    public abstract int read(ByteBuffer dst) throws IOException;

    /**
     * Reads a sequence of bytes from this channel into a subsequence of the
     * given buffers.
     *
     * <p> Bytes are read starting at this channel's current file position, and
     * then the file position is updated with the number of bytes actually
     * read.  Otherwise this method behaves exactly as specified in the {@link
     * ScatteringByteChannel} interface.  </p>
     */
    public abstract long read(ByteBuffer[] dsts, int offset, int length)
        throws IOException;

    /**
     * Reads a sequence of bytes from this channel into the given buffers.
     *
     * <p> Bytes are read starting at this channel's current file position, and
     * then the file position is updated with the number of bytes actually
     * read.  Otherwise this method behaves exactly as specified in the {@link
     * ScatteringByteChannel} interface.  </p>
     */
    public final long read(ByteBuffer[] dsts) throws IOException {
        return read(dsts, 0, dsts.length);
    }

    /**
     * Writes a sequence of bytes to this channel from the given buffer.
     *
     * <p> Bytes are written starting at this channel's current file position
     * unless the channel is in append mode, in which case the position is
     * first advanced to the end of the file.  The file is grown, if necessary,
     * to accommodate the written bytes, and then the file position is updated
     * with the number of bytes actually written.  Otherwise this method
     * behaves exactly as specified by the {@link WritableByteChannel}
     * interface. </p>
     */
    public abstract int write(ByteBuffer src) throws IOException;

    /**
     * Writes a sequence of bytes to this channel from a subsequence of the
     * given buffers.
     *
     * <p> Bytes are written starting at this channel's current file position
     * unless the channel is in append mode, in which case the position is
     * first advanced to the end of the file.  The file is grown, if necessary,
     * to accommodate the written bytes, and then the file position is updated
     * with the number of bytes actually written.  Otherwise this method
     * behaves exactly as specified in the {@link GatheringByteChannel}
     * interface.  </p>
     */
    public abstract long write(ByteBuffer[] srcs, int offset, int length)
        throws IOException;

    /**
     * Writes a sequence of bytes to this channel from the given buffers.
     *
     * <p> Bytes are written starting at this channel's current file position
     * unless the channel is in append mode, in which case the position is
     * first advanced to the end of the file.  The file is grown, if necessary,
     * to accommodate the written bytes, and then the file position is updated
     * with the number of bytes actually written.  Otherwise this method
     * behaves exactly as specified in the {@link GatheringByteChannel}
     * interface.  </p>
     */
    public final long write(ByteBuffer[] srcs) throws IOException {
        return write(srcs, 0, srcs.length);
    }


    // -- Other operations --

    public abstract long position() throws IOException;

    public abstract FileChannel position(long newPosition) throws IOException;

    public abstract long size() throws IOException;

    public abstract FileChannel truncate(long size) throws IOException;

    public abstract void force(boolean metaData) throws IOException;

    public abstract long transferTo(long position, long count,
                                    WritableByteChannel target)
        throws IOException;

    public abstract long transferFrom(ReadableByteChannel src,
                                      long position, long count)
        throws IOException;

    public abstract int read(ByteBuffer dst, long position) throws IOException;

    public abstract int write(ByteBuffer src, long position) throws IOException;


    // -- Memory-mapped buffers --

    /**
     * A typesafe enumeration for file-mapping modes.
     *
     * @since 1.4
     *
     * @see java.nio.channels.FileChannel#map
     */
    public static class MapMode {

        /**
         * Mode for a read-only mapping.
         */
        public static final MapMode READ_ONLY = new MapMode("READ_ONLY");

        /**
         * Mode for a read/write mapping.
         */
        public static final MapMode READ_WRITE = new MapMode("READ_WRITE");

        /**
         * Mode for a private (copy-on-write) mapping.
         */
        public static final MapMode PRIVATE = new MapMode("PRIVATE");

        private final String name;
        private MapMode(String name) {
            this.name = name;
        }
        public String toString() {
            return name;
        }
    }

    public abstract MappedByteBuffer map(MapMode mode, long position, long size)
        throws IOException;

    // -- Locks --

    public abstract FileLock lock(long position, long size, boolean shared)
        throws IOException;

    public final FileLock lock() throws IOException {
        return lock(0L, Long.MAX_VALUE, false);
    }

    public abstract FileLock tryLock(long position, long size, boolean shared)
        throws IOException;

    public final FileLock tryLock() throws IOException {
        return tryLock(0L, Long.MAX_VALUE, false);
    }

}
```



#### 创建FileChannel

在使用FileChannel之前必须先创建它，但是不能直接创建它。可以通过FileInputStream、FileOutputStream或RandomAccessFile获取一个FileChannel示例：

```
fileInputStream.getChannel();
fileOutputStream.getChannel();
fileRandomAccessFile.getChannel();
```

另外FileChannel提供可静态open()方法也可以打开对一个文件的通道实例：

```java
public static FileChannel open(Path path,
                               Set<? extends OpenOption> options,
                               FileAttribute<?>... attrs) throws IOException {
    FileSystemProvider provider = path.getFileSystem().provider();
    return provider.newFileChannel(path, options, attrs);
}

public static FileChannel open(Path path, OpenOption... options) throws IOException {
    Set<OpenOption> set = new HashSet<OpenOption>(options.length);
    Collections.addAll(set, options);
    return open(path, set, NO_ATTRIBUTES);
}
```

这两个API在JDK1.7被提供。包含三个参数：

- `Path path`：
- `Set<? extends OpenOption> options`：
- `FileAttribute<?>... attrs`：



#### 读写数据

FileChannel可以通过read和write从缓冲区中进行读写数据。

- read()：将一个字节序列从这个通道读入给定的缓冲区。从这个通道的当前文件位置开始读取字节，然后根据实际读取的字节数更新文件位置。否则，此方法的行为与ReadableByteChannel接口中指定的完全一致。返回一个int型，表示读取的字节数量。
- write()：从给定的缓冲区将一个字节序列写入该通道。字节从这个通道的当前文件位置开始写入，除非通道处于追加模式，在这种情况下，该位置首先被推进到文件的末尾。如果有必要，文件会增长以适应写入的字节，然后文件位置会根据实际写入的字节数更新。否则，该方法的行为与WritableByteChannel接口指定的完全一致。返回一个int型，表示写入的字节数量。

示例代码如下所示：

```java
FileChannel channel = FileChannel.open(Paths.get("1.txt"),
                           StandardOpenOption.READ, StandardOpenOption.WRITE);
// reading
ByteBuffer buffer = ByteBuffer.allocate(1024);
channel.read(buffer);
// writing
buffer.flip();
buffer.put(0, (byte) 'W');
channel.write(buffer);
```

#### 从指定的位置开始读写数据

FileChannel具有文件位置（position）的概念，即FileChannel所读取文件的字节的位置。为了更好的理解，假设所读取文件的整个内容是一个字节数组，初始位置position等于0，即没有任何读取，随着调用read()函数读取数据，position向后移动，假设读取了5字节数据，那么position将等于5。

FileChannel提供了获取和设置文件位置（position）函数：

```java
public abstract long position() throws IOException;
public abstract FileChannel position(long newPosition) throws IOException;
```

<font style="background-color:#02aaf4">position对读写影响</font>

- write()函数将从文件的position位置开始写数据，如果position位置之后已经存在数据，将被覆盖。

  注意，仅仅覆盖所写数据的数量。例如假设position等于20，position位置之后有100字节的数据，而write()仅仅写了10个字节数据，那么只有`[20~30]`的部分会被覆盖，而`[30~100]`保持不变。

- read()函数将从文件的position位置开始读数据。

所以，如果想要从文件的指定位置开始读写，那么只需要设置文件位置（position）即可。

> 1. 设置的position不能小于0。
> 2. 如果将position设置到文件结束符之后，read()函数将返回-1。
> 3. 如果将position设置到文件结束符之后，然后向通道写入数据，文件将被撑大，并产生“文件空洞”。

#### 获取关联文件大小

FileChannel提供了size()函数，返回此通道文件的当前大小，以字节为单位。

示例：

```java
FileChannel channel = ....
long size = channel.size();
System.out.println(size);
```

#### 截取文件

FileChannel提供了truncate()函数可以截取文件，即将指定长度后面的部分删除。

示例：

```java
FileChannel channel = ...
channel.truncate(5);
```

这个例子截取前5个字节。假设文件内容为ABCDEFG，截取之后为ABCDE。

#### 强制写数据

FileChannel提供了fore()函数强制将通道文件的任何更新写入到存储设备上。

对于write()系统调用，出于性能方面的考虑，操作系统并不会将要写的数据立刻写入，而是先将要写的数据放入内核缓冲区中，然后马上返回。这也就意味着，当write()调用返回时，可能数据并没有写入到文件。

虽然fore()函数的语义如此，但由于环境不同还是会有所差异：

1. 如果此通道的文件是存在本地存储设备上，那么当此方法返回时，可以保证自创建此通道或自上次调用此方法以来对该文件所做的所有更改都已写入该设备。这对于确保在系统崩溃时不会丢失关键信息非常有用。
2. 如果文件不在本地设备上，则不会做出这样的保证。

fore()函数有一个Boolean类的参数metaData。取值true和false的含义如下：

- TRUE：表示必须写入对文件内容和元数据的更新。
- FALSE：表示只对文件内容的更新才需要写入存储设备。

> 注意：
>
> 当metaData取值为true时，需要将文件的元数据进行更新，这意味着需要进行额外的I/O操作。另外对于元数据的更新是否生效还取决于操作系统是否支持。
>
> 调用此方法可能会导致发生I/O操作，即使通道仅为读取而打开。例如，一些操作系统将最后一次访问时间作为文件元数据的一部分，每当读取文件时，这个时间就会更新。是否真的这样做取决于系统，因此没有指定。
>
> 此方法只能保证强制通过该类中定义的方法对通道文件进行更改。它可能强制也可能不强制通过修改通过调用map方法获得的映射字节缓冲区的内容而进行的更改。调用映射字节缓冲区的force方法将强制写入对缓冲区内容所做的更改。

#### 映射文件到直接缓冲区

通过FileChannel#map()方法可以将文件的一个区域直接映射到内存中。

一个文件的一个区域可以通过以下三种方式映射到内存：

1. Read-only：只读模式→任何修改结果缓冲区的尝试都将导致抛出java.nio.ReadOnlyBufferException异常。（MapMode.READ_ONLY）
2. Read/write：读写模式→对结果缓冲区所做的更改将传播到文件；它们可能对映射了相同文件的其他程序可见，也可能不可见。（MapMode.READ_WRITE）
3. Private：私有模式→对结果缓冲区所做的更改不会传播到文件，并且对映射了相同文件的其他程序也不可见；相反，它们将导致创建缓冲区中被修改部分的私有副本。（MapMode.PRIVATE）

对于read-only映射，此通道必须已经打开可以进行读取；对于read/write映射或private映射，必须为读和写同时打开此通道。

这个方法返回的MappedByteBuffer的position为0，limit和capacity为size；它的mark是未定义的。缓冲区和它所代表的映射将一直有效，直到缓冲区本身被垃圾回收。

映射一旦建立，就不依赖于用于创建它的文件通道。特别是关闭通道对映射的有效性没有影响。

内存映射文件的许多细节本质上依赖于底层操作系统，因此没有指定。当请求的区域未完全包含在此通道文件中时，此方法的行为未指定。此程序或其他程序对基础文件的内容或大小所做的更改是否传播到缓冲区是未指定的。将对缓冲区的更改传播到文件的速率未指定。

对于大多数操作系统，将一个文件映射到内存中比通过通常的读写方法读取或写入几十kb的数据更昂贵。从性能的角度来看，通常只值得将较大的文件映射到内存中。

```java
public abstract MappedByteBuffer map(MapMode mode, long position, long size)
        throws IOException;
```

- mode：一个常量→READ_ONLY，READ_WRITE和PRIVATE。在FileChannel.MapMode类中定义，代表含义分别是只读，读/写和私有(写时复制)。
- position：映射区域在文件中的开始位置；必须是非负整数。
- size：要映射区域的大小；必须是非负整数且不大于Integer.MAX_VALUE。

示例：

```java
fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```

这个例子映射文件到内存，映射模式为读/写，从文件位置0开始，映射1024字节。

#### 通过直接缓冲写文件





```java
FileChannel fileChannel = FileChannel.open(Paths.get("1.txt"), StandardOpenOption.READ, StandardOpenOption.WRITE);
MappedByteBuffer buffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, fileChannel.size());
buffer.put(0, (byte) 'A');
buffer.put(1, (byte) 'B');
buffer.put(2, (byte) 'C');
System.out.println("修改完毕");
```

#### 通道之间的数据传输

在Java NIO中，如果两个channel中有一个是FileChannel，那么可以直接将数据从一个Channel传输到另一个Channel。

##### mmap实现文件传输

其底层是基于mmap()系统调用。

```java
FileChannel readChannel = FileChannel.open(Paths.get("1.txt"), StandardOpenOption.READ, StandardOpenOption.WRITE);
MappedByteBuffer buffer = readChannel.map(FileChannel.MapMode.READ_WRITE, 0, readChannel.size());
FileChannel writeChannel = FileChannel.open(Paths.get("2.txt"), StandardOpenOption.CREATE, StandardOpenOption.WRITE);
writeChannel.write(buffer);
```

##### sendfile实现文件传输

FileChannel提供了两个函数，`transferFrom()`和`transferTo()`，用于将数据从源通道传输到目标通道。

其底层是基于sendfile()系统调用。

transferFrom()

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();
toChannel.transferFrom(fromChannel, 0, fromChannel.size());
```

transferTo()

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel fromChannel = fromFile.getChannel();
RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel toChannel = toFile.getChannel();
fromChannel.transferTo(0, fromChannel.size(), toChannel);
```



#### 分散(Scatter)/收集(Gather)

Channel操作支持Scatter/Gather，Scatter/Gather用于描述从通道中读取和写入的操作。

- 分散(Scatter)：指将从channel中读取的数据写入到多个Buffer。
- 收集(Gather)：指将多个Buffer中的数据写入同一个channel。

<font style="background-color:#02aaf4">适用场景</font>

用于传输数据需要分开的场合。例如传输一个由消息头和消息体组成的消息，可以将消息头和消息体分别分散到不同的Buffer，这样可以方便处理。

<font style="background-color:#02aaf4">分散读</font>

```java
FileChannel channel = FileChannel.open(Paths.get("1.txt"), StandardOpenOption.READ);
ByteBuffer header = ByteBuffer.allocate(4);
ByteBuffer body = ByteBuffer.allocate(100);
ByteBuffer[] messages = {header, body};
channel.read(messages);
System.out.printf("header : %s; body : %s;\n",
                  new String(header.array()), new String(body.array()));
```

首先将多个Buffer组装成一个数组，然后作为channel#read()的参数。read()方法安照Buffer在数组中的顺序将数据从channel中写入Buffer，当一个Buffer被写满之后，紧接着向下一个Buffer中写入。

由于Scatter Reads在向下一个Buffer写入之前，当前Buffer必须被写满，因此，这表明Scatter Reads不适用于动态消息（消息大小不固定）。例如，如果整个消息存在消息头和消息体，并且需要消息头和消息体分别写入不同的Buffer，那么消息头大小必须固定，如果不固定，则必须填充为固定大小。

<font style="background-color:#02aaf4">收集写</font>

```java
FileChannel channel = FileChannel.open(Paths.get("1.txt"), StandardOpenOption.WRITE);
ByteBuffer header = ByteBuffer.wrap("PENG".getBytes());
ByteBuffer body = ByteBuffer.wrap("我是一只小小鸟".getBytes());
ByteBuffer[] messages = {header, body};
channel.write(messages);
```

首先将多个Buffer组装成一个数组，然后作为channel#write()的参数。write()方法安照Buffer在数组中的顺序将数据写入channel。

注意，只有position和limit之间的数据才会被写入，如果一个Buffer的容量为128字节，但只包含58字节的数据，那么只有这58字节的数据才会被写入channel。因此，与Scatter Reads相反，Gather Writes能较好的支持动态消息。





### SelectableChannel





```java
public abstract class SelectableChannel extends AbstractInterruptibleChannel
    														implements Channel {
    public abstract SelectorProvider provider();

    public abstract int validOps();

    // Internal state:
    //   keySet, may be empty but is never null, typ. a tiny array
    //   boolean isRegistered, protected by key set
    //   regLock, lock object to prevent duplicate registrations
    //   boolean isBlocking, protected by regLock

    public abstract boolean isRegistered();
    //
    // sync(keySet) { return isRegistered; }

    public abstract SelectionKey keyFor(Selector sel);
    //
    // sync(keySet) { return findKey(sel); }

    public abstract SelectionKey register(Selector sel, int ops, Object att)
        throws ClosedChannelException;
    //
    // sync(regLock) {
    //   sync(keySet) { look for selector }
    //   if (channel found) { set interest ops -- may block in selector;
    //                        return key; }
    //   create new key -- may block somewhere in selector;
    //   sync(keySet) { add key; }
    //   attach(attachment);
    //   return key;
    // }

    public final SelectionKey register(Selector sel, int ops)
        										throws ClosedChannelException {
        return register(sel, ops, null);
    }

    public abstract SelectableChannel configureBlocking(boolean block)
        throws IOException;
    //
    // sync(regLock) {
    //   sync(keySet) { throw IBME if block && isRegistered; }
    //   change mode;
    // }

    public abstract boolean isBlocking();

    public abstract Object blockingLock();

}
```



### ServerSocketChannel



```java
public abstract class ServerSocketChannel extends AbstractSelectableChannel
    												implements NetworkChannel {
    protected ServerSocketChannel(SelectorProvider provider) {
        super(provider);
    }

    public static ServerSocketChannel open() throws IOException {
        return SelectorProvider.provider().openServerSocketChannel();
    }

    public final int validOps() {
        return SelectionKey.OP_ACCEPT;
    }


    // -- ServerSocket-specific operations --

    public final ServerSocketChannel bind(SocketAddress local) throws IOException {
        return bind(local, 0);
    }

    public abstract ServerSocketChannel bind(SocketAddress local, int backlog)
        throws IOException;

    public abstract <T> ServerSocketChannel setOption(SocketOption<T> name, T value)
        throws IOException;

    public abstract ServerSocket socket();

    public abstract SocketChannel accept() throws IOException;

    @Override
    public abstract SocketAddress getLocalAddress() throws IOException;

}
```

## Selector

Selector(选择器)是Java NIO中能够检测一到多个NIO通道，并能够监听通道读写等事件是否准备好的组件。通过Selector，一个单独的线程可以管理多个通道，从而管理多个网络连接。

### Selector创建

Selector提供了静态方法`open()`用于创建选择器，实际上其还是通过`java.nio.channels.spi.SelectorProvider`的`openSelector()`方法创建的。

open()方法源码：

```java
public static Selector open() throws IOException {
    return SelectorProvider.provider().openSelector();
}
```

示例 - 创建Selector：

```java
Selector selector = Selector.open();
```

### 注册通道到Selector

java.nio.channels.SelectableChannel提供了两个重载的register方法用于将Channel注册到Selector。只有当Channel注册到Selector之后，Selector才能监听通道的事件。

register()方法源码：

```java
public abstract SelectionKey register(Selector sel, int ops, Object att)
    throws ClosedChannelException;

public final SelectionKey register(Selector sel, int ops)
    throws ClosedChannelException 
{
    return register(sel, ops, null);
}
```

示例 - 将channel注册到selector：

```java
// 打开一个selector
Selector selector = Selector.open();
// 将通道注册到selector，并监听ACCEPT事件
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
```

### 监听事件

Selector通过了三个方法用于监听就绪事件，分别是：

1. int selectNow()：监听通道就绪事件，不管是否有通道就绪，都不阻塞直接返回。
2. int select(long timeout)：监听通道就绪事件，如果没有通道准备就绪，将阻塞timeout毫秒，然后返回。
3. int select()：监听通道就绪事件，如果没有通道准备就绪，将一直阻塞，直到有通道准备就绪为止。

三个select方法都返回一个int类型的数值，表示自上次调用select之后有多少通道有事件准备就绪。

### 获取事件

Selector提供了keys()和selectedKeys()获取注册的通道事件：

- `Set<SelectionKey> keys()`：获取当前Selector上的所有通道的选择键（无论是否有事件准备就绪）。
- `Set<SelectionKey> selectedKeys()`：获取当前Selector上的所有准备就绪的通道事件的选择键。

### wakeup()

wakeup()方法的作用是使第一个尚未返回的select()操作立即返回。

wakeup与select跟`LockSupport`的`park()`和`unpark(Thread)`的关系是一样的。

当某个线程调用select()方法阻塞了（没有通道准备就绪），在另一个线程调用wakeup()方法将会使其立即返回。如果提前调用了wakeup()方法，再调用select()方法（没有通道准备就绪），select()将会立刻“醒来”，不会阻塞。

### 关闭Selector

当Selector使用完毕之后，调用`close()`方法关闭选择器。`close()`方法是所有注册到Selector上的通道的`SelectionKey`无效，但通道本身不会关闭。

### Selector类

```java
public abstract class Selector implements Closeable {
    public static Selector open() throws IOException {
        return SelectorProvider.provider().openSelector();
    }
    public abstract boolean isOpen();
    public abstract SelectorProvider provider();
    public abstract Set<SelectionKey> keys();
    public abstract Set<SelectionKey> selectedKeys();
    public abstract int selectNow() throws IOException;
    public abstract int select(long timeout) throws IOException;
    public abstract int select() throws IOException;
    public abstract Selector wakeup();
    public abstract void close() throws IOException;
}
```

## SelectionKey

一个令牌，表示向selector注册一个SelectableChannel。

每次向selector注册通道时都会创建选择键。在通过调用键的cancel方法、关闭它的channel或关闭它的selector来取消它之前，键仍然有效。取消一个键并不会立即将其从选择器中移除；相反，它被添加到选择器的取消键集，以便在下一个选择操作期间删除。可以通过调用键的isValid方法来测试键的有效性。

选择键包含两个以整数值表示的操作集。操作集的每一位都表示键通道支持的一类可选择操作。

- interest set确定在下一次调用选择器的选择方法之一时，将测试哪些操作类别以备就绪。使用创建键时给出的值初始化兴趣集；它可能稍后会通过`interestOps(int)`方法进行更改。

- 就绪集标识键选择器检测到的键通道已经准备就绪的操作类别。当创建键时，ready set被初始化为0；它可能稍后在选择操作期间由选择器更新，但不能直接更新。

选择键的就绪集表明它的通道为某个操作类别准备好了，这是一个提示，但不是保证，线程可以在不导致线程阻塞的情况下执行此类类别中的操作。在选择操作完成后，一个现成的集最有可能是准确的。外部事件和在相应通道上调用的I/O操作很可能使其变得不准确。

这个类定义了所有已知的操作集位，但确切地说，给定通道支持哪些位取决于通道的类型。`SelectableChannel`的每个子类都定义了一个`validOps()`方法，该方法返回一个标识通道支持的操作集。如果试图设置或测试key通道不支持的操作集位，将导致适当的运行时异常。

通常需要将一些特定于应用程序的数据与选择键关联起来，例如表示高级协议状态并处理准备通知以实现该协议的对象。因此，选择键支持将单个任意对象附加到键。可以通过`attach`方法附加对象，然后再通过`attachment`方法检索对象。

选择键对于多个并发线程使用是安全的。通常，读写兴趣集的操作将与选择器的某些操作同步。具体如何执行这个同步取决于实现：在一个简单的实现中，如果一个选择操作已经在进行中，读取或写入兴趣集可能会无限阻塞；在高性能实现中，读或写兴趣集可能会短暂阻塞(如果阻塞的话)。在任何情况下，选择操作将始终使用操作开始时的当前interest-set值。。

SelectionKey源码

```java
public abstract class SelectionKey {
    // -- Channel and selector operations --
    public abstract SelectableChannel channel();
    public abstract Selector selector();
    public abstract boolean isValid();
    public abstract void cancel();
    // -- Operation-set accessors --
    public abstract int interestOps();
    public abstract SelectionKey interestOps(int ops);
    public abstract int readyOps();
    // -- Operation bits and bit-testing convenience methods --
    public static final int OP_READ = 1 << 0;
    public static final int OP_WRITE = 1 << 2;
    public static final int OP_CONNECT = 1 << 3;
    public static final int OP_ACCEPT = 1 << 4;
    public final boolean isReadable() {
        return (readyOps() & OP_READ) != 0;
    }
    public final boolean isWritable() {
        return (readyOps() & OP_WRITE) != 0;
    }
    public final boolean isConnectable() {
        return (readyOps() & OP_CONNECT) != 0;
    }
    public final boolean isAcceptable() {
        return (readyOps() & OP_ACCEPT) != 0;
    }
    // -- Attachments --
    private volatile Object attachment = null;
    private static final AtomicReferenceFieldUpdater<SelectionKey,Object>
        attachmentUpdater = AtomicReferenceFieldUpdater.newUpdater(
            SelectionKey.class, Object.class, "attachment"
        );
    public final Object attach(Object ob) {
        return attachmentUpdater.getAndSet(this, ob);
    }
    public final Object attachment() {
        return attachment;
    }
}
```

### Channel和selector操作

- `SelectableChannel channel()`

返回为其创建此键的通道。该方法将继续返回通道，即使键被取消。

- `Selector selector()`

返回为其创建此键的选择器。此方法将继续返回选择器，即使键被取消。

- `boolean isValid()`

测试该键是否有效。键在创建时是有效的，并一直保持有效，直到它被取消、其通道被关闭或其选择器被关闭。

- `void cancel()`

请求取消对该键的通道及其选择器的注册。一旦返回，该键将是无效的，并将被添加到其selector的取消键集。在下一次选择操作期间，该键将从所有选择器的键集中删除。

如果此键已被取消，则调用此方法无效。一旦取消，key将永远无效。

这个方法可以在任何时候被调用。它对selector的取消键集进行同步，因此如果同时调用涉及同一选择器的取消或选择操作，可能会短暂阻塞。

### 操作集访问

- `int interestOps()`

检索此键的兴趣集。

它保证返回的集合将只包含对该键的通道有效的操作位。

这个方法可以在任何时候被调用。它是否阻塞以及阻塞多长时间取决于实现。

- `SelectionKey interestOps(int ops)`

将该键的interest设置为给定的值。

这个方法可以在任何时候被调用。它是否阻塞以及阻塞多长时间取决于实现。

- `readyOps()`

检索此键的就绪操作集。它保证返回的集合将只包含对该键的通道有效的操作位。

### 操作位和位测试

`OP_READ`：用于读取操作的操作集位。

假设选择键的interest-set在选择操作开始时包含OP_READ。如果selector检测到相应的通道已经准备好进行读取，已经到达流的末尾，已经远程关闭以进行进一步读取，或者有一个等待读取的错误，那么它将把OP_READ添加到键的就绪操作集，并将该键添加到其选择键集。

`OP_WRITE`：用于写操作的操作集位。

假设选择键的interest-set在选择操作开始时包含OP_WRITE。如果selector检测到相应的通道已经准备好写入，已经远程关闭以进行进一步写入，或者有一个等待的错误，那么它将把OP_WRITE添加到键的就绪集，并将键添加到其选择的键集。

`OP_CONNECT`：套接字连接操作的操作集位。

假设选择键的interest-set在选择操作开始时包含OP_CONNECT。如果selector检测到相应的套接字通道已经准备好完成它的连接序列，或者有一个等待处理的错误，那么它将把OP_CONNECT添加到键的就绪集，并将键添加到它的已选择键集。

`OP_ACCEPT`：套接字接受操作的操作集位。

假设选择键的interest-set在选择操作开始时包含OP_ACCEPT。如果selector检测到相应的服务器-套接字通道已经准备好接受另一个连接，或者有一个等待处理的错误，那么它将将OP_ACCEPT添加到密钥的就绪集，并将密钥添加到其选择的密钥集。

#### interest-set

interest-set实质上是一个int类型的数值，其集合元素是通过位来表示。

interest-set定义在SelectionKeyImpl类中，成员变量interestOps表示感兴趣的interest-set，readyOps表示就绪的interest-set。

```java
public class SelectionKeyImpl extends AbstractSelectionKey {
    final SelChImpl channel;
    public final SelectorImpl selector;
    private int index;
    private volatile int interestOps;
    private int readyOps;
    ......
}
```

集合元素为定义在SelectionKey类中的四个常量，如下：

```java
public static final int OP_READ = 1 << 0;  //aka 1
public static final int OP_WRITE = 1 << 2;  //aka 4
public static final int OP_CONNECT = 1 << 3;  //aka 8
public static final int OP_ACCEPT = 1 << 4;  //aka 16
```

假设SelectionKey感兴趣所有interest-set，那么interestOps属性的值就是29（十进制），二进制表示为0001 1101。

因此，通过位运算可以判断是否有对应就绪的interest，例如，如果想要判断是否有`OP_READ`，那么可以通过`readyOps & OP_READ !=0`来判断。

SelectionKey提供了四个Boolean方法，用来判断interest-set中是否有对应的interest，代码如下：

```java
public final boolean isReadable() {
    return (readyOps() & OP_READ) != 0;
}
public final boolean isWritable() {
    return (readyOps() & OP_WRITE) != 0;
}
public final boolean isConnectable() {
    return (readyOps() & OP_CONNECT) != 0;
}
public final boolean isAcceptable() {
    return (readyOps() & OP_ACCEPT) != 0;
}
```

### 附件

通常情况下当SelectionKey产生就绪操作集时，需要根据不同的就绪操作执行不同的逻辑处理。可以将这一段处理操作与SelectionKey关联起来。SelectionKey提供了`attach`方法附加对象，但产生就绪操作时，通过`attachment`方法检索对象，用以执行逻辑处理。

示例：

```java
selectionKey.attach(Object);
Object attachment = selectionKey.attachment();
```

另外，附件的添加还可以通过在Channel向Selector注册时附加，接口如下：

```java
public abstract SelectionKey register(Selector sel, int ops, Object att)
    throws ClosedChannelException;
```

其中，第三个参数`att`就是附加对象。

示例：

```java
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT,Object);
```

## 零拷贝

Linux操作系统中传统的I/O操作是基于数据的拷贝操作的，即I/O操作会将数据在内核空间的内核缓冲区和用户空间的用户缓冲区之间相互拷贝传输。这样做的好处在于可以减少磁盘I/O操作，因为如果所需读取的数据已经存储在操作系统的高速缓冲存储器中，那么就不需要再进行实际的物理磁盘I/O操作。

### read+write

传统的I/O操作模型：

![](images/IO-Linux-Model-tradition.png)

如上图所示，传统的I/O操作需要进行4次内核态与用户态的上下文切换，以及4次拷贝（两次CPU和两次DMA拷贝）。

流程如下：

传入文件描述符，应用进程切换至内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。

1. 进程调用read()系统调用，应用进程由用户态切换为内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。
2. CPU将数据从内核缓冲区拷贝至用户缓冲区，read()系统调用返回，应用进程由内核态切换为用户态。
3. 进程调用write()系统调用，应用进程由用户态切换为内核态，CPU将数据从用户缓冲区拷贝至内核缓冲区（数据被拷贝到另一个不同的缓冲区，这个缓冲区与套接字相关）。write()调用完毕返回，应用进程由内核态切换为用户态。
4. DMA将Socket缓冲区中的数据拷贝至网卡缓冲区。

> 在第三步write()系统调用一旦将数据从用户缓冲区拷贝至内核缓冲区就立即返回，而并不会等待第四步DMA拷贝完毕，因此write()系统调用在返回时并不能保证数据已经传输出去，甚至不能保证数据已经开始传输。write()系统调用的返回只是意味着以太网驱动程序在其队列中有空闲的fd并接受了当前传输的数据，在这之前可能会有很多其他数据包正在排队。除非驱动/硬件实现了优先级环或队列，否则数据将以FIFO（先入先出）的方式传输。

### mmap+write

传统的I/O操作虽然减少了磁盘I/O的操作，但所带来的问题是，数据传输过程中的数据拷贝操作导致了极大的性能开销，同时也带来了多次用户态和内核态之间上下文的切换。这些问题限制了操作系统有效进行数据传输操作的能力。

为了解决这些问题，Linux系统提供了mmap()系统调用，**通过mmap()可以消除内核缓冲区和用户缓冲区的数据拷贝**。

基于mmap的I/O操作：

![](images/IO-Linux-Model-mmap.png)

如上图所示，基于mmap()+write()的I/O操作操作需要进行4次内核态与用户态的上下文切换，以及3次拷贝（一次CPU和两次DMA拷贝）。

流程如下：

1. 进程调用mmap()系统调用，应用进程由用户态切换为内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。然后mmap()系统调用返回，应用进程由内核态切换为用户态。用户进程与内核共享缓冲区，因此不再需要CPU将数据从内核缓冲区拷贝至用户缓冲区。
2. 进程调用write()系统调用，应用进程由用户态切换为内核态，CPU将数据从内核缓冲区拷贝至Socket缓冲区。write()调用完毕返回，应用进程由内核态切换为用户态。
3. DMA将Socket缓冲区中的数据拷贝至网卡缓冲区。

通过mmap减少了一次缓冲区数据的拷贝，无疑提升了效率。但是使用mmap也是有代价的。

**mmap的缺陷**

mmap()系统调用的作用是映射文件或设备到内存。当通过mmap()使内存映射一个文件，然后调用write()系统调用，此时，文件被另一个进程截断，write()系统调用会因为访问非法地址而被SIGBUS信号终止，SIGBUS信号默认会杀死进程并产生一个coredump。

解决方案：

- 为SIGBUS信号建立信号处理程序

  当遇到SIGBUG信号时，信号处理程序简单地返回。write()系统调用将返回在被中断之前已经写入的字节数，并将errno设置为success（这是一种糟糕的解决方式，因为只治了标，没有治本）。

- 使用文件租借锁

  在文件描述符上添加租借锁，当其他进程想要截断这个文件时，该进程会被阻塞。同时内核会向持有租借锁的进程发送一个实时的`RT_SIGNAL_LEASE`信号，告知有其他进程正在访问该文件，在程序访问非法地址而被SIGBUS信号终止之前，write()系统调用会被中断，然后返回已经写入的字节数，并将errno设置为success。

> <font style="background-color:#02aaf4">租借锁</font>
>
> 当进程尝试打开一个被租借锁保护的文件时，该进程会被阻塞，同时，在一定时间内拥有该文件租借锁的进程会收到一个信号。收到信号之后，拥有该文件租借锁的进程会首先更新文件，从而保证了文件内容的一致性，接着，该进程释放这个租借锁。如果拥有租借锁的进程在一定的时间间隔内没有完成工作，内核就会自动删除这个租借锁或者将该锁进行降级，从而允许被阻塞的进程继续工作。
>
> 系统默认的这段间隔时间是 45 秒钟，定义如下：
>
> ```c
> 137 int lease_break_time = 45; 
> ```
>
> 这个参数可以通过修改`/proc/sys/fs/lease-break-time`进行调节（当然，`/proc/sys/fs/leases-enable`必须为`1`才行）。

### sendfile

Linux提供了[sendfile()](https://man7.org/linux/man-pages/man2/sendfile.2.html)系统调用，其作用是在文件描述符之间传递数据。sendfile()系统调用首次出现在Linux 2.2。

sendfile()简介：

```c
#include <sys/sendfile.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

基于sendfile的I/O操作：

![](images/IO-Linux-Model-sendfile1.png)

如上图所示，基于sendfile()系统调用的I/O操作操作需要进行2次内核态与用户态的上下文切换，以及3次拷贝（一次CPU和两次DMA拷贝）。

流程如下：

1. 进程调用sendfile()系统调用，传入文件描述符，应用进程切换至内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。
2. CPU将缓冲区中的数据拷贝至Socket缓冲区。
3. DMA将Socket缓冲区中的数据拷贝至网卡缓冲区。

sendfile()的优点很明显，相对mmap()又减少了两次内核态的切换。但是它的优点也是它的缺点所在，即不能对数据进行修改，只能用于两个不同I/O源的数据传输。

> <font style="background-color:#02aaf4">sendfile()系统调用被截断</font>
>
> 与mmap()系统调用一样，在调用时所访问的文件也可能被其他线程截断。对于这种情况，sendfile()系统调用有两种响应：
>
> 1. 如果不注册任何信号处理信号，在被其他线程截断时，sendfile()系统调用会返回被中断之前已经写入的字节数，并将errno设置为success。
> 2. 如果在调用sendfile()系统调用之前，获的了文件的租借锁，那么在被其他线程截断时，sendfile()系统调用会返回被中断之前已经写入的字节数，并将errno设置为success，同时还会收到`RT_SIGNAL_LEASE`信号。

### sendfile+SG-DMA

在Linux 2.4提供了，进一步消除了内核缓冲区内的一次数据拷贝，而这才是真正意义上的“**零拷贝**”。

基于sendfile+SG-DMA的I/O操作：

![](images/IO-Linux-Model-sendfile2.png)

如上图所示，基于sendfile()+SG-DMA系统调用的I/O操作操作需要进行2次内核态与用户态的上下文切换，以及2次拷贝（一次DMA和一次SG-DMA拷贝）。

流程如下：

1. 进程调用sendfile()系统调用，传入文件描述符，应用进程切换至内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。
2. 将缓冲区文件描述符（包含数据长度和位置等信息）传输给Socket缓冲区，SG-DMA通过这些信息直接将内核缓冲区中的数据直接拷贝至网卡缓冲区。

sendfile+SG-DMA相对sendfile进一步减少了内核缓冲区中的一次数据拷贝。但是它强依赖硬件的支持，并且同样不支持对数据进行修改。

### splice 

Linux提供了[splice()](https://man7.org/linux/man-pages/man2/splice.2.html)系统调用，其作用是在内核缓冲区之间建立管道，将数据与管道拼接。splice()系统调用首次出现在Linux 2.6.17。

splice()简介：

```c
#define _GNU_SOURCE         /* See feature_test_macros(7) */
#include <fcntl.h>

ssize_t splice(int fd_in, off64_t *off_in, int fd_out,
               off64_t *off_out, size_t len, unsigned int flags);
```

基于splice()的I/O操作：

![](images/IO-Linux-Model-splice.png)

如上图所示，基于splice()系统调用的I/O操作操作需要进行2次内核态与用户态的上下文切换，以及2次拷贝（一次DMA和一次SG-DMA拷贝）。

流程如下：

1. 进程调用splice()系统调用，传入文件描述符，应用进程切换至内核态，并通过DMA将磁盘上的数据拷贝至内核缓冲区。

1. CPU在内核空间的读缓冲（read buffer）区和Socket缓冲区（socket buffer）之间建立管道（pipeline）。
2. DMA将Socket缓冲区（socket buffer）中的数据拷贝至网卡缓冲区。
3. 上下文从内核态（kernel space）切换回用户态（user space），splice()系统调用执行返回。

splice()系统调用不需要硬件的支持，同时跟sendfile+SG-DMA一样实现了两个文件描述符之间的零拷贝。不过splice()同样不支持对数据进行修改。

splice()系统调用使用了Linux的管道缓冲机制，可以用于任意两个文件描述符之间的数据传输，但是这个两个文件描述符参数必须有一个是管道设备。

### 总结

五种I/O操作特性：

| I/O操作模型     | 硬件支持 | 数据修改 | 内核切换次数 | 拷贝次数 | 必要条件                               | 适用范围                       |
| --------------- | -------- | -------- | ------------ | -------- | -------------------------------------- | ------------------------------ |
| read+write      | 需要     | 可以     | 4            | 4        | 无                                     | 数据修改读写。                 |
| mmap+write      | 需要     | 可以     | 4            | 3        | 无                                     | 数据修改读写。                 |
| sendfile        | 需要     | 不可以   | 2            | 3        | 无                                     | 两个文件描述符之间的数据传输。 |
| sendfile+SG-DMA | 需要     | 不可以   | 2            | 2        | 无                                     | 两个文件描述符之间的数据传输。 |
| splice          | 不需要   | 不可以   | 2            | 2        | 两个文件描述符参数必须有一个是管道设备 | 两个文件描述符之间的数据传输。 |







https://blog.csdn.net/m0_45364328/article/details/124832572



注意：这里向从Reactor注册的只是read事件，并没有注册write事件，因为read事件是由epoll内核触发的，而write事件则是由用户业务线程触发的（什么时候发送数据是由具体业务线程决定的），所以write事件理应是由用户业务线程去注册。

用户线程注册write事件的时机是只有当用户发送的数据无法一次性全部写入buffer时，才会去注册write事件，等待buffer重新可写时，继续写入剩下的发送数据、如果用户线程可以一股脑的将发送数据全部写入buffer，那么也就无需注册write事件到从Reactor中。





Redis 单Reactor单线程

Nginx 多Readtor多进程

Memchche and Netty 对Reactor多线程

select、poll、epoll、kqueue

惊群效应



![](images/IO-Model-SIGIO.png)

信号驱动I/O是指用户进程预先告知内核，向内核注册一个号处理函数，然后用户进行返回不阻塞。当内核缓冲区数据准备就绪时通过注册的信号处理函数回调进程，当用户进程收到通知后便调用recvfrom读取数据。

特点：并不符合异步I/O的要求，只能算是伪异步，并且实际并不常用。

典型应用：应用场景较少。

优点：

缺点：实现和开发难度较大。

![](images/IO-Model-Async.png)

用户进程向内核发起aio_read操作，传递与read操作相同的描述符、缓冲区指针、缓冲区大小以及文件偏移等参数，内核收到aio_read请求之后直接返回不阻塞。内核将等到数据包准备好，并将数据从内核拷贝到用户空间。当这一切都完成之后，内核会给用户进程发送一个信号，通知aio_read操作完成。

特点：真正实现了异步I/O，是五中I/O模型中唯一的异步模型。

典型应用：Java 7 AIO，高性能服务器应用。

优点：

- 不阻塞，数据一步到位，采用Proactor模式。

- 非常适合高性能，高并发应用。

缺点：

- 需要操作系统底层支持，Linux2.5内核首现，Linux2.6产品的内核标准特性。

- 实现和开发应用难度大。





## 总结

本文所描述的五种I/O模型中，其中前四种是属于同步I/O操作。它们的区别在于第一阶段，而第二阶段都是一样的（将数据从内核空间拷贝到用户空间期间，进程阻塞与recvfrom调用）。

*各I/O模型的阻塞状态对比*

![](images/IO-Model-Compare.png)



### 同步和异步

同步和异步是指CPU时间片的利用，主要看请求发起方<span alt="emp">对消息结果的获取</span>是主动发起的还是被动等待通知的。如果对应答结果的获取是由请求方主动发起，则是同步；如果对应答结果的获取是由服务方主动通知，则是异步。

- 同步：请求方主动发起，获取应答结果。
- 异步：请求方主动发起，服务方同步应答结果。

### 阻塞和非阻塞

阻塞和非阻塞在计算机的世界里，通常是针对I/O的操作，如网络I/O和磁盘I/O等。请求方调用服务方，在等待返回结果之前，当前线程/进程是处于挂起状态还是运行状态。

阻塞：请求方调用服务方，在返回结果之前，当前线程/进程处于挂起状态。

非阻塞：请求方调用服务方，在返回结果之前，当前线程/进程处于运行状态。



同步和异步以及阻塞和非阻塞可以组成四种情况：

- 同步阻塞：请求方主动发起，一直等待应答结果。

- 同步非阻塞：请求方主动发起，但需要不断轮询获取应答结果。

上述两种情形由于不管如何都需要请求方主动发起获取应答结果，因此其形式上是同步操作。

- 异步阻塞：请求方发出请求后，一直等到服务方通知。
- 异步非阻塞：请求方发出请求后，不需要等待服务方通知，当服务方准备完毕之后，通知请求方。

上述两种情形由于不管如何都需要服务方主动通知应答结果，因此其形式上是异步操作。





