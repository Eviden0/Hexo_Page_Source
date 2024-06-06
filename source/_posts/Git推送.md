---
title: Git推送远程仓库
date: '2024-4-06 17:18'
categories: 新手村的操作
tags: Git
description: Git推送远程仓库(仓库已含文件)
abbrlink: f35c27cf
---

> ### **git命令行推送到远端仓库**

==命令行模式,启动!==

```shell
#进入需要推送的目录
初始化: git init
#把文件添加到版本库中 
git add +文件名
如推送整个文件夹:则推送所有文件 git add .
#用命令 git commit "提交说明" 告诉Git，把文件提交到仓库。引号内为提交说明
git commit  -m "XXX说明"
# 关联到远程库 git remote add origin 你的远程库地址 
 git remote add origin https://:..
#此处有个坑,`https`...url每次推送到Github时需要输入密码,而git现在不支持仅密码验证,你又得去输一个token,而且token用起来很麻烦
#因此在此说明,除非是你在服务器上的仓库,不然不推荐这样做!(服务器通过ssh密码一遍过就行)
#所以怎样推送到github仓库呢?
#我们remote设置连接url时,设置为:
git remote add origin git@github.com:Eviden0/Webpage.git
#选择github ssh下面那个url即可别选`https`!请看下面图片实例

#git pull 获取远程库与本地同步合并(如果远程库不为空必须做这一步，否则后面的提交会失败) 
git pull --rebase origin master(同步合并的分支) (master 或者main这里自己选)
git push -u origin master(推送到远程的分支)

```

==OK,一切搞定!舒舒服服==*到这里你就好好享受git这个工具带来的便捷吧!*

![](https://s1.vika.cn/space/2024/04/06/9eab35437e5a4451b195df9feffbdb72)

```shell
命令需要三个参数，github用户名(jack),第一步生成的token（12345）,仓库名称（exampleRepo）
根据上述参数的例子，运行以下代码
curl -u jack:12345 https://api.github.com/user/repos -d '{ "name": "exampleRepo" }'
```





==坑:==被推的仓库已经包含文件时

此时需要同步以下本地和远端仓库的文件

==坑:==

[免密推送](https://blog.csdn.net/hhgggggg/article/details/77853665)

​	



