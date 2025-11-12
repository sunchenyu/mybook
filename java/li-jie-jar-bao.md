# 理解jar包

## 最简单的jar包示例

Teacher.java

```java
public class Teacher {
    public static void greeting() {
        System.out.println("Welcome!");
    }
}
```

Welcome.java

```java
public class Welcome {
    public static void main(String[] args) {
        Teacher.greeting();
    }
}
```

使用javac将这两个java文件构建成class文件

<div align="left"><figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure></div>

将两个class文件打成jar包，执行如下命令

```
jar -cvf test.jar Teacher.class Welcome.class
```

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

修改jar包里面的MANIFEST.MF文件，新增Main-Class入口配置

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

执行jar包

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

## 如果带有包名

包名即目录，本地需要存在目录，打的jar包当中会按照目录去构建

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

修改jar包里面的的MANIFEST.MF文件，添加入口为com.test.Welcome

<div align="left"><figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

执行jar包

<div align="left"><figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure></div>

## 如果存在依赖jar包

需要配置Class-Path参数

将主类文件、依赖jar和配置文件都打进jar包

```
jar -cvf test.jar Prefix.class lib/mysql-connector-java-8.0.20.jar prefix.properties
```

修改里面的META-INF/MANIFEST.MF文件

```
Manifest-Version: 1.0
Created-By: 1.8.0_412 (Red Hat, Inc.)
Main-Class: Prefix
Class-Path: lib/mysql-connector-java-8.0.20.jar
```

新增的参数如下：

<mark style="color:red;">Main-Class: Prefix</mark>\ <mark style="color:red;">Class-Path: lib/mysql-connector-java-8.0.20.jar</mark>

执行jar包运行

<div align="left"><figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure></div>

## 执行jar包当中的其他main方法

线上需要我提供一个客户端，去调用自己的服务端方法，我不想每次都构建新的客户端jar包，那么就可以直接在原有项目里写个客户端的main方法，然后在线上执行的时候指定入口。

示例如下：

```
package com.ultra.isg.iwf.kernel.client;

import com.ultra.isg.iwf.kernel.innitializer.KernelServerInitializer;
import com.ultra.isg.iwf.kernel.msg.SipTestMessage;
import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.PooledByteBufAllocator;
import io.netty.channel.Channel;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

import java.nio.charset.StandardCharsets;

public class KernelClient {

    public static void main(String[] args) throws InterruptedException {
        if (args.length < 5) {
            System.out.println("Usage: KernelClient <ecnum> <mobile> <content> <type>");
            return;
        }
        byte[] content;
        Integer type = Integer.parseInt(args[3]);
        if (type == 0 || type == 4) {
            content = args[4].getBytes(StandardCharsets.ISO_8859_1);
        } else {
            content = args[4].getBytes(StandardCharsets.UTF_16BE);
        }
        String ecnum = args[0];
        String mobile = args[1];
        String sipAddr = args[2];


        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        try {
            Bootstrap b = new Bootstrap();
            b.group(bossGroup)
                    .channel(NioSocketChannel.class)
                    .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                    .handler(new KernelServerInitializer());
            ChannelFuture channelFuture = b.connect("127.0.0.1", 9034).sync();
            Channel channel = channelFuture.channel();

            SipTestMessage sipTestMessage = new SipTestMessage();
            sipTestMessage.setEcnum(ecnum);
            sipTestMessage.setMobile(mobile);
            sipTestMessage.setSipAddr(sipAddr);
            sipTestMessage.setType(type);
            sipTestMessage.setContent(content);
            String finalMobile = mobile;
            String finalEcnum = ecnum;
            channel.writeAndFlush(sipTestMessage).addListener(future -> {
                if (future.isSuccess()) {
                    System.out.println("send rpc success;mobile:" + finalMobile + ";ecnum" + finalEcnum);
                } else {
                    System.out.println("send rpc fail;mobile:" + finalMobile + ";ecnum" + finalEcnum);
                }
            });
            channel.close();
        } finally {
            bossGroup.shutdownGracefully();
        }
    }
}
```

使用的时候只需要如下调用

{% code overflow="wrap" fullWidth="false" %}
```
java -cp ./ultra-smsc-isg.jar com.ultra.isg.iwf.kernel.client.KernelClient 1065xxxxxxxx 173xxxxxxxx sip:+86173xxxxxxxx@xx.ims.mnc011.mcc460.3gppnetwork.org 8 中文短信发送测试
```
{% endcode %}

避免了线上存在很多脚本和测试工具，也可以用来模块隔离、便于测试

