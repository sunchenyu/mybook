# Netty编程

## Netty客户端

```java
package com.ultra.script.netty;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.PooledByteBufAllocator;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LengthFieldBasedFrameDecoder;
import io.netty.handler.codec.LengthFieldPrepender;

import java.util.Scanner;


public class KernelClient {
    public static void main(String[] args) throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)
                    .option(ChannelOption.TCP_NODELAY, true)
                    .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                    .handler(new ChannelInitializer<Channel>() {
                        @Override
                        protected void initChannel(Channel ch) {
                            ChannelPipeline p = ch.pipeline();
                            p.addLast(new LengthFieldBasedFrameDecoder(1024, 0, 4, 0, 4));
                            p.addLast(new LengthFieldPrepender(4));
                            p.addLast(new KernelClientHandler());
                        }
                    });

            ChannelFuture f = b.connect("127.0.0.1", 8080).sync();
            System.out.println("Client connected.");

            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNextLine()) {
                String line = scanner.nextLine();
                CustomMessage message = new CustomMessage((short) 2, line);
                f.channel().writeAndFlush(message.encode());
            }

            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }

    static class KernelClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
        @Override
        protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) {
            CustomMessage response = CustomMessage.decode(msg);
            System.out.println("Received from server: " + response);
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            System.out.println("Client error:" + cause);
            ctx.close();
        }
    }
}

```

## Netty服务端

```java
package com.ultra.script.netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.PooledByteBufAllocator;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LengthFieldBasedFrameDecoder;
import io.netty.handler.codec.LengthFieldPrepender;

public class KernelServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup boss = new NioEventLoopGroup(1);
        EventLoopGroup worker = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(boss, worker)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.TCP_NODELAY, true)
                    .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) {
                            ChannelPipeline p = ch.pipeline();
                            // [解码]：最大帧长、长度字段偏移、长度字段长度、长度调整、去除长度字段字节数
                            p.addLast(new LengthFieldBasedFrameDecoder(
                                    1024, 0, 4, 0, 4));
                            // [编码]：自动在消息前加4字节长度字段
                            p.addLast(new LengthFieldPrepender(4));
                            p.addLast(new KernelServerHandler());
                        }
                    });

            int port = 8080;
            ChannelFuture f = b.bind(port).sync();
            System.out.println("Server started on port " + port);
            f.channel().closeFuture().sync();
        } finally {
            boss.shutdownGracefully();
            worker.shutdownGracefully();
        }
    }

    static class KernelServerHandler extends SimpleChannelInboundHandler<ByteBuf> {
        @Override
        protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) {
            CustomMessage customMessage = CustomMessage.decode(msg);
            System.out.println("Received from client: " + customMessage);

            // 回复客户端
            CustomMessage response = new CustomMessage((short) 2, "Server ACK: " + customMessage.getMsg());
            ctx.writeAndFlush(response.encode());
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            System.out.println("exception: " + cause);
            ctx.close();
        }
    }
}
```

客户端结果

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

服务端结果

<div align="left"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

## 组件设计

### 引导器

封装底层的 Channel、EventLoop、Handler、Pipeline 等复杂初始化逻辑，提供一个流式 API 来启动整个网络程序。

#### group()

配置两个线程组

bossGroup：负责接收连接（accept）

workerGroup：负责处理读写事件（read/write）

#### channel()

指定使用的 Channel 类型

#### option() / childOption()

option()：配置 ServerSocketChannel&#x20;

childOption()：配置 SocketChannel（每个连接）

#### childHandler()

配置新连接建立后，如何初始化它的ChannelPipeline。

#### bind() / connect()

服务端调用 bind(port) → 绑定端口，启动监听；

客户端调用 connect(host, port) → 发起连接。

### EventLoop

EventLoop 是一个不断循环执行任务的线程（事件循环），负责处理 Channel 的所有 IO 事件与任务。

### Channel

Channel 是 Java NIO 的一个基本构造。 它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接

### ChannelFuture

在 Netty 中，所有 IO 操作都是异步的（非阻塞）。比如 connect()、write()、bind()、close() 等操作不会立即完成，而是立即返回一个 ChannelFuture 对象。

ChannelFuture 表示一个“还没完成、但将来会完成”的结果。

### ChannelPipeline

ChannelPipeline 提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站和出站事件流的API。当 Channel被创建时，它会被自动地分配到它专属的ChannelPipeline。

### ChannelHandler

从应用程序开发人员的角度来看，Netty的主要组件是ChannelHandler，它充当了所有处理入站和出站数据的应用程序逻辑的容器。因为ChannelHandler的方法是由网络事件触发的。事实上，ChannelHandler 可专门用于几乎任何类型的动作，例如将数据从一种格式转换为另外一种格式，或者处理转换过程中所抛出的异常。

建议阅读书籍：《Netty实战》
