---
title: V2ray 终端代理
date: 2020-07-01 08:40:09
tags:
- V2ray
---
# V2ray 终端代理设置

我可能是个白痴吧，从业这么久了，都没有研究过在终端加速 HTTP/HTTPS，特别是下载 **github** 的开源代码，或者使用 **brew** 更新或者安装软件。

昨天竟然才明白，终端使用了代理之后，**ping** 不通谷歌或者非死不可是正常的。所以之前的郁闷一下就解决了。下面说说在三个操作系统上简单的设置吧。  



## Mac

### 拷贝代理地址

如图，右键选择菜单项，地址就会被拷贝，然后在终端执行就 OK 了。

![右键选中Copy代理地址](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/image-20200627234616476.png)
<!--more-->

### 终端执行

然后在终端执行即可：

![image-20200627234858279](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/image-20200627234858279.png)

### 拉取测试

然后尝试拉取 github 代码，以 **etcd** 为例吧。

![image-20200627235113454](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/image-20200627235113454.png)

要知道之前可是 **30KB** 以内的速度，真的让人想死。

### 永久生效

将代理地址拷贝到 ~/.zprofile 文件中即可，具体根据自己的机器进行配置，我的系统是最近才安装的，默认是 **zsh**.

## Windows

### Termial/PowerShell
```powershell
$env:http_proxy="http://127.0.0.1:10808"
$env:https_proxy="https://127.0.0.1:10808"
```
### git bash

**虽然 Windows Terminal 现有已经非常强大了**，但是有部分人依然还在使用 git bash，设置方法比较简单

```bash
export http_proxy=http://127.0.0.1:10809
export https_proxy=http://127.0.0.1:10809
```

永久生效的就将将这两个设置放进 ~/.bashrc 文件中即可

## Linux

由于我并没有在实体机上安装 Linux，所以此处就用本地的虚拟机吧。

### 最关键的设置

勾选本机 **v2rayN 设置** 中的“允许来自局域网的连接”。Mac 的我不清楚，因为穷，并没有办法在 128GB 的机器上安装虚拟机。

![v2rayN设置](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/image-20200628074406279.png) 

### 终端执行

```shell
export HTTP_PROXY=192.168.0.108:10809 # IP 使用本机IP
export HTTPS_PROXY=192.168.0.108:10809 # IP 使用本机IP
```

**抄作业不要连名字也抄了！！！**

### 永久生效

将前面两句回到对应 shell 的启动配置文件中。

由于我惯用的都是 bash，所以以 bash 为例，放置在 ~/.bashrc 文件的末尾即可。

```shell
echo -e "export HTTP_PROXY=192.168.0.108:10809\nexport HTTPS_PROXY=192.168.0.108:10809" >> ~/.bashrc
```

**开启新的终端窗口就可以生效了，或者在当前窗口执行以下语句也可以生效**

```shell
. ~/.bashrc # source ~/.bashrc 有同样的效果
```



## 修改 HTTPS 为 SSH

提示：**并不能通过 HTTP/HTTPS 来代理 github 的 SSH 流量，可以通过使用 HTTPS 的方式来下载，下载完了之后修改成 SSH 的地址**

如图：

![修改HTTPS为SSH](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/image-20200628074624577.png)

用自己熟悉的工具将这个 url 修改为 etcd SSH 的 url: "**git@github.com:etcd-io/etcd.git**"，如果你足够细心，你会发现，你根本不用去项目主页去看地址是什么，因为规则是非常明确的。
