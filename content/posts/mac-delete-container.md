---
title: Mac 重装之后,删除容器中的其它卷宗
date: 2020-06-27 10:44:10
tags:
- Mac
---
# 问题
128GB 的机器，重新安装机器之后，安装完一些必备的软件，发现只有 8GB 可用了，要知道我是为了节省一个空间才重新安装的，结果成这样，恶心程度可想而知啊。

如图：
![可怜的存储空间](https://img-blog.csdnimg.cn/20200627093013328.png)
## 存储空间查看
关于本机 -> 存储空间
发现『容器中的其它卷宗』占用了 70GB+

##  一番谷歌
Apple 论坛上并没有找到解决方案，让我重启，重启并没有任何改善。
<!--more-->

## 重新查看重新时的界面
Command + R 按下，再按开机键，出现加载界面之后，放开 Command + R。**因为我记得当时重装时，抹掉硬盘时，显示了两块虚拟硬盘。我当初只抹掉的是第一块硬盘,果然硬盘工具选项里面，第二块硬盘是没有加载的，点击加载之后，需要输入密码，就是这块硬盘当初账号使用和密码，输入完密码之后，显示加载成功。**

### 这样设定的可能性
这样的设定可能是，若电脑丢了或者被偷了，你用的存储空间永远是安全的，别人只能用你剩下的空间重新安装系统使用。

## 解决
**看来解决问题还得靠自己啊**
![终端截图](https://img-blog.csdnimg.cn/20200627093902680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2w3bDFsMGw=,size_16,color_FFFFFF,t_70)
![存储空间截图](https://img-blog.csdnimg.cn/2020062709392528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2w3bDFsMGw=,size_16,color_FFFFFF,t_70)
