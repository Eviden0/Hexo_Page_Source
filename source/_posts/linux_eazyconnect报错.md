---
title: Linux_eazyconnect使用
date: '2024-4-19 22:18'
categories: 新手村的操作
tags: Linux
description: Linux下安装Eazyconnect
abbrlink: dc7789b0
---


参考了许多网上的文章，发现都无法很好彻底的指导小白进行解决问题。这里完整的记录下如何解决的方案： 首先是下载EasyConnect.deb文件

![Screenshot from 2022-07-17 14-37-58.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/246f1e829e974c70a9b57a92b7ddc10e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

然后使用dpkg -i命令进行安装：

```css
css
复制代码sudo dpkg -i [name of installation package]
```

安装完成后进行启动：这个时候大概率是无法启动成功的。小弟看了下报错：

![Screenshot from 2022-07-17 08-44-35.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d68f493ffbb642a08f3e1718f0bc105e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

不过大概率是看不懂的，因为我看来看去也就是DMR0异常中断，这玩意就是非正常的缺页中断，也就是无法找到某个物理地址下的文件。小弟功力有限，只能继续在百度上搜索解决方案，下面这篇文章讲的还算不错，但还是有些关键的地方一笔代过了，其实主要的原因就是libpango这个包在Ubuntu22.04（或者18.04等更高的版本中）版本太高了，需要进行降版本处理。 [www.cnblogs.com/zj420255586…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fzj420255586%2Fp%2F14211474.html)

下面我具体将下处理步骤： 1.查看涉及到需要降级的文件

1.1 先查看EasyConnect的安装位置

由于我是通过dpkg命令来安装的，因此这里使用如下命令查看（显示于软件包关联的文件的命令），这个命令的主要作用就是 “查看软件安装到什么地方”

```
复制代码dpkg -L easyconnect
```

小弟这边执行后，显示结果如下

```bash
bash复制代码Vostro-5471:~$ dpkg -L easyconnect
/.
/etc
/etc/init
/etc/init/EasyMonitor.conf
/usr
/usr/lib
/usr/lib/systemd
/usr/lib/systemd/system
/usr/lib/systemd/system/EasyMonitor.service
/usr/share
/usr/share/pixmaps
/usr/share/pixmaps/EasyConnect.png
/usr/share/applications
/usr/share/applications/EasyConnect.desktop
/usr/share/sangfor
/usr/share/sangfor/EasyConnect
/usr/share/sangfor/EasyConnect/LICENSE
/usr/share/sangfor/EasyConnect/content_resources_200_percent.pak
/usr/share/sangfor/EasyConnect/version
/usr/share/sangfor/EasyConnect/LICENSES.chromium.html
/usr/share/sangfor/EasyConnect/resources
/usr/share/sangfor/EasyConnect/resources/electron.asar
/usr/share/sangfor/EasyConnect/resources/conf
/usr/share/sangfor/EasyConnect/resources/conf/need_hook_dns_server.ini
/usr/share/sangfor/EasyConnect/resources/conf/ConfModuleMap.xml
/usr/share/sangfor/EasyConnect/resources/conf/Version.xml
/usr/share/sangfor/EasyConnect/resources/conf/LogConf.xml
/usr/share/sangfor/EasyConnect/resources/conf/SurpportBrowser.xml
/usr/share/sangfor/EasyConnect/resources/conf/Module.xml
/usr/share/sangfor/EasyConnect/resources/default_app.asar
/usr/share/sangfor/EasyConnect/resources/shell
/usr/share/sangfor/EasyConnect/resources/shell/list_dns.sh
/usr/share/sangfor/EasyConnect/resources/shell/sslservice.sh
/usr/share/sangfor/EasyConnect/resources/shell/startrapp.sh
/usr/share/sangfor/EasyConnect/resources/shell/sslcheck.sh
```

EasyConnect的安装位置在 /usr/share/sangfor/EasyConnect

1.2查看涉及降级的文件

需要先进入到EasyConncet的安装目录 然后使用

```perl
perl
复制代码ldd EasyConnect | grep pango 
```

命令可以查看相关的依赖文件信息

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/062dff24164a4d9cbd9f590cef85acb3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

接下来需要作的就是去下载对应的低版本文件，去覆盖高依赖版本 注意有三个依赖包需要下载

```
复制代码libpangocairo-1.0.so.0
libpango-1.0.so.0
libpangoft2-1.0.so.0
```

2.下载对应依赖的低版本包

下载地址：[kr.archive.ubuntu.com/ubuntu/pool…](https://link.juejin.cn/?target=http%3A%2F%2Fkr.archive.ubuntu.com%2Fubuntu%2Fpool%2Fmain%2Fp%2Fpango1.0%2F) 请根据自己的CPU架构类型，选择具体的版本，小弟是AMD64所以需要下载以下三个文件

```
复制代码libpangocairo-1.0-0_1.40.14-1_amd64.deb
libpangoft2-1.0-0_1.40.14-1_amd64.deb
libpango-1.0-0_1.40.14-1_amd64.deb
```

下载完成之后对这三个文件进行解压缩，提取出 data.tra.xz -> usr -> lib -> x86_64-linux-gnu 下面的所有文件

3.覆盖EasyConnect安装目录下的pango依赖

将提取出来的文件，复制到EasyConnect的安装目录下，也就是在1.1步骤中的位置 使用cp 命令即可

```bash
bash
复制代码cp source /usr/share/sangfor/EasyConnect
```

最后在/usr/share/sangfor/EasyConnect下确认下是否覆盖完成

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/523cfb9810b24554b3a584705ecd6cc0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?) 发现所有的文件地址都指向了EasyConnet的安装位置 重启EasyConnect即可。

附录：

（1）ldd命令解释

[zhuanlan.zhihu.com/p/386776402](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F386776402)

（2）dpkg命令说明

[cloud.tencent.com/developer/a…](https://link.juejin.cn/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1484940) 该命令的常用参数如下表格所列

| -i   | 安装软件包             |
| ---- | ---------------------- |
| -r   | 删除软件包             |
| -l   | 显示已安装软件包列表   |
| -L   | 显示于软件包关联的文件 |
| -c   | 显示软件包内文件列表   |