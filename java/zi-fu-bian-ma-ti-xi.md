# 编码体系

## ASCII到Unicode

计算机最初只服务于英语用户，ASCII（American Standard Code for Information Interchange） 因此成为最早的通用字符编码标准。它只定义了 128 个字符（0–127），包括英文字母、数字、标点符号和控制字符。ASCII 采用 7 位二进制编码（最高位为 0），占用 1 字节。

然而，随着计算机走向全球，ASCII 无法表示中文、日文、韩文等复杂文字，各国纷纷制定自己的编码体系。这导致“同一个字节内容在不同系统下显示不同字符”的混乱现象。为了解决这种“编码碎片化”问题，Unicode 联盟（Unicode Consortium） 在 1991 年提出了统一全球字符集的标准——Unicode 标准。

## Unicode编码 <a href="#dzge-1761886988313" id="dzge-1761886988313"></a>

Unicode 的目标是：为世界上每一个字符分配唯一编号（码点 Code Point）。但是Unicode 只定义字符与编号的映射，并不规定这些编号在计算机中如何存储。

于是，产生了多种实现方式（即不同的“存储格式”或“编码方式”）：

* UTF-8（互联网标准）
* UTF-16（Java 内部采用）
* UTF-32
* UCS-2（早期固定两字节实现）

## Unicode的“平面”概念

Unicode 理论上可表示的字符范围是U+000000 \~ U+10FFFF。也就是 21 位（2¹¹⁰¹⁶）≈ 111 万个字符。

为了方便管理，Unicode 把整个编码空间分为 17 个平面（Plane 0～16）

<table><thead><tr><th width="112">平面编号</th><th>名称</th><th width="191">范围</th><th>内容简介</th></tr></thead><tbody><tr><td><strong>Plane 0</strong></td><td>BMP（Basic Multilingual Plane）基本多文种平面</td><td>U+0000～U+FFFF</td><td>绝大多数常用字符：英文字母、数字、标点、汉字、日文、韩文等</td></tr><tr><td>Plane 1</td><td>SMP（Supplementary Multilingual Plane）补充多文种平面</td><td>U+10000～U+1FFFF</td><td>历史文字、音乐符号、表情符号（Emoji）等</td></tr><tr><td>Plane 2</td><td>SIP（Supplementary Ideographic Plane）补充表意文字平面</td><td>U+20000～U+2FFFF</td><td>扩展汉字（CJK Extension B～D）</td></tr><tr><td>Plane 3～13</td><td>保留</td><td>——</td><td>预留给未来使用</td></tr><tr><td>Plane 14</td><td>SSP（Supplementary Special-purpose Plane）</td><td>U+E0000～U+EFFFF</td><td>特殊用途字符（如标签、控制符）</td></tr><tr><td>Plane 15～16</td><td>Private Use Areas（私人使用区）</td><td>U+F0000～U+10FFFF</td><td>用户自定义符号</td></tr></tbody></table>

### 常见Unicode实现方式对比 <a href="#o7jv-1761887025795" id="o7jv-1761887025795"></a>

<table data-header-hidden><thead><tr><th width="115"></th><th></th><th></th><th></th></tr></thead><tbody><tr><td>编码方式</td><td>特性</td><td>字节长度</td><td>是否兼容 ASCII</td></tr><tr><td>UTF-8</td><td>可变长编码（1～4 字节），互联网标准</td><td>1–4 字节</td><td>完全兼容</td></tr><tr><td>UTF-16</td><td>可变长编码（2 或 4 字节），Java 内部使用</td><td>2 或 4 字节</td><td>不兼容</td></tr><tr><td>UTF-32</td><td>固定长度编码，每字符 4 字节</td><td>4 字节</td><td>不兼容</td></tr><tr><td>UCS-2</td><td>固定长度编码，每字符 2 字节（不支持扩展字符）</td><td>2 字节</td><td>不兼容</td></tr></tbody></table>

## ISO-8859系列

