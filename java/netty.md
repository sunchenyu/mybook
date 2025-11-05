# Netty

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

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

服务端结果

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



## 组件设计

服务器引导器



客户端引导器



EventLoop



Channel



ChannelFuture



ChannelPipeline



ChannelHandler



建议阅读书籍：《Netty实战》
