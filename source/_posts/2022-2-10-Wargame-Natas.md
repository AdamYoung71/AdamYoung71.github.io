---
layout:	post
title:  "Wargame Natas"
date: 2022-2-10 17:00
author: "Adam"
header-img: "img/post-bg-cyberwar.jpg"
catalog: true
tags:
   - Security learning
   - Wargame
   - OverTheWire
---

今天开始学习第二个wargame Natas。

Natas teaches the basics of serverside web-security.

Each level of natas consists of its own website located at **http://natasX.natas.labs.overthewire.org**, where X is the level number. There is **no SSH login**. To access a level, enter the username for that level (e.g. natas0 for level 0) and its password.

Each level has access to the password of the next level. Your job is to somehow obtain that next password and level up. **All passwords are also stored in /etc/natas_webpass/**. E.g. the password for natas5 is stored in the file /etc/natas_webpass/natas5 and only readable by natas4 and natas5.

Start here:

```
Username: natas0
Password: natas0
URL:      http://natas0.natas.labs.overthewire.org
```

这个游戏不再是ssh连接远程服务器来进行了，而是在网页上进行。打开网页看看是什么样先。

#### Level 0 > 1

![image-20220210171019481](https://tva1.sinaimg.cn/large/008i3skNgy1gz8wzsh0zcj312u0fggmj.jpg)

打开后是一个很简单的网站，写着密码就在这一页。习惯性用DevTool看一下，找到了在注释中的key：gtVrDuiDfck831PqWsLEZy5gyDz1clto

![image-20220210171200170](https://tva1.sinaimg.cn/large/008i3skNgy1gz8x1h6q7lj30t005swf6.jpg)

#### Level 1 > 2

这一关的提示为右键已经被禁止。点一下右键会出现以下提示：

![image-20220210171625511](https://tva1.sinaimg.cn/large/008i3skNgy1gz8x6374ijj30p007amxg.jpg)

这可难不倒我们，直接一个F12就可以轻松找到key：ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi

![image-20220210171717153](https://tva1.sinaimg.cn/large/008i3skNgy1gz8x6zl9y2j30tg06mdgn.jpg)

#### Level 2 > 3

这一关的说明为：这个页面上什么都没有。查看html代码后发现key确实不在网页中了。但是可以看到在说明旁边有一个图片，只不过渲染大小为1*1像素，所以正常情况下看不到。

![image-20220210172405681](https://tva1.sinaimg.cn/large/008i3skNgy1gz8xe2kz3pj30qi0ceta8.jpg)

观察此图片的来源，可以看到此网站还有files网页，尝试进入后找到一个`users.txt`，内容如下。

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

Key: sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14

#### Level 3 > 4

这一题的提示仍然是网页上什么都没有，但是有一个提示。

`<!-- No more information leaks!! Not even Google will find it this time... -->`

不能用Google搜索到其实是一个附加信息。现在的网站大多有相关是否允许爬虫的设置，通常存放在robots.txt中，所以我们找一下有没有这个文件。输入后得到以下信息。

```
User-agent: *
Disallow: /s3cr3t/
```

查阅[相关文档](https://robot-txt.com/sem-glossary/a-guide-to-robot-txt-files/?gclid=Cj0KCQiAjJOQBhCkARIsAEKMtO2POL91mPRQJiY7pzygDjt58ul4z7XaLzXk4yN2L83-LlisgdRg-GsaAgsrEALw_wcB)，上面两行的意思是屏蔽除了第二行子文件夹之外的爬虫，所以我们打开此文件夹。

`http://natas3.natas.labs.overthewire.org//s3cr3t/`

得到user.txt，同时得到key：Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

#### Level 4 > 5

这一题是说只有从指定网站的访问才可以，页面上有一个刷新的按钮，按一下就会检测是从哪里访问的，突破口应该就在这里。使用Burp截取请求即可发现有一个Referer表名从哪里来的请求，将其改为要求的网址即可。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gz8yh7qvv1j30mc0g6q5n.jpg" alt="image-20220210180141294" style="zoom:50%;" />

得到key：iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq

#### Level 5 > 6

打开网页后提示拒绝访问，未登录。既然是登录问题，就可以抓请求看一下cookie设置。用Burp看到

`Cookie: loggedin=0`

将其值设置为1后发送，即可得到key：aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1

#### Level 6 > 7

这一题需要我们输入一个秘密来获得key。旁边还有一个链接查看源码，点开看一下发现是网页源码，其中就包括了判断秘密的逻辑。

```php
include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
```

可以看到这里include了一个文件，里面应该就是秘密的内容，访问这个文件得到`$secret = "FOEIUWGHFEEUHOFUOIU";`，输入即可得到key：7z3hEENjQtflzgnT29q7wAvMNfZdh0i9

#### Level 7 > 8

进入网页后可以看到源码中有一条提示：hint: password for webuser natas8 is in /etc/natas_webpass/natas8。网站上有两个链接，分别是主页和关于页，其url为?page=home，利用此形式打开key文件。直接将home替换为上面的地址后输出了错误信息：

**Warning**: include(/etc/natas_pass/natas8): failed to open stream: Permission denied in **/var/www/natas/natas7/index.php** on line **21**

说明需要先回到根目录才行。在地址前增加5个../后，得到key：DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe

#### Level 8 > 9

这一题又是需要输入秘密来获得key，同样也有查看源代码的选项。

```php
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

这一段代码是说，编码后的秘密已经给出，我们的输入会先base64加密后反向最后转为16进制。我们只需要逆序执行这些操作即可。

```python
import base64

secret = "3d3d516343746d4d6d6c315669563362"
secret = bytes.fromhex(secret)
secret = secret[::-1]
secret = base64.decodebytes(secret)

print(secret)

# Result = oubWYf2kBq
```

最终获得key：W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl