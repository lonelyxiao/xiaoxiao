

# BIO

- 阻塞型IO,同一时间内，线程只能处理一个socket（同一个线程连接到断开期间不能处理其他socket）
- 一个线程对应一个连接,并发量上来了，线程开销就大了
- 线程切换成本高
- 适用于连接数少，短连接的场景（http）

![](../../image/java/netty/20200729222717.jpg)

代码解析

- 来一个连接启动一个线程

```java
@Slf4j
public class TestBio {
    public void createCocket() throws Exception {
        //创建一个线程池，维护连接进来的通讯
        ExecutorService executorService = Executors.newCachedThreadPool();
        //建立一个socket
        ServerSocket serverSocket = new ServerSocket(8000);
        log.debug("服务器启动");
        while (true) {
            Socket accept = serverSocket.accept();
            //来一个连接启动一个线程
            log.debug("建立一个连接");
            executorService.execute(()->{
                handler(accept);
            });
        }
    }

    /**
     * 与客户端通信handler
     * @param socket
     */
    public void handler(Socket socket)  {
        try {
            log.debug("开始接受信息");
            InputStream inputStream = socket.getInputStream();
            while (true) {
                byte[] bytes = new byte[1024];
                int read = inputStream.read(bytes);
                if(read!=-1){
                    log.debug("收到信息：{}", new String(bytes, "UTF-8"));
                } else {
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        new TestBio().createCocket();
    }
}
```

- 测试BIO的端口
  - telnet 127.0.0.1 8000

```shell
Escape 字符为 'CTRL+]'


Microsoft Telnet> send 好的
发送字符串 好的
Microsoft Telnet> send 111
发送字符串 111
Microsoft Telnet>
```

# NIO

- 同步非阻塞（channel连接后，没有发生事件，当前线程可以处理其他的）
- server端启动一个线程，线程维护一个selector
- selector一直轮询，查询通道有没有连接事件(selector负责监听这通道的所有事件)
- 一个selector负责拖个channel
- 也可以有多个线程维护selector
- 适合连接数多且连接比较短，如聊天服务器，弹幕系统，（如果一个channel发送了很长的数据，那么，其他channel可能就要等待很久）

![](../../image/java/netty/20200729223449.png)

## 简介

- NIO有三个核心部分channel(通道)， buffer(缓冲区), selector(选择器)
- 每一个channel对应一个buffer，channel可以通过buffer进行读写，程序只与buffer进行交互

![](../../image/java/netty/20200731085919.png)



# 三大组件

## Buffer

- 当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据

以intbuffer为例

```java
@Slf4j
public class TestBuffer {
    public static void main(String[] args) {
        //创建一个buffer，可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);
        //将i设置进入buffer，将buffer塞满
        for(int i=0; i<intBuffer.capacity(); i++){
            intBuffer.put(i);
        }
        //转化读操作
        intBuffer.flip();
        //判读是否还有数据
        while (intBuffer.hasRemaining()) {
            log.info("取出数据：{}", intBuffer.get());
        }

    }
}
```

### buffer几个参数

```java
//标记
private int mark = -1;
//当前下标
private int position = 0;
//操作过程中，下标不能超过limit
//作用：读数据不能超过这个数，flip时，会将position赋值给limit
private int limit;
//容量
private int capacity;
```

### 分配内存

```java
//class java.nio.HeapByteBuffer
//使用堆内存，读写效率低，受GC影响
ByteBuffer.allocate(5).getClass();
//class java.nio.DirectByteBuffer
//直接内存，读写效率高（少一次拷贝）
ByteBuffer.allocateDirect(5).getClass();    
```

### 存和读

```java
ByteBuffer buffer = ByteBuffer.allocate(10);
//存入数据，也可以用 channel.read(buffer)
buffer.put(new byte[] {'a', 'b', 'c', 'd', 'e'});
buffer.flip();
buffer.get(new byte[4]);
//[pos=4 lim=5 cap=10]
//位置已经读取到了4位置
System.out.println(buffer);
//获取坐标1的数据，但是pos不会动
System.out.println((char)buffer.get(1));
//读取坐标重置
buffer.rewind();
System.out.println((char)buffer.get());
//标记当前位置
buffer.mark();
//中间做N个读取操作后，指针回到mark标记处
buffer.reset();
```

### 字符串和buffer转换

```java
//这些方式都是直接切换到读模式的
ByteBuffer buffer1 = StandardCharsets.UTF_8.encode("hello");
ByteBuffer buffer2 = ByteBuffer.wrap("hello".getBytes());
```

### 分散读集中写

### 黏包半包

网络发送三条数据

hello \n

world \n

zhangsang \n

结果变成了下面的两条(黏包，半包)

hello\nworld\nzhan

gsang\n



### MappedByteBuffer

能够直接在内存中进行修改，操作系统不需要再拷贝一次

### Buffer 分散聚合

- scattering : 将数据写入buffer时，采用buffer数组依次写入
  - channel.read(buffers[])
- gathering： 从buffer读数据，采用buffer数组，依次读
  - channel.write(buffers[])

## Channel

- 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的
- 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入

![](../../image/java/netty/20200803231057.jpg)

- FileChannel  用来对本地文件进行IO操作

举例：使用通道的方式，将目标文件写入文件夹中

### compact方法

- java.nio.ByteBuffer#compact
- 将剩余没读的数据压缩，然后切换写模式

### 文件编程

- 通过FilelnputStream获取的channel只能读
- 通过FileoutputStream获取的channel 只能写
- 通过RandomAccessFile是否能读写根据构造RandomAccessFile时的读写模式决定
- channel需要close

### 文件读取

```java
File file = new File("d:\\1.txt");
FileInputStream fileInputStream = new FileInputStream(file);
FileChannel channel = fileInputStream.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(10);
while (true) {
    int read = channel.read(buffer);
    if(read == -1) {
        break;
    }
    //切换读模式
    buffer.flip();
    //查询是否还有数据没读
    while (buffer.hasRemaining()) {
        System.out.println(buffer.get());
    }
    //清空缓存区
    buffer.clear();
}
```

### 文件Copy

- 方式1：

