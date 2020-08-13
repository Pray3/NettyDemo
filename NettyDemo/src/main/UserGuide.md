本文基于netty官网 User guide for 4.x  参考链接:https://netty.io/wiki/user-guide-for-4.x.html

本文中，很多不是按照原文翻译，为了阅读更通畅，更便于理解，做了一些缩略，说明，
在阅读程序时，可以从代码的后面的数字注释如(1)(2)(3)，找到对该注释的具体说明，这里跟国内的很多文章不太一样，
对于我来说这样的方式感觉会更好理解一点
如果您更希望按照原生文档作者的语义阅读，请阅读英文版，
其实英文文档结构逻辑都非常清晰，建议有时间的同学去看看官方文档，也可以去官方社区交流你的看法。
作者(也许是一个团队)的表达都非常谦虚，也许这也是netty成功的原因之一，希望大家技术成长的路上也能像这些大牛一样一直保持谦虚的心态不断进步
Stay Hungry. Be Humble. Always Hustle

如果有翻译错误或者理解错误，欢迎指正。


#### 问题 The Problem

如今，我们常常在应用程序，或者服务间进行通讯。例如，我们经常使用HTTP客户端库从Web服务器获取信息、在本地调用远程Web的服务。
但是，很多通用协议不能很好地扩展。就像我们不能使用HTTP服务器来交换大文件，电子邮件和近乎实时的消息（例如财务信息和多人游戏数据）。
而需要对该场景进行专门的高度优化。例如，基于AJAX的聊天应用程序，媒体流或大文件传输。
您甚至可能需要设计一个全新的协议来完成您的需求。与此同时，您必须处理旧的专有协议，以确保与旧系统的兼容。
我们需要在不牺牲应用程序的稳定性和性能的前提下，快速的实现一个符合要求的协议。
#### 解决方案 The Solution

Netty项目致力于提供一个异步事件驱动的网络应用程序框架和工具，支持快速开发可维护的高性能和高可扩展性协议服务器和客户端。
从技术上来说，Netty是一个基于NIO的客户端服务器框架，可以快速轻松地开发网络应用程序，例如协议服务器和客户端。
同时极大地简化和简化了网络编程，例如TCP和UDP socket服务器的开发。
Netty经过精心设计，并结合了许多协议（例如FTP，SMTP，HTTP以及各种基于二进制和文本的旧式协议）的经验。
在提供快速开发能力的同时，netty还保障了应用程序的高性能，高稳定性，并且非常灵活可维护。当然您可能知道一些其他声称具有相同优势的网络应用程序框架，
netty跟它们有什么不同呢？答案是它所基于的哲学。 Netty从诞生的第一天开始就为您在API和实施方面提供最舒适的体验。
这不是实际的东西，但是您在阅读本指南时会感受到这种哲学，在您阅读指南时、使用Netty时、甚至在您的日常生活中这种哲学都会让您更加轻松。

#### 开始 Getting Started
本章将通过简单的示例介绍Netty的核心构造，以使您快速入门。 在本章结束时，您将可以立即在Netty之上编写客户端和服务器。 如果您喜欢自上而下的学习方法，则可能要从第2章，体系结构概述开始，然后回到此处。

