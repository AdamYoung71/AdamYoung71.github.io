---
title:  "Week 1 Introduction"
date: 2019-1-17 18:40
author: "Adam"
index-img: "/img/bg/post-bg-ai.jpg"
banner-img: "/img/bg/post-bg-ai.jpg"
tags:
    - AI
    - Notes
    - Coursera
---

# Week 1 Introduction

## What is Machine Learning?

​	Tom Mitchell provides a more modern definition: "A computer program is said to learn from experience E with respect to some class of tasks T and performance measure P, if its performance at tasks in T, as measured by P, improves with experience E."(对某一任务表现水平随不断经历某种过程而提高)

Example: playing checkers.

E = the experience of playing many games of checkers

T = the task of playing checkers.

P = the probability that the program will win the next game.

In general, any machine learning problem can be assigned to one of two broad classifications:

### Types

#### Supervised Learning

**对于给定的数据，已知其正确结果的形态和输入和输出的关系。**

##### 分类：

1. “regression"回归：连续型（广义，不一定绝对连续，如销量）

2. ”classification"分类：离散型（可能性通常很少，如0-1）

3. example：

   (a) Regression - Given a picture of a person, we have to predict their age on the basis of the given picture

   (b) Classification - Given a patient with a tumor, we have to predict whether the tumor is malignant or benign.

#### Unsupervised Learning

**未知结果。可以在不知道中间的各种变量的意义的情况下，根据变量的结构、关系做出推断。**

Example：

Clustering: Take a collection of 1,000,000 different genes, and find a way to automatically group these genes into groups that are somehow similar or related by different variables, such as lifespan, location, roles, and so on.

Non-clustering: The "Cocktail Party Algorithm", allows you to find structure in a chaotic environment. (i.e. identifying individual voices and music from a mesh of sounds at a [cocktail party](https://en.wikipedia.org/wiki/Cocktail_party_effect)).

## Model and Cost Function

### Model Representation

1. 数据集表示：![1547720243394](assets\1547720243394.png)

2. 预测结果h(x)：”A good predictor"，Hypothesis

3. 流程：![img](assets\\H6qTdZmYEeaagxL7xdFKxA_2f0f671110e8f7446bb2b5b2f75a8874_Screenshot-2016-10-23-20.14.58.png)

   ### Cost Function

![1547721315822](assets\1547721315822.png)

**代价方程，又称平方误差方程，是衡量预测方程和真实值之间误差的函数。**