```java
public static void main(String[] args) throws Exception {
    FileInputStream input = new FileInputStream("E:\\1.avi");
    FileChannel channelSource = input.getChannel();
    FileOutputStream outputStream = new FileOutputStream("d:\\1.mp4");
    FileChannel channelTarget = outputStream.getChannel();
    ByteBuffer byteBuffer = ByteBuffer.allocate(5);
    while (true) {
        byteBuffer.clear();
        int read = channelSource.read(byteBuffer);
        if(read == -1){
            //读取完成
            break;
        }
        byteBuffer.flip();
        channelTarget.write(byteBuffer);
    }
    input.close();
    outputStream.close();
}
```

- 方式2
  - 一次，最多传输2g内容
  - 想要优化，可以采用多次传输

```java
try(FileChannel from = new FileInputStream("d:\\1.txt").getChannel();
    FileChannel to = new FileOutputStream("d:\\2.txt").getChannel();
) {
    //操作效率高，底层会调用0拷贝进行优化
    from.transferTo(0,from.size(), to);
} catch (Exception e) {
    e.printStackTrace();
}
```

## Selector

- 一般称 为选择器 ,也可以翻译为 多路复用器 。
- 它是用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个channels,也就是可以管理多个网络链接
- 当有事件发生时，返回select Key 数组，通过selectKey可以获取对应channel

对应方法

```java
int select()：阻塞到至少有一个通道在你注册的事件上就绪了。
int select(long timeout)：和select()一样，但最长阻塞时间为timeout毫秒。
int selectNow()：非阻塞，只要有通道就绪就立刻返回、。
```

```java
//获取有事件发生的key
Set selectedKeys = selector.selectedKeys();
//获取所有注册的key
Set<SelectionKey> keys = selector.keys();

```

### Selector Key

```java
//有新的网络连接
SelectionKey.OP_ACCEPT
//连接已建立
SelectionKey.OP_CONNECT
//读操作
SelectionKey.OP_READ
//写操作
SelectionKey.OP_WRITE
```

# NIO编程

## 无selector示例

- 可以看到，accept是阻塞的，read也是阻塞的
- 如果没有selector的配置，那么线程会一直阻塞，知道有连接产生，有数据发送

```java
ByteBuffer buffer = ByteBuffer.allocate(16);
try (ServerSocketChannel socketChannel = ServerSocketChannel.open();) {
    socketChannel.bind(new InetSocketAddress(80));
    while (true) {
        log.debug("连接中....");
        //连接的过程中是阻塞的，只有有连接了才会进行下一步操作
        SocketChannel channel = socketChannel.accept();
        log.debug("连接完成");
        //read的过程也是阻塞的
        channel.read(buffer);
        buffer.flip();
        log.debug("获取到客户端数据: {}", new String(buffer.array()));
        buffer.clear();
    }
} catch (Exception e) {
}
```

## 非阻塞模式

- 当没有连接时，socketChannel.accept()返回为null
- read方法也变成非阻塞的，只有i>0时，表示有数据读取

```java
socketChannel.configureBlocking(false);
while (true) {
    SocketChannel channel = socketChannel.accept();
    int i = channel.read(buffer);
```

- 非阻塞的缺点：没有连接和数据传输，也一直在循环

## NIO网络编程原理

- 客户端连接，通过serversocketChannel 得到 socketChannel
- socketChannel注册到selector上

![](../../image/java/netty/20200811213415.jpg)

- NIO server

## 多路复用

### 服务器端

ServerSocketChannel：负责监听客户端连接
SocketChannel：负责读写操作

- selector.select(1000):如果没有调用socketChannel.accept()/channel.read(buffer)(**buffer没有可读的时候**)/ key.cancel()方法，它会任务事件还未处理，不进行阻塞
- 为什么要调用keys.remove():因为每次有时间发生，都会往keys里面添加，但是它不会主动删除事件
- 当调用socketChannel.accept()时，他会把他和他关联的selector的key的事件标记为已处理，处理后，无法获取当前关联的channel了
- 客户端正常的端口和异常断开，都会产生读事件，这时，我们需要调用key.cancel()方法来进行处理

```java
ServerSocketChannel socketChannel = ServerSocketChannel.open();
Selector selector = Selector.open();
//绑定一个服务器监听端口
socketChannel.bind(new InetSocketAddress(7070));
//设置为非阻塞
socketChannel.configureBlocking(false);
//将selector与channel关联
//SelectionKey: 之后发生的事件都集中这里
SelectionKey sscKey = socketChannel.register(selector, 0, null);
//注册一个连接事件
sscKey.interestOps(SelectionKey.OP_ACCEPT);
//循环获取连接事件
while (true) {
    //1s没有获取到事件就重新获取
    if(selector.select(1000) == 0) {
        //System.out.println("没有人连接....");
        continue;
    }
    //获取发生的事件集合
    Iterator<SelectionKey> keys = selector.selectedKeys().iterator();
    while (keys.hasNext()) {
        SelectionKey key = keys.next();
        keys.remove();
        //如果是连接事件，注册读事件,并关联一个buffer
        if(key.isAcceptable()){
            //有新的客户端连接，注册一个生成一个channel，
            SocketChannel socketChannelRead = socketChannel.accept();
            socketChannelRead.configureBlocking(false);
            //注册一个读事件(如果接下来需要写，可以注册写事件)
            socketChannelRead.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
        } else if(key.isReadable()) {
            try {
                SocketChannel channel = (SocketChannel) key.channel();
                ByteBuffer buffer = (ByteBuffer) key.attachment();
               int i = channel.read(buffer);
                if(i ==-1) {
                    //客户端正常断开
                    key.cancel();
                } else {
                    System.out.println("客户端传来： "+ new String(buffer.array()));
                }
            } catch (IOException e) {
                //将当前事件消除，否则select()方法会认为这个事件还没有处理
                key.cancel();
            } 
        }
    }
}
```



### 客户端

```java
public static void main(String[] args) throws Exception {
    //打开选择器
    Selector selector = Selector.open();
    //打开套字接通道
    SocketChannel channel = SocketChannel.open();
    //设置非阻塞
    channel.configureBlocking(false);
    //注册通道，设置为链接就绪
    channel.register(selector, SelectionKey.OP_CONNECT);
    //绑定IP，端口
    if(!channel.connect(new InetSocketAddress("127.0.0.1", 7070))){
        while (!channel.finishConnect()) {
            System.out.println("客户端还未连接，不会阻塞，可以做其他事");
        }
    }
    ByteBuffer byteBuffer = ByteBuffer.wrap("hello, 老肖".getBytes());
    channel.write(byteBuffer);
    System.out.println("写入完毕");
}
```

### 边界值处理问题

- 上述代码有编辑值处理问题
  - 如：如果客户端发送的字节超过1024，则服务器端需要分多次接收，这时，则会产生拆包黏包的问题
- 解决方案

1. 服务器端和客户端都约定一个长度，如果超过这个长度，则做两次发送
   1. 弊端：如果数据不超过这个预定长度，则每次都会浪费空间
2. 按分隔符拆分，每一段结束，都给定一个特殊的分隔符，服务器端按照这个特定的分隔符来进行解析
   1. 如： hello word \n 你好\n
   2. 弊端：效率低，每次都要一次一次的去寻找分隔符
3. 没段字符分为两段：头部和数据部，头部标记数据部的长度
   1. 如： 5bytehello  2bytehi

### 事件的附件

- 可以注意到，在注册事件时，我们造了一个bytebuffer,这个buffer是与当前事件关联的

```java
socketChannelRead.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
```

- 如果事件没有读取完，下一次读取，可以对这个bytebuffer进行重新的扩容处理，如：

```java
key.attach(ByteBuffer.allocate(2048))
```

- 获取附件的内容

```java
key.attachment()
```

## NIO和BIO比较

- NIO 以块的方式处理数据，BIO以流的方式处理

# NIO多线程版

- boss线程只处理客户端的accept事件
- 当有读/写事件发生时，交给work线程池处理（一个work线程表示一个selectorkey事件集合）

![](../../image/java/netty/20210615165848.png)

# AIO

- 异步非阻塞
- 异步意味着，在进行读写操作时，线程不必等待结果，而是将来由操作系统来通过回调方式由另外的线程来获得结果

# IO模型

- 阻塞IO(如下的过程，从read到获取数据是一个阻塞过程，用户线程是被阻塞的)

```mermaid
graph LR
用户线程 --read--> linux内核
linux内核 --等待数据--> linux内核
linux内核 --复制数据 --> 用户线程
```

- 非阻塞IO（用户线程发现在等待数据的过程中，是非阻塞的）
- 多路复用
- 异步IO:线程自己获取不到结果，由其他线程去获取结果

# NIO与零拷贝

## 传统IO

传统IO进行了四次拷贝三次切换

- 拷贝过程

1. 从硬盘 经过 DMA 拷贝（**直接内存拷贝**） 到 kernel buffer （内核buferr）
2. 从kernel buffer 经过**cpu 拷贝**到 user buffer ,比如拷贝到应用程序(这个时候我们可以对流进行修改)
3. 从user buffer 拷贝到 socket buffer
4. 从socket buffer 拷贝到 protocol engine 协议栈

- 状态切换
- 用户态---》 内核状 （或者叫着 用户上下文----》 内核上下文）
- 内核状---》 用户状
- 用户状---》 内核状

![](../../image/java/netty/20210615203422.png)

## sendFile优化

零拷贝指没有cpu拷贝,NIO中使用transferTo操作

![](../../image/java/netty/20210615203948.png)

1. DMA拷贝，将数据从硬盘拷贝到kernel buffer
2. DMA拷贝，将数据从kernel buffer拷贝到protocol engine
3. cpu copy 其实拷贝的是一些基本的信息（数据量少）

# Netty介绍

netty是一个基于异步的，基于事件的网络应用框架

- 基于一个事件，这个事件可能是连接，或者断开,然后寻找对应的方法做对应的事情
- 异步：在服务器操作一些事情的时候，客户端不等待。等服务器有响应的时候进行回调

# Netty设计

## 线程模型

- 传统阻塞IO模型
  - 每个客户端都创建线程请求
  - 当服务器端的数据没有可读时，处于阻塞状态
  - 当并发数越大，服务器端线程越多

![](../..\image\java\netty\20210613125141.jpg)

- Reactor模式

Reactor模式是处理并发I/O常见的一种模式，用于同步I/O，其中心思想是将所有要处理的I/O事件注册到一个中心I/O多路复用器上，同时主线程阻塞在多路复用器上，一旦有I/O事件到来或是准备就绪，多路复用器将返回并将相应 `I/O`事件分发到对应的处理器中

- 单reactor单线程

![](../../image/java/netty/20200901084951.jpg)

- 单reactor多线程
  - reactor监控客户端请求
  - 发生连接，穿件一个handler
  - 如果不是连接，则reactor分发连接对应的handler
  - handler只负责响应，不做业务处理
  - worker线程池分配独立线程池完成真正的业务,并将结果返回handler

![](../../image/java/Netty/20200901090022.jpeg)

- 主从reactor多线程
  - 可以让reactor在多线程中运行
  - **main负责连接，sub负责其他**
  - Reactor 主线程 MainReactor 对象通过 Select 监控建立连接事件，收到事件后通过 Acceptor 接收，处理建立连接事件。
  - Acceptor 处理建立连接事件后，MainReactor 将连接分配 Reactor 子线程给 SubReactor 进行处理。
  - SubReactor 将连接加入连接队列进行监听，并创建一个 Handler 用于处理各种连接事件。
  - 当有新的事件发生时，SubReactor 会调用连接对应的 Handler 进行响应。
  - 一个mian可以管理多个sub

![](../image/java/Netty/20200901091012.jpeg)

## Netty模型

### 简单版

- BossGroup 线程维护Selector, 只关注Accecpt。
- 当接收到Accept事件，获取到对应的SocketChannel, 封装成 NIOScoketChannel并注册到Worker 线程(事件循环), 并进行维护。
- 当Worker线程监听到selector 中通道发生自己感兴趣的事件后，就进行处理(就由handler)， 注意handler 已经加入到通道。

![](../../image/java/netty/20200902084032.png)

### 详细版

- 两组group，这两个group都属于NIOEventLoopGroup
  - boss  负责客户端连接
  - worker  负责网络读写
- NIOEventLoop表示不断循环的执行处理任务的 线程, 每个NIOEventLoopGroup都有一个Selector，同于监听其绑定的socket网络通道（channel）
- NIOEventLoopGroup可以有多个NIOEventLoop（多个线程）
- 处理业务时，会使用pipeline

