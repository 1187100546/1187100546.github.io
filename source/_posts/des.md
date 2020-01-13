---
title: 分组密码DES
date: 2020-01-03 17:02:50
tags: [密码学]
categories: 密码学
mathjax: true
---

密码学中的DES加密。

<!--more-->

## DES算法原理

### 基本结构

DES是分组加密，将明文分成64位一组，密钥长度 64 比特（其中有效长度为 56 比特），8 的倍数位为奇校验位（保证每 8 位有奇数个 1）。

如图，64 比特的密钥经过置换选择和循环移位操作可生成 16 个 48 比特的子密钥。明文 m 经过初始置换 IP 后划分为左右两部分（各32 比特），经过 16 轮 Feistel 结果（其中最后一轮不做左右交换）再做一次逆置换 IP-1得到密文 c 。

![loHUSg.png](https://s2.ax1x.com/2020/01/12/loHUSg.png)

加密方程：
$$
L_0R_0 \leftarrow IP\\
L_n\leftarrow R_{n-1}\\
R_n\leftarrow L_{n-1}\oplus f(R_{n-1},K_n)\\
c=\leftarrow IP^{-1}
$$
解密方程：
$$
R_{16}L_{16}\leftarrow IP\\
R_{n-1}\leftarrow L_n\\
L_{n-1}\leftarrow R_n\oplus f(L_nK_n)\\
m=IP^{-1}
$$
由此可见DES是一个对合运算。

### 密钥扩展

用于生成迭代的子密钥。具体过程为：

64位初始密钥经过置换选择1 ( PC-1 ) 后变成 56 位，经过循环左移和置换选择2 ( PC-2 ) 后分别得到 16 个 48 位子密钥 Ki 用做每一轮的迭代运算。

PC-1 去掉了校验位， PC-2 去掉了9, 18, 22, 25, 35, 38, 43, 54 位。

![loLkes.png](https://s2.ax1x.com/2020/01/12/loLkes.png)

### F函数

-  先进行E-扩展：32比特扩展为48比特

- 将E盒扩展得到的48位输出与子密钥𝐾𝑖进行异或运算

- l压缩替换S-盒，由8个S-盒构成, 每个S-盒都是6比特的输入，4比特的输出

-  P-置换对8个S-盒的输出进行变换

![loLmWT.png](https://s2.ax1x.com/2020/01/12/loLmWT.png)

## 代码实现

### 密钥扩展

```python
#PC-1盒
def key_change_1(st):
    key1_list = [57,49,41,33,25,17,9,1,58,50,42,34,26,18,10,2,59,51,43,35,27,19,11,3,60,52,44,36,63,55,47,39,31,23,15,7,62,54,46,38,30,22,14,6,61,53,45,37,29,21,13,5,28,20,12,4]
    #得到56位，去掉校验位
    res = ""
    for i in key1_list:
        res+=st[i-1]
    return res
#PC-2盒
def key_change_2(st):
    key2_list = [14,17,11,24,1,5,3,28,15,6,21,10,23,19,12,4,26,8,16,7,27,20,13,2,41,52,31,37,47,55,30,40,51,45,33,48,44,49,39,56,34,53,46,42,50,36,29,32]
    #压缩置换
    #去掉9，18，22，25，35，38，43，54位得到48位
    res = ""
    for i in key2_list:
        res+=st[i-1]
    return res
#16轮迭代生成子密钥
def key_gen(st):
    key_list = []#子密钥表
    key_change_res = key_change_1(st)
    #PC-1置换分成C，D两块
    key_c = key_change_res[0:28]
    key_d = key_change_res[28:]
    for i in range(1,17): #共16轮
        if (i==1) or (i==2) or (i==9) or (i==16):#按轮数循环左移
            key_c = zuoyiwei(key_c,1)
            key_d = zuoyiwei(key_d,1)
        else:
            key_c = zuoyiwei(key_c,2)
            key_d = zuoyiwei(key_d,2)
        key_yiwei = key_c+key_d
        #压缩置换
        key_res = key_change_2(key_yiwei)
        key_list.append(key_res)
return key_list

```

### F函数

```python
#E盒扩展置换，将32位输出扩展至48位
def box_e(st):
    #E盒置换表
    e_list = [32,1,2,3,4,5,4,5,6,7,8,9,8,9,10,11,12,13,12,13,14,15,16,17,16,17,18,19,20,21,20,21,22,23,24,25,24,25,26,27,28,29,28,29,30,31,32,1]
    res = ""
    for i in e_list:
        res +=st[i-1]
    return res
#S盒代换盒
def box_s(st):
    j = 0
    #8个S盒代换表
    s_list = [[14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7,0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8,4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0,15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13],
              [15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10,3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5,0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15,13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9],
              [10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8,13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1,13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7,1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12],
              [7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15,13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9,10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4,3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14],
              [2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9,14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6,4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14,11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3],
              [12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11,10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8,9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6,4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13],
              [4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1,13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6,1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2,6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12],
              [13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7,1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2,7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8,2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11]]
    res = ""
    for i in range(0,len(st),6):#6位输入，4位输出
        begin_s = st[i:i+6]
        row = int(begin_s[0]+begin_s[5],2)
        col = int(begin_s[1:5],2)
        num = bin(s_list[j][row*16+col])[2:]
        for padd in range(0,4-len(num)):
            num = "0"+num
        res += num
        j = j+1
    return res
#P盒置换
def box_p(st):
    res = ""
    p_list = [16,7,20,21,29,12,28,17,1,15,23,26,5,18,31,10,2,8,24,14,32,27,3,9,19,13,30,6,22,11,4,25]
    for i in p_list:
        res +=st[i-1]
    return res
#封装成F函数
def funcF(st,key):
    str_e_res = box_e(st)
    xor_res = xor(str_e_res,key)
    str_s_res = box_s(xor_res)
    str_p_res = box_p(str_s_res)
    return str_p_res

```

### IP置换与IP逆置换

```python
#初始IP置换
def begin_change(st):
    #置换表
    change_list = [58,50,42,34,26,18,10,2,60,52,44,36,28,20,12,4,62,54,46,38,30,22,14,6,64,56,48,40,32,24,16,8,57,49,41,33,25,17,9,1,59,51,43,35,27,19,11,3,61,53,45,37,29,21,13,5,63,55,47,39,31,23,15,7]
    res =""
    for i in change_list:
        res+=st[i-1] #因为列表是1-64，而数组是0-63，所以减一
return res
#IP逆置换
def mov_IP(st):
    res = ""
    ip_list = [40,8,48,16,56,24,64,32,39,7,47,15,55,23,63,31,38,6,46,14,54,22,62,30,37,5,45,13,53,21,61,29,36,4,44,12,52,20,60,28,35,3,43,11,51,19,59,27,34,2,42,10,50,18,58,26,33,1,41,9,49,17,57,25]
    for i in ip_list:
        res += st[i-1]
return res

```

### 加解密封装

```python
加密：
#封装64位加密
def DESenc_test(mes,key):
    mes_bin = strtobin(mes)#明文转二进制
    mes_IP = begin_change(mes_bin)#IP置换
    key_bin = strtobin(key)#密钥转二进制
    key_list = key_gen(key_bin)#生成子密钥
    mes_left = mes_IP[0:32]#明文分两组32位
    mes_right = mes_IP[32:]
    for i in range(0,15):#16轮F函数迭代
        mes_tmp = mes_right
        right_f_res = funcF(mes_right,key_list[i])
        mes_right = xor(right_f_res,mes_left)
        mes_left = mes_tmp
    fin_right = mes_right
    fin_left = xor(funcF(mes_right,key_list[15]),mes_left)
    fin = fin_left+fin_right
    fin = mov_IP(fin)#IP逆置换
return fin#返回密文
解密：
#封装64位解密
def DESdec_test(cipher,key):  #密文直接输64位2进制
    #cipher = strtobin(str)
    key_bin = strtobin(key)
    key_list = key_gen(key_bin)
    cipher = begin_change(cipher)
    i = 15
    cipher_left = cipher[0:32]
    cipher_right = cipher[32:]
    while i>0:
        cipher_tmp = cipher_right
        cipher_right = xor(cipher_left,funcF(cipher_right,key_list[i]))
        cipher_left = cipher_tmp
        i = i -1
    fin_left = xor(cipher_left,funcF(cipher_right,key_list[0]))
    fin_right = cipher_right
    fin = fin_left+fin_right
    fin = mov_IP(fin)
    my_plain = ""
    for j in range(0,len(fin),8):
        my_plain += chr(int(fin[j:j+8],2))
return my_plain

```

## 运行结果

![loL50s.png](https://s2.ax1x.com/2020/01/12/loL50s.png)

## CBC工作模式DES加密

### 设计思路  

对于DES分组加密，每次加密完的结果与下一组的明文进行异或，这就是CBC（密码分组链接模式）。

![loLxB9.png](https://s2.ax1x.com/2020/01/12/loLxB9.png)

### 代码实现

这里采用的初始向量是“aaaaaaaa”.

```python
#cbc模式下加密
def cbc_desenc(mes,key):
    IV="aaaaaaaa"
    res=""
    i=0
    cns=""
    while mes[i:i+8]!="":
        if i==0:
            res+=DESenc_test(bintostr(xor(strtobin(IV),strtobin(mes[i:i+8]))),key)
            cns=DESenc_test(bintostr(xor(strtobin(IV),strtobin(mes[i:i+8]))),key)
        else:
            res+=DESenc_test(bintostr(xor(cns,strtobin(mes[i:i+8]))),key)
            cns=DESenc_test(bintostr(xor(cns,strtobin(mes[i:i+8]))),key)
        i=i+8
return res
#cbc模式下解密
def cbc_desdec(cipher,key):
    res=""
    IV="aaaaaaaa"
    i=0
    cns=""
    while cipher[i:i+64]!="":
        if i==0:
            res+=bintostr(xor(strtobin(DESdec_test(cipher[i:i+64],key)),strtobin(IV)))
            cns=cipher[i:i+64]
        else:
            res+=bintostr(xor(strtobin(DESdec_test(cipher[i:i+64],key)),cns))
            cns=cipher[i:i+64]
        i=i+64
return res

```

### 运行结果

![loOAje.png](https://s2.ax1x.com/2020/01/12/loOAje.png)

## 安全性分析

安全性争论：

-  S盒的设计准则还没有完全公开，人们仍然不知道S盒的构造中是否使用了进一步的设计准则

-  DES存在一些弱密钥和半弱密钥

-  DES的56位密钥无法抵抗穷举攻击

-  代数结构存在互补对称性

弱密钥：

给定初始密钥𝐾生成子密钥时，将种子密钥分成两个部分，如果𝐾使得这两部分的每一部分的所有位置全为0或1，则经子密钥产生器产生的各个子密钥都相同，即𝐾1=𝐾2=…=𝐾16，则称密钥𝐾为弱密钥（共有4个）

若𝐾为弱密钥，则对任意的64比特信息有：
$$
E_k(E_k(m))=m和D_k(D_k(m))=m
$$
半弱密钥：

把明文加密成相同的密文，即存在两个不同的密钥𝑘和𝑘′,使得𝐸_𝑘 (𝑚)=𝐸_(𝑘^′ ) (𝑚)

具有下述性质：

若𝑘和𝑘′为一对弱密钥，𝑚为明文组，则有：
$$
E_{k'}(E_k(m))=E_k(E_{k'}(m))=m
$$
互补性：

对明文𝑚逐位取补，记为𝑚 ̅，密钥𝐾逐位取补，记为𝑘 ̅ ， 若𝑐=𝐸𝑘(𝑚)，则有𝑐 ̅=𝐸_𝑘 ̅ (𝑚 ̅) ，称为算法上的互补性

由算法中两次异或运算的配置决定：两次异或运算一次在S盒之前，一次在P盒置换之后

若对DES 的明文和密钥同时取补，则扩展运算E的输出和子密钥产生器的输出也都取补，因而经异或运算后的输出和未取补时的输出一样，即到达S盒的输入数据未变，输出自然也不变，但经第二个异或运算时，由于左边数据已取补，因而输出也就取补

互补性使DES在选择明文攻击下所需的工作量减半（2^55）

对选择的明文𝑚和𝑚 ̅ 加密后得到密文如下：
$$
c_1=E_k(m)\\
c_2=E_k(m^-)
$$
由对称互补性可得
$$
c_2^-=E_{k^-}(m)
$$
所以对𝑚加密，如果密文为𝑐_1，则加密密钥为𝑘, 如果密文为(𝑐_2 ) ̅，则加密密钥为𝑘 ̅

差分分析法：

通过分析特定明文差对结果密文差的影响来获得可能性最大的密钥。这种攻击方法主要适用于攻击迭代分组密码，最初是针对DES提出的一种攻击方法，虽然差分攻击方法对破译16轮的DES不能提供一种实用的方法，但对破译轮数较低的DES是很成功的。

线性分析法：

寻找一个给定密码算法的有关明文比特、密文比特和密钥比特的有效线性近似表达式，通过选择充分多的明－密文对来分析密钥的某些比特，用这种方法破译DES比差分分析方法更有效。可用247个已知明文破译8－轮DES。

三重DES：

![loXtMD.png](https://s2.ax1x.com/2020/01/12/loXtMD.png)

两密钥的3DES称为加密-解密-加密方案，简记为EDE(encrypt-decrypt-encrypt)

破译它的穷举密钥搜索量为2<sup>112 </sup>量级，用差分分析破译也要超过10<sup>35</sup>sup>量级。此方案仍有足够的安全性。

## 完整代码

```python
#二进制转字符串
def bintostr(st):
    res=""
    l=int(len(st)/8)
    for i in range(l):
       res+=chr(int(st[i*8:i*8+8],2))
    return res
#字符串转二进制
def strtobin(st):
    res = ""
    for i in st:
        tmp = bin(ord(i))[2:]
        for i in range(0,8-len(tmp)):
            tmp = '0'+tmp
        res+=tmp
    return res
#循环左移
def zuoyiwei(st,num):
    my = st[num:len(st)]
    my = my+st[0:num]
    return my
#异或
def xor(str1,str2):
    res = ""
    for i in range(0,len(str1)):
        xor_res = int(str1[i],10)^int(str2[i],10)
        if xor_res == 1:
            res +='1'
        else:
            res +='0'
    return res
#PC-1盒
def key_change_1(st):
    key1_list = [57,49,41,33,25,17,9,1,58,50,42,34,26,18,10,2,59,51,43,35,27,19,11,3,60,52,44,36,63,55,47,39,31,23,15,7,62,54,46,38,30,22,14,6,61,53,45,37,29,21,13,5,28,20,12,4]
    #得到56位，去掉校验位
    res = ""
    for i in key1_list:
        res+=st[i-1]
    return res
#PC-2盒
def key_change_2(st):
    key2_list = [14,17,11,24,1,5,3,28,15,6,21,10,23,19,12,4,26,8,16,7,27,20,13,2,41,52,31,37,47,55,30,40,51,45,33,48,44,49,39,56,34,53,46,42,50,36,29,32]
    #压缩置换
    #去掉9，18，22，25，35，38，43，54位得到48位
    res = ""
    for i in key2_list:
        res+=st[i-1]
    return res
#16轮迭代生成子密钥
def key_gen(st):
    key_list = []#子密钥表
    key_change_res = key_change_1(st)
    #PC-1置换分成C，D两块
    key_c = key_change_res[0:28]
    key_d = key_change_res[28:]
    for i in range(1,17): #共16轮
        if (i==1) or (i==2) or (i==9) or (i==16):#按轮数循环左移
            key_c = zuoyiwei(key_c,1)
            key_d = zuoyiwei(key_d,1)
        else:
            key_c = zuoyiwei(key_c,2)
            key_d = zuoyiwei(key_d,2)
        key_yiwei = key_c+key_d
        #压缩置换
        key_res = key_change_2(key_yiwei)
        key_list.append(key_res)
    return key_list
#初始IP置换
def begin_change(st):
    #置换表
    change_list = [58,50,42,34,26,18,10,2,60,52,44,36,28,20,12,4,62,54,46,38,30,22,14,6,64,56,48,40,32,24,16,8,57,49,41,33,25,17,9,1,59,51,43,35,27,19,11,3,61,53,45,37,29,21,13,5,63,55,47,39,31,23,15,7]
    res =""
    for i in change_list:
        res+=st[i-1] #因为列表是1-64，而数组是0-63，所以减一
    return res
#E盒扩展置换，将32位输出扩展至48位
def box_e(st):
    #E盒置换表
    e_list = [32,1,2,3,4,5,4,5,6,7,8,9,8,9,10,11,12,13,12,13,14,15,16,17,16,17,18,19,20,21,20,21,22,23,24,25,24,25,26,27,28,29,28,29,30,31,32,1]
    res = ""
    for i in e_list:
        res +=st[i-1]
    return res
#S盒代换盒
def box_s(st):
    j = 0
    #8个S盒代换表
    s_list = [[14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7,0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8,4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0,15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13],
              [15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10,3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5,0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15,13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9],
              [10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8,13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1,13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7,1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12],
              [7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15,13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9,10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4,3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14],
              [2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9,14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6,4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14,11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3],
              [12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11,10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8,9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6,4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13],
              [4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1,13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6,1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2,6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12],
              [13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7,1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2,7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8,2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11]]
    res = ""
    for i in range(0,len(st),6):#6位输入，4位输出
        begin_s = st[i:i+6]
        row = int(begin_s[0]+begin_s[5],2)
        col = int(begin_s[1:5],2)
        num = bin(s_list[j][row*16+col])[2:]
        for padd in range(0,4-len(num)):
            num = "0"+num
        res += num
        j = j+1
    return res
#P盒置换
def box_p(st):
    res = ""
    p_list = [16,7,20,21,29,12,28,17,1,15,23,26,5,18,31,10,2,8,24,14,32,27,3,9,19,13,30,6,22,11,4,25]
    for i in p_list:
        res +=st[i-1]
    return res
#封装成F函数
def funcF(st,key):
    str_e_res = box_e(st)
    xor_res = xor(str_e_res,key)
    str_s_res = box_s(xor_res)
    str_p_res = box_p(str_s_res)
    return str_p_res
#IP逆置换
def mov_IP(st):
    res = ""
    ip_list = [40,8,48,16,56,24,64,32,39,7,47,15,55,23,63,31,38,6,46,14,54,22,62,30,37,5,45,13,53,21,61,29,36,4,44,12,52,20,60,28,35,3,43,11,51,19,59,27,34,2,42,10,50,18,58,26,33,1,41,9,49,17,57,25]
    for i in ip_list:
        res += st[i-1]
    return res
#封装64位加密
def DESenc_test(mes,key):
    mes_bin = strtobin(mes)#明文转二进制
    mes_IP = begin_change(mes_bin)#IP置换
    key_bin = strtobin(key)#密钥转二进制
    key_list = key_gen(key_bin)#生成子密钥
    mes_left = mes_IP[0:32]#明文分两组32位
    mes_right = mes_IP[32:]
    for i in range(0,15):#16轮F函数迭代
        mes_tmp = mes_right
        right_f_res = funcF(mes_right,key_list[i])
        mes_right = xor(right_f_res,mes_left)
        mes_left = mes_tmp
    fin_right = mes_right
    fin_left = xor(funcF(mes_right,key_list[15]),mes_left)
    fin = fin_left+fin_right
    fin = mov_IP(fin)#IP逆置换
    return fin#返回密文
#Alice_bob
def DESenc_ab(mes,key):
    mes_bin = strtobin(mes)#明文转二进制
    mes_IP = begin_change(mes_bin)#IP置换
    key_bin = key#密钥转二进制
    key_list = key_gen(key_bin)#生成子密钥
    mes_left = mes_IP[0:32]#明文分两组32位
    mes_right = mes_IP[32:]
    for i in range(0,15):#16轮F函数迭代
        mes_tmp = mes_right
        right_f_res = funcF(mes_right,key_list[i])
        mes_right = xor(right_f_res,mes_left)
        mes_left = mes_tmp
    fin_right = mes_right
    fin_left = xor(funcF(mes_right,key_list[15]),mes_left)
    fin = fin_left+fin_right
    fin = mov_IP(fin)#IP逆置换
    return fin#返回密文
#封装64位解密
def DESdec_test(cipher,key):  #密文直接输64位2进制
    #cipher = strtobin(str)
    key_bin = strtobin(key)
    key_list = key_gen(key_bin)
    cipher = begin_change(cipher)
    i = 15
    cipher_left = cipher[0:32]
    cipher_right = cipher[32:]
    while i>0:
        cipher_tmp = cipher_right
        cipher_right = xor(cipher_left,funcF(cipher_right,key_list[i]))
        cipher_left = cipher_tmp
        i = i -1
    fin_left = xor(cipher_left,funcF(cipher_right,key_list[0]))
    fin_right = cipher_right
    fin = fin_left+fin_right
    fin = mov_IP(fin)
    my_plain = ""
    for j in range(0,len(fin),8):
        my_plain += chr(int(fin[j:j+8],2))
    return my_plain
#alice_bob
def DESdec_ab(cipher,key):  #密文直接输64位2进制
    #cipher = strtobin(str)
    key_bin = key
    key_list = key_gen(key_bin)
    cipher = begin_change(cipher)
    i = 15
    cipher_left = cipher[0:32]
    cipher_right = cipher[32:]
    while i>0:
        cipher_tmp = cipher_right
        cipher_right = xor(cipher_left,funcF(cipher_right,key_list[i]))
        cipher_left = cipher_tmp
        i = i -1
    fin_left = xor(cipher_left,funcF(cipher_right,key_list[0]))
    fin_right = cipher_right
    fin = fin_left+fin_right
    fin = mov_IP(fin)
    my_plain = ""
    for j in range(0,len(fin),8):
        my_plain += chr(int(fin[j:j+8],2))
    return my_plain
#DES加密
def DESenc(mes,key):
    res = ""
    i = 0
    while mes[i:i+8] != "":
        res += DESenc_test(mes[i:i+8],key)
        i = i+8
    return res
#alice_bob
def DESenc_a(mes,key):
    res = ""
    res += DESenc_ab(mes,key)
    return res
#DES解密
def DESdec(cipher,key):
    res = ""
    i = 0
    while cipher[i:i + 64] != "":
        res += DESdec_test(cipher[i:i + 64], key)
        i = i + 64
    return res
#cbc模式下加密
def cbc_desenc(mes,key):
    IV="aaaaaaaa"
    res=""
    i=0
    cns=""
    while mes[i:i+8]!="":
        if i==0:
            res+=DESenc_test(bintostr(xor(strtobin(IV),strtobin(mes[i:i+8]))),key)
            cns=DESenc_test(bintostr(xor(strtobin(IV),strtobin(mes[i:i+8]))),key)
        else:
            res+=DESenc_test(bintostr(xor(cns,strtobin(mes[i:i+8]))),key)
            cns=DESenc_test(bintostr(xor(cns,strtobin(mes[i:i+8]))),key)
        i=i+8
    return res
#cbc模式下解密
def cbc_desdec(cipher,key):
    res=""
    IV="aaaaaaaa"
    i=0
    cns=""
    while cipher[i:i+64]!="":
        if i==0:
            res+=bintostr(xor(strtobin(DESdec_test(cipher[i:i+64],key)),strtobin(IV)))
            cns=cipher[i:i+64]
        else:
            res+=bintostr(xor(strtobin(DESdec_test(cipher[i:i+64],key)),cns))
            cns=cipher[i:i+64]
        i=i+64
    return res
#main
def main():
    m = input("Please enter your plaintext:\n\t")
    lm = len(m)
    # 若不是8的倍数，则用0填充
    lm_mod = lm % 8
    if lm_mod != 0:
        FillLength = 8 - lm_mod
        m += FillLength * "0"#padding
        lm += FillLength
    k = input("Please enter your key:\n\t")
    c_t=bintostr(DESenc(m,k))
    print("DES:\n\t",c_t)
    print("CBC_DES:\n\t",bintostr(cbc_desenc(m,k)))
    '''c = input("Please enter your ciphertext:\n\t")
    k_1 = input("Please enter your key:\n\t")
    print(DESdec(strtobin(c),k_1))'''
    print("DES解密\n\t",DESdec(DESenc(m,k),k))
    c2= input("Please enter your cbc_ciphertext:\n\t")
    k_2=input("Please enter your key:\n\t")
    print(cbc_desdec(cbc_desenc(m,k),k_2))
if __name__ == '__main__':
main()
实验四
RSA加密
import random
def FastMod(x, n, m):
    a = 1
    b = x
    while True:
        temp = n
        if n % 2 == 1 :
            a = a * b % m
        b = b * b % m
        n = n//2
        if temp < 1 :
            return a
def computeD(e, phi_n):
    (x, y, r) = extendedGCD(phi_n, e)
    if y < 0:
        return phi_n + y
    return y
def extendedGCD(a, b):
    if b == 0:
        return (1, 0, a)
    x1 = 1
    y1 = 0
    x2 = 0
    y2 = 1
    while b != 0:
        q = a // b
        r = a % b
        a = b
        b = r
        x = x1 - q*x2
        x1 = x2
        x2 = x
        y = y1 - q*y2
        y1 = y2
        y2 = y
    return(x1, y1, a)
def str2Hex(m):
    return "".join("{:02x}".format(ord(x)) for x in m)
# 素性检验：采用 Miler-Rabin 检验法
def miller_rabin(n,k=80):
    if n == 2 or n == 3:
        return True
    if n % 2 == 0:
        return False
    r, s = 0, n - 1
    while s % 2 == 0:
        r += 1
        s //= 2
    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, s, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True
# 生成 b 位的素数
def genPrime(b=1024):
    while True:
        # the highest bit is 1
        ans = "1"
        for i in range(b-2):
            ans += str(random.randint(0,1))
        # the lowest bit is 1
        ans += "1"
        ans = int(ans,2)
        if miller_rabin(ans):
            return ans
def genE(phi_n):
    while True:
        e = genPrime(b=random.randint(3,13))
        if e == 3 or e == 5:
            continue
        if phi_n%e != 0:
            return e
def RSAEncrypt(m,n,e):
    m = int(str2Hex(m),16)
    c = pow(m,e,n)
    return c
def RSADecrypt(c,d,n):
    m = pow(c,d,n)
    m = bytes.fromhex('{:x}'.format(m))
    return m
def main():
    # 生成两个大素数p和q
    print ("Generate p and q ......")
    p = genPrime()
    q = genPrime()
    print ("p = "+str(p))
    print ("q = "+str(q))
    # 计算n = p*q
    n = p*q
    print ("n = "+str(n))
    # 计算φ(n) = p*q
    phi_n = (p-1)*(q-1)
    print ("\nGenerate e ......")
    # 生成一个和φ(n)互素的数e
    e = genE(phi_n)
    print ("e = "+str(e))
    m = "Hello world!"
    # 加密算法
    print ("\n"+8*"*"+" Encryption "+8*"*")
    Ciphertext = RSAEncrypt(m,n,e)
    print ("The Ciphertext is:\n\t"+str(Ciphertext))
    # 解密算法
    print ("\n"+8*"*"+" Decryption "+8*"*")
    # 使用私钥d，d是e模φ(n)的逆
    d = computeD(e,phi_n)
    print ("d = "+str(d))
    Plaintext = RSADecrypt(Ciphertext,d,n)
    print ("The Plaintext is:\n\t"+str(Plaintext))
if __name__ == '__main__':
main()

```

