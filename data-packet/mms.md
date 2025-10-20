# 彩信数据包分析

## 一、wap-push发送彩信通知包分析

**相关协议Wireless Application Protocol Downloads** [http://www.openmobilealliance.org/wp/Affiliates/WAP.html](http://www.openmobilealliance.org/wp/Affiliates/WAP.html)&#x20;

**说明**

以下数据均为16进制

### 1、WAP PUSH编码分析

#### **第一部分  WDP协议**

```
**单包情况下**
06          user-data-head
05          udh ie identifier  port numbers
04          udh port number ie length
0b84        wap push port  2498
23f0        wap-wsp port  9200
**多包情况下**
06 =》0b
同时多出5字节
00
03
03
xx     total number of segmetns in datagram 分成几个短信发送  例如：01
yy     segment count        当前包的序列号   例如：01
```

#### **第二部分  WSP部分**

参考 wap-230-wsp-20010705-a.pdf wsp 信息头

```
06         transatcion id 流水号
06         pdu type   为6 表示（push）

wsp 信息体部分
wbxml格式
06          头域部分长度  不包括pdu类型 和 流水号（Length of Content type + Header）
03 ae       03 表示头域  ae表示头域值  application/vnd.wap.sic  
81 ea       chartset = utf-8
8d 4a       8d = content-length    4a =  wbxml 模块长度 （0x7E,//Push消息体的长度 16）

mms通知格式
0x22          表示下面content type的长度
61 70 70 6C 69 63 61 74 69 6F 6E 2F 76 6E 64 2E 77 61 70 2E 6D 6D 73 2D 6D 65 73 73 61 67 65 00        application/vnd.wap.mms-message（这个值不是内置的，所以用特殊字符串编码方式进行编码）
0xb487      push-flag
AF          2f=X-Wap-Application-Id  这个key是内置的 所以用af表示
84          84
```

#### **第三部分  根据上面第二部分的信息体**

**如果是彩信协议 MMSEP**&#x20;

参考WAP-209-MMSEncapsulation-20010601-a.pdf文档

```
第三部分  
0x8c82          0c=X-Mms-Message-Type:    82=m-notification-ind
98 4A 51 32 35 78 55 5A 51 5A-6D 42 46 00    98表示 transactionId   值为JQ25xUZQZmBF
0x8d90     MMS-Version: 为1.0
83 68 74 74 70 3A 2F 2F 67 32 2E 7A 7A 78 39 2E 63 6E 2F 37 32 38 39 35 32 2E 6D 6D 73 3F 31 31 7A 37 46 44 53 6A 43 64 54 00       83表示Content-Location  后面的为值：http://g2.zzx9.cn/728952.mms?11z7FDSjCdT
8805810300a8c0       08=Expiry    05=Length  81=Relative-token   03=Delta-secs    00a8c0=12
```

**如果是wbxml（测试 用不了）**

```
第三部分 
0x02        开始
0x05         //'-//WAPFORUM//DTD SI 1.0//EN
6a        UTF-8 
00         标记开始
45        <si>
c6        <indication
08        <action=signal-high>
0c         href=\"http://
03          URL = (链接字符传开始)'
xxxx
00        url字符串结束
01        >
03        字符串开始，标题 UTF-8编码
xxxx
00        字符串结束,在编码函数中已经有结束符0
01        </indication>
01        </si>
```

**一个可用的彩信通知包**

```
第一部分
0605040b8423f0
第二部分
0006
226170706c69636174696f6e2f766e642e7761702e6d6d732d6d65737361676500
//push flag
b487
//长度
af84
第三部分
//xMmsMessageType
8c82
//transactionId
984a51323578555a515a6d424600
//mmsVersion
8d90
//contentLocation
83687474703a2f2f3132372e302e302e313a363535352f3233343200
//时间
8805810300a8c0
//from
890a803130363535303031303000
//msg Class
8a80
//msg size
8e020000
```

## 二、彩信下载包

彩信数据包，包含特定编码和smil文件格式

```
//一个可以发送的彩信数据包
8c84
8d90
//时间
850460dd1ebb
//from
890f803131312f545950453d504c4d4e00
//to
972b38363131312f545950453d504c4d4e00
//subject
9605ea31313100
//content type
841cb3896170706c69636174696f6e2f736d696c008a3c53746172743e00
// 开始smil相关格式
//文件长度  包含smil布局结构   和彩信素材文件
02
//smil文件布局
//header length
28
//body length
8171
//11 表示长度
11
// application/smil
6170706c69636174696f6e2f736d696c00
//contentId为   "<Strart>
c0223c53746172743e00
//contentLocation 为Start.smil
8e53746172742e736d696c003c736d696c
//smil 文件结构
3e3c6865616465723e3c6c61796f75743e3c726f6f742d6c61796f7574202f3e3c726567696f6e2069643d22496d61676522206865696768743d2231303025222077696474683d223130302522206669743d226d65657422202f3e3c726567696f6e2069643d225465787422206865696768743d2231303025222077696474683d223130302522206669743d227363726f6c6c22202f3e3c2f6c61796f75743e3c626f64793e3c706172206475723d223132303073223e3c74657874207372633d22612e7478742220726567696f6e3d2254657874222f3e3c2f7061723e3c2f626f64793e3c2f736d696c3e
//素材内容
//header length
11
//body length
03
// 03 表示长度
03
//wsp type
8381ea
//contentId为   "<1>
c0 22 3c 31 3e 00 
contentLocation   1.txt
8e 61 2e 74 78 74 00
//txt内容  为 123
313233
```

**参考文档：**&#x20;

wap-230-wsp-20010705-a.pdf&#x20;

WAP-209-MMSEncapsulation-20010601-a.pdf&#x20;

OMA-MMS-CONF-V1\_2-20030929-C.pdf