在 Unicode 出现前，欧洲也遇到了 ASCII 不足的问题。因此，ISO 制定了一系列 8 位编码标准：ISO-8859 系列。其中最常见的是：

* ISO-8859-1（Latin-1）：覆盖西欧语言（英语、法语、德语、西班牙语等）。
* 字符范围：0–255，共 256 个字符。
* 前 128 个与 ASCII 一致。
* 扩展部分（128–255）增加了带重音符号的拉丁字母。

## 中文编码的演化：GB系列标准 <a href="#yzev-1761887109821" id="yzev-1761887109821"></a>

<table data-header-hidden><thead><tr><th width="91"></th><th width="83"></th><th width="82"></th><th width="167"></th><th></th></tr></thead><tbody><tr><td>编码</td><td>年代</td><td>字符数</td><td>字节长度</td><td>特性</td></tr><tr><td>GB2312</td><td>1980年</td><td>6763</td><td>1 字节（ASCII）或 2 字节（汉字）</td><td>仅支持简体中文，覆盖常用汉字</td></tr><tr><td>GBK</td><td>1995年</td><td>21003</td><td>1 字节（ASCII）或 2 字节（扩展汉字）</td><td>向下兼容 GB2312，增加繁体、日文、俄文等</td></tr><tr><td>GB18030</td><td>2005年</td><td>7 万+</td><td>1、2 或 4 字节</td><td>国家强制标准，支持少数民族文字，全面覆盖 Unicode</td></tr></tbody></table>

这些编码采用 双字节或多字节变长编码，并与 ASCII 兼容，保证英文部分仍占 1 字节。

## Java 与字符编码 <a href="#pv6b-1761887134734" id="pv6b-1761887134734"></a>

Java 的设计目标是“平台无关”，因此内部统一使用 UTF-16 存储字符串。每个 char 占用 2 个字节。

### 编码示例

```java
public static void main(String[] args) throws Exception {
   String text = "中文ABC";

   // 以不同编码查看原字符串的字节表示
   System.out.println("原字符串：" + text);
   printBytes(text, StandardCharsets.UTF_16);
   printBytes(text, StandardCharsets.UTF_16BE);
   printBytes(text, StandardCharsets.UTF_16LE);
   printBytes(text, StandardCharsets.UTF_8);
   printBytes(text, Charset.forName("GBK"));
   printBytes(text, StandardCharsets.ISO_8859_1);
}

private static void printBytes(String s, Charset charset) {
   byte[] bytes = s.getBytes(charset);
   System.out.printf("%s -> %s%n", charset.displayName(), Arrays.toString(bytes));
}
```

执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

### 问题

**为什么UTF-16、UTF-16BE、UTF-16LE得到的结果不一样？**

String.getBytes(StandardCharsets.UTF\_16) 会在字节前加上 BOM（Byte Order Mark）（即 FE FF 或 FF FE），用于指明字节序；

UTF-16BE（Big Endian）与 UTF-16LE（Little Endian）不会自动添加 BOM；

因此，UTF-16 与 UTF-16BE / UTF-16LE 的输出会不同。

**为什么中文经过ISO\_8859\_1编码之后得到固定的63？**

ISO\_8859\_1 是单字节编码，不能表示中文字符。 所以 text.getBytes(StandardCharsets.ISO\_8859\_1) 会发生编码降级（不可映射字符），结果是每个中文被替换成 ?（即 0x3F）

## 短信系统中的编码

很多短信协议要求的编码是UCS2，比如SGIP协议当中会有如下说明：

<div align="left"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

### **那UCS2又是什么呢？**

在 Unicode 早期，所有字符都在BMP内（U+0000 \~ U+FFFF）。那时的UTF-16实际上与UCS-2 一模一样：每个字符占 2 字节。所以很多系统“支持 UCS-2”，其实就是使用UTF-16BE的早期实现。

随着 Unicode 扩展到U+10FFFF，字符数量超过了 65536（2¹⁶），UCS-2 就无法再容纳所有字符。
