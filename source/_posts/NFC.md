---
title: NFC
date: 2020-03-19 08:08:06
tags: [NFC]

mathjax: true
---

对NFC安全机制进行分析和攻防测试，了解最新NFC标准和安全机制的改进

# NFC技术原理

NFC(Near Field Communication，近场通信 ) 技术由非接触式射频识别 (RFID) 演变而来，是飞利浦 半导体 ( 现恩智浦半导体公司 )、诺基亚和索尼共同 研制开发的一种短距高频无线电技术。NFC 技术运行于13.56 MHz 射段，使用特殊的射频衰减技术，通过缩短通信距离来保证通信场景的安全与可靠， 有效距离约 10 cm。

## NFC通信过程

![8yuJMD.png](https://s1.ax1x.com/2020/03/19/8yuJMD.png)

如图所示

NFC控制器：也称 NFC芯片，负责将数字信号转换为射频信号，并通过13.56ＭＨｚ天线发送；同时负责接收射频信号，并将其转换为数字信号，与主控制器和安全元件进行通信。

主控制器：负责实现对NFC控制器的控制和操作以及与安全元件之间通过私有接口进行数据交互。

安全元件：用于存储敏感数据，例如密钥、余额等，通过 NFC控制器与外界设备进行通信，保证数据存储和交易过程的安全性。

天线：通过无线接口与NFC控制器进行通信，实现13.56ＭＨｚ射频信号的发射与接收。

NFC电子标签：是存储数据的IC卡，能够被NFC读写设备读取，可作为通信的目标。

## NFC协议栈

NFC是在RFID的基础上演变而来的，其协议规范也是基于RFID的众多标准制定和扩展的，可分为物理层 、数 据链路层、协议层和应用层。

物理层，是NFC协议参考模型的最底层，它包括物理通信媒介及其物理特性，该层主要的作用是产生并检测磁场以便发送和接收携带数据的信号，为上层提供一个通信媒介以及它们的机械、电气、功能和规程特性。例如邻近集成电路卡特性、天线、交变磁场、工作频率、输出功率等。物理层的协议主要有ISO14443 A-1/ISO 18092,ISO 14443 B-1、Felica JIS X6319-4/ISO 18092等。

其中NFC A，NFC B和NFC F是指NFC Tag的三种协议，分别遵循ISO 14443 Type A、ISO 14443 Type B 和ISO 18092（JIS X 6319-4）标准的协议，三者是主要区别在于调制方式，编码方案和协议初始化程序有所不同。

数据链路层可分为两层：媒体接入控制层和逻辑链路控制层

媒体接入控制层，主要负责控制与连接物理层的物理介质，描述了工作频率、磁场强度、不同设备之间的通信信号调制和解调等特性。

逻辑链路控制层，主要定义了磁场内轮询、帧格式、命令的请求和应答、冲突检测机制、通信链路初始化 等，与媒体接入控制层一起控制协议层与物理层之间的通信。

协议层，该层类似于OSI七层模型中的传输层，负责非接触环境下数据的半双工数据块的传输。协议层是在数据通信过程中真正传输和发送数据的层，通信载荷可以是任意格式的数据，载荷数据一般由应用层协议定义。

 应用层为NFC协议参考模型的顶层，用于为用户提供各种应用服务。该层协议主要指NFC数据交换格式（ NFC Data Exchange Format，NDEF）协议， 其定义了用户数据交换的数据格式。

## NDEF协议

在NFC协议栈的应用层中提到，NDEF是NFC数据交换的格式。

为实现NFC标签、NFC设备以及NFC设备之间的交互通信，NFC论坛(NFC Forum)定义了称为NFC数据交换格式（NDEF）的通用数据格式。

NDEF是轻量级的紧凑的二进制格式，可带有URL，vCard和NFC定义的各种数据类型。

NDEF使NFC的各种功能更加容易的使用各种支持的标签类型进行数据传输，因为NDEF已经封装了NFC标签的种类细节信息，使得应用不用关心是在与何种标签通信。 

上层应用产生由一个或多个文件生成的NDEF信息，该消息交由底层LLC层传送给对方，对方可以接受后直接处理或作为中间阶段写入Tag中。当其他设备接近该tag时，会读到该tag中的内容，并把读到的NDEF消息传给上层应用分析和处理。 

消息格式：

![8yNSqx.png](https://s1.ax1x.com/2020/03/19/8yNSqx.png)

NDEF交换的信息由一系列记录（Record）组成。每条记录包含一个有效载荷，记录内容可以是URL、MIME媒质或者NFC自定义的数据类型。使用NFC定义的数据类型，载荷内容必须被定义在一个NFC记录类型定义（RTD）文档中。 

记录中的数据类型和大小由记录载荷的头部（Header）注明。 这里的头部包含：1、类型域。用来指定载荷的类型。2、载荷的长度数。单位是字节。3、可选的指定载荷是否带有一个NDEF记录。 

 NDEF完整封包格式如下： 

![8ytLiF.png](https://s1.ax1x.com/2020/03/19/8ytLiF.png)

 如果Short Flag为1时，对应的封包格式如下： 

![8yNKdf.png](https://s1.ax1x.com/2020/03/19/8yNKdf.png)

 其中各标记说明如下： 

|      ***Flag***      |      ***说明***      |                     ***补充说明***                     |
| :------------------: | :------------------: | :----------------------------------------------------: |
|       ***MB***       |    Message Begin     |              第一个NDEF record设置，1bit               |
|       ***ME***       |     Message End      |             最后一个NDEF record设置，1bit              |
|       ***CF***       |      Chunk Flag      |  当负载被截断时，第一个和中间的负载会设置此Flag，1bit  |
|       ***SR***       |     Short Record     | 负载长度小于255，设置此Flag；否则负载长度大于255，1bit |
|       ***IL***       |   Id Length Exist    |               表示后续ID域是否存在，1bit               |
|      ***TNF***       |   Type Name Format   |                设置type中的类型，3bits                 |
|  ***TYPE_LENGTH***   |  Type field length   |                 表示Type域长度，8bits                  |
|   ***ID_LENGTH***    |   ID field length    |                  表示ID域长度，8bits                   |
| ***PAYLOAD_LENGTH*** | Payload field length |    表示Payload域长度，若SR为1为8bits，为0则为4Bytes    |
|      ***TYPE***      |      Type field      |              TYPE信息，与TYPE_LENGTH对应               |
|       ***ID***       |       ID field       |               ID信息，与TYPE_LENGTH对应                |
|    ***PAYLOAD***     |    Payload filed     |         有效负载信息，与PAYLOADTYPE_LENGTH对应         |

 关于TNF，具体值信息如下： 

|         ***TNF***         | ***Value*** |   ***说明***    |
| :-----------------------: | :---------: | :-------------: |
|        ***Empty***        |     00      |     空负载      |
| ***NFC Well-known type*** |     01      |   参考RTD信息   |
|     ***Media type***      |     02      |                 |
|    ***Obsolute URI***     |     03      |     绝对URI     |
|    ***External type***    |     04      |                 |
|       ***Unknown***       |     05      | 无法解析的格式  |
|      ***Unchanged***      |     06      | 用于Chunked消息 |
|       ***Reserve***       |     07      | 保留，暂无使用  |

## NFC工作模式

按照通信的发起者划分，工作模式可分为主动模式和被动模式 ，在主动模式下， 通信双方均产生RF场；在被动模式下，只有通信发起者产生RF场。

按照通信的对象划分，可分为点对点模式、读卡器模式和卡模拟模式。

# 近场通信的安全技术（NFC-SEC） 

NFC-SEC的基本架构

![8Lb3DS.png](https://s1.ax1x.com/2020/03/24/8Lb3DS.png)

NFC-SEC用户通过NFC-SEC服务访问点（NFC-SEC-SAP）来激活和访问NFC-SEC服务。NFC-SEC实体从NFC-SEC用户出获取NFC-SEC-SDU（请求）并且向NFC-SEC用户返回NFC-SEC-SDU（确认）。

NFC-SEC提供两种安全服务，分别是安全通道服务（SCH）和共享秘密服务（SSE）。

为了提供NFC-SEC服务，对等NFC-SEC实体根据NFC-SEC协议要求，在NFC-SEC连接上交换NFC-SEC-PDU

对等NFC-SEC实体之间通过NFCCIP-1服务访问点（NFCIP-1-SAP）来互相通信，访问NFCIP-1数据服务，发送和接收（NFC-SEC-PDU，一个NFC-SEC-PDU包含NFC-SEC协议控制信息（NFC-SEC-PCI）和一个单独的NFC-SEC-SDU。

## 共享秘密服务（SSE）

SSE在两个对等用户之间建立一个共享秘密，用户可以自行使用

通过密钥协商和密钥确认机制建立共享秘密

## 安全通道服务（SCH）

SCH提供一个安全通道

通过密钥协商和密钥确认机制建立的共享秘密来导出连接密钥，并且保护之后的双向通信安全。

## NFC-SEC服务常规流程

![8LXhJH.png](https://s1.ax1x.com/2020/03/24/8LXhJH.png)

## 密钥协定和密钥确认概述

![8Lj79J.png](https://s1.ax1x.com/2020/03/24/8Lj79J.png)

SSE和SCH服务是通过两个NFC-SEC实体之间共享秘密来调用的，共享秘密通过密钥协商和密钥确认来建立，下面进一步详细解释整个过程。

## 前提

在开始服务前，对于每个NFC-SEC实体， 根据SM2椭圆曲线公钥密码算法生成自身的公钥和私钥，以及用于密钥交换的临时密钥对。

##  密钥协商

通信双方协商出一个固定长度k的共享密钥，这个密钥不能提前预知。

过程如下：

![8Lx1RH.png](https://s1.ax1x.com/2020/03/24/8Lx1RH.png)

发送者（A）转换

a)生成随机数NA

b)利用 SM2椭圆曲线公钥密码算法生成临时密钥QA

c)发送QA||NA作为ACT_REQ PDU（激活请求PDU）的有效载荷

d)从ACT_RES PDU（激活响应PDU）的有效载荷中接收QB‘||NB’

e)根据QB‘解压出Q<sub>B</sub>’验证是否有效

f)计算出共享秘密Z

接收者（B）转换

a)从ACT_REQ PDU（激活请求PDU）有效载荷中接收QA‘||NA’

b)生成随机数NB

