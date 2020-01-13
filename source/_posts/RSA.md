---
title: 公钥密码RSA
date: 2020-01-12 18:16:06
tags: [密码学]
categories: 密码学
mathjax: true
---

密码学中的RSA加密。

<!--more-->

## RSA算法原理

1. 密钥的生成

- 选择两个大素数 𝑝和𝑞,（𝑝≠𝑞，需要保密，步骤4以后建议销毁）
- 计算𝑛=𝑝×𝑞， (𝑛)=(𝑝－1)×(𝑞－1)

-   选择整数 𝑒 使 ((𝑛)，𝑒) =1,  1<𝑒< (𝑛) 

- 计算𝑑，使𝑑=𝑒－1 𝑚𝑜𝑑 (𝑛),

​    得到：公钥为{𝑒, 𝑛}； 私钥为{𝑑}

2. 加密(用𝒆，𝒏)： 明文𝑀<𝑛， 密文𝐶=𝑀^𝑒 (𝑚𝑜𝑑 𝑛).

3. 解密(用𝒅，𝒏)： 密文𝐶， 明文𝑀 =𝐶^𝑑 (𝑚𝑜𝑑 𝑛)

## 大素数生成

Miler-Rabin 算法：

对于大整数的素性测试，一般用 Miller-Rabin 算法。它是一个基于概率的算法，是费马小定理（若 n 是一个素数，则 an-1 ≡ 1 (mod n) ）的一个改进。要测试 n 是否为素数，首先将 n−1 分解为 2sd 。在每次测试开始时，先随机选一个介于 [1,n−1] 的整数 a ，之后如果对所有的 r∈[0,s−1] ，若admodn≠1 且 a2rd mod n≠−1，则 n 是合数。否则，n 有 3/4 的概率为素数。增加测试的次数，该数是素数的概率会越来越高。这样，我们就可以给定位数 n 的情况下随机生成数，然后再用 Miller-Rabin 算法验证它是不是素数，若是，则就用它，否则再随机生成其他数字，循环。Python 脚本如下：

```python
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

```

## 带模的幂运算

原理：模重复平方运算，Python 代码如下：

```python
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

```

## 求逆运算

扩展欧几里得法

```python
def computeD(e, phi_n):
    (x, y, r) = extendedGCD(phi_n, e)
    # y maybe < 0, so convert it
    if y < 0:
        return phi_n + y
    return y
# Extended Euclidean algorithm
def extendedGCD(a, b):
    # a*xi + b*yi = ri
    if b == 0:
        return (1, 0, a)
    # a*x1 + b*y1 = a
    x1 = 1
    y1 = 0
    # a*x2 + b*y2 = b
    x2 = 0
    y2 = 1
    while b != 0:
        q = a // b
        # ri = r(i-2) % r(i-1)
        r = a % b
        a = b
        b = r
        # xi = x(i-2) - q*x(i-1)
        x = x1 - q*x2
        x1 = x2
        x2 = x
        # yi = y(i-2) - q*y(i-1)
        y = y1 - q*y2
        y1 = y2
        y2 = y
return(x1, y1, a)

```

## 运行结果

![lov9Nn.png](https://s2.ax1x.com/2020/01/12/lov9Nn.png)

## 安全性分析

RSA的安全性依赖于大数分解问题，目前，还未能从数学上证明由𝑐和𝑒计算出𝑚一定需要分解𝑛，然而，如果新方法能使密码分析者推算出𝑑，它也就成为大数分解的一个新方法

非对称加密算法中 1024 bit 密钥的强度相当于对称加密算法 80bit 密钥的强度。但是，从效率上，密钥长度增长一倍，公钥操作所需时间增加约 4 倍，私钥操作所需时间增加约 8 倍，公私钥生成时间约增长16倍。所以，我们要权衡一下效率和安全性。一般来说， 1024 bit 只能用于加密 最多117 字节的明文。

低加密指数攻击：

为了使加密高效，一般希望选取较小的加密指数 e ，但是 e 不能太小，否则容易遭到低加密指数攻击。

假设用户使用的密钥 e=3 。考虑到加密关系满足：
$$
C\equiv m^3\ mod\ n\\
$$
则容易得到：
$$
m^3=c+k\times n\\
m=\sqrt[3]{c+k\times n}
$$
攻击者可以从小到大枚举 k ，依次开三次根，直到开出整数为止。

低加密指数广播攻击：

还有一种情况是如果给 k 个用户发的都是同个低加密指数比如 e=3 ，在不同的模数 n1.n2,n3下 ，可由 CRT（中国剩余定理） 解出 m3 ，从而直接开三次根解出 m。

共模攻击：

场景：n 相同（让多个用户使用相同的模数 n ），但他们的公私钥对不同。这样，我们可以在已知 n,e1,e2,c1,c2 的情况下解出 m 。过程如下：

其实有个隐形的前提条件是：
$$
gcd(e_1,e_2)=1
$$
存在 s1,s2 使得：
$$
s_1e_1+s_2e_2=1
$$
又由 RSA 定义可知：
$$
c_1\equiv m^{e_1}\ mod\ n\\
c_2\equiv m^{e_2}\ mod \ n
$$
可得出：
$$
c_1^{s_1}c_2^{s_2}\equiv m\ mod\ n
$$
解出明文。

## 完整代码

```python
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

