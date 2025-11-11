# 短信中心DCS编码

Bit 1 Bit 0 Message Class

0   0     Class 0      普通短信\
0    1     Class 1        Default meaning: ME-specific. 手机存储\
1     0    Class 2       SIM specific message SIM卡存\
1     1     Class 3       Default meaning: TE specific (see GSM TS 07.05) ：该消息是给终端设备的，例如调制  解调器、串口设备、电脑终端等



Bits 3 and 2 indicate the alphabet being used, as follows :\
Bit 3 Bit2 Alphabet:\
0    0       Default alphabet 7位编码\
0    1        8bit 8位编码\
1     0       UCS2 (16bit) \[10] 中文编码\
1      1        Reserved 无

0100..1011 Reserved coding groups\
0    0   0   0     不压缩 无class含义（不需要处理bit1 bit0）\
0    0   0    1 不压缩 有class含义\
0    0    1     0    压缩（不存在压缩，没有这种情况）\
0 011 压缩（不存在压缩，没有这种情况）

&#x20;        Bit 5, if set to 0, indicates the text is uncompressed\
&#x20;        Bit 5, if set to 1, indicates the text is compressed using the GSM standard\
&#x20;        Bit 4, if set to 0, indicates that bits 1 to 0 are reserved and have no message class meaning 代表没有  class 含义 也就不需要设置bit 1 bit0\
&#x20;        Bit 4, if set to 1, indicates that bits 1 to 0 have a message class meaning : 代表有class 含义

&#x20;        Bit 1 Bit 0 Message Class\
&#x20;        0 0 Class 0\
&#x20;        0 1 Class 1 Default meaning: ME-specific.\
&#x20;        1 0 Class 2 SIM specific message\
&#x20;        1 1 Class 3 Default meaning: TE specific (see GSM TS 07.05)

&#x20;         Bits 3 and 2 indicate the alphabet being used, as follows :\
&#x20;         Bit 3 Bit2 Alphabet:\
&#x20;         0 0 Default alphabet 7 bit编码\
&#x20;          0 1 8 bit 8 bit编码\
&#x20;          1 0 UCS2 (16bit) \[10] ucs2中文\
&#x20;           1 1 Reserved 保留

0100 保留编码组

0101 保留编码组

0110 保留编码组

0111 保留编码组

1000 保留编码组

1001 保留编码组

1010 保留编码组

1011 保留编码组

1100 仅提示消息，不存储消息 手机用户收到消息进作为提示，看完不会自动保存到收件箱里 7bit

1101 指示手机有t特殊消息存储在某处 7bit

&#x20;         Bits 3 indicates Indication Sense:\
&#x20;         Bit 3\
&#x20;         0 Set Indication Inactive 设置指示无效\
&#x20;          1 Set Indication Active 设置指示有效\
&#x20;          Bit 2 is reserved, and set to 0 无类型，设置为0\
&#x20;          Bit 1 Bit 0 Indication Type:\
&#x20;           0 0 Voicemail Message Waiting 语音消息等待\
&#x20;           0 1 Fax Message Waiting 传真留言等待\
&#x20;           1 0 Electronic Mail Message Waiting 邮件消息等待\
&#x20;          11 Other Message Waiting\* 求他消息等待\
1110 指示手机有t特殊消息存储在某处 同1101 文本为中文ucs2编码\
1111\
&#x20;         Bit 3 is reserved, set to 0.\
&#x20;        Bit 2 Message coding:\
&#x20;        0 Default alphabet 7 bit 编码\
&#x20;        1 8-bit data 8 bit编码\
&#x20;         Bit 1 Bit 0 Message Class:\
&#x20;         0 0 Class 0\
&#x20;         0 1 Class 1 default meaning: ME-specific.\
&#x20;        1 0 Class 2 SIM-specific message.\
&#x20;         1 1 Class 3 default meaning: TE specific (see GSM TS 07.05)
