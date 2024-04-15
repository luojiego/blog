---
title: AWS re-invent 12-23
date: 2020-12-27 11:45:53
tags:
- aws
- re-invent
- 西安
---
# 巅峰科技 重塑未来

这次举办的比上一次参加的 10 月 23 的能好一些，人也多一些。不过总体的感觉就是每次都是新产品推销会议。但是对于我们小公司而言，了解趋势有帮助。不然一直用着那些古老的技术也不太合适。

## 2020 主题分享

算是对于 re:invent 发布的内容的总体概览，讲解者的职位最高，也是最能将东西讲解的最清楚的。

自 2006 年发布 S3，AWS 已经持续创新 15 年。

2019，在拉斯维加斯举办的线上有高达 6.7 万的参会者，而且门票将近 1.8K 美金。

2020 年因为疫情的原因，在线上举行超过 3 周，有超过 50w 的注意用户观看。现在线上的版本已经有了中文版本，对于有需要的可以点击链接去观看了。

AWS 云市场的份额高达 45 %。

目前云计算本地部署高达 96%，而云上部署只有 4%，这个数字让我特别震惊。
<!--more-->

利用云，可以让我们的 Workload 更加合理，可以充分利用实例特性，比自己购买机器更加灵活，更加方便。

2017 年，AWS 在芯片级改造，将原来需要在软件层级处理的内容迁移到硬件处理，这样可能给客户提供更好性能的实例。

2020 年，AWS 新发布的实例超过 400 种。

同时，AWS 已经也提供了 Mac Instance，也将在 2021 年提供 M1 芯片支持。

目前处理 AWS 使用的处理器分为：Intel, AMD 和 AWS Gravtion(arm 芯片级的改造)，众所周知，Intel 的价格的确是个问题，需要降低成本的话，可以选择非 Intel 的处理器。

Intel 发布的新实例分为 M5ZN(高达 4.5GHz), R5B(应对高 IOPS 和 EBS), D3EN(高密度存储，高达 336TB)

AMD 发布的有 C5A,...,G4AD(AMD 的 GPU)

AWS Gravtion 有 R6G, M6G, C6G, T4G, CG6N，价格相比 Intel CPU，最高可达到 40%

EKS Distro (AWS 维护的 K8S 版本）

ECS/EKS AnyWhere 可在自己的数据中心搭建，在 AWS 的后台进行管理

Serverless 的内存现在可达 10GB，另外现在运行以 ms 计费，之前为 100ms

存储方面发布的 GP3，相比之前的 GP2(不用先增加存储才能提高吞吐）

数据库方面，Aurora Serverless V2，不需要根据峰值选择，扩展速度极快，最高节省 90%

Aurora PostgreSQL，Babelfish 2021 将开源

Glue Elastic View, 用 SQL 写逻辑

QuickSight Q BI, 可以使用自然语言获取报表

机器学习三层架构(AI, ML, ML基础架构)，AWS 都做了极大的努力，让用户非常方便入手。

Habana Intel GPU 和 Trainiums (AWS 芯片，节省成本)

Monitron, 端到端的机器监控， lookout

## 加速上云，高频创新

完全不知道讲解的什么玩意，话筒拿的一会近，一会远，压根没有听的心情。只希望他尽快滚下去。

AWS 网络原则：安全易扩展；抽象复杂性；无缝连接，集中可视化

Nitro: 提供虚拟机最大化

## 黑科技分享

将机器学习交到每一位开发者手上

构建机器学习爱好者社区

机器学习未来的需要职位是 5.8 千万，但是目前仅有 30 万

Deeplens, DeepRacer, Composer

DonkyerCar

## 现代应用程序开发及云安全

AWS 十四条军规

EKS/ECS Anywhere，AWS 在混合云上的努力

ECR Public & Public Gallery 代替 docker hub

Proto（模板）: 无服务与微服务的管理

## 数据库与数据分析新趋势

用正确的工具做正确的事情

图数据库、时序数据库，目前我们能想到的，AWS 均有提供

大数据，数据分析能力，数据湖以及数据迁移

Glue DataBrew, RedShift Data sharing

EMR Studio, EMR on EKS

## 人工智能与机器学习创新之旅

SageMaker 端到端的服务

Data Wrangler

JumpStar

Redshift ML

Amaxon Neptune MP

## 笔记

![WechatIMG1](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/WechatIMG1.jpeg)

![WechatIMG2](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/WechatIMG2.jpeg)
