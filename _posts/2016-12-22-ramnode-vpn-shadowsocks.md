---
layout: post
title:  "RamNode VPS搭建VPN之路"
date:   2016-12-22 23:03:01 +0800
categories: 七八日常
tag: 白菜炖粉条
---

* content
{:toc}


# 在RamNode 购买vps

这是产品的截图，第一次购买，选择了最弱的一个套餐，一年15刀（网站顶部常年挂有一个10% off的优惠码），一个月500G，杠杠的～ 
![whats]({{ '/styles/images/post/16122201.png' | prepend: site.baseurl  }})

购买的过程也是逗，被fraud checking system列为欺诈购买行为，估计规则里面对中国ip做了某种限制，真是气愤得，哼！ 想想，不能忍啊，作为一个黄色的脸，黑色的眼的中国人，怎么能怂？立马用蹩脚的英语写了一个ticket发了过去。

![whats]({{ '/styles/images/post/16122202.png' | prepend: site.baseurl  }})


好在歪果仁的效率还是飞速的，三分钟就给我解决了。帐号到手，着手准备搭梯子～

# 修改密码

略过。。。收到RamNode的邮件后，根据提示改密码就好了 

![whats]({{ '/styles/images/post/16122203.png' | prepend: site.baseurl  }})

# 搭建vpn

  第一想法是“RamNode 搭建 VPN”，于是搜了一堆资料，找了几个靠谱的照猫画虎的描了一遍，主要都是通过PPTP方式实现。 具体配置我就不贴了，因为我最后不是通过这条路走通的，没有参考性，自行百度吧。
这些是我参考的文章 
http://www.tuicool.com/articles/YjYnim 
http://blog.bossma.cn/server/linux-vps-debian-vpn-server-pptp/ 
按着方子走了一遍，自己又调理了一遍，发现还是没治好这货，依旧不通。。折腾了一天还是留坑。 有种感觉这种方法已经很不靠谱了，但秉承“through it”的精神，我还是决定坚持下。这时候看日志很关键啦～

依旧有问题，这时候看日志

Dec 19 00:34:24 vps pptpd[862]: CTRL: Starting call (launching pppd, opening GRE)
Dec 19 00:34:24 vps pppd[863]: Plugin /usr/lib/pptpd/pptpd-logwtmp.so loaded.
Dec 19 00:34:24 vps pppd[863]: Couldn't open the /dev/ppp device: Permission denied
Dec 19 00:34:29 vps pppd[863]: Sorry - this system lacks PPP kernel support
Dec 19 00:34:29 vps pptpd[862]: GRE: read(fd=6,buffer=7f143b8954a0,len=8196) from PTY failed: status = -1 error = Input/output error, usually caused by unexpected termination of pppd, check option syntax and pppd logs
Dec 19 00:34:29 vps pptpd[862]: CTRL: PTY read or GRE write failed (pty,gre)=(6,7)
Dec 19 00:34:29 vps pptpd[862]: CTRL: Reaping child PPP[863]
Dec 19 00:34:29 vps pptpd[862]: CTRL: Client 59.61.92.115 control connection finished
root@vps:~# pppd
Couldn't open the /dev/ppp device: Permission denied
modprobe: ERROR: ../libkmod/libkmod.c:507 kmod_lookup_alias_from_builtin_file() could not open builtin file '/lib/modules/2.6.32-042stab113.11/modules.builtin.bin'
modprobe: FATAL: Module ppp_generic not found.
pppd: Sorry - this system lacks PPP kernel support
关键就是Module ppp_generic not found 以及 his system lacks PPP kernel support，网上查了下解决方法，试了还是不行，但是知道跟系统内核多少有点关系，这可能跟我选的是openVZ类型有关。 
在山穷水尽之时，求助高人，那人只说了句“ppp被长城防火墙破解了，你需要更高级的工具，比如shadowsocks”。 醍醐灌顶！！立马抛弃pptp转投shadowsocks。

# 使用shadowsocks欢墙

这个就简单多了，贴个连接https://xulog.com/2014/07/31/openvz-vps-build-shadowsocks-and-optimization.html 看了你就懂了。 
看完了吧，是不是so easy~

又能谷歌了，真的是哈皮 :) 

![whats]({{ '/styles/images/post/16122204.png' | prepend: site.baseurl  }})