c)利用 SM2椭圆曲线公钥密码算法生成临时密钥QB

d)发送QB||NB作为ACT_RES PDU（激活响应PDU）的有效载荷

e)根据QA‘解压出Q<sub>A’验证是否有效

f)计算出共享秘密Z

## 详细密钥交换协议参见GM/T0003

[http://www.gmbz.org.cn/main/viewfile/20180108023456003485.html]: GM/T 0003

设用户A和B协商获取密钥数据的长度为klen比特，用户A是发起方，用户B是响应方。用户A和B双方为了获取相同的密钥，应实现如下运算步骤：

w=[[log2(n)]/2]-1。（注：此处的[]指的是顶函数）

#### 用户A：

A1：用随机数发生器产生随机数rA∈[1,n-1]；

A2：计算椭圆曲线点RA=[rA]G=（x1，y1）；

A3：将RA发送给用户B。

#### 用户B：

B1：用随机数发生器产生随机数rB∈[1,n-1]；

B2：计算椭圆曲线点RB=[rB]G=（x2，y2）；

B3：从RB中取出域元素x2，按本文第一部分的4.2.7给出的方法将x2的数据类型转换为整数，计算x2`=2^w+(x2&(2^w-1))；

B4：计算tB=（dB+x2`*rB）mod n；

B5：验证RA是否满足椭圆曲线的方程，若不满足则协商失败；否则从从RB中取出域元素x1，按本文第一部分的4.2.7给出的方法将x2的数据类型转换为整数，

计算x1`=2^w+(x1&(2^w-1))；

B6：计算椭圆曲线点V=[h*tB] (PA+[x1`]RA)=(xv,yv)，若V是无穷远点，则B协商失败；否则按本文第一部分4.2.5和4.2.4给出的方法将xv，yv的数据类型转换为比特串；