![](../../image/java/netty/20200902084801.jpg)

## TCP代码

- 服务器端

```java
public static void main(String[] args) throws InterruptedException {
    //管理连接
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    EventLoopGroup workGroup = new NioEventLoopGroup();

    //服务器启动对象
    ServerBootstrap bootstrap = new ServerBootstrap();
    //设置两个线程组
    bootstrap.group(bossGroup, workGroup)
            //选择通道类型
            .channel(NioServerSocketChannel.class)
            //设置线程队列得到的连接数
            .option(ChannelOption.SO_BACKLOG, 128)
            //保持活动连接状态
            .childOption(ChannelOption.SO_KEEPALIVE, true)
            //设置一个处理事情的工作handler
        	//ChannelInitializer负责添加别的handler
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel socketChannel) throws Exception {
                    socketChannel.pipeline().addLast(new NettyServerHandler());
                }
            });
    //绑定一个端口并且同步
    ChannelFuture sync = bootstrap.bind(6666).sync();
    //对关闭通道进行监听
    sync.channel().closeFuture().sync();
}
```

- 服务器端业务处理

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {


    /**
     * 从客户端读取数据
     * @param ctx 上下文，含pipeline，channel, 客户端送的数据
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("客户端地址："+ctx.channel().remoteAddress());
        //msg是一个buf类型
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送的消息："+ buf.toString(CharsetUtil.UTF_8));
    }

    /**
     * 读取数据完毕，往客户端发送数据
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //write+flush方法
        //写入缓存，并冲刷
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello : "+ ctx.channel().remoteAddress(), CharsetUtil.UTF_8));
    }
}
```

- 客户端

```java
public static void main(String[] args) throws InterruptedException {
    EventLoopGroup loopGroup = new NioEventLoopGroup();
    try {

        Bootstrap bootstrap = new Bootstrap();
        bootstrap.group(loopGroup)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        socketChannel.pipeline().addLast(new NettyClientHandler());
                    }
                });
        ChannelFuture future = bootstrap.connect("127.0.0.1", 6666).sync();
        future.channel().closeFuture().sync();
    } finally {
        loopGroup.shutdownGracefully();
    }
}
```

- 客户端业务处理

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("启动客户端，发送消息....");
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello serve:", CharsetUtil.UTF_8));
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务端发送： "+ ctx.channel().remoteAddress() + " : " + buf.toString(CharsetUtil.UTF_8));
    }
}
```

## 执行一个初步流程

![](../../image/java/netty/0040.png)

## 源码分析

### 工作线程

- workgroup里面的线程分配客户端是循环分配的过程，如果分配满了，又从头开始分配
- LoopGroup使用EventExecutor来管理线程
- group包含多个NioEventLoop

  - NioEventLoop包含一个selector，一个taskqueue
  - 一个selector可以注册多个NioChannel
  - 每个NioChannel只会绑定他对应的selector上（N:1）

![](../../image/java/netty/20210613160350.png)

### 上下文分析

- 每个context包含一个pipeline和channel
- channel中包含了pipeline

![](../../image/java/netty/20210613162309.png)


- pipeline本质是一个双向链表（包含head和tail）,也包含了channel

## 异步执行

服务器端中，如果handler中的执行业务时间很久，就会与客户端阻塞

1. 解决方案1：使用eventloop中的taskqueue执行(用户程序自定义的普通任务)

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    ctx.channel().eventLoop().execute(() -> {
        try {
            Thread.sleep(10*1000);
            ctx.writeAndFlush(Unpooled.copiedBuffer("hello1 : "+ ctx.channel().remoteAddress(), CharsetUtil.UTF_8));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```

2. 解决方案2：用户定时任务（提交到scheduleTaskQueue中执行）

- 新建 runnable 用于执行调度

```java
//延时5秒执行
ctx.channel().eventLoop().schedule(() -> {
    try {
        Thread.sleep(1*1000);
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello1 time : "+ ctx.channel().remoteAddress(), CharsetUtil.UTF_8));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}, 5, TimeUnit.SECONDS);
```

3. 解决方案3：非当前Reactor线程调用Channel的各种方法


# Netty核心组件

## 常用组件

### EventLoop

- EventLoop本质是一个单线程执行器（同时维护了一个Selector)，里面有run方法处理Channel上源源不断的io事件。

![](../../image/java/netty/20210615230702.png)

### EventLoopGroup 

- 时间循环组，一组EventLoop

- NioEventLoopGroup可以处理io事件，普通任务，定时任务

- NioEventLoopGroup默认的子线程线程数是：cpu核心数*2，new NioEventLoopGroup构造方法默认使用了NettyRuntime.availableProcessors() * 2

  如果构造参数有值，则使用构造参数的线程数

![](../../image/java/netty/20210613130404.png)

- **不过我们一般bossGroup设置为1个**
- 获取下一个eventLoop

```java
EventLoopGroup workGroup = new NioEventLoopGroup();
EventLoop eventLoop = workGroup.next();
```

- 执行一个普通任务

```java
NioEventLoopGroup executors = new NioEventLoopGroup();
executors.next().submit(() -> {
    System.out.println(Thread.currentThread().getId());
});
```

- 执行一个定时任务

```java
executors.next().scheduleAtFixedRate(() -> {
    log.debug("定时任务");
}, 0, 1, TimeUnit.SECONDS);
```

### ServerBootstrap，Bootstrap

1. 意思是引导，一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个Netty程序，串联各个组件，Netty中 Bootstrap类是客户端程序的启动引导类
2. ServerBootstrap是服务端启动引导类

- 用于设置EventLoopGroup

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workGroup = new NioEventLoopGroup();
try {
    ServerBootstrap bootstrap = new ServerBootstrap();
    bootstrap.group(bossGroup, workGroup)
```

- channel:设置服务器端通道的实现

```java
.channel(NioServerSocketChannel.class)
```

- option 给serverchannel添加配置

```java
//设置线程队列得到的连接数
                .option(ChannelOption.SO_BACKLOG, 128)
```

- childHandler: 用于添加workergroup对应的自定义的handler

```java
//设置一个处理事情的工作handler
.childHandler(new ChannelInitializer<SocketChannel>() {
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        socketChannel.pipeline().addLast(new NettyServerScheduleHandler()).addLast();
    }
});
```

- handler：添加bossgroup对应的handler

### Channel

- Netty 网络通信的组件，能够用于执行网络I/O操作
- 通过Channel可获得当前网络连接的通道的状态
- 通过Channel可获得网络连接的配置参数（例如接收缓冲区大小)
- Channel提供异步的网络l/О操作(如建立连接，读写，绑定端口)
- channel类型与协议有关

### Selector

- Netty基于Selector对象实现I/O多路复用，通过Selector一个线程可以监听多个连接的Channel事件。
- 当向一个Selector中注册 Channel后，Selector内部的机制就可以自动不断地查询(Select)这些注册的Channel是否有已就绪的l/O事件（例如可读，可写，网络连接完成等)，这样程序就可以很简单地使用一个线程高效地管理多个Channel

### ChannelHandler

- ChannelHandler是一个接口，处理I/O事件或拦截I/О操作，并将其转发到其ChannelPipeline(业务处理链)中的下一个处理程序。

ChannelInboundHandler:负责出站（客户端发送到服务器端）

```tex
//通道就绪事件
ChannelInboundHandlerAdapter#channelActive
//通道读取事件
ChannelInboundHandlerAdapter#channelRead
//读取完成
ChannelInboundHandlerAdapter#channelReadComplete
//通道发生异常
ChannelInboundHandlerAdapter#exceptionCaught
```

ChannelOutboundHandler: 负责入站
![](../../image/java/Netty/20200920104532.png)

- 出站入站的顺序

1. 调用顺序：head -> i1 ->i2 -> o4 -> o3 -> tail

```java
ch.pipeline().addLast("I1",new ChannelInboundHandlerAdapter() {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        log.debug("I1");
        super.channelRead(ctx,msg);
    }
});
ch.pipeline().addLast("I2",new ChannelInboundHandlerAdapter() {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        log.debug("I2");
        super.channelRead(ctx,msg);
        ctx.channel().writeAndFlush(ctx.alloc().buffer().writeBytes("server...".getBytes()));
    }
});
ch.pipeline().addLast("o3",new ChannelOutboundHandlerAdapter() {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        log.debug("o3");
        super.write(ctx,msg,promise);
    }
});
ch.pipeline().addLast("o4",new ChannelOutboundHandlerAdapter() {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        log.debug("o4");
        super.write(ctx,msg,promise);
    }
});
```

- super.channelRead(ctx,msg);表示将数据处理好后，交给下一个入站处理器
- out通道中，必须要有数据，才会触发

- writeAndFlush的调用方式不同，则处理结果也不同，如：

1. ctx调用，他是从当前节点往前找有没有出站handler
   1. 假如代码顺序是： i1 -> o3 -> i2 ->o4，则此时，I2往前调用用，只发现了o3是out可以调用

```java
log.debug("I2");
super.channelRead(ctx,msg);
ctx.writeAndFlush(ctx.alloc().buffer().writeBytes("server...".getBytes()));
```

2. channel调用，则是tail节点往前推送，发现out则调用

### 测试Handller

- 在开发中，启动服务端很耗时，可以借助netty提供的工具来测试handler的出站和入站

```java
EmbeddedChannel embeddedChannel = new EmbeddedChannel(i1, i2, o3, o4);
//测试入站操作
embeddedChannel.writeInbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("hello".getBytes()));
//测试出站
embeddedChannel.writeOutbound(ByteBufAllocator.DEFAULT.buffer().writeBytes("hello".getBytes()));
```

### 线程安全的Handller

- netty在他内置的handler里标记了一个注解，表示这个handler是可以共享的

如：

```java
@Sharable
public class LoggingHandler
```

### PipeLine

ChannelPipeline：

- 也可以这样理解:ChannelPipeline是保存ChannelHandler的 List，用于处理或拦截Channel的入站事件和出站操作

pipeline是一个包含头和尾的类似双向链表

![](../../image/java/Netty/20200920111503.jpg)

### ChannelHandlerContext

- ChannelHandlerContext中包含一个具体的事件处理器ChannelHandler ,同时ChannelHandlerContext中也绑定了对应的 pipeline和 Channel的信息，方便对ChannelHandler进行调用

```tex
### 
ChannelOutboundInvoker#write(java.lang.Object)
###
ChannelOutboundInvoker#flush
```

### ChannelOption

- ChannelOption.SO_BACKLOG
  - 对应TCP/IP协议listen函数中的backlog 参数，用来初始化服务器可连接队列大小。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog 参数指定了队列的大小。
- ChannelOption.SO_KEEPALIVE
  - 一直保持连接活动状态

## EventLoop详解

- 每个EventLoopGroup会对应一个Selector
- 通常是OP_ACCEPT事件，然后将接收到的SocketChannel交给WorkerEventLoopGroup
- WorkerEventLoopGroup会由next选择其中一个EventLoopGroup来将这个SocketChannel注册到其维护的Selector并对其后续的IO事件进行处理

![](../../image/java/netty/20210614154824.png)

- 当客户端的一个channel和服务端的EventLoop建立绑定，那么，服务器处理这个channel的EventLoop则一直是同一个
- boss EventLoop只会ServerSocketChannel绑定

### 额外的线程组处理业务

- 有时候，一个handler执行的业务时间特别长，这时，肯定会影响同一个EventLoop的其他channel的工作，此时，我们需要将此业务放入一个非IO的EventLoop中，防止它影响其他channel的使用
- 如下，我们造DefaultEventLoopGroup一个线程组，让handler2在其上执行，注意，需要调用ctx.fireChannelRead(msg);将消息传递这个里面

```java
//非IO的线程组
EventLoopGroup defaultEventLoop = new DefaultEventLoopGroup();
new ServerBootstrap()
        .group(new NioEventLoopGroup(1), new NioEventLoopGroup())
        .channel(NioServerSocketChannel.class)
        .childHandler(new ChannelInitializer<NioSocketChannel>() {
            @Override
            protected void initChannel(NioSocketChannel ch) throws Exception {
                ch.pipeline().addLast("handler1",new ChannelInboundHandlerAdapter() {
                    @Override
                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                        log.debug("handler1 进入 : {}", Thread.currentThread().getName());
                        //将消息传递给下一个handler
                        ctx.fireChannelRead(msg);
                    }
                });
                ch.pipeline().addLast(defaultEventLoop, "handler2", new ChannelInboundHandlerAdapter() {
                    @Override
                    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                        log.debug("handler2 是个很长的业务 {}", Thread.currentThread().getName());
                    }
                });
            }
        }).bind(80);
```

- 如图

![](../../image/java/netty/0041.png)

### handler切换源码

- io.netty.channel.AbstractChannelHandlerContext#invokeChannelRead(io.netty.channel.AbstractChannelHandlerContext, java.lang.Object)

```java
//获取msg信息
final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
//获取下一个handler
EventExecutor executor = next.executor();
//判断是否是同一个执行eventloop
if (executor.inEventLoop()) {
    next.invokeChannelRead(m);
} else {
    //不是，调用的代码封装为一个任务对象，由下一个 handler 的线程来调用
    executor.execute(new Runnable() {
        @Override
        public void run() {
            next.invokeChannelRead(m);
        }
    });
}
```

## 异步模型

- Netty的异步模型是建立在future上的
- 异步过程，调用者不能马上获取结果，等处理组件处理完后，通过状态、通知、回调来通知调用者
- Netty中的I/O操作是异步的，包括Bind、Write、Connect等操作会简单的返回一个ChannelFuture.
- Netty的异步模型是建立在future和 callback的之上的。
- ChannelFuture是一个接口，我们可以在此添加监听器

例如：添加一个监听，判断是否绑定端口成功

```java
//绑定一个端口并且同步
ChannelFuture sync = bootstrap.bind(6666).sync();
sync.addListener((future) -> {
    if(future.isSuccess()) {
        System.out.println("绑定端口 6666 成功....");
    }
});
```

### Connect说明

- 这段代码是异步非阻塞的
- main线程调用，真正去建立连接的是NIO相关的线程

```java
bootstrap.connect("127.0.0.1", 80)
```

- sync表示，mian线程等待nio线程建立连接后，返回结果

```java
bootstrap.connect("127.0.0.1", 80).sync()
```

- 如果不想使用sync的方式获取channel，亦可以采用监听的方式

```java
ChannelFuture future = bootstrap.connect("127.0.0.1", 80);
future.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture future) throws Exception {
        Channel channel = future.channel();
        log.debug("channel：{}", channel);
        channel.writeAndFlush("我是监听发过来的");
    }
});
```

### Channel关闭

1. 通过监听的方式获取关闭事件

```java
Channel channel = future.sync().channel();
new Thread(() -> {
    Scanner scanner = new Scanner(System.in);
    String next = scanner.next();
    if("q".equals(next)) {
        channel.close();
    }
}).start();
ChannelFuture channelFuture = channel.closeFuture();
channelFuture.addListener(closeFuture -> {
    log.debug("channel 已关闭..");
});
```

2. 通过 channelFuture.syn的方式

### 优雅的停止EventLoop组

```java
loopGroup.shutdownGracefully();
```

### 异步的好处

- 异步并没有缩短响应时间，反而有所增加
- 合理进行任务拆分，也是利用异步的关键
- 它提升的是单位时间内，处理的连接数的提升（吞吐量）

## Future & Promise

###   Jdk Future

```java
ExecutorService threadPool = Executors.newFixedThreadPool(2);
Future<Integer> future = threadPool.submit(() -> {
    log.debug("进入future中...");
    Thread.sleep(1000);
    return 10;
});
log.debug("准备获取结果...");
Integer integer = future.get();
log.debug("获取到结果...");
```

- future.get():异步阻塞的获取结果

### Netty Future

```java
DefaultEventLoopGroup loopGroup = new DefaultEventLoopGroup();
EventLoop eventLoop = loopGroup.next();
Future<Integer> future = eventLoop.submit(() -> {
    log.debug("进入future中...");
    Thread.sleep(1000);
    return 10;
});
log.debug("准备获取结果...");
log.debug("获取到结果 {}...", log.getName());
```

- getNow:获取任务结果，非阻塞，还未产生结果时返回 null  
- await:等待任务结束，如果任务失败，不会抛异常，而是通过 isSuccess 判断  
- sync:等待任务结束，如果任务失败，抛出异常  
- addLinstener: 添加回调，异步接收结果  

### Promise

- 它相当于一个容器，一个线程往这个里面设置数据，另一个线程从中取数据

```java
EventLoop eventLoop = new DefaultEventLoopGroup().next();
Promise<Integer> promise = new DefaultPromise<>(eventLoop);
new Thread(() -> {
    log.debug("开始计算结果设置数据进入promise");
    promise.setSuccess(100);
}).start();
log.debug("获取结果中....");
log.debug("获取到结果：{} ...", promise.get());
```

- setFailure:设置失败结果

# ByteBuf

- 创建

```java
//创建初始容量为10
ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(10);
```

- 直接内存 vs 堆内存

```java
ByteBuf buffer = ByteBufAllocator.DEFAULT.heapBuffer(10);
//默认使用的直接内存
ByteBuf buffer = ByteBufAllocator.DEFAULT.directBuffer(10);
```

## 池化功能

- 池化的最大意义在于可以重用 ByteBuf（有点类似连接池的思想）

- 池化功能是否开启，可以通过下面的系统环境变量来设置

  ```java
  -Dio.netty.allocator.type={unpooled|pooled}
  ```

- 4.1 以后，非 Android 平台默认启用池化实现，Android 平台启用非池化实现

- 4.1 之前，池化功能还不成熟，默认是非池化实现

## 结构

- bytebuf由四个部分组成，它读写分为两个指针，这样就不用flip了
- 最开始读写指针都在 0 位置

![](../../image/java/netty/0010.png)

## 扩容

再写入一个 int 整数时，容量不够了（初始容量是 10），这时会引发扩容

```java
buffer.writeInt(6);
log(buffer);
```

扩容规则是

* 如何写入后数据大小未超过 512，则选择下一个 16 的整数倍，例如写入后大小为 12 ，则扩容后 capacity 是 16
* 如果写入后数据大小超过 512，则选择下一个 2^n，例如写入后大小为 513，则扩容后 capacity 是 2^10=1024（2^9=512 已经不够了）
* 扩容不能超过 max capacity 会报错

## 内存回收

- UnpooledHeapByteBuf 使用的是 JVM 内存，只需等 GC 回收内存即可
- UnpooledDirectByteBuf 使用的就是直接内存了，需要特殊的方法来回收内存
- PooledByteBuf 和它的子类使用了池化机制，需要更复杂的规则来回收内存

Netty 这里采用了**引用计数法**来控制回收内存，每个 ByteBuf 都实现了 ReferenceCounted 接口

- netty中，有head和tail对ByteBuf进行回收，如果中间有没传到这两个hanlder的bytebuf，则需要自己进行回收

## Unpooled 

- 提供了非池化的 ByteBuf 创建、组合、复制等操作

- 通过两个指针，readerIndex与writerIndex分别指向已经读到的位置和写入的位置，比JDK提供的ByteBuffer 省了flip操作

```java
//创建一个buff长度为10的对象
ByteBuf buffer = Unpooled.buffer(10);
//插入数据
Stream.iterate(0, i -> i+1)
        .limit(10)
        .forEach(buffer::writeByte);
for(int i=0; i<buffer.capacity(); i++) {
    System.out.println(buffer.getByte(i));
}
```

- readerindex---writerIndex , 可读的区域
- writerIndex -- capacity,可写的区域

### Unpooled相关Api

```tex
## 创建一个buff
Unpooled#copiedBuffer(byte[])
## 返回当前数组内容
ByteBuf#array
```

## ByteBuf 优势

* 池化 - 可以重用池中 ByteBuf 实例，更节约内存，减少内存溢出的可能
* 读写指针分离，不需要像 ByteBuffer 一样切换读写模式
* 可以自动扩容
* 支持链式调用，使用更流畅
* 很多地方体现零拷贝，例如 slice、duplicate、CompositeByteBuf

# Netty中的零拷贝

## slice

对原始 ByteBuf 进行切片成多个 ByteBuf，切片后的 ByteBuf 并没有发生内存复制，还是使用原始 ByteBuf 的内存

- 此时，buf1/buf2和buf是同一内存，所以，修改buf1的值会影响buf的值
- 切片后的对象容量不允许再添加，也就是buf1/buf2不能扩容

```java
ByteBuf buf = ByteBufAllocator.DEFAULT.buffer(6);
buf.writeBytes(new byte[] {'a', 'b', 'c', 'd', 'e', 'f'});
//采用切片的方式
ByteBuf buf1 = buf.slice(0, 3);
ByteBuf buf2 = buf.slice(3, 3);
```

- 切片后，buf调用release回收内存会有影响，配合buf1.retain则buf.release不会报错，这样buf1/buf2和buf都需要调用release方法进行回收

## CompositeByteBuf

- 将小的bytebuf合并成新的bytebuf

```java
ByteBuf buf1 = ByteBufAllocator.DEFAULT.buffer(5);
buf1.writeBytes(new byte[]{1, 2, 3, 4, 5});
ByteBuf buf2 = ByteBufAllocator.DEFAULT.buffer(5);
buf2.writeBytes(new byte[]{6, 7, 8, 9, 10});

CompositeByteBuf buf3 = ByteBufAllocator.DEFAULT.compositeBuffer();
// true 表示增加新的 ByteBuf 自动递增 write index, 否则 write index 会始终为 0
buf3.addComponents(true, buf1, buf2);
```



# 心跳机制

IdleStateHandler是netty处理空闲状态的处理器

- readerIdleTime:多久时间没读，发送一个心跳检测包，是否连接
- writerIdleTime：多久时间没写，发送一个心跳检测包
- allIdleTime：多久时间没有读写

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workGroup = new NioEventLoopGroup();
try {

    ServerBootstrap bootstrap = new ServerBootstrap();
    bootstrap.group(bossGroup, workGroup)
            .channel(NioServerSocketChannel.class)
            .handler(new LoggingHandler(LogLevel.INFO))
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();
                    //在pipeline中 加入解码器
                    pipeline.addLast(new IdleStateHandler(3,5,7, TimeUnit.SECONDS));
                    pipeline.addLast(new IdleStateServerHandler());
                }
            });
    log.debug("==>聊天服务器端启动");
    ChannelFuture sync = bootstrap.bind(this.port).sync();
    sync.channel().closeFuture().sync();
} finally {
    bossGroup.shutdownGracefully();
    workGroup.shutdownGracefully();
}
```

```java
public class IdleStateServerHandler extends ChannelInboundHandlerAdapter {
    /**
     *
     * @param ctx 上下文
     * @param evt 事件
     * @throws Exception
     */
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        //如果是心跳检测机制
        if(evt instanceof IdleStateEvent ) {
            IdleStateEvent event = (IdleStateEvent) evt;
            String msg = null;
            switch (event.state()) {
                case ALL_IDLE:
                    msg = "读写空闲";
                    break;
                case READER_IDLE:
                    msg = "读空闲";
                    break;
                case WRITER_IDLE:
                    msg = "写空闲";
                    break;
            }
            log.info(msg);
        }
    }
}
```

# websorcket

- websocket数据以 帧frame传输

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workGroup = new NioEventLoopGroup();
try {

    ServerBootstrap bootstrap = new ServerBootstrap();
    bootstrap.group(bossGroup, workGroup)
            .channel(NioServerSocketChannel.class)
            .handler(new LoggingHandler(LogLevel.INFO))
            .childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline pipeline = ch.pipeline();
                    //使用http解码和编码
                    pipeline.addLast(new HttpServerCodec());
                    //以块方式进行读写
                    pipeline.addLast(new ChunkedWriteHandler());
                    //http传输是分段的，HttpObjectAggregator多段聚合
                    //所以在http传输中，大量数据会有多个http请求
                    pipeline.addLast(new HttpObjectAggregator(8192));
                    // 浏览器访问 ws://127.0.0.1/hello
                    //WebSocketServerProtocolHandler 将http升级 为 websocket
                    pipeline.addLast(new WebSocketServerProtocolHandler("/hello"));
                    //以文本帧的方式进行处理
                    pipeline.addLast(new TextWebSocketFrameHandler());

                }
            });
    log.debug("===>服务器端启动");
    ChannelFuture sync = bootstrap.bind(this.port).sync();
    sync.channel().closeFuture().sync();
} finally {
    bossGroup.shutdownGracefully();
    workGroup.shutdownGracefully();
}
```

