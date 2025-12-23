# 短信网关拆包粘包解决方案

### 一、长度字段协议如何正确拆包 <a href="#ahcn-1766479361710" id="ahcn-1766479361710"></a>

### 在基于 TCP 的通信中，拆包与粘包 是所有网络程序绕不开的问题。 <a href="#x5di-1766479361712" id="x5di-1766479361712"></a>

Netty 提供了多种内置解码器，其中 LengthFieldBasedFrameDecoder 是工程中使用频率最高、同时也是最容易配置错误的一个。

本文将从原理、参数含义、长度修正值的真实作用出发，结合两个完整示例，讲清楚：

解码器到底是如何“知道一整个包有多长的”

***

### 二、LengthFieldBasedFrameDecoder 是什么？ <a href="#aesj-1766479361721" id="aesj-1766479361721"></a>

LengthFieldBasedFrameDecoder 是 Netty 提供的一个通用拆包解码器：

根据消息头中的“长度字段”，自动从 ByteBuf 中切割出一整帧数据它不关心协议语义，只关心三件事：

长度字段在哪里

长度字段占多少字节

长度字段的值“代表什么”

***

### 三、构造器与参数总览 <a href="#ucjl-1766479361736" id="ucjl-1766479361736"></a>

public LengthFieldBasedFrameDecoder(ByteOrder byteOrder,int maxFrameLength,int lengthFieldOffset,int lengthFieldLength,int lengthAdjustment,int initialBytesToStrip,boolean failFast)

***

### 四、构造参数详细说明 <a href="#vrfn-1766479361758" id="vrfn-1766479361758"></a>

#### byteOrder（字节序） <a href="#jcyb-1766479361760" id="jcyb-1766479361760"></a>

* BIG\_ENDIAN（大端）
* LITTLE\_ENDIAN（小端）

必须和协议保持一致。

***

#### maxFrameLength（最大帧长度） <a href="#oksx-1766479361769" id="oksx-1766479361769"></a>

单个数据包允许的最大长度，防止恶意或错误数据导致内存溢出超过该长度会直接抛异常（或延迟抛出，取决于 failFast）

***

#### lengthFieldOffset（长度字段偏移量） <a href="#ivxt-1766479361776" id="ivxt-1766479361776"></a>

长度字段在整个消息中的起始位置例如：| flag(2) | length(4) | body |lengthFieldOffset = 2

***

#### lengthFieldLength（长度字段长度） <a href="#id-4jd5-1766479361787" id="id-4jd5-1766479361787"></a>

长度字段本身占多少字节常见取值：

* 2（short）
* 4（int）
* 8（long）

***

#### initialBytesToStrip（剥离字节数） <a href="#wth2-1766479361800" id="wth2-1766479361800"></a>

解码完成后，交给下游 Handler 前，需要跳过的字节数典型用途：不希望下游再看到协议头

***

#### failFast（是否立即抛异常） <a href="#h3wq-1766479361807" id="h3wq-1766479361807"></a>

true：一发现超长立即抛异常false：等接收完整帧后再抛异常

***

### 五、最容易误解的参数：lengthAdjustment <a href="#id-9jxj-1766479361814" id="id-9jxj-1766479361814"></a>

#### 它解决的是什么问题？ <a href="#pyke-1766479361816" id="pyke-1766479361816"></a>

虽然 长度字段的位置和长度是固定的，但长度字段“表示的含义”并不统一。

常见两种情况：

长度字段表示 包体长度

长度字段表示 整个包长度

于是 Netty 引入了 lengthAdjustment（长度修正值）。

***

#### 解码器如何计算一帧的总长度？ <a href="#id-9xil-1766479361830" id="id-9xil-1766479361830"></a>

Netty 的内部计算公式为：

帧长度 = 长度字段的值 + lengthFieldOffset + lengthFieldLength + lengthAdjustment

***

#### lengthAdjustment 的本质 <a href="#s3ul-1766479361837" id="s3ul-1766479361837"></a>

用于“纠正”长度字段值的真实含义。

它不是魔法值，而是一个数学补偿项。

***

### 六、示例一：长度字段表示“包体长度” <a href="#ryos-1766479361842" id="ryos-1766479361842"></a>

#### 协议格式 <a href="#al3i-1766479361844" id="al3i-1766479361844"></a>

\| type(2) | length(4) | body |

length = body.length

***

#### 解码器配置 <a href="#id-7dct-1766479361851" id="id-7dct-1766479361851"></a>

new LengthFieldBasedFrameDecoder( ByteOrder.BIG\_ENDIAN, 1024, 2, 4, 0, true )

不需要修正 ，所以lengthAdjustment 填0

***

#### 数据构造 <a href="#azno-1766479361861" id="azno-1766479361861"></a>

buf.writeShort(1);

buf.writeInt(msg.length);

buf.writeBytes(msg);

***

#### 为什么 lengthAdjustment = 0？ <a href="#aiqo-1766479361870" id="aiqo-1766479361870"></a>

因为：

帧长度 = bodyLength + 2 + 4 + 0

正好等于整个包长度。

***

### 七、示例二：长度字段表示“整个包长度” <a href="#pz0i-1766479361879" id="pz0i-1766479361879"></a>

#### 协议格式 <a href="#smyz-1766479361881" id="smyz-1766479361881"></a>

\| type(2) | length(4) | type(2) | body |

length = 整个包长度

***

#### 解码器配置 <a href="#rgm6-1766479361888" id="rgm6-1766479361888"></a>

new LengthFieldBasedFrameDecoder( 1024, 2, 4, -6, 0 )

***

#### 为什么 lengthAdjustment = -6？ <a href="#twqz-1766479361894" id="twqz-1766479361894"></a>

因为：

lengthFieldOffset = 2

lengthFieldLength = 4

Netty会默认再加一次这 6 个字节，但这里长度字段已经包含了它们。

因此必须减回去：lengthAdjustment = -(2 + 4) = -6

***

#### 最终计算结果 <a href="#umpz-1766479361911" id="umpz-1766479361911"></a>

帧长度 = 长度字段值 + 2 + 4 - 6 = 长度字段值

正好匹配真实帧长。

***

### 八、一个记忆公式 <a href="#czu0-1766479361918" id="czu0-1766479361918"></a>

lengthAdjustment =长度字段“没有算进去”的字节数 − 已经多算的字节数

长度字段只算 body → 通常是 0

长度字段算了整个包 → 通常是 -(offset + length)

***

### 八、常见错误总结 <a href="#vtgx-1766479361927" id="vtgx-1766479361927"></a>

把 lengthAdjustment 当成“拍脑袋参数”

不搞清楚长度字段代表什么

忽略 initialBytesToStrip 的作用

长度字段算重复，导致半包或拆包错误

***

### 九、结语 <a href="#dyrp-1766479361938" id="dyrp-1766479361938"></a>

LengthFieldBasedFrameDecoder 并不复杂，复杂的是协议本身。

一旦你搞清楚三件事：

长度字段在哪里长度

字段占多少字节

长度字段“到底表示什么”

那么：lengthAdjustment 就是一个简单的数学题
