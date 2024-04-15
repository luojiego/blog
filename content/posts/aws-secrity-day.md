---
title: aws secrity day 10 月 23 日笔记
date: 2020-10-25 10:30:11
tags:
---

# Security Day

**责任共担** 的安全模型

AWS 的安全责任：计算，存储，数据库和网络以及 全球基础设施



- 安全可视化和安全可控
- 服务之间高度集成，达到自动化布署 Macie 。。
- 建立隐私数据保护
- 丰富的使用场景

<!--more-->


外部：

AWS Shield Standard 和 **Shield Advanced** 的优势：盾牌服务

WAF 7 层安全防护：动态阻止机器人和爬虫 自动化的解决方案，WebACL

GuardDuty: 安全守卫 机器学习 账号的行为 异常的行为会通知给管理员

Detective: 进一步分析异常的行为



内部：

Secret Manager: 密钥的记录与轮换 需要账号和密码先去找该服务

CloundWatch: 监控平台 收集日志 分析

AWS Config: 配置合规性检测

AWS KMS: 密钥加密服务

SecurityHub: 合规检测 检测环境是否达到要求



安全管理：

Certificate Manager: TLS 证书免费申请，自动续签

Cognito: 和第三方做联合登录认证

Amazon Fraud Detector: 欺诈检测，适应不同的场景



# 权限边界

IAM & 权限边界

开发中密切相关 devops 发展的方向

IAM 用户：可以设置别名，方便记忆与登录

IAM 角色：非人（应用）的访问，给 EC2 访问 S3 的权限

分配 AWS 托管策略、自定义的策略

建议：**至少拥有两个 IAM 用户**

ARN：**AWS 所有资源的唯一标识**



## 权限边界

安全授权管理，更安全的委派自己的权限

不同部门有不同的权限边界：权限边界下的账号只有该权限边界之内

通过编辑 json 来控制



# Game Day

## 数据安全

传输 网络加密



存储加密 S3 和 EBS



使用中的数据 应用级别的加密



CloundTrail 可以用来审核 AWS KMS 的使用情况



VPC 

安全组 有状态的防火墙 **按照角色划分安全组** **可以附加多个安全组**



NACL 无状态的防火墙 安全级别非常高

子网是不能跨 AZ 的



SSO: 

Directory Service:

Firewall Manager:

Amazon Inspector: 自动化安全评估服务 网络层和操作系统/中间件

云管理挑战

Systems Manager：好多工具。。



S3 ALB EC2 RDS VPC

简报：不规则的突增访问流量

S3 静态文件被删除

账号数据也异常了



恢复业务，最佳实践

