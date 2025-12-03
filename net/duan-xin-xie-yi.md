---
description: 短信协议的简单介绍
---

# 短信协议

## 介绍

SMPP协议短信中心对外使用的协议消息头如下

<table data-header-hidden><thead><tr><th width="189"></th><th width="114"></th><th width="100"></th><th></th></tr></thead><tbody><tr><td>字 段</td><td>长度 字节</td><td>类 型</td><td>描 述</td></tr><tr><td>Command_Length</td><td>4</td><td>Integer</td><td>消息包的长度 包括此字段</td></tr><tr><td>Command_ID</td><td>4</td><td>Integer</td><td>这个字段表明此条短消息的类型 例如ESME_SUB_SM表示此消息为ESME向SMSC提交短消息</td></tr><tr><td>Command_status</td><td>4</td><td>Integer</td><td>此字段表示一个命令的成功与失败如失败 指示引起失败的错误类型等信息 它在请求中必须设为0</td></tr><tr><td>Sequence_No</td><td>4</td><td>Integer</td><td>此字段表示消息的序列号 它由ESME产生 它是消息和它的应答之间的对应标志 它的数值在0到0XFFFFFFFF间必须保证严格单调的递增 当达到0XFFFFFFFF时 从0开始下一循环</td></tr></tbody></table>

CMPP协议中国移动使用的短信网关协议消息头如下

<table data-header-hidden><thead><tr><th width="191"></th><th width="122"></th><th width="109"></th><th></th></tr></thead><tbody><tr><td>字 段</td><td>长度 字节</td><td>类 型</td><td>描 述</td></tr><tr><td>Total_Length</td><td>4</td><td>Unsigned Integer</td><td>消息总长度(含消息头及消息体)</td></tr><tr><td>Command_Id</td><td>4</td><td>Unsigned Integer</td><td>命令或响应类型</td></tr><tr><td>Sequence_Id</td><td>4</td><td>Unsigned Integer</td><td>消息流水号,顺序累加,步长为1,循环使用（一对请求和应答消息的流水号必须相同）</td></tr></tbody></table>

SGIP协议中国联通使用的短信网关协议

<table data-header-hidden><thead><tr><th width="184"></th><th width="125"></th><th width="118"></th><th></th></tr></thead><tbody><tr><td>字 段</td><td>长度 字节</td><td>类 型</td><td>描 述</td></tr><tr><td>Message Length</td><td>4</td><td>Integer</td><td>消息的总长度(字节)</td></tr><tr><td>Command ID</td><td>4</td><td>Integer</td><td>命令ID</td></tr><tr><td>Sequence Number</td><td>12</td><td>Integer</td><td>序列号</td></tr></tbody></table>

SMGP协议中国电信使用的短信网关协议

<table data-header-hidden><thead><tr><th width="162"></th><th width="132"></th><th width="105"></th><th></th></tr></thead><tbody><tr><td>字 段</td><td>长度 字节</td><td>类 型</td><td>描 述</td></tr><tr><td>PacketLength</td><td>4</td><td>Integer</td><td>数据包长度</td></tr><tr><td>RequestID</td><td>4</td><td>Integer</td><td>请求标识</td></tr><tr><td>SequenceID</td><td>12</td><td>Integer</td><td>消息流水号</td></tr></tbody></table>

IWPP协议互通网关使用的短信网关协议

<table data-header-hidden><thead><tr><th width="144"></th><th width="118"></th><th width="105"></th><th></th></tr></thead><tbody><tr><td>字 段</td><td>长度 字节</td><td>类 型</td><td>描 述</td></tr><tr><td>Total_Length</td><td>4</td><td>Integer</td><td>消息总长度(含消息头及消息体)</td></tr><tr><td>Command_ID</td><td>4</td><td>Integer</td><td>命令或响应类型</td></tr><tr><td>Sequence_ID</td><td>12</td><td>Integer</td><td>消息流水号,顺序累加,步长为 1,循环使用（一对请求和应答消息的流水号必须相同）</td></tr></tbody></table>

## 相同点和区别

都属于消息头+消息头的协议，消息头基本上会携带命令字、包长度、消息序号数据区别消息头的细微区别。

消息体依据命令字的不同会有不同的格式，比如短信提交、提交响应、状态报告、状态报告响应等数据。具体需要参见短信相关协议文档。

## 示例如下

SMGP协议的登录包说明

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

SGIP协议的登录包说明

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

CMPP的登录响应包说明

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

