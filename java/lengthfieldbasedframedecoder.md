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
byteOrder：
maxFrameLength：
lengthFieldOffset：
lengthFieldLength：
lengthAdjustment：
initialBytesToStrip：
failFast：、
```