# 编解码

## Protobuf

- 支持跨平台夸语言的编解码
- 很时候RPC或者数据存储

## 生成代码

- 引入pom

```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.4.0</version>
</dependency>
```

- 下载生成客户端工具

[https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases)

# netty出站入站机制

- channelPipeline 提供了 channelhandler链的容器
- 写入 socket叫 出站，读socket叫入站
- netty发送或者 接受一个消息，会发生一次解码、编码
- 出站 需要 编码，  入站 需要解码

![](..\image\java\Netty\20201006141408.jpg)

## 解码器

- ByteToMessageCodec

由于不可能知道远程节点是否会一次性发送完整的信息，所有tcp可能会出现粘包拆包问题，这个类会对入站数据进行缓冲，知道他完全被处理好

![](..\image\java\Netty\20201006143240.png)

举例：

有个int类型的解码器

重写decode方法，socket发送两个int数据，入站解码开始解码，读到四个字节就将其转化为int放入list中，直到全部读完

然后丢给下一个handler执行

![](../..\image\java\Netty\20201006145150.png)

![](../..\image\java\Netty\20201006164012.png)

# TCP粘包拆包

TCP是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的socket，因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（Nagle算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于分辨出完整的数据包了，因为面向流的通信是无消息保护边界的

- 因为TCP的消息是有应答的，所以每次消息发出都会要有响应，这样带来了吞吐量的问题
- 所以TCP将详细封装成一个大的数据块，发给服务器端（Nagle算法）
- TCP的滑动窗口也会造成这个现象

