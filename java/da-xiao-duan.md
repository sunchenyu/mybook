# 大小端

## 一、大小端概念

### 1、高位低位

一个数，左边是高位，右边是低位。比如十进制数183，1是百位，8是十位，3是个位，左边是高位，右边是低位。&#x20;

同理，一个16进制数0x12345678，0x12表示高位字节，0x78表示低位字节。

### 2、高地址低地址

一个字节数组byte\[] a = new byte\[4]&#x20;

byte\[0] 表示低地址&#x20;

byte\[3]表示高地址

### 3、大小端

以0x12345678为例&#x20;

0x12为高位数据&#x20;

0x78为低位数据

**（1）Little-Endian**&#x20;

低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。

**示例图：**&#x20;

```
栈底 （高地址） 
buf[3] (0x12) -- 高位数据 
buf[2] (0x34) 
buf[1] (0x56) 
buf[0] (0x78) -- 低位数据 
栈顶 （低地址）
```

**（2） Big-Endian**&#x20;

高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

**示例图：**&#x20;

```
栈底 （高地址） 
buf[3] (0x78) -- 低位数据 
buf[2] (0x56) 
buf[1] (0x34) 
buf[0] (0x12) -- 高位数据 
栈顶 （低地址）
```

### 4、java默认是采用大端存储

测试方法，首先先写出一个大端存储的方法，如下

```
    public static byte[] int2Byte4(int i) {
        byte result[] = new byte[4];
        result[3] = (byte) (0xff & i);                //低位字节存放在高地址，表示大端
        result[2] = (byte) ((i >> 8) & 0xff);
        result[1] = (byte) ((i >> 16) & 0xff);
        result[0] = (byte) ((i >> 24) & 0xff);
        return result;
    }
```

打印byte数组的16进制字符串的方法

```
    public static String bytesToHex(byte[] bytes) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(bytes[i] & 0xFF);
            if (hex.length() < 2) {
                sb.append(0);
            }
            sb.append(hex);
        }
        return sb.toString();
    }
```

编写测试方法

```
package com.suncy.netty.bl;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;

import java.io.IOException;

public class Test {
    public static void main(String[] args) throws IOException {
        ByteBuf byteBuf = Unpooled.buffer(8);
        byteBuf.writeInt(0x12345678);
        byteBuf.writeBytes(BigEndianUtil.int2Byte4(0x12345678));

        byte[] a = byteBuf.array();
        System.out.println(ByteUtils.bytesToHex(a));
    }
}
```

打印结果如下

```
1234567812345678
```

也就是直接写入int数据和写入的int转byte的大端数据是一样的，说明java默认采用大端存储方式。
