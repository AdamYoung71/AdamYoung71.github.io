---
layout: post
title:  "Review on: TOWARDS FEDERATED LEARNING AT SCALE: SYSTEM DESIGN"
date: 2019-10-22 20:30
author: "Adam"
header-img: "img/google-federated-learning.jpg"
catalog: true
tags:
    - Federated Learning
    - Reviews
---

> Bonawitz, Keith et al. “Towards Federated Learning at Scale: System Design.” (2019): n. pag. Web.

## 1. 介绍

- 趋势：同步的大批量训练。
- 增强安全性：DP，SecAgg。
- 场景：使用FL在安卓手机上训练模型并用Server进行聚合。

## 2. Protocol

![Federated Learning Protocol](https://adamyoung71.github.io/img/in-post/Reviews/Towards-Federated-Learning-at-Scale-System-Design/Federated-Learning-Protocol.png)

1. **基本概念**：Server在时间窗口内从声明可用的device中选择一个子集（典型大小为几百个，SecAgg所限制），在训练期间device与设备保持连接。之后，server向device发送当前的Model checkpoint，device本地更新模型后上传。

2. **训练阶段：**

   1. 选择（Selection）：根据一定条件选择设备（充电中、使用WiFi），未选择的设备会被告知稍后特定时间点重连。
   2. 配置（Configuration)：server向device发送当前计划和模型参数。
   3. 报告（reporting）：server等待device上传数据，进行Federated Averaging并告知下次连接时间。若成功则更新全局模型，失败则放弃此轮训练。掉队节点会被直接忽略（协议具有一定容忍度）
   4. 选择和报告阶段由一系列参数控制：device数量、超时、最小目标（决定本轮是否被丢弃）。

3. **流控制机制Pace Steering**

   ​	对于小规模FL，PS主要用来保证有足够数量的device同时连接server。对于大规模FL，PS主要用来使随机化报告时间，避免“Thundering Herd”问题。同时调整时间窗口减小昼夜等设备活性差异对训练造成的影响.

   

## 3. Device

![Device Architecture](https://adamyoung71.github.io/img/in-post/Reviews/Towards-Federated-Learning-at-Scale-System-Design/Device-Architecture.png)

1. **程序配置（Programmatic Configuration）:** 应用向FL Runtime提供其训练信息和数据集。重要条件是要对设备使用影响较小。当某些条件不满足时（不再充电等），Runtime会放弃本轮训练。
2. **任务调用（Job Invocation）: **Runtime向Server报告准备就绪，请求训练方案和数据。
3. **任务执行（Task Execution）:** 被选中的设备按照下发的计划进行训练。
4. **汇报（Reporting）: **训练结束后上传参数更新。



**特点**

1. 多租户（Multi-Tenancy）：共性被共享，个性被隔离，从而可以在单个应用中进行多种训练。
2. 验证性（Attestation）：使用Google SaftyNet认证机制。

## 4. Server

**结构**

1. Coordinators: 最高级别角色，有多个，每一个分管一个设备群。总体调度。

2. Selector：按照Coordinator的需求允许设备接入，并转发给Aggregators。

3. Master Aggregators：只有主聚合者完成聚合后才会写入新的checkpoint。

4. Pipelining：配置和报告是序列化的，故因为选择过程不需要之前轮次的输入，可以与上一轮的配置报告一起进行。

5. 错误模型：Aggregator 或Selector失效时，只有其连接的部分设备会丢失；MA失效时本轮会失败但是可以由Coordinator重启。若C失效，Selector会探测到并重新生成。

   ![actor-model](https://adamyoung71.github.io/img/in-post/Reviews/Towards-Federated-Learning-at-Scale-System-Design/actor-model.png)



## 6. SecAgg

- 前两轮：Server和每一个device建立共享秘密（DH）
- 第三轮：device上传加密数据。
- 第四轮：device发送特定信息帮助server解密模型（非必须，只要达到规定最小数量即可）

**问题**

- 计算成本随着device数量急剧增加。

![algorithm](https://adamyoung71.github.io/img/in-post/Reviews/Towards-Federated-Learning-at-Scale-System-Design/algorithm.png)