## 基本介绍

- 服务端分别收到了D1和D2，没有粘包和拆包
- 服务端一次性收到了D1和D2，称为TCP粘包
- 服务端两次读取到了两个数据包，第一次读到了D1的完整部分和D2的部分数据，第二次读到了D2的剩余部分。 这称为TCP拆包
- 服务端两次读取到两个数据包，第一次是D1的部分，第二次是D1的剩余部分和D2的完整部分。

![](../..\image\java\Netty\20201006171253.png)

## 解决方案

- 短连接
  - 可以解决粘包问题，不能解决拆包问题

- 定长的消息解码器
  - io.netty.handler.codec.FixedLengthFrameDecoder
  - 需要注意的是，定长一定要客户端发送消息的最大长度
- 分割符解码器 
  - io.netty.handler.codec.LineBasedFrameDecoder
- 基于字段长度（LTC）

  - io.netty.handler.codec.LengthFieldBasedFrameDecoder

# netty源码解析

## netty启动过程

- 先看example下的示例代码

EchoServer
EchoServerHandler
EchoClient
EchoClientHandler

### EchoServer

```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
```

bossGroup用于接收TCP请求，他会将请求交给workgroup，workgroup获取真正的连接进行通信

EventLoopGroup是一个时间循环组（线程组）， 含有多个EventLoop