B7：计算KB=KDF（xv||yv||ZA||ZB，klen）；

B8：（选项）按本文第一部分4.2.5和4.2.4给出的方法将RA的坐标x1、y1和RB的坐标x2、y2的数据类型转换为比特串，计算SB==Hash.......；

B9：将RB、（选项SB）发送给用户A；

#### 用户A：

A4：从RA中提取出域元素x1，按本文第一本分4.2.7给出的方法将x1的数据类型转换为整数，计算x1`=2^w+(x1&(2^w-1))；

A5：tA=（dA+x2`*rA）mod n；

A6：验证RB是否满足椭圆曲线的方程，若不满足则协商失败；否则从从RB中取出域元素x2，按本文第一部分的4.2.7给出的方法将x2的数据类型转换为整数，

计算x2`=2^w+(x2&(2^w-1))；

A7：计算椭圆曲线点U=[h*tA] (PB+[x2`]RB)=(xu,yu)，若U是无穷远点，则A协商失败；否则按本文第一部分4.2.5和4.2.4给出的方法将xu，yu的数据类型转换为比特串；

A8：计算KA=KDF（xu||yu||ZA||ZB，klen）；

A9：（选项）计算S1，并检验S1=SB是否成立，若不成立则从B到A的秘钥确认失败；

A10：（选项）计算SA，并将SA发送给用户B。

