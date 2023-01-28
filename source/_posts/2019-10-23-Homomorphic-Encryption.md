---
layout: post
title:  "Homomorphic Encryption: principle and application"
date: 2019-10-22 20:30
author: "Adam"
header-img: "img/post-bg-encryption.jpg"
catalog: true
tags:
    - Cryptography
---

>转载自：同态加密的实现原理是什么？在实际中有何应用？ - 刘巍然-学酥的回答 - 知乎       https://www.zhihu.com/question/27645858/answer/37598506

# This is a test.

## 1. 概览：同态加密的概念

同态加密（Homomorphic Encryption）是很久以前密码学界就提出来的一个Open Problem。早在1978年，Ron Rivest, Leonard Adleman, 以及Michael L. Dertouzos就以银行为应用背景提出了这个概念[RAD78]。对，你没有看错，Ron Rivest和Leonard Adleman分别就是著名的RSA算法中的R和A。至于中间的S，Adi Shamir，现在仍然在为密码学贡献新的工作。

### **什么是同态加密？**

提出第一个构造出全同态加密（Fully Homomorphic Encryption）[Gen09]的Craig Gentry给出的直观定义最好：

> A way to delegate  of your data, without giving away access to it.

这是什么意思呢？一般的加密方案关注的都是**数据存储安全**。即，我要给其他人发个加密的东西，或者要在计算机或者其他服务器上存一个东西，我要对数据进行加密后在发送或者存储。没有密钥的用户，不可能从加密结果中得到有关原始数据的任何信息。只有拥有密钥的用户才能够正确解密，得到原始的内容。我们注意到，这个过程中**用户是不能对加密结果做任何操作的**，只能进行存储、传输。对加密结果做任何操作，都将会导致错误的解密，甚至解密失败。

同态加密方案最有趣的地方在于，其关注的是**数据处理安全**。同态加密提供了一种**对加密数据进行处理的功能**。也就是说，其他人可以对加密数据进行处理，但是处理过程不会泄露任何原始内容。同时，拥有密钥的用户对处理过的数据进行解密后，得到的正好是处理后的结果。

有点抽象？我们举个实际生活中的例子。有个叫Alice的用户买到了一大块金子，她想让工人把这块金子打造成一个项链。但是工人在打造的过程中有可能会偷金子啊，毕竟就是一克金子也值很多钱的说… 因此能不能有一种方法，让工人可以对金块进行加工（delegate processing of your data），但是不能得到任何金子（without giving away access to it）？当然有办法啦，Alice可以这么做：

- Alice将金子锁在一个密闭的盒子里面，这个盒子安装了一个手套。
- 工人可以带着这个手套，对盒子内部的金子进行处理。但是盒子是锁着的，所以工人不仅拿不到金块，连处理过程中掉下的任何金子都拿不到。
- 加工完成后。Alice拿回这个盒子，把锁打开，就得到了金子。

这个盒子的样子大概是这样的：

