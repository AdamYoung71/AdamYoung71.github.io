---
layout: post
title:  "Review on: Federated machine learning: Concept and applications"
date: 2019-10-23 21:04
author: "Adam"
header-img: "img/google-federated-learning.jpg"
catalog: true
tags:
    - Federated Learning
    - Reviews
---

### 1. 提出背景

当前AI面临着两大挑战：信息孤岛、隐私保护。而在传统的模型训练中没有考虑数据聚合的可行性，所以提出FL。



### 2. 概述

Federated Learning这一概念最早由Go在防止数据泄露的前提下从分布式数据中训练学习模型。

> To build machine learning models based on data sets that are distributed across multiple devices while preventing data leakage.

#### 2.1 定义

定义N个数据拥有者$\{F_1...F_n\}$，都愿意贡献相应数据$\{D_1...D_n\}$共同训练一个机器学习模型。在传统的模式下，我们会把这些数据组合起来$D=D_1\cup ...\cup D_n$训练一个模型$M_{SUM}$。而在FL的机制中，数据拥有者可以在不暴露自己数据的情况下训练模型$M_{FED}$。除此之外，我们将$M_{FED}$这一模型的精度（记为$V_{FED}$）应该与$V_{SUM}$非常接近。设$\delta$是一个非负实数，若

$|V_{FED}-V{SUM}|<\delta$

我们说此FL算法具有$\delta $精度损失（$\delta-accuracy\  loss$）。



#### 2.2 FL的安全性

安全性是FL所要关注的一个重要问题，在目前的研究中有以下几种方案：

1. 安全多方计算（Secure Multiparty Computation）：可以在模型包含的多方上保证**完全零知识**：每一方只知道其自身的输入和输出。

2. 差分隐私（Differential Privacy)：在数据中添加噪声或使用一些泛化方法使得数据中的某些敏感属性变得更模糊，直到第三方不能够分辨出其中的个体。但此种方法仍需要进行数据转移，所以不可避免的仍然在一定程度上存在安全性和准确性两方面的trade-offs.

3. 同态加密（Homomorphic Encryption)：参考[同态加密](https://adamyoung71.github.io/2019/10/22/Homomorphic-Encryption/)，此种方法在隐私保护方面比较好，但在进行非线性方程计算式需要对其进行多项式近似，使得其准确性收到一定影响。

4. 区块链

	

#### 2.3 FL的分类

1. **Horizontal Federated Learning:** 两个数据集拥有相同的特征空间和不同的样本。例如两家异地公司业务相似，但却拥有交集很小的客户。![horizontal-federated-learning](https://adamyoung71.github.io/img/in-post/Reviews/FL-concepts-and-applications/horizontal-federated-learning.png)
2. **Vertical Federated Learning:** 两个数据集拥有相同的样本ID但特征空间不同。例如两家同地公司业务不同，但用户交集较大。![vertical-federated-learning](https://adamyoung71.github.io/img/in-post/Reviews/FL-concepts-and-applications/vertical-federated-learning.png)
3. **Federated Transfer Learning:** 数据集在样本和特征空间上均不同。通常的做法是先在比较有限的overlap上学习然后再应用到单侧特征的预测中。![federated-transfer-learning](https://adamyoung71.github.io/img/in-post/Reviews/FL-concepts-and-applications/federated-transfer-learning.png)

#### 2.4 FL的系统结构

1. **Horizontal Federated Learning：** 

	![architecture-of-horizontal-federated-learning](https://adamyoung71.github.io/img/in-post/Reviews/FL-concepts-and-applications/architecture-of-horizontal-federated-learning.png)

2. **Vertical Federated Learning**：![architecture-of-vertical-federated-learning](https://adamyoung71.github.io/img/in-post/Reviews/FL-concepts-and-applications/architecture-of-vertical-federated-learning.png)