#### 用户B;

B10：计算S2，并检验S2=SA是否成立，若不成立则从A到B的秘钥确认失败。

## 密钥导出

作用：利用协商好的共享秘密Z，来生成其它用途的密钥，

使用的算法为SM4算法

密钥导出函数（KDF）

KDF（K，S）=SM4<sub>K</sub>(s)

SSE的KDF：

![8OEea9.png](https://s1.ax1x.com/2020/03/24/8OEea9.png)

SCH的KDF：

![8OEDsS.png](https://s1.ax1x.com/2020/03/24/8OEDsS.png)

密钥用途：

![8OEHo9.png](https://s1.ax1x.com/2020/03/24/8OEHo9.png)

发送者（A）转换

对于SSE服务，密钥导出为：MK<sub>SSE</sub>=KDF-SSE(NA,NB',Z,ID<sub>A</sub>,ID<sub>B</sub>)

对于SCH服务，密钥导出为：{MK<sub>SCH</sub>,KE<sub>SCH</sub>,KE<sub>SCH</sub>}=KDF-SCE(NA,NB',Z,ID<sub>A</sub>,ID<sub>B</sub>)

接收者（B）转换

对于SSE服务，密钥导出为：MK<sub>SSE</sub>=KDF-SSE(NA‘,NB,Z,ID<sub>A</sub>,ID<sub>B</sub>)

对于SCH服务，密钥导出为：{MK<sub>SCH</sub>,KE<sub>SCH</sub>,KE<sub>SCH</sub>}=KDF-SCE(NA’,NB,Z,ID<sub>A</sub>,ID<sub>B</sub>)



## 密钥确认

当一个KDF生成一个密钥时，对等的NFC-SEC实体都要检查双方是否确实拥有相同的密钥

![8OZ3AH.png](https://s1.ax1x.com/2020/03/24/8OZ3AH.png)

发送者（A）转换

a)计算从A到B的密钥确认标识，MacTagA=SM4（MK，IDA||IDB||QA||QB‘）

b)发送MacTagA作为VFY_REQ PDU（验证请求PDU）的有效载荷

c)从VFY_RES PDU（验证响应PDU）的有效载荷接收MacTagB‘

d)检验从B收到的密钥确认标识

e)设置共享秘密为MKSSE

接收者（B）转换

a)从VFY_REQ PDU（验证请求PDU）的有效载荷接收MacTagA’

