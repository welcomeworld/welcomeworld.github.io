# ByteBuffer详解
@[toc]
## 概述
ByteBuffer顾名思义就是byte缓冲区，实际上**底层就是byte[ ]**。ByteBuffer有两种，一种是HeapByteBuffer，还有一种是DirectByteBuffer。我们知道ByteBuffer实际就是一块内存，前者是属于JVM的内存，后者则是申请的JVM以外的内存，也就是运行JVM的系统的内存。没有特殊需求一般都是用HeapByteBuffer吧（大概，毕竟我也没工作，不知道大佬们的世界）。
## ByteBuffer属性
### capacity
容量（capacity），由我们根据需要指定，也就是这个ByteBuffer在内存中分配的（也即是可使用的最大）空间。比如我们`new byte[233];`这里233就是capacity。类比数组，capacity在指定后不允许改变。`capacity()`有参则是setter，无参则是getter，下面的limit和position属性同理。mark则稍有不同。
### limit
界限（limit，原谅我不能信达雅），初始化时等于capacity，我们申请了capacity那么大的空间，但是我们未必时时刻刻都有那么大的数据。java就是用limit来界定我们的数据界限在哪，比如limit=10，capacity=20，那么10到20之间就是空闲空间（但是我们也不能直接使用这片空间）。我们所有的读写操作都受limit限制，不能操作超过limit的那片空间。如果我们想用上面那片10到20之间的空间，那么我们就要使limit等30。然后我们就能操作30之前的空间。limit最大能等于capacity，因为我们就申请了capacity那么大的空间，如果limit超过capacity那就会操作了不属于ByteBuffer，跟数组越界一个道理。记住一句话，我们所有的读写操作都起于position，终于limit，最大能等于capacity。
### position
定位（position），初始化时等于0，指向我们下一个要操作的位置，有点类似PC指针。无论我们是要读还是写，操作的都是position指向的位置，操作完position就加一，指向你下一个要操作的位置。position最大能等于limit，此时就不能再操作了，因为你操作完position加一就大于limit了，那就越界了，会报` java.nio.BufferOverflowException`异常。
### Mark
标记（Mark），初始化时等于-1，类似书签，我们在某时刻调用`mark()`方法，那么mark将等于position。然后我们想回到我们mark的地方时可以调用`reset()`方法。网络用语马克就是mark音译。
## ByteBuffer方法
### allocate()

> public static ByteBuffer allocate(int capacity)
Allocates a new byte buffer.
The new buffer's position will be zero, its limit will be its capacity, its mark will be undefined, and each of its elements will be initialized to zero. It will have a backing array, and its array offset will be zero.

ByteBuffer初始化方法，参数是int，也就capacity。比如`ByteBuffer byteBuffer=ByteBuffer.allocate(20);`就是为ByteBuffer开辟了一块20字节大小的内存空间。注意指定capacity后就不能更改了，除非你重新new一个ByteBuffer，不过那就是另外一个ByteBuffer了。
### order()
> public final ByteBuffer order(ByteOrder bo)
Modifies this buffer's byte order.
Parameters:
bo - The new byte order, either BIG_ENDIAN or LITTLE_ENDIAN

ByteBuffer是有字节序的，所谓字节序就是把字节放进ByteBuffer的顺序。比如我们ByteBuffer的内存地址是0xFFFF 0000到0xFFFF 00FF，我们现在要把一个byte数组放进ByteBuffer，那么我们是从0xFFFF 00FF开始放起，还是从0xFFFF 0000开始放起呢？ByteOrder就是指明我们按照什么顺序放的。byte还看不出什么不同，但是当我们读写那些不止一个字节的对象时就体现出不同了，比如读int类型从0xFFFF 0000到0xFFFF 0004跟从0xFFFF 0004到0xFFFF 0000完全就是两个不同的数。`order()`有参数就是设置字节序，没参数就是返回ByteBuffer当前字节序。不过这个感觉一般来说不用管。
### put()

> public abstract ByteBuffer put(byte b)
Relative put method  (optional operation).
Writes the given byte into this buffer at the current position, and then increments the position.

就是把数据放进ByteBuffer的方法，有多种变种，比如`put(byte),put(byte[]),put(byte[] src,int offset,int length),put(ByteBuffer src)`等等。你写入多少个字节的数据position就加多少。
### flip()
> public final Buffer flip()
Flips this buffer. The limit is set to the current position and then the position is set to zero. If the mark is defined then it is discarded.