MultithreadEventExecutorGroup
------ MultithreadEventExecutorGroup()方法

```
//创建时间循环组
children = new EventExecutor[nThreads];
```

引导类

```
ServerBootstrap b = new ServerBootstrap();
```

channel方法目的通过反射创建channelFactory

```
b.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
```

### EventLoopGroup

## EventLoop组件

- 一个EventLoopGroup 包含一个或者多个EventLoop；
- 一个EventLoop 在它的生命周期内只和一个Thread 绑定；
- 所有由EventLoop 处理的I/O 事件都将在它专有的Thread 上被处理；
- 一个Channel 在它的生命周期内只注册于一个EventLoop；
- 一个EventLoop 可能会被分配给一个或多个Channel。

NioEventLoopGroup对象可以理解为一个线程池，内部维护了一组线程，每个线程负责处理多个Channel上的事件，而一个Channel只对应于一个线程，这样可以回避多线程下的数据同步问题。



# 更换编解码

## Google 的 ProtoBuf
# 搭建Http服务

- SimpleChannelInboundHandler可以快速的过滤msg的类型

```java
public static void main(String[] args) {
    EventLoopGroup bossGroup = new NioEventLoopGroup(1);
    EventLoopGroup workGroup = new NioEventLoopGroup();
    try {
        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(bossGroup, workGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new HttpServerInitializer());
        ChannelFuture channelFuture = bootstrap.bind(new InetSocketAddress(6666)).sync();
        channelFuture.channel().closeFuture().sync();
    } catch (Exception e) {

    } finally {
        bossGroup.shutdownGracefully();
        bossGroup.shutdownGracefully();
    }
}
```