![HE-illustrator](https://adamyoung71.github.io/img/in-post/post-homo-encryp/HE-illustrator.jpg)

这里面的对应关系是：

- 盒子：加密算法
- 盒子上的锁：用户密钥
- 将金块放在盒子里面并且用锁锁上：将数据用同态加密方案进行加密
- 加工：应用同态特性，在无法取得数据的条件下直接对加密结果进行处理
- 开锁：对结果进行解密，直接得到处理后的结果

### 同态加密哪里能用？

这几年不是提了个云计算的概念嘛。同态加密几乎就是为云计算而量身打造的！我们考虑下面的情景：一个用户想要处理一个数据，但是他的计算机计算能力较弱。这个用户可以使用云计算的概念，让云来帮助他进行处理而得到结果。但是如果直接将数据交给云，无法保证安全性啊！于是，他可以使用同态加密，然后让云来对加密数据进行直接处理，并将处理结果返回给他。这样一来：

- 用户向云服务商付款，得到了处理的结果；
- 云服务商挣到了费用，并在不知道用户数据的前提下正确处理了数据；

这方法简直完美啊有没有？！但是，这么好的特性肯定会带来一些缺点。同态加密现在最需要解决的问题在于：效率。效率一词包含两个方面，一个是加密数据的处理速度，一个是这个加密方案的数据存储量。我们可以直观地想一想这个问题：

- 工人戴着手套加工金子，肯定没有直接加工来得快嘛~ 也就是说，隔着手套处理，精准度会变差（现有构造会有误差传递问题），加工的时间也会变得更长（密文的操作花费更长的时间），工人需要隔着操作，因此也需要更专业（会正确调用算法）。
- 金子放在盒子里面，为了操作，总得做一个稍微大一点的盒子吧，要不然手操作不开啊（存储空间问题）。里面也要放各种工具吧，什么电钻啦，锉刀啦，也需要空间吧？

### **这种加密方案真的有在研究？**

我举3个简单的例子：

1. 第一个构造出全同态加密方案的人是Gentry，这是他在Stanford攻读博士学位的研究成果。Gentry毕业后去哪里了呢？IBM。大家知道IBM可是一个云服务提供商啊！在IBM，Gentry和另一个密码学大牛Halevi继续进行同态加密及其相关的研究，并实现了一些同态加密方案。如果IBM真的做出了可以在实际使用的同态加密方案，那么其他云服务提供商就可以拜拜了啊！这游戏不用玩了啊，人家能在不知道数据内容得前提下处理数据啊，毕竟谁都不想把数据泄露给其他公司啊！
2. 国内的某个大公司（具体是哪个我就不透露了…）对这方面的研究非常感兴趣，我也和他们做了一次交流，并且初步达成了一定的研究大方向。要不怎么我现在也去弄这个头大的东西呢。要知道，国内的公司也没闲着，这是制高点，拿到了就是一家独大，而且是超级技术垄断，不公开源代码或者不了解内部构造的话想仿造都仿造不了啊…不过，这方面的研究说实话Gap确实大，入门起码要3个月的时间，还不一定做的出来…
3. 即使没有实现全同态加密，也可以得到其他一些很有趣的结论。而每一个结论都可能引发技术垄断。这些结论由于涉及到了一定的基础知识，我在后面中会进行介绍。

业界如何评价全同态加密的构造？

在此引用一个前辈的话：

> 如果未来真的做出了Practical Fully Homomorphic Encryption，那么Gentry一定可以得到图灵奖。

剩下的，我也就不用多说了吧… 

## 二、 同态加密的定义、安全性和简单实例

下面的内容，如果可以接受符号表述，具有一点密码学的知识，对抽象代数有一定的了解的话，可能体会的更深刻哦。

### 同态加密具体如何定义？

我们在云计算应用场景下面进行介绍：

![cloud-computing-scenario](https://adamyoung71.github.io/img/in-post/post-homo-encryp/cloud-computing-scenario.jpg)

Alice通过Cloud，以Homomorphic Encryption（以下简称HE）处理数据的整个处理过程大致是这样的：

1. Alice对数据进行加密。并把加密后的数据发送给Cloud；
2. Alice向Cloud提交数据的处理方法，这里用函数f来表示；
3. Cloud在函数f下对数据进行处理，并且将处理后的结果发送给Alice；
4. Alice对数据进行解密，得到结果。

据此，我们可以很直观的得到一个HE方案应该拥有的函数：

1. KeyGen函数：密钥生成函数。这个函数应该由Alice运行，用于产生加密数据Data所用的密钥Key。当然了，应该还有一些公开常数PP（Public Parameter）；
2. Encrypt函数：加密函数。这个函数也应该由Alice运行，用Key对用户数据Data进行加密，得到密文CT（Ciphertext）；
3. Evaluate函数：评估函数。这个函数由Cloud运行，在用户给定的数据处理方法f下，对密文进行操作，使得结果相当于用户用密钥Key对f(Data)进行加密。
4. Decrypt函数：解密函数。这个函数由Alice运行，用于得到Cloud处理的结果f(Data)。

那么，f应该是什么样子的呢？HE方案是支持任意的数据处理方法f？还是说只支持满足一定条件的f呢？根据f的限制条件不同，HE方案实际上分为了两类：

- Fully Homomorphic Encryption (FHE)：这意味着HE方案支持任意给定的f函数，只要这个f函数可以通过算法描述，用计算机实现。显然，FHE方案是一个非常棒的方案，但是计算开销极大，暂时还无法在实际中使用。
- Somewhat Homomorphic Encryption (SWHE)：这意味着HE方案只支持一些特定的f函数。SWHE方案稍弱，但也意味着开销会变得较小，容易实现，现在已经可以在实际中使用。

### **什么叫做安全的HE？**

HE方案的最基本安全性是语义安全性（Semantic Security）。直观地说，就是密文（Ciphertext）不泄露明文（Plaintext）中的任意信息。这里密文的意思就是加密后的结果；明文的意思就是原始的数据。如果用公式表述的话，为：

$\forall m_0,m_1,{Encrypt}(PK,m_0)\approx {Encrypt}(PK,m_1)​$

这里PK代表公钥（Public Key），是非对称加密体制中可以公开的一个量。公式中的"约等于"符号，意味着**多项式不可区分性**，即不存在高效的算法，可以区分两个结果，即使已知m0, m1和pk。有人说了，这怎么可能？我已经知道m0, m1了，我看到加密结果后，对m0或者m1在执行一次加密算法，然后看哪个结果和给定结果相同不就完了？注意了，加密算法中还用到一个很重要的量：随机数。也就是说，对于同样的明文m进行加密，得到的结果都不一样，即一个明文可以对应多个密文（many ciphertexts per plaintext）。

在密码学中，还有更强的安全性定义，叫做选择密文安全性（Chosen Ciphertext Security）。选择密文安全性分为非适应性（None-Adaptively）和适应性（Adaptively），也就是CCA1和CCA2。 

HE方案是不可能做到CCA2安全的。那么，HE方案能不能做到CCA1安全呢？至今还没有CCA1安全的FHE方案，但是在2010年，密码学家们就已经构造出了CCA1的SWHE方案了[LMSV10]。



HE方案还有一方面的安全性，就是函数f是不是也可以保密呢？这样的话HE就更厉害了！Cloud不仅不能够得到数据本身的内容，现在连数据怎么处理的都不知道，只能按照给定的算法执行，然后返回的结果就是用户想要的结果。如果HE方案满足这样的条件，我们称这个HE方案具有Function-Privacy特性。不过，仅我个人所了解到的，现在还没有Function-privacy FHE，甚至Function-privacy SWHE也没有。

不过，Function-privacy引入了另一个很有趣的概念，那就是我们能不能反过来，就做到Function-privacy，但是不用做到数据隐私呢？这其实也有很好的应用场景：比如一个天才设计了一个算法（想象Jeffrey Dean设计了历史上第一个O(1/n)复杂度算法，或者设计了一个O(n^2)算法，但是是用来解决旅行商问题的），但是他不想把这个算法公开。他只提供一个程序，这个程序不泄露任何算法本身的内容，人们只能调用这个算法，然后得到输出的结果。这个特别像什么？对啦，就是程序的编译与反编译嘛。如果Function-privacy的加密设计出来了，那么计算机科学家们就可以一劳永逸地阻止程序反编译，甚至连破解都杜绝了。满足这样条件的加密方案，即，给算法加密的方案，叫做**Obfuscation**。很遗憾，2001年，密码学家们已经证明，不可能实现严格意义上的Obfuscation [BGIRSVY01]。但是，可以做到一个称为Indistinguishability Obfuscation的东西。这个东西是密码学家们研究同态加密过程中的一个产物，现在已经有了一些候选方案了[GGHRSB13]。这个就不展开说了，是另一个领域的内容。

### **举个SWHE的例子？**

在2009年Graig Gentry给出FHE的构造前，很多加密方案都具有Somewhat Homomorphism的性质。实际上，最最经典的RSA加密，其本身对于乘法运算就具有同态性。Elgamal加密方案同样对乘法具有同态性。Paillier在1999年提出的加密方案也具有同态性，而且是可证明安全的加密方案哦！后面还有很多啦，比如Boneh-Goh-Nissim方案[BGN05], Ishai-Paskin方案等等。不过呢，2009年前的HE方案要不**只具有加同态性**，要不**只具有乘同态性**，但是不能同时具有加同态和乘同态。这种同态性用处就不大了，只能作为一个性质，这类方案的同态性一般也不会在实际中使用的。

在此我们看一下Elgamal加密方案，看看怎么个具有乘同态特性。Elgamal加密方案的密文形式为：

${CT=(C_1,C_2)=(g^T,h^r\cdot m)}$

其中r是加密过程中选的一个随机数，g是一个生成元，h是公钥。如果我们有两个密文：

${CT_1=(g^{r_1},h^{r_1}\cdot m_1), CT_2=(g^{r_2},h^{r_2}\cdot m_2)}$

我们把这两个密文的第一部分相乘，第二部分相乘，会得到：

${CT=(g^{r1}\cdot g^{r_2},h^{r_1}\cdot m_1\cdot h^{r_2}\cdot m_2)=(g^{r_1+r_2},h^{r_1+r_2}\cdot m_1m_2)}​$

也就是说，相乘以后的密文正好是m1m2所对应的密文。这样，用户解密后得到的就是m1m2的结果了。而且注意，整个运算过程只涉及到密文和公钥，运算过程不需要知道m1m2的确切值。所以我们说Elgamal具有乘同态性质。但是很遗憾，其没有加同态性质。

### **HE的效率如何？**

2011年，Gentry和Halevi在IBM尝试实现了两个HE方案：Smart-Vercauteren的SWHE方案[SV10]以及Gentry的FHE方案[Gen09]，并公布了效率。结果如何呢？我们给出Gentry公布的数据（原始数据可以在[2nd Bar-Ilan Winter School on Cryptography](https://link.zhihu.com/?target=http%3A//crypto.biu.ac.il/winterschool2012/)找到）Smart-Vercauteren的SWHE方案效率如下：

![SWHE-efficiency](https://adamyoung71.github.io/img/in-post/post-homo-encryp/SWHE-efficiency.jpg)

看着好像还行，不过这Dimension有点夸张啊…也就是说公钥很长…那么，Gentry的FHE方案如何呢？效率如下：

![FHE-efficiency](https://adamyoung71.github.io/img/in-post/post-homo-encryp/FHE-efficiency.jpg)

公钥2.3GB，KeyGen需要2个小时，也是醉了…

## 三、 现有HE方案的安全假设和构造概览

如果你致力于HE的研究，我们给出一些可用的资料。



### **如何证明HE方案的安全性？**

对于现在的密码学方案，安全性证明要把它规约到解决一个公开的困难问题上。简单地说，就是如果方案被破解了，那么攻击者可以用破解算法解决一个困难问题。然而，由于这个困难问题还没有找到高效的（多项式复杂度的）算法，因此方案是安全的。

那么，2009年以后的HE方案是建立在哪个困难问题上呢？是一个被称作Learning With Errors（LWE）的困难问题[Reg05]。后来，随着另一个新的工具出现，密码学家们又致力于基于Ring Learning With Errors（Ring-LWE）问题的HE构造[LPR10]。

Ring-LWE涉及到抽象代数中Ring以及Ideal的概念，稍显复杂。我们这里简单介绍一下LWE问题，Ring-LWE问题和它有点像。LWE问题分为两类，一个叫做Search-LWE，一个叫做Decision-LWE。Search-LWE可以简单地用下图来表示，其中A是一个m*n的矩阵，由Zp中的元素组成；s是一个n维向量；e是一个m维向量；b是一个m维向量：

![Ring-LWE](https://adamyoung71.github.io/img/in-post/post-homo-encryp/Ring-LWE.jpg)

这个问题大致为：选择一个秘密（secret）值s，并选择一个范数很小的扰乱（error）向量e，计算b = As + e mod q。这个问题是：只给定矩阵A和计算的结果b（图中红色部分），不给定s和e（途中蓝色部分），反过来求秘密值s的大小。Decision-LWE问题有点类似：给定A和b，算法需要判断，b是由某个s通过As + e计算得来的呢，还是就是一个随机量呢？这里有几个小问题：

1. m和n有多大？这取决于我们要求安全度有多高了。实际上这还取决于一些其他因素。
2. e的范数要多么小？LWE要求e的取值要满足离散高斯分布（Discrete Gaussian Distribution）。
3. 怎么想到的这么个问题？实际上，LWE问题是Lattice中的一个问题。Lattice是什么呢？这个展开说就有点累了…



### **介绍一下构造FHE的思路？**

FHE最重要的一点是Fully，就是说要支持任意的函数f。因此我们也可以很明显看出，想要构造FHE，就需要了解计算机是如何计算的。一般来说，我们有两种思路：

- **从计算机原理考虑**。计算机无论做何种运算，归根到底都是位运算。那么，计算机至少要支持哪些位运算，才能够支持所有的运算呢？实际上，一个计算机只要支持逻辑与运算（AND），以及异或运算（XOR），那么这个计算机理论上就可以实现计算机的其他运算了（我们称之为图灵完备性，Turing Completeness）
- **从抽象代数考虑**。我们只需要加法和乘法就可以完成全部运算了。但其实更严格的说，只要我们在一个域（Field）上构造HE，理论上我们就可以支持所有的f。

基于LWE问题的FHE只能针对1 bit进行加密，因此现在的构造都是从计算机原理考虑。也就是在bit的层面上实现FHE方案，或者更严谨地说，从电路层（Circuit）实现FHE方案。具体构造呢，大家刻意参考下面给出的参考文献了。实话实说，我自己也没有都消化，或者更严格地说，Regev的LWE构造论文我还没有完全看明白。因此，我在此也号召密码学爱好者一起研究啦~



参考文献

[RAD78] Ron Rivest, Leonard Adleman, and Michael L. Dertouzos. On data banks and privacy homomorphisms. Foundations of Secure Computation, 1978.

[Gen09] Craig Gentry. Fully homomorphic encryption using ideal lattices. STOC 2009. Also, see “A fully homomorphic encryption scheme”, PhD thesis, Stanford University, 2009.

[LMSV10] Jake Loftus, Alexander May, Nigel P. Smart, and Frederik Vercauteren. On CCA-Secure Fully Homomorphic Encryption. Cryptology ePrint Archive 2010/560.

[BGIRSVY01] Boaz Barak, Oded Goldreich, Russell Impagliazzo, Steven Rudich, Amit Sahai, Salil Vadhan, and Key Yang. On the (Im)possibility of Obfuscating Programs. Crypto 2001.

[GGHRSB13] Sanjam Garg, CraigGentry, Shai Halevi, MarianaRaykova, Amit Sahai, and Brent Waters. Candidate indistinguishability obfuscation and functional encryption for all circuits. Foundations of Computer Science, 2013.

[Paillier99] Pascal Paillier. Public-Key Cryptosystems Based on Composite Degree Residuosity Classes. Eurocrypt 1999.

[BGN05] Dan Boneh, Eu-Jin Goh, and Kobbi Nissim. Evaluating 2-DNF formulas on ciphertexts. TCC 2005.

[GH11a] Craig Gentry and Shai Halevi. Implementing gentry’s fully-homomorphic encryption scheme. Eurocrypt 2011.

[SV10] Nigel P. Smart and Frederik Vercauteren. Fully homomorphic encryption with relatively small key and ciphertext sizes. PKC 2010.

[Reg05] Oded Regev. On lattices, learning with errors, random linear codes, and cryptography. STOC 2005.

[LPR10] Vadim Lyubashevsky, Chris Peikert, and Oded Regev. On ideal lattices and learning with errors over rings. Eurocrypt 2010.
