---

title: msf攻击windows
date: 2019-10-11 13:55:25
tags: [msf][渗透]
categories: msf渗透攻击
---

利用msf对windows进行渗透攻击。

<!--more-->

### 一、利用ms08_067攻击xp系统

在kali终端输入：

```bash
msfconsole
```

进入metasploit

首先搜索这个漏洞

![1570774738028](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570774738028.png)

然后使用模块

![1570774772837](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570774772837.png)

查看选项

![1570774821418](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570774821418.png)

设置目标IP和自身IP，及被攻击系统版本

![1570774842853](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570774842853.png)

直接攻击返回shell

![1570774863433](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570774863433.png)

### 二、利用永恒之蓝（ms17_010）攻击win7

首先搜索该漏洞

![1570775052329](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775052329.png)

使用该漏洞模块

![1570775067326](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775067326.png)

设置目标IP和自身IP，目标系统默认

![1570775084979](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775084979.png)

攻击

![1570775102714](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775102714.png)

攻击成功，建立会话

![1570775118532](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775118532.png)

打开绘画，进入shell

![1570775133539](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775133539.png)

### 三、利用metasploit渗透win10

首先通过msfvenom生成木马文件，并放入win10靶机中

![1570775167414](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775167414.png)

![1570775173831](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775173831.png)

然后在kali上开启监听

![1570775193761](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775193761.png)

然后在win10靶机上运行木马程序，监听便能收到，获取shell

![1570775209578](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570775209578.png)

攻击完成。

以上三个攻击均拿到了shell便可以在对方机器上设置远程用户，然后就可以通过本机的Windows系统远程登陆被攻击的机器，也可以记录对方的键盘记录，远程关机等等