```
 * @Description 在某个Channel注册到EventLoop后，对这个Channel执行一些初始化操作
 * @createTime 2020年09月16日 09:23:00
 */
public class HttpServerInitializer extends ChannelInitializer<SocketChannel> {
    /**
     * 向管道加入处理器
     * @param socketChannel
     * @throws Exception
     */
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        ChannelPipeline pipeline = socketChannel.pipeline();
        pipeline.addLast("httpServerCodec", new HttpServerCodec());
        pipeline.addLast("httpServerHandler", new HttpServerHandler());
    }
}
```

```java
* @Description HttpObject:客户端与服务器端通讯的工具封装成：HttpObject
 * @createTime 2020年09月16日 09:38:00
 */
@Slf4j
public class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    /**
     * 读取数据
     * @param ch
     * @param object
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ch, HttpObject object) throws Exception {
        if(object instanceof HttpRequest) {
            log.debug("msg: {} ", object.getClass());
            log.debug("客户端地址: {} ", ch.channel().remoteAddress());
            ByteBuf content = Unpooled.copiedBuffer("netty, 服务器", CharsetUtil.UTF_8);
			//设置版本等
            DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain; charset=UTF-8");
            //告诉相应的结果长度
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());
            ch.writeAndFlush(response);
        }
    }
}
```

# 自定义协议

* 魔数，用来在第一时间判定是否是无效数据包（如：jvm字节码以cafebaby开头）
* 版本号，可以支持协议的升级
* 序列化算法，消息正文到底采用哪种序列化反序列化方式，可以由此扩展，例如：json、protobuf、hessian、jdk
* 指令类型，是登录、注册、单聊、群聊... 跟业务相关
* 请求序号，为了双工通信，提供异步能力（如：发送123，消息不一定以123这个顺序来发）
* 正文长度
* 消息正文