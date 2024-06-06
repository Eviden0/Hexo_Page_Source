---
title: Linux下环境变量配置
date: '2024-4-19 22:18'
categories: 新手村的操作
tags: Linux
description: Linux下配置环境变量
abbrlink: 4be92c95
---

常用修改环境变量的两种方式：

1. 临时设置
   比如我们刚刚安装了golang，要把GOROOT加入到环境变量中：`export PATH=$PATH:/usr/lib/go-1.9`。如果原来环境变量是`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin`的话，执行过`export`命令后的环境变量就会变成：`/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/usr/lib/go-1.9`。所有我们执行`export PATH=$PATH:/usr/lib/go-1.9`相当于：

```
$PATH = "/usr/local/sbin:/usr......"
$GOROOT = "/usr/lib/go-1.9"
$PATH = $PATH + $GOROOT
```

如何用命令查看修改后的环境变量：
`echo $PATH` 或者 `env`

1. 永久性设置
   找到`profile`文件，然后编辑它：

```
vi /etc/profile
#添加以下内容
export GOROOT=/usr/lib/go-1.9
export GOBIN=$GOROOT/bin
export GOAPTH=$GOROOT/src
export GO_WORK_PATH=/home/workspace/go #自定义的工作空间
#前面只是定义了变量，最后一句是关键
export PATH=$PATH:$GOROOT:$GOBIN:$GOPATH:$GO_WORK_PATH
```

要想立即生效就要执行：
`source /ect/profile`
重启`reboot`然后`echo $PATH`发现环境变量还是我们重启前设置的，并没有因为重启而失效。

**然而**，有时候，你会发现重启之后，环境变量和我们设置完全不一行。这个时候，就需要找到`.bashrc`文件：

```
vi ~/.bashrc
#发现最下面有以下几行：
export GOROOT=/home/lib
export GOBIN=$GOROOT/bin
export GOAPTH=$GOROOT/src
```

原来是因为`.bashrc`文件里的设置“覆盖”了我们在`profile`里的设置，好，我们现在注释（删除）这几行，重启。我们设置的环境变量终于生效了。

> 总之,bashrc里面的环境变量是一直会执行的
>
> 而我们在命令行中 export 执行的话只针对当前shell有用
>
> 关闭后重新开启又会执行默认配置文件里的配置选项