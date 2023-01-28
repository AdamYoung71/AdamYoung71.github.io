---
layout:	post
title:  "XSS Attacks"
date: 2022-2-21 21:00
author: "Adam"
header-img: "img/post-bg-xss.jpg"
catalog: true
tags:
    - Security learning
    - XSS	
    - Client security
---

> 本文是我的XSS学习笔记
>
> https://portswigger.net/web-security/cross-site-scripting

## 什么是跨站脚本（Cross-site Scripting XSS)

Cross-site scripting (also known as XSS) is a web security vulnerability that allows an attacker to compromise the interactions that users have  with a vulnerable application. It allows an attacker to circumvent the  same origin policy, which is designed to segregate different websites  from each other. Cross-site scripting vulnerabilities normally allow an  attacker to masquerade as a victim user, to carry out any actions that  the user is able to perform, and to access any of the user's data. If  the victim user has privileged access within the application, then the  attacker might be able to gain full control over all of the  application's functionality and data.        

XSS是一种允许攻击者绕过同源策略，在渲染过程中发生了不在预期过程中的JavaScript代码执行。XSS通常被用于假扮受害者用户，从而进行只有用户才能进行的操作或获取用户数据如Cookie，如果此用户拥有高权限，攻击者甚至可能据此获得Web应用的全局控制权。

## XSS的工作原理是什么

XSS通过操控存在相关漏洞的网页使其将可以Javascript代码返回给受害者，当此代码在受害者的浏览器中运行时，攻击者就可以完全掌控受害者于应用的交互。

![XSS原理](https://portswigger.net/web-security/images/cross-site-scripting.svg)

## XSS验证

可以通过注入一个有效载荷，使自己的浏览器执行一些任意的JavaScript来确认大多数类型的XSS漏洞。长期以来，使用alert()函数是一种常见的做法，因为它很短，无害，而且当它被成功调用时很明显。事实上，你可以通过在模拟受害者的浏览器中调用alert()来解决我们大多数的XSS实验。

然而，如果你使用Chrome浏览器，会有一个小麻烦。从92版开始（2021年7月20日），跨源iframe被阻止调用alert()。由于这些被用来构建一些更高级的XSS攻击，你有时需要使用一个替代的PoC有效载荷。在这种情况下，我们推荐使用print()函数。如果你有兴趣了解更多关于这一变化以及为什么我们喜欢print()，请查看我们关于这一主题的[博文](https://portswigger.net/research/alert-is-dead-long-live-print)。

## 反射型XSS (Reflected XSS)

反射型XSS是最简单的一种，其过程为当Web应用收到一个HTTP请求的数据时，以不安全的方式处理这些数据。反射型XSS通常出现在搜索等功能中，需要被攻击者点击对应的链接才能触发，且受到XSS Auditor、NoScript等防御手段的影响较大。一个简单的例子：

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

由于Web应用不对message的内容进行处理，而是直接显示，所以就可以构造以下攻击：

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

>[Lab: Reflected XSS into HTML context with nothing encoded](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)

第一个Lab很简单，只要在搜索域将搜索关键词换成脚本即可。

![image-20220221223746391](https://tva1.sinaimg.cn/large/e6c9d24egy1gzlw9x3f1uj20tg0goq3h.jpg)

### 反射型XSS的影响

如果攻击者可以在受害者的浏览器上运行脚本，则说明攻击者可以完全掌控受害者的一切信息，包括：

- 在Web应用内以受害者的身份进行操作。
- 浏览所有受害者可以浏览的信息。
- 修改所有受害者可以修改的信息。
- 以受害者的身份于其他应用进行交互。

由于反射型XSS需要外部提交机制（需要受害者点击恶意链接），所以其影响相较于储存型XSS较小。

### 反射型XSS的上下文

现实中由于反射数据位置的不同，payload的种类和影响也会不同。同时网站在反射之前对提交数据的处理也会影响payload的选择。



### 如何寻找并测试反射型XSS漏洞

- **测试所有接入点**： 包括参数，URL请求和消息体的数据，URL文件目录，HTTP头（有些头不能实施XSS）
- **提交随机字母数字组合值：**对每一个接入点都提交一个不同的随机值并判断是否反射，此值应该尽量绕过尽可能多的检查，所以一般尽量短且只含有数字和字母的组合，同时又需要足够长使得意外匹配的可能性足够低，因此8个字符左右通常比较合适。
- **确定反射上下文：**例如在HTTP tags之间，在tag的属性中，在JS字符串中等等。
- **测试候选payload：**根据反射的上下文，对候选payload进行测试，如果反射没有改变payload则可以使用如Burp Repeater等工具测试不同payload下的回应。
- **测试其他payload：**如果反射改变了payload，则需要使用其他payload或方法进行测试。详情见这里[cross-site scripting contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- **在浏览器中进行攻击测试：**最后，当我们成功找到有效的payload后，在真实的浏览器环境中对其进行测试来确定JS代码是否真正执行。通常最简单的测试方法为`alert(document.domain)`。

### 反射型XSS的常见问题

1. 于储存型XSS的区别：当应用程序从HTTP请求中获取一些输入并以不安全的方式将该输入嵌入到即时响应中时，就会产生反射式XSS。对于存储的XSS，应用程序存储输入并以不安全的方式将其嵌入到以后的响应中。
2. 于Self-XSS的区别：Self-XSS涉及的应用行为与常规反映的XSS类似，但它不能通过精心设计的URL或跨域请求以正常方式触发。相反，只有当受害者自己从他们的浏览器中提交XSS有效载荷时才会触发该漏洞。提供自我XSS攻击通常涉及社会工程，让受害者在他们的浏览器中粘贴一些攻击者提供的输入。因此，它通常被认为是一个蹩脚的、低影响的问题。

## 存储型XSS（Stored cross-site scripting)

存储型XSS，又称永久XSS，意思是web应用从不信任的来源获取数据后以不安全的方式将其包含在应用之后的HTTP应答中。这样的数据可能来自用户的提交（如评论，用户昵称）等，这里是一个简单的存储型XSS漏洞，直接在网页上显示用户提交的数据。

`<p>Hello, this is my message!</p>`

此时，我们可以通过