b检验从A收到的密钥确认标识

c)计算从B到A的密钥确认标识，MacTagB=SM4（MK，IDA||IDB||QA‘||QB）

d)发送MacTagB作为VFY_RES PDU（验证响应PDU）的有效载荷

e)设置共享秘密为MKSSE

## SCH数据交换

![8OuuPH.png](https://s1.ax1x.com/2020/03/24/8OuuPH.png)

先生成IV=SM4（MK，KI（完整性密钥）||NA||NB）

初始化序号变量SNV

![8OMc8J.png](https://s1.ax1x.com/2020/03/24/8OMc8J.png)

消息鉴别码（MAC）=SM4<sub>KI</sub>(SNV||DataLen||EncData)--------用来鉴别消息完整性

CTR模式进行分组加密

参考标准：

国标：

​	GB/T 33746.1-2017

​	GB/T 33746.2-2017

​	GB/T 17964-2008

​	GM/T 0003-2012

​	GM/T 0002-2012

国际标：

​	ISO/IEC 13157-1-2014

​	ISO/IEC 13157-2-2014

# 攻防测试

## NFCdrip攻击

NFC的通信距离在20厘米以内，但是在范围之外仍然可以进行渗透。

应用安全公司Checkmarx高级研究员 Pedro Umbelino 发现，NFC的实际有效范围比10厘米长得多，如果目标设备物理隔离但装有其他通信系统，比如WiFi、蓝牙和GSM，NFC在秘密渗漏数据方面可谓非常高效。

Pedro Umbelino 为自己的攻击方法命名NFCdrip，通过修改NFC的工作模式来调制数据。在安卓系统中，修改NFC工作模式不需要任何特殊权限，发起NFCdrip攻击更加容易。

NFCdrip采用振幅键控(ASK)调制的最简形式——开关键控(OOK)，也就是将载波信号的有无定义为“1”和“0”。由于一个字符由8比特构成，渗漏一个字符需要发出8个比特，但研究人员通常建议添加额外的校验位。 

Umbelino的实验证明，安卓手机上安装恶意软件便可向几十米外连接了普通调幅(AM)收音机的另一部安卓手机传输数据。 

在实验中，2.5米距离内，该攻击可以每秒10-12比特的速率实现无错传输。随着距离的延伸，2.5-10米距离内，虽然速率不变，但会出现一些错误，不过这些错误可以修正。距离超过10米，信号会衰减，错误会增多，但Umbelino仍成功在60米距离上传输了一些数据。甚至连墙都挡不住这种攻击，10米距离内穿墙传输无碍。 

## Android Beam

谷歌在Android 4.0系统引入Android Beam功能，能够最大程度地利用NFC进行分享。  NFC Beam允许Android设备使用无线电波将图像、文件、视频或者应用程序之类的数据发送到附近的另一台设备，以替代WiFi或蓝牙。 

漏洞名为CVE-2019-2114，黑客可以利用此漏洞将恶意软件传播到附近的手机。 

一般情况下，通过NFC传输的应用程序（APK文件）会存储在磁盘上，并在屏幕上显示通知。此外该通知会询问手机持有人是否同意NFC服务从未知来源安装应用程序。 

但是研究人员发现在Android 8（Oreo）及更高版本的手机上并没有此项通知服务。也就是说，在这种情况下，NFC会默认允许设备一键安装应用程序，而不会发出任何安全警告。 

由于没有提示从未知来源进行安装，恶意软件会直接开始安装，很多用户会误以为是来自官方商店的消息，然而在安装之后则会面临新的危险。 

## NFC Tag

工具：一张空白的NFC Tag,一个支持NFC的手机，TagWriter软件

利用TagWriter软件可以向空白的NFC Tag中写入恶意网站的链接，当NFC Tag靠近手机是，便会强制手机打开写入的恶意网址，从而对手机进行监听。

[![8vt3tA.jpg](https://s1.ax1x.com/2020/03/25/8vt3tA.jpg)](https://imgchr.com/i/8vt3tA)

