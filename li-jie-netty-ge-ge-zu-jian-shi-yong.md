# 理解netty各个组件使用

服务端

```
package com.ultra.wap;

import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;

public class NettyServerExample {

    public static void main(String[] args) throws InterruptedException {
        // 1. 创建 EventLoopGroup：boss 处理 accept 事件，worker 处理客户端读写
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1); // boss 线程组，只处理连接接收
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(); // worker 线程组，处理客户端读写

        try {
            // 2. 手动创建 NioServerSocketChannel（服务端监听通道）
            NioServerSocketChannel serverChannel = new NioServerSocketChannel();

            // 3. 配置服务端通道参数
            ChannelConfig config = serverChannel.config();
            config.setOption(ChannelOption.SO_BACKLOG, 128); // 连接队列大小
            config.setOption(ChannelOption.SO_REUSEADDR, true); // 端口复用
            config.setOption(ChannelOption.SO_KEEPALIVE, true); // 保活

            // 4. 给 ServerSocketChannel 添加处理器：处理 accept 事件（接收客户端连接）
            serverChannel.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    // msg 是客户端连接的 SocketChannel（NioSocketChannel）
                    SocketChannel clientChannel = (SocketChannel) msg;
                    System.out.println("新客户端连接: " + clientChannel.remoteAddress());

                    // 5. 配置客户端通道的 pipeline（添加解码器、编码器、业务处理器）
                    clientChannel.pipeline().addLast(
                            // 按行解码，最大帧长度 1024
                            new LineBasedFrameDecoder(1024),
                            // 字符串解码器，指定编码格式（避免乱码）
                            new StringDecoder(StandardCharsets.UTF_8),
                            // 字符串编码器
                            new StringEncoder(StandardCharsets.UTF_8),
                            // 自定义业务处理器
                            new SimpleChannelInboundHandler<String>() {
                                @Override
                                protected void channelRead0(ChannelHandlerContext ctx, String msg) {
                                    // 接收客户端消息并回复
                                    System.out.println("收到客户端[" + ctx.channel().remoteAddress() + "]消息: " + msg);
                                    ctx.writeAndFlush("服务端回复: " + msg + "\n");
                                }

                                @Override
                                public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
                                    // 异常处理，关闭连接
                                    System.out.println("客户端连接异常: " + cause.getMessage());
                                    ctx.close();
                                }

                                @Override
                                public void channelInactive(ChannelHandlerContext ctx) {
                                    System.out.println("客户端断开连接: " + ctx.channel().remoteAddress());
                                }
                            }
                    );

                    // 6. 将客户端通道注册到 workerGroup，并设置为非阻塞
                    workerGroup.register(clientChannel).addListener(future -> {
                        if (future.isSuccess()) {
                            System.out.println("客户端通道注册成功: " + clientChannel.remoteAddress());
                        } else {
                            System.err.println("客户端通道注册失败: " + future.cause().getMessage());
                        }
                    });
                }
            });

            // 7. 将服务端通道注册到 bossGroup
            bossGroup.register(serverChannel).sync();

            // 8. 绑定端口并启动监听
            ChannelFuture bindFuture = serverChannel.bind(new InetSocketAddress(8083)).sync();
            System.out.println("Netty 服务端启动成功，监听端口: 8083");

            // 9. 阻塞主线程，直到服务端通道关闭
            bindFuture.channel().closeFuture().sync();
        } finally {
            // 10. 优雅关闭线程组
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
            System.out.println("Netty 服务端已关闭");
        }
    }
}
```

客户端

```
package com.ultra.wap;

import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.util.Scanner;

public class Test {
    public static void main(String[] args) throws InterruptedException, IOException {
        NioSocketChannel ch = new NioSocketChannel();
        ch.config().setOption(ChannelOption.SO_KEEPALIVE, true);
        ch.config().setOption(ChannelOption.TCP_NODELAY, true);

        ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
        ch.pipeline().addLast("a", new StringEncoder());
        ch.pipeline().addLast("b", new StringDecoder());

        NioEventLoopGroup loop = new NioEventLoopGroup(1); // 手动拿一个 EventLoop
        loop.register(ch).sync();  // 注册 Channel 到 EventLoop
        ch.connect(new InetSocketAddress("127.0.0.1", 8083)).sync();
        ch.writeAndFlush("hello world\n");

        Scanner scanner = new Scanner(System.in);
        while (true) {
            String next = scanner.next();
            ch.writeAndFlush(next + "\n");
        }
    }
}
```
