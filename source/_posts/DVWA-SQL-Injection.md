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

## 级别：low

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

