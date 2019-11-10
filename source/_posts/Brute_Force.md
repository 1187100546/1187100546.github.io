---
title: DVWA-Brute_Force暴力破解
date: 2019-10-22 20:53:42
tags: [暴力破解]
categories: 暴力破解
---

利用burpsuite暴力破解DVWA的Brute_Force。

<!--more-->

### 一、配置burpsuite

打开火狐浏览器，工具栏里找到preferences,找到network proxy,点击settings,填写如下配置

![~`KHUGAWJ8ZXHWWKAC``3~S.png](https://i.loli.net/2019/10/22/dbGx16ywKJ32Y9t.png)

![DH136K_XS4P1DIU_UWK7_R5.png](https://i.loli.net/2019/10/22/oag5jxbHnJhIQy8.png)

打开burpsuite,勾选代理即可

![2KIR_2_A1X2TVG54_Z_@C96.png](https://i.loli.net/2019/10/22/zeU52IaT8qSYhpN.png)

先关闭proxy里的intercept，先不拦截

### 二、寻找注入点

先进入DVWA页面，安全等级设置位low,再进入暴力破解的页面

![C0_0_CWB_ISDUO_RCNCS45I.png](https://i.loli.net/2019/10/22/jEcwLoTW9AgiKdR.png)

输入用户名

```bash
admin
```

密码

```bash
123
```

开启burp suite拦截，登陆

![H_HLQGD19_DF_ODPW_@385T.png](https://i.loli.net/2019/10/22/OjQFxUDV3PBmiuE.png)

选中全部内容，右键发送给intruder

![2Y19__1WHZ__RQM@9K__BLX.png](https://i.loli.net/2019/10/22/TdNYEPaJ9vpGfeK.png)

点clear清除全部变量，选中123，点add，添加变量

![O@P_AL@_P1IU`E~MU5A7X_Q.png](https://i.loli.net/2019/10/22/Opm7XKfaS1otTZE.png)

在payloads里添加破解字典

![0EA__K`9T_QD_WXGC7_1_@S.png](https://i.loli.net/2019/10/22/837G4kexKdzlRaS.png)

下面就可以攻击了

burp suite的攻击类型

![5C`_M8L_Q_D_X_AT_`___@J.png](https://i.loli.net/2019/10/22/Gm5AEUr6lTiKd2Y.png)

#### 第一种：

Sniper标签 这个是我们最常用的，Sniper是狙击手的意思。这个模式会使用单一的payload【就是导入字典的payload】组。它会针对每个position中$$位置设置payload。这种攻击类型适合对常见漏洞中的请求参数单独地进行测试。攻击中的请求总数应该是position数量和payload数量的乘积。

#### 第二种：

Battering ram – 这一模式是使用单一的payload组。它会重复payload并且一次把所有相同的payload放入指定的位置中。这种攻击适合那种需要在请求中把相同的输入放到多个位置的情况。请求的总数是payload组中payload的总数。简单说就是一个playload字典同时应用到多个position中

#### 第三种：

Pitchfork – 这一模式是使用多个payload组。对于定义的位置可以使用不同的payload组。攻击会同步迭代所有的payload组，把payload放入每个定义的位置中。比如：position中A处有a字典，B处有b字典，则a【1】将会对应b【1】进行attack处理，这种攻击类型非常适合那种不同位置中需要插入不同但相关的输入的情况。请求的数量应该是最小的payload组中的payload数量

#### 第四种：

Cluster bomb – 这种模式会使用多个payload组。每个定义的位置中有不同的payload组。攻击会迭代每个payload组，每种payload组合都会被测试一遍。比如：position中A处有a字典，B处有b字典，则两个字典将会循环搭配组合进行attack处理这种攻击适用于那种位置中需要不同且不相关或者未知的输入的攻击。攻击请求的总数是各payload组中payload数量的乘积。

### 三、攻击

选择sniper攻击方式，开始攻击

根据攻击结果的length来判断是否成功

![R_OTHC_P6_X_`_ZWAN_O_`I.png](https://i.loli.net/2019/10/22/JFGd3PfibMpaTZ6.png)

攻击成功，密码为password

此法适用各个安全等级