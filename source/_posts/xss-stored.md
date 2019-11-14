---
title: DVWA--xss_stored
date: 2019-11-14 21:12:02
tags: [xss攻击]
categories: xss攻击
---

DVWA的存储型xss。

<!--more-->

## 级别：low

### Stored XSS Source

```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = stripslashes( $message );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitize name input
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```

### 分析

 `isset() ` 函数在php中用来检测变量是否设置，该函数返回的是布尔类型的值，即true/false 

 `trim() ` 函数作用为移除字符串两侧空白字符或其他预定义字符 

 `stripslashes() ` 函数用于删除字符串中的反斜杠 

 `mysqli_real_escape_string() ` 函数会对字符串中的特殊号`(\x00，\n，\r，\，'，"，\x1a) ` 进行转义 

 在代码中对message，name输入框内容  `没有进行XSS方面的过滤和检查 `

 且通过  `query ` 语句插入到数据库中。所以存在存储型XSS漏洞 

### 方法

由于name和message输入框均存在xss。但name输入框有字符限制，这里可以使用burpsuite抓包修改name输入框内容： 

```
<script>alert(document.cookie)</script>
```

![MUAK3Q.png](https://s2.ax1x.com/2019/11/14/MUAK3Q.png)

![MUAQjs.png](https://s2.ax1x.com/2019/11/14/MUAQjs.png)

![MUA1un.png](https://s2.ax1x.com/2019/11/14/MUA1un.png)

**由于提交的结果存储在数据库中，所以每次刷新页面，输入的恶意代码就会被执行一次 **

## 级别：medium

### Stored XSS Source

```php

<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = str_replace( '<script>', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```

### 分析

 `strip_tags() ` 函数剥去字符串中的 HTML、XML 以及 PHP 的标签，但允许使用 `<b>` 标签。 

 `addslashes() ` 函数返回在预定义字符（单引号、双引号、反斜杠、NULL）之前添加反斜杠的字符串。 

 `htmlspecialchars() ` 函数把预定义的字符&、"、'、<、>转换为 HTML 实体，防止浏览器将其作为HTML元素 

 对message输入内容进行检测过滤，因此无法再通过message参数注入XSS代码
但是对于name参数，只是简单过滤了`<script>`字符串，仍然存在存储型的XSS。 

### 方法

 抓包修改name输入内容:

1.  使用双写绕过，输入  `<scr<script>ipt>alert(document.cookie)</script> `
2.  使用大小写绕过，输入`  <sCript>alert(document.cookie)</script> `
3.  输入其他标签，如  `<IMG src=1 onerror=alert(document.cookie)> `

![MUmpy4.png](https://s2.ax1x.com/2019/11/14/MUmpy4.png)

**由于low级别已经注入过，所以打开medium级别会直接弹出cookie**

## 级别：high

### Stored XSS Source

```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?>
```

### 分析

和上面两个级别一样，抓包修改name内容即可，代码对name输入内容利用正则匹配删除所有关于`<script>`标签

### 方法

抓包修改name内容为

```
<IMG src=1 onerror=alert(document.cookie)>
```

