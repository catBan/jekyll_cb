---
layout: post
title:  "在sublime text上打造python开发环境"
date:   2016-11-06 18:43:34 +0800
categories: 学无止境
tag: Python
---

* content
{:toc}


Sublime Text一直只是拿来简单的用作代码查看工具，这次的python开发就用sublime练练手，顺便见识下ST的诱人之处。







### 安装插件管理工具

-----



为了方便管理插件，需要安装`Package Control`。点击这里看官方的安装说明。   安装好后，点这里看使用文档。不想看文档的... 贴了几条常用的：







1) 打开方式`ctrl+shift+p` (Win, Linux)   / `cmd+shift+p` (OS X)



2) 输入packege后，可以看到列出了所有package的相关命令，相信你一眼就知道这些命名是干嘛的，是不是很简单.  :)



### 安装python编码相关的插件

----





#### 1) 安装sublimelinter







>  _SublimeLinter 只是一个代码检测的 **框架**，本身不含任何语言检测插件，它容纳其他的语言检测插件，点[这里](http://www.sublimelinter.com/en/latest/about.html "instruction" )是它的官方介绍。_







 调出Package Control，输入`Package Control:Install Package`安装，稍等片刻出现列表后，输入`sublimelinter`,找到对应的插件安装。







### 2) 安装sublimelinter-flake8 







> _sublimelinter-flake8 是一款python的语言检测插件，更多的其他语言检测插件可以点[这里](https://github.com/SublimeLinter "sublimelinter" )查看_







首先查看了`sublimelinter-flake8` 的[安装说明](https://github.com/SublimeLinter/SublimeLinter-flake8)，它需要安装flake8 2.1以上。而安装flake8又需要Python3及pip。



在安装pip时出了点状况，提示错误:



     











提示安装需要 SSL传输协议，网上搜了一圈，说是有的linux 版本是有默认安装openssl,用命令查了下，果然有，那又是为什么报错，其中定有蹊跷：











再摸索了一下，找到这样的解释：



```



Redhat在封装openssl的时候，把openssl分成了几个部分，执行码部分就是 openssl-1.0.0-27.el6.x86_64 这种包。

openssl-devel-1.0.0-27.el6.x86_64 这个就是包含了头文件，头文件参考，某些库文件等跟开发相关的东西。



mod_ssl-2.2.15-26.el6.x86_64 这个不是open ssl 本身的东西，是apache的模块。



你在http://www.openssl.org/source/上下载的源码编译安装后得到的东西就是openssl-1.0.0-27.el6.x86_64和openssl-devel-1.0.0-27.el6.x86_64这两个包加在一起的内容。



另外，OpenSSL是分系列的，每个系列下再分版本 a b c d e。。。。



目前常用的是 0.9.8 1.0.0 1.0.1 三个系列。



RHEL 6.4 是openssl 1.0.0 系列的版本。

RHEL 6.5 是 openssl 1.0.1 系列的版本。



Redhat 提供的openssl升级包的版本一般是 openssl-1.0.0-27.el6.X.x86_64.rpm 这种。 Redhat 会把OpenSSL发布的补丁整合到现有版本中去，叫做backport。



例如，RHEL 6.4 目前的最新的OpenSSL就是2014-06-05发布的 openssl-1.0.0-27.el6_4.4.x86_64.rpm 和 openssl-devel-1.0.0-27.el6_4.4.x86_64.rpm 

RHEL 6.5 则是2014-08-13发布的 openssl-1.0.1e-16.el6_5.15.x86_64.rpm 和 openssl-devel-1.0.1e-16.el6_5.15.x86_64.rpm。



因为不同系列的OpenSSL，存在的安全漏洞或者BUG不一定相同，所以版本要根据系列来判断。

当然，如果你愿意手动编译安装openssl，那么也可以，只是注意相关软件的依赖。



```



简单来说就是`openssl`跟 `openssl-devel` 是不一样的，`openssl`不满足`openssl-devel`。所以需要安装的是dev版的`openssl`。注意,redhat/centos 使用 `openssl-devel`,有些linux 需要安装的是`libssl-dev`



#### 3) Anaconda

Anaconda是一个神器，有了它，sublime上也能像在IDE上使用python那样便捷了，什么代码自动补全不在话下。

就像官方说的:"_Anaconda turns your Sublime Text 3 into a full featured Python development IDE_"



它的安装很简单，跟安装Sublimelinter一样。

说多了也是乱，原汁原味的更有助于吸收：[点我](http://damnwidget.github.io/anaconda/#carousel-features "Anaconda" )看Anaconda的介绍、安装、使用等。



#### 4)SublimeREPL -　调试python程序

 聪明的你一定已经装好了。

 安装后在`Tools -> SublimeREPL -> Python` 中 可以调试当前文件，具体的使用请自行查阅文档。

其实就是懒的把查过的整理出来了，有问题再找我。

#### 5)辅助插件



`SideBarEnhancements` ：丰富了侧边栏菜单。

`Markdown Preview`:预览和编辑markdown。



### 其他配置

----

>  vim



sublime text支持vim,需要修改Vintage配置。

Vintage默认是禁用的， 通过`ignored_packages `配置。如果要从ignored packages列表中移除"Vintage"的话可以通过下面的方式编辑:选择`Preferences/Settings - Default`菜单编辑`ignored_packages`配置, 修改:  

```

  "ignored_packages": ["Vintage"]

```

 成:

```

    "ignored_packages": []

```

 然后保存文件。Vintage模式则已启用——你可以看到"INSERT MODE"显示在状态栏了。

Vintage默认是插入模式。可以添加:

   "vintage_start_in_command_mode": true 这项配置到User Settings里。



### # 总结

----



折腾了半天，配合了好几个插件终于是用起来比较顺手了，过程中发现sublime基于json的配置挺方便的，不仅是sublime自身，安装的插件也是一样的配置规则。

比如想修改Anaconda里面goto definition的快捷键（跳转的函数定义的地方）：

在`Preferences -> Package Settings -> Anaconda -> Key Bindings -User`中覆盖掉`Key Bindings -Default `的相关键值就可以了，是不是很方便清晰..



不过为了弄个顺手的环境还是要装不少东西，总的来说我还是会用IDE来编码，省心...