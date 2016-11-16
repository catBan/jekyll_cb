---
layout: post
title:  "迷之python版本"
date:   2016-10-23 21:03:55 +0800
categories: 学无止境
tag: Python
---

* content
{:toc}


&emsp;&emsp;初次跟Python打交道，准备约会python 3.4 ；下载并安装好后，开始了探索之旅。

&emsp;&emsp;但生活总是在意外中彰显她的性感。。在执行一些例子时，提示需要使用python3.4的版本，当时我就纳闷了，没错呀，我装的是3.4的版本呀，怎么能说我当前是2.7版本?  不管了，反正这2.7也不是我用的，说不定是多出来的，卸了再说... 

```

    sudo apt-get remove python2.7

```

&emsp;&emsp; 不对啊，卸载的时候怎么出现了deepin-menu , deepin-music这些deepin自己的程序？难道……灵光一闪，搜了下"deepin python版本",果然，deepin某些程序是有依赖python2.7的，我 这么一弄，岂不是会导致程序出错 - - 　    不管，先把2.7替换成3.4再说...  

```

    mv /usr/bin/python /usr/bin/python2.7

    ln -s /usr/local/bin/python3.4 /usr/bin/python

```

&emsp;&emsp;看下ptyhon现在的版本是什么再来看依赖到的程序有没有问题。一敲`python`,提示命令不存在，这下好了，不存在。哎哟喂，我做了什么吗。。

&emsp;&emsp; 继续看deepin论坛里的相关帖子，找到原来已经有人跟我一样这么干的，找到一句话“要运行 python3，是需要在命令行中输入 python3 的，而不是输入 python。”呵呵，文档里似乎看到过,怎么就忘了。。到 /usr/bin下一查，发现deepin系统包含了2.7 /3.4 /3.5的Python版本，控制台输入python3 果然出来了。

&emsp;&emsp; 好了，把默认的2.7还原回去，敲python默认使用的版本还原回2.7了。至于依赖到的深度音乐/深度影院。。。本来也不用，不管你了 - - 　 哎呀码，回头想了下，遇到个白吃的问题跟`寄几`。