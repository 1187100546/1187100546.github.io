---
title: hexo发生error：spawn failed错误的解决方法
date: 2019-11-24 18:54:46
tags: [记录]
categories: 记录
---

hexo突然上传不了，出错了。

<!--more-->

今天hexo突然部署不了文章了，错误页面如下。

![MO2Q8s.png](https://s2.ax1x.com/2019/11/24/MO2Q8s.png)

然后就去github仓库看了一下，发下之前的ssh key没了，重新设了一个也连接不上。最后找到一个方法。

在存放key的目录下新建config文件。

![MO22Ie.png](https://s2.ax1x.com/2019/11/24/MO22Ie.png)

填入以下内容

```
Host github.com
User 你GitHub的邮箱
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

然后用 `ssh -T git@github.com`命令测试能否连接

如果没有出现ssh不能连接的话，忽略以上内容

接着回到博客的根目录

第一种方法：

删除.deploy_git文件

然后输入`git config --gloabl core.autocrlf false`

重新hexo clean

hexo g

hexo d

部署

但是发现好像只是一次性的。并不能永久解决

第二种方法：

打开_config.yml配置文件

修改以下内容

deploy:

type: git

repo: https://github.com/yourname/yourname.github.io.git

branch: master

其中的repo修改为

git@github.com:yourname/yourname.github.io.git