flip()方法就是把limit设置成当前position，然后position=0，附带把mark置-1，也就是舍弃你可能有的标记。简单来说这个方法的作用就是把写模式变成读模式。比如你初始化一个ByteBuffer，此时position=0，limit=capacity。然后你向其中写入了5个字节的数据（假设capacity大于5），此时position=5，limit=capacity。现在你想把写入的5字节数据读出来，假如你直接读，那么你会读到位与5的数据，然后position=6。这显然不符合我们的需求。我们需要把position置0，因为我们希望下个操作的数据位于0位置。这样我们能正确读出那五个数据了，但是我们没有设置limit，那么我们完全有可能读到5以后的数据。这样也是不行的，那么我们就需要把limit设置成5，也就是写完后position的位置。这样我们就只会读5个字节的数据，觉不会多读。
### array()
> public final byte[] array()
Returns the byte array that backs this buffer  (optional operation).
Modifications to this buffer's content will cause the returned array's content to be modified, and vice versa.
Invoke the hasArray method before invoking this method in order to ensure that this buffer has an accessible backing array.

前面说过ByteBuffer实质就是一个`byte[]`，这个方法就是返回ByteBuffer底层的byte数组，也就是说你对ByteBuffer的所有操作实质就是操作这个byte数组，反之亦然。需要注意的是操作数组不会影响ByteBuffer的position，limit，mark等属性，同时对数组的操作和一般数组没有任何区别，不受ByteBuffer那些属性影响。
###get()

> public abstract byte get()
Relative get method. Reads the byte at this buffer's current position, and then increments the position.

用`flip()`或者手动准备好读后，可以使用`get()`方法读取ByteBuffer的数据，和put()方法一样`get()`有很多延伸，可以根据自己需要选取。
### wrap()
> public static ByteBuffer wrap(byte[] array)
Wraps a byte array into a buffer.
The new buffer will be backed by the given byte array; that is, modifications to the buffer will cause the array to be modified and vice versa. The new buffer's capacity and limit will be array.length, its position will be zero, and its mark will be undefined. Its backing array will be the given array, and its array offset> will be zero.

文档说的很清楚了，就是以你给定的字节数组为底层数组构建一个ByteBuffer。

```
//wrap()创建ByteBuffer
byte[] array=new byte[233];
ByteBuffer byteBuffer=ByteBuffer.wrap(array);

//allocate创建ByteBuffer
ByteBuffer byteBuffer=ByteBuffer.allocate(233);
byte[] array=byteBuffer.array();
```
上面两种创建ByteBuffer的方式，前者不会开辟新的内存空间，而是直接使用array的内存空间，同时保留array数组原有数据。而后者会开辟新的内存空间。

### clear()
> public final Buffer clear()
Clears this buffer. The position is set to zero, the limit is set to the capacity, and the mark is discarded.

`clear()`做的就是把position=0，limit=capacity，mark=-1，`clear()`只是移动指针，不改变ByteBuffer里面的数据。此时如果你要写，那么ByteBuffer就相当于空的，如果你要读，那么ByteBuffer就相当于写满数据的。
### rewind()
> public final Buffer rewind()
Rewinds this buffer. The position is set to zero and the mark is discarded.

如上，position=0，mark被舍弃，也就是说你读写到一半或读写完了，现在想从头开始，那就调用`rewind()`方法，从头开始。
### campact()
> public abstract ByteBuffer compact()
Compacts this buffer  (optional operation).
The bytes between the buffer's current position and its limit, if any, are copied to the beginning of the buffer. That is, the byte at index p = position() is copied to index zero, the byte at index p + 1 is copied to index one, and so forth until the byte at index limit() - 1 is copied to index n = limit() - 1 - p. The buffer's position is then set to n+1 and its limit is set to its capacity. The mark, if defined, is discarded.
The buffer's position is set to the number of bytes copied, rather than to zero, so that an invocation of this method can be followed immediately by an invocation of another relative put method.

如上所说，调用campact()方法会把position和limit之间的数据移动到ByteBuffer开始的地方，然后position=limit-position，也就是position接在被转移的数据后面，然后limit=capacity。这个方法的作用是把我们未读完的数据放到前面，以便我们接着后面写入数据，因为我们不能期望一次把所有数据读完。总会出现在还未读完数据的时候需要写入数据的情况，此时如果你不需要未读的那些数据，可以`clear()`,如果需要保留那些数据则`campact()`。
## 最后
其实写博客很难受，要考虑写得对写得全，又要通俗易懂，写到最后我都有点偷工减料了，如果有哪里不清楚，自己写段简单代码验证一下就是了，更多ByteBuffer特性用法请参阅官方API文档。同时如果有哪里写错了还请不吝赐教。最后祝各位能找到自己的真物。