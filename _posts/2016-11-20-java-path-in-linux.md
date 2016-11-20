---
layout: post
title:  "linux下配置java 环境变量"
date:   2016-11-21 +0800
categories: 学无止境
tag: linux
---

* content
{:toc}


### #1.查看java环境变量

```

whereis java

which java

echo $JAVA_HOME

echo $PATH

```

### #2.配置

要修改环境变量可以通过三种方式：

#### (1) 只对当前shell生效的修改

直接在命令行修改`export JAVA_HOME='XXXXX'`;

换个shell后修改的变量就不生效了，一般不用。

#### (2) 针对用户生效的修改

修改 `~/.bash_profile`或`~/.bash_login` 或`~/.profile`配置文件

> 这三个配置属于用户偏好设置；在login shell的bash环境下，读取完`/etc/profile`后，就按顺序查找这三个文件，先找到哪个就用哪个

#### (3)对所有用户生效的修改

修改`/etc/profile`;

在文末添加上要设置的环境变量：

```

export JAVA_HOME="/usr/share/jdk1.8.0" 
export PATH="$JAVA_HOME/bin:$PATH"
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

```



注销或执行`source 配置文件名`：`source /etc/profile`或`source ~/.profile`


