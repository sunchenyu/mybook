# LengthFieldBasedFrameDecoder

## 概述

根据消息头中的“长度字段”自动分割 ByteBuf，提取完整的一帧

## 构造器

```java
    public LengthFieldBasedFrameDecoder(
            ByteOrder byteOrder, int maxFrameLengthmaxFrameLength：, int lengthFieldOffset, int lengthFieldLength,
            int lengthAdjustment, int initialBytesToStrip, boolean failFast) {

        this.byteOrder = checkNotNull(byteOrder, "byteOrder");

        checkPositive(maxFrameLength, "maxFrameLength");

        checkPositiveOrZero(lengthFieldOffset, "lengthFieldOffset");

        checkPositiveOrZero(initialBytesToStrip, "initialBytesToStrip");

        if (lengthFieldOffset > maxFrameLength - lengthFieldLength) {
            throw new IllegalArgumentException(
                    "maxFrameLength (" + maxFrameLength + ") " +
                    "must be equal to or greater than " +
                    "lengthFieldOffset (" + lengthFieldOffset + ") + " +
                    "lengthFieldLength (" + lengthFieldLength + ").");
        }

        this.maxFrameLength = maxFrameLength;
        this.lengthFieldOffset = lengthFieldOffset;
        this.lengthFieldLength = lengthFieldLength;
        this.lengthAdjustment = lengthAdjustment;
        this.lengthFieldEndOffset = lengthFieldOffset + lengthFieldLength;
        this.initialBytesToStrip = initialBytesToStrip;
        this.failFast = failFast;
    }
```

### 构造参数详细说明

```
byteOrder：字节序
maxFrameLength：数据帧的最大长度
lengthFieldOffset：长度字段偏移量
lengthFieldLength：长度字段的长度
lengthAdjustment：长度字段的补偿值
initialBytesToStrip：处理完毕给到下游解析的数据需要跳过的字节数
failFast：是否立即抛出异常
```

#### 长度修正值

长度修正值如何正确填写，他是一个什么参数，解码器如何知道整个包的长度？

因为长度偏移量和长度所占字节是固定的和已知的，唯一区别就是长度域的值可能代表的含义不同。

有的代表整个包的长度，有的只代表包体的长度，基于这个长度可能代表的含义不同，所以增加了一个字段，用于调整这个解码器读取帧长问题，这个字段就是长度修正值。

计算规则：帧长度 = 长度域的值 + 长度字段偏移量的值 + 长度字段的长度值 + lengthAdjustment



示例一

```
EmbeddedChannel channel = new EmbeddedChannel(
        new LengthFieldBasedFrameDecoder(
                ByteOrder.BIG_ENDIAN,
                1024,  // 最大帧长度1024
                2,     // 长度字段偏移量，从2开始
                4,     // 长度字段占4字节（这个地方用的是内容的长度：为5）
                0,     // 长度调整值为0
                0,     // 解码后不跳过任何字节
                true
        )
);

ByteBuf buf = Unpooled.buffer();

// 第一条消息: "hello"
byte[] msg1 = "hello".getBytes(StandardCharsets.UTF_8);
buf.writeShort(1);
buf.writeInt(msg1.length);  //长度仅代表内容的长度
buf.writeBytes(msg1);

// 第二条消息: "world"
byte[] msg2 = "world".getBytes(StandardCharsets.UTF_8);
buf.writeShort(1);
buf.writeInt(msg2.length);  //长度仅代表内容的长度
buf.writeBytes(msg2);

channel.writeInbound(buf);

ByteBuf byteBuf1 = channel.readInbound();
ByteBuf byteBuf2 = channel.readInbound();

System.out.println("Frame1 = " + ByteBufUtil.prettyHexDump(byteBuf1));
System.out.println("Frame2 = " + ByteBufUtil.prettyHexDump(byteBuf2));

channel.finish();
```

结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

示例二

```
EmbeddedChannel channel = new EmbeddedChannel(
        // 配置 LengthFieldBasedFrameDecoder
        new LengthFieldBasedFrameDecoder(
                1024,  // 最大帧长度
                2,     // 长度字段偏移量，从2开始
                4,     // 长度字段占4字节
               -6,     // 长度修正值
                0      // 解码后不跳过任何数据
        )
);

ByteBuf buf = Unpooled.buffer();

// 第一条消息: "hello"
byte[] msg1 = "hello".getBytes(StandardCharsets.UTF_8);
buf.writeShort(1);
buf.writeInt(msg1.length + 8); //长度字段代表整个包的长度
buf.writeShort(1);
buf.writeBytes(msg1);

// 第二条消息: "world"
byte[] msg2 = "world".getBytes(StandardCharsets.UTF_8);
buf.writeShort(1);
buf.writeInt(msg2.length + 8); //长度字段代表整个包的长度
buf.writeShort(1);
buf.writeBytes(msg2);

channel.writeInbound(buf);

ByteBuf byteBuf1 = channel.readInbound();
ByteBuf byteBuf2 = channel.readInbound();

System.out.println("Frame1 = " + ByteBufUtil.prettyHexDump(byteBuf1));
System.out.println("Frame2 = " + ByteBufUtil.prettyHexDump(byteBuf2));

channel.finish();
```

结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure></div>
