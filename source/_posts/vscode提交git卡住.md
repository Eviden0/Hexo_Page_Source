---
title: vscode 提交git一直卡着转圈圈
abbrlink: 38f255c6
---

**打开设置
把use Editor As commit input的勾选框去掉，ok，重新提交就可以了 。在commit却不添加任何消息时，勾选了这个会默认生成一个文件来替代消息并提交，而服务器无法接受这样的消息**
![](https://s1.vika.cn/space/2024/03/28/d6d3efc6e6ae469194bb020bd3848e80)

取消勾选后重启
![](https://s1.vika.cn/space/2024/03/28/c1be662048a4472aa0cf7e8fec7b5cdb)
成功!