#### 准备环境 Before Getting Started
启动当前章节中的例子只需要准备两个环境：最新的netty版本以及JDK1.6或以上
最新版本的netty可以在[这里](https://netty.io/downloads.html)找到，jdk版本请访问您使用的JDK供应商的官网

在阅读时，如果您对本章介绍的类有疑惑，请参考我们提供的API文档，为了您的方便，我们提供了在线的版本，同时，
非常欢迎您在[netty项目社区](https://netty.io/community.html)跟我们交流，包括任何可以帮助改进文档的idea，以及文档中语法或者跟错别字。

#### 写一个丢弃请求的服务器 Writing a Discard Server 
世界上最简单的协议不是'Hello, World!' 而是 [DISCARD](https://tools.ietf.org/html/rfc863.),这个协议会丢弃所有收到的数据，并且没有任何回复
实现这个协议只需要忽略掉所有收到的数据就可以了，下面让我们使用netty提供的处理器实现吧

```java
package io.netty.example.discard;
 
 import io.netty.buffer.ByteBuf;
 
 import io.netty.channel.ChannelHandlerContext;
 import io.netty.channel.ChannelInboundHandlerAdapter;
 
 /**
  * Handles a server-side channel.
  */
 public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)
 
     @Override
     public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
         // Discard the received data silently.
         ((ByteBuf) msg).release(); // (3)
     }
 
     @Override
     public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
         // Close the connection when an exception is raised.
         cause.printStackTrace();
         ctx.close();
     }
 }
 
 ```
 1.DiscardServerHandler继承了ChannelInboundHandlerAdapter,ChannelInboundHandlerAdapter实现了
 ChannelInboundHandler. ChannelInboundHandler这个两个接口，您可以通过覆盖这两个接口的实现类，
 来处理各种事件，在这个例子中我们只需要继承ChannelInboundHandlerAdapter就可以了。
 
 2.我们重写了 channelRead() 这个事件处理方法，这个方法会在收到消息后被调用，这个例子中我们收到的 消息类型是 ByteBuf。
 
 3.为了实现Discard协议，我们需要忽略掉任何收到的信息，ByteBuf是一个引用计数对象，必须通过release()方法显式释放。
 请注意，我们的处理程序中必须要对所有引用计数类型的对象做处理。通常channelRead()处理方法会是下面这个样子
 
 ```java
 
 @Override
 public void channelRead(ChannelHandlerContext ctx, Object msg) {
     try {
         // Do something with msg
     } finally {
         ReferenceCountUtil.release(msg);
     }
 }
 ```
 4.在netty抛出io异常或者程序在处理事件时抛出异常后,会调用exceptionCaught()这个事件方法，大部分情况下
 caught到的异常记录到日志，且要关闭其他相关的channel，您也可以根据您想要的其他方式来处理异常，比如
 在关闭连接通道前，返回一个错误码

非常好，我们已经实现了Discard Server程序的前半部分了，接下来我们来写一个main()方法，启动我们的Discard Server

```java
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }

        new DiscardServer(port).run();
    }
}

``` 
1.NioEventLoopGroup是一个多线程的I/O操作处理器，netty提供了多种EventLoopGroup实现类，用于处理不同的
传输协议，在这个例子中我们使用了两个NioEventLoopGroup，第一个称为'boss'，用来接收传入的链接
第二个称为'worker',worker首先需要注册到boss中，当boss接收到请求后，由worker去处理具体的请求。
具体使用多少个线程，以及他们是怎么映射到channel的取决于EventLoopGroup的实现，或者通过构造函数配置

2.ServerBootstrap是一个用来设置服务器的帮助类，你也可以直接设置channel，大多数情况下程序中不需要通过帮助类设置。

3.这里我们指定使用 NioServerSocketChannel这个类去初始化一个channel并用来接收传入的连接

4.这里指定了每次请求都会被一个新的channel处理， ChannelInitializer 是一个特殊的处理器，用来帮助用户配置新的channel
您可能希望通过添加一些处理程序例如(DiscardServerHandler)来实现新的来实现新的Channel的ChannelPipeline，以实现您的网络
应用程序。 随着应用程序变得复杂，您可能会向Channel添加更多处理程序，并最终将此匿名类提取到顶级类中。

5.你也可以设置channel实现类的参数，我们是基于TCP/IP的协议，所以我们设置tcpNoDelay跟keepAlive.这两个参数
请参考文档中的ChannelOption类跟ChannelConfig接口的实现类，文档中记录了所有支持的ChannelOptions.

6. option()方法用来设置NioServerSocketChannel的参数，childOption()可以让实现类接收来自实现接口的参数，
比如这个例子中的 NioServerSocketChannel实现了ServerChannel接口，可以通过这个方法传递参数。

7.到这里我们的程序第一个基于netty的程序就写好了，接下来我们去绑定一个端口并启动服务，例子中我们选择了8080端口，
你也可以多次调用bind()方法(使用不同的端口）。

#### 查看接收到的数据 Looking into the Received Data

现在我们写好了一个程序，来测试一下是不是真的能用吧，最简单的方法是通过 telnet 命令，比如使用
telnet localhost 8080 并输入一些内容。

然而虽然我们发送了一些内容到服务上，但我们的程序会将所有收到的数据丢弃，所以我们稍微改造一下
DiscardServerHandler的channelRead()方法。

```java

@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}

```

1.这个低效循环可以用System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII)) 代替

2.这里可以用in.release() 方法代替

再次使用telnet命令就可以看见服务端打印出数据了。
#### 编写一个有回应的服务 Writing an Echo Server

目前为止，我们一直在使用客户端发来的数据但没有回应，但大部分的服务器都会响应客户端的请求，接下来让我们
实现一个服务 Echo Server，将所有接收到的数据原样返回给客户端。这个服务跟discard server服务的区别就是，
将discard server服务中打印数据部分改成返回就可以了，我们将channelRead()改成如下的样子、

```java
 @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```
1. ChannelHandlerContext提供了很多操作用于响应各类IO事件与操作，这里我们调用write(Object)来逐字的返回接收到的数据，
这里我们没有像discard server一样调用release释放掉引用对象，这是因为当我们将数据写到网络时，netty会为您释放掉改该对象。

2.ctx.write(Object)方法并没有将数据写到网络，而是将数据写到缓冲区中，当调用ctx.flush()后,才会从缓冲区中写到网络，
也可以使用ctx.writeAndFlush(msg) 将两个方法合二为一。

再次使用telnet命令就可以在客户端看到您发出去的数据了。

#### 写一个获取时间的服务

在这个小节总我们要去实现一个获取时间的服务，跟之前的例子不同的是，这个例子只会发送一个32位整数的数据
然后在发送完成后就关闭，通过这个例子，您将学习怎么去构造和发送消息，以及怎么在完成时关闭连接

因为我们要在建立连接之后忽略所有的请求，并发送一条消息，所以不能再使用之前的channelRead()，我们通过重写
channelActive()方法实现，具体实现参考下面的代码

```java
package io.netty.example.time;

public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1.就像之前说的，channelActive()方法会在连接建立时被调用，这个方法里我们会写一个32位的integer数代表当前的时间。

2.因为是一个32位的正数，我们需要一个4个byte的缓冲区来存储，这里通过ChannelHandlerContext.alloc()来获得。
ChannelHandlerContext是一个实现了 ByteBufAllocator 接口的类，ByteBufAllocator负责分配所有ByteBuf 类型的内存。

3.一般来说我们现在要编写构造方法了，但是再此之前我们先来看看NIO中的flip方法。传统的NIO中，有三个概念，Capacity,Position和Limit
Capacity是缓冲区的大小，Position是当前读写指针的位置，Limit表示当前缓冲区只能写到这个位置为止，通过调用flip方法来设置limit跟Position的位置
防止数据读取不全或者写数据时覆盖了之前的缓冲区中的数据。

但是在netty中我们并不需要处理这么麻烦的事情，因为ByteBuf 中引入了两个指针，写指针只会在写数据的时候移动，
写指针跟读指针就表示了消息的起始位置。netty中我们为不同操作类型提供了不同的指针，你会发现生活变得如此简单，
享受没有flipping的生活吧！

另有一个关键点是ChannelHandlerContext.write(),writeAndFlush()方法返回了一个ChannelFuture对象，这个对象表示
当前还未发生的事件，就是说所有netty的请求响应都是异步的，因此可能尚未执行任何请求的操作，比如下面的代码可能会在
发送消息之前就被关闭

```java
Channel ch = ...;
ch.writeAndFlush(message);
ch.close();
```
所以您需要在ChannelFuture完成后再调用close方法，该方法由write方法返回，并在完成写操作后通知所有的listeners,
请注意，调用close()方法可能也不会立即关闭连接，因为他的返回值也是一个 ChannelFuture。

4.那么我们怎么能在请求结束后得到通知呢？下面是一个添加listener到ChannelFuture的例子，这里我们创建了一个
匿名类 ChannelFutureListener,在操作完成后，用来接收通知并关闭channel，当然您也可以使用您预先定义好的listener
```java
f.addListener(ChannelFutureListener.CLOSE);
```

可以通过下面的unix系统命令测试服务，port是您在main方法里定义的端口，host是localhost

```shell
$ rdate -o <port> -p <host>
```

#### 写一个接收时间的客户端

跟DISCARD和ECHO服务不通的是，我们并不能将日历上的一个日期转换成32位的数据，这个小节我们来讨论怎么
确保服务器返回正确的时间，以及怎么基于netty写一个客户端

客户端与服务端最大的区别是使用不同的 Bootstrap 跟 Channel的实现类，请看下面的代码

```java
package io.netty.example.time;

public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            
            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

1.Bootstrap 跟 ServerBootstrap 非常类似，除了他用于非服务器通道，例如客户端跟无连接通道

2.如果您只指定了一个EventLoopGroup，那么这个EventLoopGroup会被同时用于boss group 与 worker group.
boss group 在服务端不会启用

3.区别于 NioServerSocketChannel, 客户端使用NioSocketChannel

4.在客户端我们不是用childOption()方法，因为客户端的 SocketChannel没有父channel;

5.在客户端我们需要使用connect()方法取代bind()方法

客户端跟服务端的代码其实相差的并不大，那么处理器 ChannelHandler怎么写呢，这里我们会接受到一个表示
时间的32位整数，我们需要将它翻译成我们看得懂的时间并打印出来，然后关闭连接。

```java

package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1.在TCP/IP协议中，netty的读取数据都是通过 ByteBuf进行的。

这看起来非常简单，而且跟服务端程序很像，但是这个handler有时候会拒绝响应，并抛出 IndexOutOfBoundsException异常
我们将在下一个部分讨论这个问题。

#### 处理基于流的传输Dealing with a Stream-based Transport

**一个关于socketbuffer的小警告**
在一个基于流传输的传输协议中(比如TCP/IP)，收到的数据会被存储在一个socket buffer中，不幸的是，这个缓存是一个存储
bytes的队列而不是一个存储packets的队列，这意味着，及时你发送了两个独立packets的数据，系统并不会按照两个message处理它，
而是将它们看作是一堆数据，所以，这里不能保证您读到的数据等于您写进来的数据，比如，我们假设基于TCP/IP协议收到了以下的三个包

![netty1](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/1.png)

由于流传输协议的特性，在您的应用中很有可能读到的是以下的形式

![netty2](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/2.png)

所以，不管在服务端还是客户端都应该讲接收到的数据整理到一个或多个有意义的分段中，应用程序可以轻松的理解这些分段
比如在这个例子中，收到的数据应该按这样的分段排列

![netty3](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/3.png)

**第一种解决方案**

我们先看看之前的TIME client的例子，32位的数字是非常小的数据量，通常不太可能被分段，但是还是会有被分段的可能性，
而且随着流量的增加，被分段的可能性也会变高。

最简单的解决方案是简历一个内部的缓冲区，等到接收到完整的4个bytes为止，下面的例子实现了解决方案

```java
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
1.一个ChannelHandler有两个生命周期listener方法，handlerAdded() 和 handlerRemoved().您可以执行任意(取消)
初始化任务，只要它不会长时间阻塞。

2.首先所有的数据都应该写入缓存

3.然后，handler必须检查buf里是否有足够的数据，我们这个例子里是4个。然后实现业务逻辑，同时，当收到数据时，
netty会调用channelRead()方法，最终所有的4个bytes数据都会被写入到缓存。

**第二种解决方案**

第一种方法虽然解决了TIME client的问题，但是修改后的handler看起来不是很干净，想象一下如果有一个更复杂的协议，
由多个字段组成，这些字段是可变长度的，我们的ChannelInboundHandler就会无法处理。

您可能注意到，您可以在 ChannelPipeline中添加多个ChannalHandler，可以将一个ChannalHandler分成多个Handler，从而降低
单个Handler的复杂度，比如您可以将TimeClientHandler分成如下两个handlers:

1.TimeDecoder用来处理数据被分段的问题，以及
2.一个初始化的简单版本

netty也提供了可继承的类，帮助您开箱即用的编写第一个类

```java
package io.netty.example.time;

public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```

1.ByteToMessageDecode是ChannelInboundHandler接口的实现类，用来更简单的处理数据分段的问题。

2.ByteToMessageDecoder会在收到新的数据后立即调用decode()方法，并将数据写入内部维护的累计buffer中

3.如果缓冲区中没有足够的数据，decode()方法可以决定不向out里写数据，ByteToMessageDecoder会在收到更多数据之后再次调用decoder()方法

4.如果decode()方法成功将一个对象写入到了out，说明decoder程序成功将一个消息解码了 ByteToMessageDecoder 会丢弃剩下的积累buffer部分，
您不需要解码多个message，因为 ByteToMessageDecoder会一直调用decode()方法，直到没有数据可以输出为止。

现在我们有另一个handler加入到ChannelPipeline中了，让我们来修改一下 TimeClient中的初始化 ChannelInitializer类的实现。

```java
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果您希望再进一步解决这个问题，您可以参照 ReplayingDecoder ，它可以进一步简化解码器，
您可以参考API获取更多的信息。

```java
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```

另外netty提供了其他的开箱即用的解码器，让您可以很轻松的实现各类协议，防止在处理数据的过程中出现无法
解析数据导致程序不可用，请查看以下的文档

[io.netty.example.factorial](https://netty.io/4.1/xref/io/netty/example/factorial/package-summary.html) for a binary protocol, and

[io.netty.example.telnet](https://netty.io/4.1/xref/io/netty/example/telnet/package-summary.html) for a text line-based protocol.

#### 使用POJO代替ByteBuf Speaking in POJO instead of ByteBuf

目前为止我们所有的例子主要用ByteBuf这个结构定义协议消息，在这个小节中我们会使用我们定义的POJO来取代ByteBuf。

在ChannelHandlers中使用POJO的优点显而易见，将提取信息的代码从ByteBuf中分离之后，您的handler的复用性跟可维护性都
会提高，在 TIME client 与 server 例子中，我们只需要读取32位的整数，所以并不需要直接使用ByteBuf，实际上自己定义
POJO的方式在实现具体协议的时候是非常必要的

首先让我们定义一个新的POJO叫做UnixTime

``` java

package io.netty.example.time;

import java.util.Date;

public class UnixTime {

    private final long value;
    
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    
    public UnixTime(long value) {
        this.value = value;
    }
        
    public long value() {
        return value;
    }
        
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}

```

现在我们可以在之前的TimeDecoder程序中使用UnixTime代替ByteBuf了

```java
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```

因为更新了decoder的代码，TimeClientHandler 也不再需要ByteBuf了

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

是不是非常简单而且优雅，现在我们同时去改造服务端的代码吧，先改造TimeServerHandler 

```java
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```

现在只剩下encoder了，encoder的左右是吧系统时间翻译成具体时间，我们用上POJO之后就更简单了，可以省略掉翻译的步骤

```java
package io.netty.example.time;

public class TimeEncoder extends ChannelOutboundHandlerAdapter {
    @OverrideChannelOutboundHandlerAdapter 
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int)m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

1.这短短的一行实际上做了一些很重要的事情
首先我们掺入了一个原始对象 ChannelPromise ，netty会通过这个对象告诉我们最终encoded成功还是失败。
然后我们并没有调用ctx.flush()方法，这里有一个单独 handler方法void flush(ChannelHandlerContext ctx) ，覆盖了flush() 操作。

进一步的简化，可以写成如下形式
```java

public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}
```

最后我们将TimeEncoder插入到服务器端的ChannelPipeline的TimeServerHandler上

#### 停止你的应用Shutting Down Your Application

停止一个netty应用就跟停止 EventLoopGroups 一样，通过调用shutdownGracefully()方法，
在EventLoopGroup 跟所有的相关的 Channels 关闭之后，将返回一个Future通知您。

####  Summary总结

在这章中，我们过了一遍netty工作的例子，并编写了整个基于netty网络应用的程序，后面我们有更多的详细介绍
netty的文章，同时也非常推荐您查看我们在 io.netty.example 包下提供的官方例子。

欢迎在官方社区提出您的问题跟想法，帮助我们不断提高netty的性能以及完善文档。

