---
title: 序列密码
date: 2020-01-03 16:56:18
tags: [密码学]
categories: 密码学
---

密码学中的序列密码。

<!--more-->

## 实验要求

选择一个15次以上的不可约多项式，编写一个线性反馈移位寄存器。验证生成序列的周期。

## 设计思路

利用python数组的pop操作可以实现移位的效果，将反馈函数写成0，1数组，便可以实现反馈效果。

## 线性反馈移位寄存器

### 算法实现

```python
def lfsr(lst, k,key):
    temp_l= lst[:]
    temp_k= key[:]
    for i in range(k):
        #temp.append(temp.pop(0))
        k_out=0
        for j in range(18):
            if(temp_k[j]==1):
                k_out+=(temp_k[j]+temp_l[j])%2
        k_out=k_out%2
        #print(k_out)
        temp_l.pop(0)
        temp_l.append(k_out)
        print("当前寄存器值：",temp_l,"循环次数：",i+1,temp_l==lst)
    #return temp
def main():
    lst=[0,1,1,1,0,0,0,1,0,1,0,0,1,0,0,1,0,1]
    k=262146
    key=[1,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]
    lfsr(lst,k,key)
print(len(key))

```

### 运行结果

18次线性移位寄存器，当运行到周期次数时开始重复。

寄存器初始值：[0,1,1,1,0,0,0,1,0,1,0,0,1,0,0,1,0,1]

本原多项式：[1,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0]

![lauAUg.png](https://s2.ax1x.com/2020/01/03/lauAUg.png)

## 安全性分析

由算法的实现可知，序列密码算法的加解密对种子秘钥的依赖十分强烈。故需要保证种子秘钥的安全性。对于此可进行相关攻击。

可以进行穷举搜素攻击，故为了保证安全强度，要求秘钥长度足够长。

弱密钥攻击，弱密钥会产生重复的密钥流，一旦子密钥序列出现了重复，密文就有可能被破解。

## 合理性分析

序列密码具有实现简单、便于硬件实施、加解密处理速度快、没有或只有有限的错误传播等特点，因此在实际应用中，特别是专用或机密机构中保持着优势，序列密码是一个随时间变化的加密变换，具有转换速度快、低错误传播的优点，硬件实现电路更简单。
