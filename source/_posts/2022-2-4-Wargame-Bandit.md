---
layout:	post
title:  "Wargame Bandit"
date: 2022-2-4 11:00
author: "Adam"
header-img: "img/post-bg-cyberwar.jpg"
catalog: true
tags:
   - Security learning
   - Wargame
   - OverTheWire
---

>  Bandit是over The Wire中最基础的一个游戏，旨在帮助游戏者熟悉linux命令行操作。

前面的几个level较为简单，所以这里只是些重点

- SSH 连接：`-p`指定端口号，格式为`ssh name@host`
- Open dash-filename: `cat ./-`
- Space in filename: `cat "space in filename"`
- Find hidden file: `cat . (tab)`

#### Level 6 → 7

> The password for the next level is stored **somewhere on the server** and has all of the following properties:

- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

- 在根目录使用find会有很多权限报警，所以使用`2>/dev/null`,原因：*2>/dev/null redirects error messages to null so that they do not show on stdout.*



`find / -user bandit7 -group bandit6 -size 33c 2>/dev/null`

得到如下结果：![image-20220204155710620](https://tva1.sinaimg.cn/large/008i3skNgy1gz1x5ts6g3j314203ggm5.jpg)

Key: HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

#### Level 7 → 8

> The password for the next level is stored in the file **data.txt** next to the word **millionth**

这一题是要找在millionth这个单词，用vim打开后可以发现文档非常大，在normal模式输入命令`/millionth`回车后即可找到答案。

Key: cvX2JJa4CFALtqS87jk27qwqGhBM9plV

方法2: `cat data.txt | grep millionth`![image-20220204160404915](/Users/tingyi/Library/Application Support/typora-user-images/image-20220204160404915.png)

#### Level 8 > 9

> The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once

这一题的任务是找到文件中只出现一次的行。自然地想到先进行排序，在统计频次，命令如下

`sort data.txt | uniq -c` 

可以顺利找到频数为1的Key：UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR， 而其他行的频数都为10。或者也可以使用`uniq -u`直接获取。![image-20220204161738775](https://tva1.sinaimg.cn/large/008i3skNgy1gz1xr3jo3pj30nw02e3ys.jpg)

#### Level 9 > 10

> The password for the next level is stored in the file **data.txt** in one of the few human-readable strings, preceded by several ‘=’ characters.

找到文件中在一些‘=’之后的可读部分。尝试使用`cat data.txt | grep ===`失败，使用vim打开后查找，输入`\===`回车后按`n`选择下一个，几次后找到key:truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk

查找资料后，发现第一次尝试不成功的原因在文件中含有二进制数据所以不可读。首先将其转为字符串格式，再选取开头为`=`的行即可。命令如下：

`cat data.txt | strings | grep =`![image-20220204163310896](https://tva1.sinaimg.cn/large/008i3skNgy1gz1y7dooz6j30s80e4wfr.jpg)

####  Level 10 > 11

> The password for the next level is stored in the file **data.txt**, which contains base64 encoded data

文件中的内容是使用base64进行编码的，![image-20220204164153180](https://tva1.sinaimg.cn/large/008i3skNgy1gz1ygauvy2j312a02eq3g.jpg)

所以需要进行解码。输入命令`base64 -d data.txt`即可得到key：IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR![image-20220204164313867](https://tva1.sinaimg.cn/large/008i3skNgy1gz1yhpa7p3j30rm02odg9.jpg)

#### Level 11 > 12

> The password for the next level is stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

文件中的所有字母都经过了ROT13加密，解密方法为$m = c+(26-13) \rm\, mod\,  26$。使用`tr`功能可以转换或删除字符。用法为`tr [SET1][SET2]`其中SET1中的每一个字符都会被替换到SET2，所以可以得到key：5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu![image-20220204170007426](https://tva1.sinaimg.cn/large/008i3skNgy1gz1yz9u4tfj30xy02imxn.jpg)

#### Level 12 > 13

> The password for the next level is stored in the file **data.txt**, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

在完成创建目录和复制操作后，使用`xxd -r`将hexdump数据转化为binary数据。再使用`file`命令，得到此文件为gzip压缩文件，解压缩后重复以上步骤直到得到最终的key：8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL

![image-20220204173953493](https://tva1.sinaimg.cn/large/008i3skNgy1gz204oabraj30rk02mq3d.jpg)

#### Level 13 > 14

> The password for the next level is stored in **/etc/bandit_pass/bandit14 and can only be read by user bandit14**. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. **Note:** **localhost** is a hostname that refers to the machine you are working on

> 参考资料：https://help.ubuntu.com/community/SSH/OpenSSH/Keys

登录后，有如下的ssh私钥<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gz20b185wgj30v20u0aks.jpg" alt="image-20220204174601218" style="zoom:50%;" />

使用`ssh`的身份验证登录模式即可，输入以下命令即可登录bandit14

`ssh bandit14@localhost -i ./sshkey.private`

#### Level 14 > 15

> The password for the next level can be retrieved by submitting the password of the current level to **port 30000 on localhost**.

首先我们需要找到本题的密码，位置如上一题所说。Key: 4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e。而向特定端口提交数据需呀用到`natcat`工具，命令如下：

`nc localhost 30000 4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e`

得到key：BfMYroe26WYalil77FoDi9qh59eK5xNr

#### Level 15 > 16

> The password for the next level can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL encryption.
>
> **Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -ign_eof and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…**

本题同样是发送消息并获取回复，但不同是要使用SSL加密。输入以下命令：

`openssl s_client -connect localhost:30001`

得到key：cluFn7wTiGryunymYOu4RcffSxQluehd![image-20220204181319532](https://tva1.sinaimg.cn/large/008i3skNgy1gz213g3sd7j30ue0cwjso.jpg)

#### Level 16 > 17

> The credentials for the next level can be retrieved by submitting the password of the current level to **a port on localhost in the range 31000 to 32000**. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

本题首先要进行端口扫描，判断在31000到32000之间有哪些端口正在监听。输入命令`nmap -p 31000-32000 localhost`得到共有5个端口开放。

` *nmap -v -A -T4 -p 31000-32000 localhost*` 可以得到更完整的结果，`-A`用来检测OS和版本，脚本等， `-v`提高结果的可读性，`T4`为速度模板。

![image-20220204182600451](https://tva1.sinaimg.cn/large/008i3skNgy1gz21gnwmrzj30zc0ggwh6.jpg)

接下来就是分别确定这些端口中那些使用ssl。输入命令：`openssl s_client -connect localhost:31790`得到如下的私钥：<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gz21qe2vswj30u00u2wo3.jpg" alt="image-20220204183522925"  />

在/tmp/下新建一个文件夹保存此私钥，并尝试使用ssh登录，结果为此私钥权限太低，应该改为只有我可以读取。修改其权限：![image-20220204185010304](https://tva1.sinaimg.cn/large/008i3skNgy1gz225svgoqj313y090q55.jpg)

此处有一个小技巧，使用`chmod`命令时，三位数字分别代表user, group和all，写，读，执行的权限分别为4，2，1.进行组合就可以方便地修改权限了。所以这里我们输入命令`chmod 600 pub.key`，可以看到，此时只有user有读写权限。![image-20220204185439412](https://tva1.sinaimg.cn/large/008i3skNgy1gz22agrrcoj30t604wmxv.jpg)

再次尝试使用ssh登录，竟然又没有登录进去，这次又提示需要输入passphrase。查阅一些资料后发现此处应该是服务器的问题，应该可以正常登录，所以我找到了进入下一题的key：xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn

#### Level 17 > 18

>  There are 2 files in the homedirectory: **passwords.old and passwords.new**. The password for the next level is in **passwords.new** and is the only line that has been changed between **passwords.old and passwords.new**

这一题是要找到passwords.old和password.new之间不同的一行。通过查看`diff`命令的手册，找到适合的选项为`--supress-common-lines` 意为不输出相同的行，所以输入以下命令即可得到key：kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd

`diff --suppress-common-lines passwords.old passwords.new`![image-20220205190549050](https://tva1.sinaimg.cn/large/008i3skNgy1gz388f77m3j313m070gmj.jpg)

#### Level 18 > 19

> The password for the next level is stored in a file **readme** in the homedirectory. Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH.

由于`.bashrc`被修改，导致登录失败并出现以下提示：

![image-20220205190842229](https://tva1.sinaimg.cn/large/008i3skNgy1gz38bd3hp6j30sa056dg3.jpg)

经过一连串的尝试后找到避免在登录时source .bashrc的方法为使用`-T`参数，意为关闭伪终端分配，意思是在建立连接时不分配shell，也就不运行.bashrc。最终得到key：IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x

![image-20220205192953748](https://tva1.sinaimg.cn/large/008i3skNgy1gz38xfzisoj30qs04kjrn.jpg)

#### Level 19 > 20

> To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

本题要求我们运行目录下的`bandit0-do`程序，此程序的作用是以其他用户的身份运行命令，而下一题的key只有bandit20可以读取，所以想到可以用此程序来打开key文件，命令如下：

`./bandit20-do cat /etc/bandit_pass/bandit20`

可以得到key：GbKksEFF4yrVs6il55v6gwY5aVje5f0j![image-20220205194140228](https://tva1.sinaimg.cn/large/008i3skNgy1gz399oxcznj30zu02kjru.jpg)

#### Level 20 > 21

> There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).
>
> **NOTE:** Try connecting to your own network daemon to see if it works as you think

这一题是需要我们将这一题的key通过给的程序发送，来得到下一题的key，想了半天，尝试用`nc -l 5000 &`新建一个端口后台监听但是没有成功。找了半天才发现还有一个提示：NOTE: To beat this level, you need to login twice: once to run the setuid command, and once to start a network daemon to which the setuid will connect. 所以解决方法为，在一个终端中新建监听`nc -l 1234 < /etc/bandit_pass/bandit20`，在另一个终端中输入`./suconnect 1234`即可获取key：gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr

#### Level 21 > 22

> A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

这一题是让我们了解一下基于时间的任务管理工具`cron`的应用，打开上述目录后，可以看到下一题的cronjob，打开后发现指向一个脚本文件，打开此脚本文件可以看到其将下一关的key写入了tmp文件夹中，打开这个文件即可获得key：t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv![image-20220206213625968](https://tva1.sinaimg.cn/large/008i3skNgy1gz4i7gf5cfj312y062jsr.jpg)![image-20220206214057342](https://tva1.sinaimg.cn/large/008i3skNgy1gz4ic38mtqj313o02kt96.jpg)

#### Level 22 > 23

> A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.
>
> **NOTE:** Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.

和上题类似，首先来到`/etc/cron.d`中查看下一题的任务，然后查看其具体脚本。

![image-20220206214306874](https://tva1.sinaimg.cn/large/008i3skNgy1gz4iecd6msj30z803m757.jpg)

![image-20220206214358164](https://tva1.sinaimg.cn/large/008i3skNgy1gz4if8gdvmj313a0aodhi.jpg)

通过脚本我们可以看到脚本新建了两个变量，`myname=bandit23`,`mytarget`是将一个字符串通过`md5`加密后在剪掉一些内容，最终下一题的key存入了最终结果的文件夹，我们也做同样的操作后得到了最终key：jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n

![image-20220206214951138](https://tva1.sinaimg.cn/large/008i3skNgy1gz4ilcl7goj313e060wfx.jpg)

#### Level 23 > 24

> A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.
>
> **NOTE:** This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!
>
> **NOTE 2:** Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

这一题不再是使用写好的脚本，而是自己写脚本，我们也要从厨子做菜变成看兵法了。第一步同样是打开原来的脚本，可以看到首先脚本会进入一个目录，然后执行目录下的所有脚本之后将其删除。所以我们要进入目录然后写一个脚本，将下一题的密码提取出来。

![image-20220206220011968](https://tva1.sinaimg.cn/large/008i3skNgy1gz4iw4b8zaj310a0mojtl.jpg)

使用vim编写以下脚本；注意需要修改权限让所有人都可以运行`chmod 777`，然后等待一分钟。

![image-20220206220843357](https://tva1.sinaimg.cn/large/008i3skNgy1gz4j50fwozj312w04adg3.jpg)

最终得到key：UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ

![image-20220206221555218](https://tva1.sinaimg.cn/large/008i3skNgy1gz4jcinl01j30ve05wwg1.jpg)

#### Level 24 > 25

> A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.

本题需要暴力破解一个四位的pin，题意应该就是想让我们写一个脚本来破解。具体想法为将每一个可能的密码组合都写入到一个文件中，然后将此文件作为输入传到监听端口，并将结果存入另一个文件中。最后在此文件中找到key。

![image-20220206230122174](https://tva1.sinaimg.cn/large/008i3skNgy1gz4knro5uxj310o0723z8.jpg)

将输出存入`adam.txt`

![image-20220206225832587](https://tva1.sinaimg.cn/large/008i3skNgy1gz4kkufvdjj313i01qglx.jpg)

由于输出中有很多错误信息，所以用`sort`功能找到唯一不同的一行，key：uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG

#### Level 25 > 26

> Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not **/bin/bash**, but something else. Find out what it is, how it works and how to break out of it.

登录后有一个ssh私钥，使用其登录后立刻又退出了。通过查阅资料，知道shell在/etc/passwd文件中指定，所以查看此文件中bandit26相关部分，发现使用的是/usr/bin/showtest而不是bash。

![image-20220209214717760](https://tva1.sinaimg.cn/large/008i3skNgy1gz7zdmljbnj313m02g3z4.jpg)

查看此文件，发现是一个脚本，作用是调用了more显示了一个文件，然后就退出了。之前没看到more是因为窗口太大了，缩小后即可看到。所以我们就要利用more来执行命令获取key。

![image-20220209215222900](https://tva1.sinaimg.cn/large/008i3skNgy1gz7ziwnunpj30qg08a3yz.jpg)

![image-20220209215228092](https://tva1.sinaimg.cn/large/008i3skNgy1gz7zizv9h7j313c08gjrt.jpg)

这里不知道该怎么办了，查阅了一些资料，知道more可以执行命令，有以下几种方式，一是直接!command，尝试输入!/bin/sh或!/bin/bash都没有成功。二是输入v进入vim模式再!command，但同样没有成功。最后使用vim模式下的:e file功能成功获取到了key：5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z

![image-20220209215742207](https://tva1.sinaimg.cn/large/008i3skNgy1gz7zog5m73j313w07qmyj.jpg)

#### Level 26 > 27

> Good job getting a shell! Now hurry and grab the password for bandit27!

再次登录，仍然是一登录就退出。同样的进入vim模式，使用:set命令将shell设置为/bin/sh，再:sh即可进入。

![image-20220209221818404](https://tva1.sinaimg.cn/large/008i3skNgy1gz809wma7bj313u07qjse.jpg)

看到进入shell后有一个文件，和之前一样是以bandit27的权限运行命令，所以直接输出key：3ba3118a22e93127a4ed485be72ef5ea

![image-20220209222041951](https://tva1.sinaimg.cn/large/008i3skNgy1gz80celbyfj30u205w0tj.jpg)

#### Level 27 > 28

> There is a git repository at `ssh://bandit27-git@localhost/home/bandit27-git/repo`. The password for the user `bandit27-git` is the same as for the user `bandit27`.
>
> Clone the repository and find the password for the next level.

这一题要求我们克隆一个repo，我们将它克隆到/tmp目录下即可，克隆时需要输入这一题的key。

![image-20220209222336783](https://tva1.sinaimg.cn/large/008i3skNgy1gz80feq3nvj311q05yab7.jpg)

得到下一题的key：0ef186ac70e04ea33b4c1853d2526fa2

#### Level 28 > 29

> There is a git repository at `ssh://bandit28-git@localhost/home/bandit28-git/repo`. The password for the user `bandit28-git` is the same as for the user `bandit28`.

> Clone the repository and find the password for the next level.

和上一题相同，先cd到/tmp，克隆时发现已经存在repo文件夹，只需要在之后重新起一个名字即可。打开文件，里面却没有有用的信息。

![image-20220209223314206](https://tva1.sinaimg.cn/large/008i3skNgy1gz80pf8g7oj30o009umy2.jpg)

既然是git仓库，我们应该可以通过查看log获取到一些信息。输入git log果然可以看到添加了信息。使用git show命令即可看到password修改前的内容：key：bbc96594b4e001778eee9975372716b2

![image-20220209224013179](https://tva1.sinaimg.cn/large/008i3skNgy1gz80woisspj30sa0kc77k.jpg)

#### Level 29 > 30

> There is a git repository at `ssh://bandit29-git@localhost/home/bandit29-git/repo`. The password for the user `bandit29-git` is the same as for the user `bandit29`.
>
> Clone the repository and find the password for the next level.

相同的步骤进行完后，得到如下信息：意思是说从来就没输入过password

![image-20220209224448996](https://tva1.sinaimg.cn/large/008i3skNgy1gz811hpaitj30qg09qdgt.jpg)

不管是log还是show都没有内容，这时就应该想到是否存在分支？输入git branch -a显示所有分支，发现一共有四个分支。使用git checkout切换到dev分支，使用git show找到key：5b90576bedb2cc04c86a9e924ce42faf![image-20220209225317103](https://tva1.sinaimg.cn/large/008i3skNgy1gz81abouguj311q0kytcj.jpg)

### Level 30 > 31

> There is a git repository at `ssh://bandit30-git@localhost/home/bandit30-git/repo`. The password for the user `bandit30-git` is the same as for the user `bandit30`.
>
> Clone the repository and find the password for the next level.

同样的工作完成后，这次的提示更搞了:laughing:

![image-20220209225616118](https://tva1.sinaimg.cn/large/008i3skNgy1gz81deiq02j30oo02iaab.jpg)

试过之前的所有命令后仍然什么都没有发现。只能在git的命令里再找找可能能给出更多信息的。最终找到以下命令：

`git show-ref`用来查看所有可用引用。

![image-20220209230545716](https://tva1.sinaimg.cn/large/008i3skNgy1gz81n9h6ycj313u05w76b.jpg)

这里看到一个secret，很可能就是答案所在位置，输入git show ref得到key：47e603bb428404d265f59c42920d81e5![image-20220209230750596](https://tva1.sinaimg.cn/large/008i3skNgy1gz81pfzowwj314203kmxv.jpg)

#### Level 31 > 32

> There is a git repository at `ssh://bandit31-git@localhost/home/bandit31-git/repo`. The password for the user `bandit31-git` is the same as for the user `bandit31`.

这一次我们需要向master push一个文件。

![image-20220209231040189](https://tva1.sinaimg.cn/large/008i3skNgy1gz81se89sfj313u098t9p.jpg)

编辑好这个文件后commit，即可得到key：56a9bf19c63d650ce78e6ec0354ee45e

![image-20220209231308433](https://tva1.sinaimg.cn/large/008i3skNgy1gz81uyoyhgj313o0bmq4s.jpg)

#### Level 32 > 33

> After all this `git` stuff its time for another escape. Good luck!

看到题目舒了一口气，终于不用git了哈哈哈。但是登录进来又傻眼了，所有命令都会被变成大写导致无法执行。输入$0执行当前shell(zsh没有)，即可退出UPPERCASE SEHLL，老地方找到key：c9c3199ddf4121b10cf581a98d51caee

![image-20220209233325973](https://tva1.sinaimg.cn/large/008i3skNgy1gz82g2rj6wj313y06ywez.jpg)

#### Level 33 > 34

> **At this moment, level 34 does not exist yet.**

完结撒花啦，能够完成真的非常非常地开心，虽然有些题确实不会所以参考了其他人的想法，但是在整个过程中真的学到了很多，不管是命令，脚本都有了从零到一的突破。接下来整理回顾一下就准备开启下一个wargame了，加油，奥利给！

![image-20220209233737315](https://tva1.sinaimg.cn/large/008i3skNgy1gz82kezszdj313y0ea770.jpg)
