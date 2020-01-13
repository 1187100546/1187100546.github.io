---
title: MD5加密
date: 2020-01-12 21:15:24
tags: [密码学]
categories: 密码学
mathjax: true
---

密码学里的MD5加密。

<!--more-->

## MD5算法原理

### 消息填充

-  使消息长度模512=448如果消息长度模512恰等于448，增加512个填充比特。即填充的个数为1~512，填充方法：第1比特为1，其余全部为0

- 将消息长度转换为64比特的数值，如果长度超过64比特所能表示的数据长度，值保留最后64比特添加到填充数据后面，使数据为512比特的整数倍

- 512比特按32比特分为16组

**注：64位数据长度存储的时候是小端序**

### 初始化链接变量

使用4个32位的寄存器A， B，C， D存放4个固定的32位整型参数，用于第一轮迭代，这里需要注意的是，寄存器的值要转化为小端序。
$$ {% raw %}
A=0x01234567\\
B=0x89abcdef\\
C=0xfedcba98\\
D=0x76543210
$$ {% endraw %}

### 分组处理

与分组密码分组处理相似，有4轮步骤，将512比特的消息分组平均分为16个子分组，每个子分组有32比特，参与每一轮的的16步运算，每步输入是4个32比特的链接变量和一个32位的的消息子分组，经过这样的64步之后得到4个寄存器的值分别与输入的链接变量进行模加。

### 步函数

![lTYEY4.png](https://s2.ax1x.com/2020/01/12/lTYEY4.png)

该函数包括 4 轮，每轮 16 步，上一步的链接变量 D, B, C 直接赋值给下一步的链接变量 A, C, D。

A 先和非线性函数的结果加一下，结果再和 M[j] 加一下，结果再和 T[i] 加一下，结果再循环左移 s 次，结果再和原来的 B 加一下，最后的得到新 B。

非线性函数：

![lTtd29.png](https://s2.ax1x.com/2020/01/12/lTtd29.png)

## 代码实现

### 消息填充

```python
length = struct.pack('<Q', len(message)*8)  #原消息长度64位比特的添加格式
    while len(message) > 64:
        solve(message[:64])
        message = message[64:]
    #长度不足64位消息自行填充
    message += b'\x80'
    message += b'\x00' * (56 - len(message) % 64)
    message += length
solve(message[:64])

```

### 初始化链接变量

```python
A, B, C, D = (0x67452301, 0xefcdab89, 0x98badcfe, 0x10325476)
```

### 分组处理及步函数

```python
def solve(chunk):
    global A
    global B
    global C
    global D
    w = list(struct.unpack('<' + 'I' * 16, chunk))  #分成16个组，I代表1组32位
    a, b, c, d = A, B, C, D
    for i in range(64):  #64轮运算
        if i < 16:  #每一轮运算只用到了b,c,d三个
            f = ( b & c)|((~b) & d)
            flag  = i      #用于标识处于第几组信息
        elif i < 32:
            f = (b & d)|(c & (~d))
            flag = (5 * i +1) %16
        elif i < 48:
            f = (b ^ c ^ d)
            flag  = (3 * i + 5)% 16
        else:
            f = c ^(b |(~d))
            flag  = (7 * i ) % 16
        tmp = b + lrot((a + f + k[i] + w[flag])& 0xffffffff,r[i]) #&0xffffffff为了类型转换
        a, b, c, d = d, tmp & 0xffffffff, b, c
    A = (A + a) & 0xffffffff
    B = (B + b) & 0xffffffff
    C = (C + c) & 0xffffffff
D = (D + d) & 0xffffffff

```

## 运行结果

![lTtW2d.png](https://s2.ax1x.com/2020/01/12/lTtW2d.png)

找一个在线加密的网站验证一下

![lTtfxA.png](https://s2.ax1x.com/2020/01/12/lTtfxA.png)

## 安全性分析

攻击者的主要目标不是恢复原始的明文，而是用非法消息替代合法消息进行伪造和欺骗，对哈希函数的攻击也是寻找碰撞的过程。

基本攻击方法：

（1）穷举攻击：能对任何类型的Hash函数进行攻击

最典型方法是“生日攻击”：给定初值𝐻0=H(M)，寻找𝑀’≠ 𝑀，使ℎ(𝑀’)= 𝐻0

（2）密码分析法：依赖于对Hash函数的结构和代数性质分析，采用针对Hash函数弱性质的方法进行攻击。这类攻击方法有中间相遇攻击、修正分组攻击和差分分析等

 

MD5算法中，输出的每一位都是输入的每一位的函数，逻辑函数F、G、H、I的复杂迭代使得输出对输入的依赖非常小

但Berson证明，对单轮的MD5算法，利用差分分析，可以在合理时间内找出碰撞的两条消息

MD5算法抗密码分析的能力较弱，生日攻击所需代价是试验264个消息

2004年8月17日，在美国加州圣巴巴拉召开的美密会（Crypto2004）上，中国的王小云、冯登国、来学嘉、于红波4位学者宣布，只需1小时就可找出MD5的碰撞（利用差分分析）

## 完整代码

```python
import struct
import math
import binascii

lrot = lambda x,n: (x << n)|(x >> 32- n)  #循环左移的操作

#初始向量
A, B, C, D = (0x67452301, 0xefcdab89, 0x98badcfe, 0x10325476)
# A, B, C, D = (0x01234567, 0x89ABCDEF, 0xFEDCBA98, 0x76543210)

#循环左移的位移位数
r = [   7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22,
        5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20,
         4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23,
         6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21
        ]

#使用正弦函数产生的位随机数，也就是书本上的T[i]
k =  [int(math.floor(abs(math.sin(i + 1)) * (2 ** 32))) for i in range(64)]


def init_mess(message):
    global A
    global B
    global C
    global D
    A, B, C, D = (0x67452301, 0xefcdab89, 0x98badcfe, 0x10325476)
    # A, B, C, D = (0x01234567, 0x89ABCDEF, 0xFEDCBA98, 0x76543210)
    length = struct.pack('<Q', len(message)*8)  #原消息长度64位比特的添加格式
    while len(message) > 64:
        solve(message[:64])
        message = message[64:]
    #长度不足64位消息自行填充
    message += b'\x80'
    message += b'\x00' * (56 - len(message) % 64)
    message += length
    solve(message[:64])


def solve(chunk):
    global A
    global B
    global C
    global D
    w = list(struct.unpack('<' + 'I' * 16, chunk))  #分成16个组，I代表1组32位
    a, b, c, d = A, B, C, D

    for i in range(64):  #64轮运算
        if i < 16:  #每一轮运算只用到了b,c,d三个
            f = ( b & c)|((~b) & d)
            flag  = i      #用于标识处于第几组信息
        elif i < 32:
            f = (b & d)|(c & (~d))
            flag = (5 * i +1) %16
        elif i < 48:
            f = (b ^ c ^ d)
            flag  = (3 * i + 5)% 16
        else:
            f = c ^(b |(~d))
            flag  = (7 * i ) % 16
        tmp = b + lrot((a + f + k[i] + w[flag])& 0xffffffff,r[i]) #&0xffffffff为了类型转换
        a, b, c, d = d, tmp & 0xffffffff, b, c
    A = (A + a) & 0xffffffff
    B = (B + b) & 0xffffffff
    C = (C + c) & 0xffffffff
    D = (D + d) & 0xffffffff



def digest():
    global A
    global B
    global C
    global D
    return struct.pack('<IIII',A,B,C,D)

def hex_digest():
    return binascii.hexlify(digest()).decode()


if __name__ == '__main__':
    while True:
        mess = input("请输入你的信息:")
        init_mess(mess.encode())
        out_put = hex_digest()
        print(type(out_put))
        print (out_put)

```

