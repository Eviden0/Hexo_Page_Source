---
title: Termux安卓机实现内网穿透
description: 浅浅手动造一台服务器把~~
categories: 骚操作
tags: Linux
abbrlink: 91facb7b
---
> [内网穿透教程1](https://www.cpolar.com/blog/public-ssh-remote-connection-to-termux-using-android-termux-cpolar-internal-network-penetration)

> [内网穿透教程2](https://www.linuxhe.com/1380.html)

[cpolar实现对termux内网穿透](https://blog.csdn.net/xianyun_0355/article/details/134207599)

`ssh u0_a113`

需要注意的问题：

1. Termux的默认ssh端口为：`9022`，而非`22`

2. 如果你已root,那么可以通过安装一个linux,且可以使用超级管理员权限,更爽,但是没root也不妨碍正常使用就是了

3. 控制手机杀后台的问题,一定一定不能把后台杀了

4. ssh内网穿透后尽量不要将重要的信息上传到Termux中,做做小测试平台还行,安全意识还得有

5. 渗透工具安装都基本被Termux官方杀了,因此别想了~很难装,我装个kali都费劲

   