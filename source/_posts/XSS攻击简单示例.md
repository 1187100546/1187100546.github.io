---
title: XSS攻击简单示例
date: 2019-10-13 19:06:06
tags: [XSS攻击]
categories: XSS攻击
---
xss
<!--more-->

### 建立漏洞页面

在www目录下新建xss.php,代码如下

```php
<!DOCTYPE html>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html;charset=utf-8">
		<title>xss漏洞简单搭建</title>
	</head>
	<body>
		<center>
			<h6>把我们输入的字符串输出到input里的value属性里</h6>
			<form action="" method="get">
				<h6>请输入你想显现的字符串</h6>
				<input type="text" name="xss_input_value" value="输入"><br />
				<input type='submit'>
			</form>
			<hr>
	
			<?php
			//php防注入和XSS攻击通用过滤
//$_GET     && SafeFilter($_GET);
function SafeFilter (&$arr) 
{
   $ra=Array('/([\x00-\x08,\x0b-\x0c,\x0e-\x19])/','/script/','/javascript/','/vbscript/','/expression/','/applet/'
   ,'/meta/','/xml/','/blink/','/link/','/style/','/embed/','/object/','/frame/','/layer/','/title/','/bgsound/'
   ,'/base/','/onload/','/onunload/','/onchange/','/onsubmit/','/onreset/','/onselect/','/onblur/','/onfocus/',
   '/onabort/','/onkeydown/','/onkeypress/','/onkeyup/','/onclick/','/ondblclick/','/onmousedown/','/onmousemove/'
   ,'/onmouseout/','/onmouseover/','/onmouseup/','/onunload/');
     
   if (is_array($arr))
   {
     foreach ($arr as $key => $value) 
     {
        if (!is_array($value))
        {
          if (!get_magic_quotes_gpc())  //不对magic_quotes_gpc转义过的字符使用addslashes(),避免双重转义。
          {
             $value  = addslashes($value); //给单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）  加上反斜线转义
          }
          $value       = preg_replace($ra,'',$value);     //删除非打印字符，粗暴式过滤xss可疑字符串
          $arr[$key]     = htmlentities(strip_tags($value)); //去除 HTML 和 PHP 标记并转换为 HTML 实体
        }
        else
        {
          SafeFilter($arr[$key]);
        }
     }
   }
}

			if (isset($_GET['xss_input_value']))
			{
				$s=$_GET['xss_input_value'];
				echo'<input type="text" value="'.$_GET['xss_input_value'].'">';
				
				
			}
			else{
				echo '<input type="text" value="输出">';
			}
			
?>

		</center>
		
		</script>
		

	</body>
</html>

```

在浏览器输入http://localhost/xss.php/

进入页面

![1570973405972](C:\Users\张寅\AppData\Roaming\Typora\typora-user-images\1570973405972.png)

输入

```bash
"><img src=1 onerror=alert(/xss/)>
```

![L0_P317WRNGPCYPGDQPG_0O.png](https://i.loli.net/2019/10/13/UonfqPdrGlsBXSt.png)

攻击成功

### 漏洞修补

将xss.php中

```bash
//$_GET     && SafeFilter($_GET);
```

注释去掉

```bash
$_GET     && SafeFilter($_GET);
```

通过SafeFilter()函数来过滤输入的内容

再输入刚才的内容

![_I9UOP3AI0448_OMY9_R_6Y.png](https://i.loli.net/2019/10/13/l2DCHsWTqGRXNfJ.png)

**注意：建议用ie浏览器打开，关闭ie浏览器的xss过滤器**