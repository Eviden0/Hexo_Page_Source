---
title: 对sqlmap时间盲注的研究
date: '2024-9-9 8:03'
description: '对sqlmap时间盲注的研究,意外之喜!'
abbrlink: 9a87762d
---

> **最近在猛刷sql注入的题,大部分类型的题目已经刷完,时间盲注这块还没深入研究,自然也就对sqlmap的检测思路进行学习学习!**

[#参考1](https://www.freebuf.com/sectool/222967.html)

[#参考2](https://mp.weixin.qq.com/s/nCvrk6NEb_lDv7MXdXn3vA)

[sqlmap代码示例](https://www.cnblogs.com/hongfei/p/sqlmap-time-based-blind.html)

[sqlmap参考资料汇总](https://blog.csdn.net/q20010619/article/details/122243790)

实际案子的时候遇到了一个注入，过狗可以使用`sqlmap`，但是是基于时间的注入和限制频率需要使用`--delay`参数，本来就是延时再加上`--delay`等的心力憔悴。所有有了下面介绍使用`sqlmap`利用`DNS`进行`oob(out of band)`注入，快速出数据。一般情况下仅适用于`windows`平台

`对mysql版本有关系,且需要开启文件权限!`

