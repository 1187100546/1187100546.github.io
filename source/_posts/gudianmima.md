---
title: 古典密码
date: 2020-01-03 16:35:15
tags: [密码学]
categories: 密码学
---

密码学中的古典密码。

<!--more-->

## 维吉尼亚密码原理

维吉尼亚密码是一种简单的多表代换密码，可以看成由一些偏移量不同的恺撒密码组成。为了掩盖字母使用中暴露的频率特征，解决的办法就是用多套符号代替原来的文字。它是一个表格，第一行代表原文的字母，下面每一横行代表原文分别由哪些字母代替，每一竖列代表我们要用第几套字符来替换原文。一共26个字母，一共26套代替法，所以这个表是一个26*26的表  .

![laA81K.png](https://s2.ax1x.com/2020/01/03/laA81K.png)

## 算法实现

### 加密算法

$$
c_i=(m_i+k_i)mod26
$$

若密钥长度小于明文长度，则密钥循环使用

```python
def VigenereEncrypt(m,k):
    k = k.upper()
    lm = len(m)
    lk = len(k)
    j = 0
    ans = ""
    for i in range(lm):
        if m[i].isupper():
            ans += chr(((ord(m[i])-ord('A'))%26+(ord(k[j%lk])-ord('A'))%26)%26+ord('A'))
            j += 1
        elif m[i].islower():
            ans += chr(((ord(m[i])-ord('a'))%26+(ord(k[j%lk])-ord('A'))%26)%26+ord('a'))
            j += 1
        else:
            ans += m[i]
return ans

```

### 解密算法

$$
m_i=(c_i+26-k_i)mod\ 26 
$$

```python
def VigenereDecrypt(c,k):
    k = k.upper()
    lc = len(c)
    lk = len(k)
    j = 0
    ans = ""
    for i in range(lc):
        if c[i].isupper():
            ans += chr(((ord(c[i])-ord('A'))%26+26-(ord(k[j%lk])-ord('A'))%26)%26+ord('A'))
            j += 1
        elif c[i].islower():
            ans += chr(((ord(c[i])-ord('a'))%26+26-(ord(k[j%lk])-ord('A'))%26)%26+ord('a'))
            j += 1
        else:
            ans += c[i]
return ans

```

## 攻击方法

破译维吉尼亚密码的关键在于它的密钥是循环重复的。如果我们知道了密钥的长度，那密文就可以被看作是交织在一起的凯撒密码，而其中每一个都可以单独破解。

多表代换密码体制的分析方法主要分为三步：第一步确定秘钥长度，常用的方法有卡西斯基（Kasiski）测试法和重合指数法（Index of Coincidence）；第二步就是确定秘钥，常用的方法是拟重合指数测试法；第三步是根据第二步确定的密钥恢复出明文。

### Kasiski测试法

卡西斯基试验是基于类似 the 这样的常用单词有可能被同样的密钥字母进行加密，从而在密文中重复出现。如果将密文中所有相同的字母组都找出来，并计算他们的最大公因数，就有可能提取出来密钥长度信息。

测试过程：搜索长度至少为2的相邻的一对对相同的密文段，记下它们之间的距离。而密钥长度d可能就是这些距离的最大公因子

### 重合指数法

利用随机文本和英文文本的统计概率差别来分析密钥长度。依据：英文中每种单词出现的频率不同。

重合指数公式：

<img src="https://s2.ax1x.com/2020/01/03/laet2t.png" alt="laet2t.png" style="zoom:50%;" />

人们已经获得了英文的26个字母的概率分布的一个估计。期望值为：

<img src="https://s2.ax1x.com/2020/01/03/laeNxP.png" alt="laeNxP.png" style="zoom: 67%;" />

将密文按n来分组，当每组的重合指数都接近0.065时，n便为密钥的长度值

### 重合指数算法

```python
def chzs(input): #定义重合指数法
    num_list = [0]*26
    for i in range(0,len(input)):
        ord_input = ord(input[i])-97
        num_list[ord_input] +=1
    n = len(input)
    res = 0
    for i in range(0,26):
        res += num_list[i]*(num_list[i]-1)
    return float(res)/((n)*n)

def len_key(input):#计算密钥长度
    may_d = 0
    index_d = 0
    for d in range(1,10):
        str_list = [""] * d
        for i in range(0,len(input)):
            str_list[i%d] += input[i]
        ch_sum = 0
        ch_time = 0
        for k in range(0, len(str_list)):
            if str_list[k] !="":
                ch_sum += chzs(str_list[k])
                ch_time +=1
            print(chzs(str_list[k]))
            print(str_list[k])
        print(d)
        k1 = abs(ch_sum / ch_time - 0.065)
        k2 = abs(may_d - 0.065)
        if k1<k2:                       #取最接近0.065的
            may_d = ch_sum / ch_time
            index_d = d
return index_d

```

### 重合互指数

重合互指数用于确定密钥字。

假定𝑥＝𝑥1𝑥2 … 𝑥𝑛和𝑦＝𝑦1 𝑦2…𝑦𝑛′,分别是长为𝑛和𝑛’的字符串。𝑥和𝑦的重合互指数是指𝑥的一个随机元素等于𝑦的一个随机元素的概率，记为𝑀𝐼𝑐(𝑥,𝑦)

将𝑥和𝑦中的字母A,B,C,……,Z出现的次数分别表示为𝑓0,𝑓1,……,𝑓25和𝑓0′, 𝑓1′,⋯,𝑓25′, 那么

<img src="https://s2.ax1x.com/2020/01/03/laeyPs.png" alt="laeyPs.png" style="zoom:50%;" />

### 重合互指数算法

```python
def xd_d(input): #相对位移
    #pi频率表
    pi=[0.082,0.015,0.028,0.043,0.127,0.022,0.02,0.061,0.07,0.002,0.008,0.04,0.024,0.067,0.075,0.019,0.001,0.06,0.063,0.091,0.028,0.01,0.023,0.001,0.02,0.001]
    print(pi)
    key_len = len_key(input)
    print("密钥长度：",key_len)
    d_list = [0]*key_len
    #weiyi = [0]*key_len
    str_list = [""]*key_len#根据密钥长度分组
    for i in range(0,len(input)):
        str_list[i%key_len] += input[i]
    for i in range(0,key_len):#查看分组结果和计算每个分组长度
        print(str_list[i],"长度：",len(str_list[i]))
        d_list[i]=len(str_list[i])
    for i in range(0,key_len):#找第i组的相对位移
        num_list_i = [0] * 26#每个字母出现的次数
        for k in str_list[i]:#计算i组每个字母出现次数
            i_c = ord(k) - 97
            num_list_i[i_c] += 1
        may_mc=0
        may_d=0
        for j in range(0,26):#位移长度
            mc=0
            for l in range(0,26):#计算最接近0.065的互重合指数
                mc+=pi[l]*float(num_list_i[(l+j)%26]/d_list[i])
                if(abs(mc-0.065)<=abs(may_mc-0.065)):
                    may_mc=mc
                    may_d=j
        print(i,"组：",may_mc,chr(97+may_d))

```

## 代码运行结果

**加密：**

![laeTi9.png](https://s2.ax1x.com/2020/01/03/laeTi9.png)

**解密：**

![laeHR1.png](https://s2.ax1x.com/2020/01/03/laeHR1.png)

**重合指数：**

![lamPzt.png](https://s2.ax1x.com/2020/01/03/lamPzt.png)

## 安全性分析

多表代换密码打破了原语言的字符出现规律，故其分析方法比单表代换密码复杂得多。多表代换密码对比单表代换密码安全性显著提高。但是仍然可以用一些统计分析法破解（具体参看上文攻击方法），但是前提是密文足够长。所以，较短的密文几乎是不可破译的。较长的密文是很容易破解的。

## 仿射密码的破解

### 加密原理

仿射密码属于单表代换密码，它使用线性方程加上一个模数。
$$
c_i=(k_1*m_i+k_2)mod26
$$
前提条件是：k1 和 26 互素

### 穷举攻击

通过密文，列举出所有可能的明文（一共 311 种情况），从中找出有特定标识或构成自然语言中有意义的单词或短语的正确明文。

```python
def BruteAffineDecrypt(c):
    for k1 in range(26):
        if gcd(k1,26) == 1:
            for k2 in range(26):
                print ("k1 = "+str(k1).zfill(2)+", k2 = "+str(k2).zfill(2)+": "+AffineDecrypt(c,k1,k2))

```

**结果：**

![lamT0S.png](https://s2.ax1x.com/2020/01/03/lamT0S.png)

### 频率分析攻击

统计密文中字母的出现次数和频率，从出现频率最高的几个字母及双字母组合、三字母组合开始，并假定它们是英语中出现频率较高的字母及字母组合对应的密文，逐步推测各密文字母对应的明文字母。

```python
def pinlv(input):
    num_list = [0]*26
    for i in range(0,len(input)):
        ord_input = ord(input[i])-97
        num_list[ord_input] +=1
    n = len(input)
    x=[]
    y=[]
    for i in range(0,26):
        x.append(chr(97+i))
        y.append(float(num_list[i]/n))
    plt.bar(x, y, 0.4, color="green")
    plt.xlabel("Letters")
    plt.ylabel("Frequency")
    plt.title("Frequency analysis")
plt.show()

```

![lantnf.png](https://s2.ax1x.com/2020/01/03/lantnf.png)

## 完整代码

### 维吉尼亚加解密

```python
def VigenereEncrypt(m,k):
    k = k.upper()
    lm = len(m)
    lk = len(k)
    j = 0
    ans = ""
    for i in range(lm):
        if m[i].isupper():
            ans += chr(((ord(m[i])-ord('A'))%26+(ord(k[j%lk])-ord('A'))%26)%26+ord('A'))
            j += 1
        elif m[i].islower():
            ans += chr(((ord(m[i])-ord('a'))%26+(ord(k[j%lk])-ord('A'))%26)%26+ord('a'))
            j += 1
        else:
            ans += m[i]
    return ans
# Decryption
def VigenereDecrypt(c,k):
    k = k.upper()
    lc = len(c)
    lk = len(k)
    j = 0
    ans = ""
    for i in range(lc):
        if c[i].isupper():
            ans += chr(((ord(c[i])-ord('A'))%26+26-(ord(k[j%lk])-ord('A'))%26)%26+ord('A'))
            j += 1
        elif c[i].islower():
            ans += chr(((ord(c[i])-ord('a'))%26+26-(ord(k[j%lk])-ord('A'))%26)%26+ord('a'))
            j += 1
        else:
            ans += c[i]
    return ans
# Main function
def main():
    while True:
        op = input("What do you want to do?\n[E]Encryption [D]Decryption\n").upper()
        if op[0] == "E":
            s = input("Please enter your plaintext:\n\t")
            k = input("Please enter your key:\n\t")
            # s = "plaintext"
            # k = "key"
            print (8*"*"+"  Encryption  "+8*"*")
            Encryption = VigenereEncrypt(s,k)
            print ("The Ciphertext is:\n\t"+Encryption)
            break
        elif op[0] == "D":
            s = input("Please enter your ciphertext:\n\t")
            k = input("Please enter your key:\n\t")
            # s = "ciphertext"
            # k = "key"
            print (8*"*"+"  Decryption  "+8*"*")
            Decryption = VigenereDecrypt(s,k)
            print ("The Plaintext is:\n\t"+Decryption)
            break
        else:
            continue
if __name__ == '__main__':
main()

```

### 重合指数

```python
def chzs(input): #定义重合指数法
    num_list = [0]*26
    for i in range(0,len(input)):
        ord_input = ord(input[i])-97
        num_list[ord_input] +=1
    n = len(input)
    res = 0
    for i in range(0,26):
        res += num_list[i]*(num_list[i]-1)
    return float(res)/((n)*n)

def len_key(input):#计算密钥长度
    may_d = 0
    index_d = 0
    for d in range(1,10):
        str_list = [""] * d
        for i in range(0,len(input)):
            str_list[i%d] += input[i]
        ch_sum = 0
        ch_time = 0
        for k in range(0, len(str_list)):
            if str_list[k] !="":
                ch_sum += chzs(str_list[k])
                ch_time +=1
            print(chzs(str_list[k]))
            print(str_list[k])
        print(d)
        k1 = abs(ch_sum / ch_time - 0.065)
        k2 = abs(may_d - 0.065)
        if k1<k2:                       #取最接近0.065的
            may_d = ch_sum / ch_time
            index_d = d
    return index_d
def xd_d(input): #相对位移
    #pi频率表
    pi=[0.082,0.015,0.028,0.043,0.127,0.022,0.02,0.061,0.07,0.002,0.008,0.04,0.024,0.067,0.075,0.019,0.001,0.06,0.063,0.091,0.028,0.01,0.023,0.001,0.02,0.001]
    print(pi)
    key_len = len_key(input)
    print("密钥长度：",key_len)
    d_list = [0]*key_len
    #weiyi = [0]*key_len
    str_list = [""]*key_len#根据密钥长度分组
    for i in range(0,len(input)):
        str_list[i%key_len] += input[i]
    #print("****",len(d_list))
    for i in range(0,key_len):#查看分组结果和计算每个分组长度
        print(str_list[i],"长度：",len(str_list[i]))
        d_list[i]=len(str_list[i])
    #sll = len(str_list)
    for i in range(0,key_len):#找第i组的相对位移
        num_list_i = [0] * 26#每个字母出现的次数
        for k in str_list[i]:#计算i组每个字母出现次数
            i_c = ord(k) - 97
            num_list_i[i_c] += 1
        may_mc=0
        may_d=0
        for j in range(0,26):#位移长度
            mc=0
            for l in range(0,26):#计算最接近0.065的互重合指数
                mc+=pi[l]*float(num_list_i[(l+j)%26]/d_list[i])
                if(abs(mc-0.065)<=abs(may_mc-0.065)):
                    may_mc=mc
                    may_d=j
            #print(mc)
        print(i,"组：",may_mc,chr(97+may_d))

    return

def main():
    s=input("Please enter your ciphertext:\n\t")
    xd_d(s)
if __name__ == '__main__':
main()

```

### 仿射频率分析

```python
import numpy as np
import matplotlib.mlab as mlab
import matplotlib.pyplot as plt
def pinlv(input):
    num_list = [0]*26
    for i in range(0,len(input)):
        ord_input = ord(input[i])-97
        num_list[ord_input] +=1
    n = len(input)
    x=[]
    y=[]
    for i in range(0,26):
        x.append(chr(97+i))
        y.append(float(num_list[i]/n))
    plt.bar(x, y, 0.4, color="green")
    plt.xlabel("Letters")
    plt.ylabel("Frequency")
    plt.title("Frequency analysis")
    plt.show()
def main():
    s = input("Please enter your ciphertext:\n\t")
    pinlv(s)
if __name__ == '__main__':
main()

```

### 仿射暴力破解

```python
def gcd(a,b):
    if a%b == 0:
        return b
    return gcd(b,a%b)
def findModReverse(a,m):
    if gcd(a,m)!=1:
        return None
    u1,u2,u3 = 1,0,a
    v1,v2,v3 = 0,1,m
    while v3!=0:
        q = u3//v3
        v1,v2,v3,u1,u2,u3 = (u1-q*v1),(u2-q*v2),(u3-q*v3),v1,v2,v3
    return u1%m
def AffineDecrypt(c,k1,k2):
    k2 %= 26
    if gcd(k1,26) != 1:
        print("k1 and 26 are not mutually prime, and decryption fails.")
        return
    k1 = findModReverse(k1,26)
    l = len(c)
    ans = ""
    for i in range(l):
        if c[i].isupper():
            ans += chr((k1*(ord(c[i])-ord('A')+26-k2))%26+ord('A'))
        elif c[i].islower():
            ans += chr((k1*(ord(c[i])-ord('a')+26-k2))%26+ord('a'))
        else:
            ans += c[i]
    return ans
def BruteAffineDecrypt(c):
    for k1 in range(26):
        if gcd(k1,26) == 1:
            for k2 in range(26):
                print ("k1 = "+str(k1).zfill(2)+", k2 = "+str(k2).zfill(2)+": "+AffineDecrypt(c,k1,k2))
def main():
    s = input("Please enter your plaintext:\n\t")
    print (7 * "*" + "  Attack  " + 7 * "*")
    BruteAffineDecrypt(s)

if __name__ == '__main__':
main()

```

