---
title: MantisBT 安装
date: 2020-07-01 08:43:41
tags:
- MantisBT
---
# MantisBT 安装

[github 地址](https://github.com/mantisbt/mantisbt) **注意，不能从 release 页面下载来安装，要从官网进行下载**

## 下载并解压

```shell
# curl -LO https://github.com/mantisbt/mantisbt/archive/release-2.24.1.zip # 这个代码是错误的
curl -LO https://nchc.dl.sourceforge.net/project/mantisbt/mantis-stable/2.24.1/mantisbt-2.24.1.zip
unzip unzip release-2.24.1.zip
```

[安装官方文档](https://www.mantisbt.org/docs/master/en-US/Admin_Guide/html-desktop/)

**建议详细阅读官方文档**，不然将会和我一样遇到一车子的坑。
<!--more-->

## PHP 环境

由于 CentOS 默认安装的是 5.4，但是 MantisBT 要求的是 5.5.9+，所以需要安装更高版本的 PHP。

此处，我们安装 7.2 吧。其实我也不知道为什么要安装 7.2

```shell
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum install yum-utils
# yum-config-manager --enable remi-php55   # Install PHP 5.5
# yum-config-manager --enable remi-php56   # Install PHP 5.6	
yum-config-manager --enable remi-php72
yum install php # 此时安装时则会是 7.2
```

[PHP 安装参考地址](https://www.tecmint.com/install-php-5-6-on-centos-7/)

## 数据库

此处使用我比较熟悉的 Mariadb，直接拷贝了之前的启动脚本

```shell
docker run --restart=always --name MariaDB -p 3307:3306 -v /data/db/mariadb:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mariadb:10.1
```

## 拷贝文件

```shell
mv mantisbt-release-2.24.1 mantisbt
cd mantisbt/config
```

## 安装

### 坑1

没有重新启动 Apache, 所以直接在浏览器里面打开了 php 文件。

**重启 Apache**

### 坑2 

```
FATAL ERROR: PHP mbstring extension is not enabled.
MantisBT requires this extension for Unicode (UTF-8) support
http://www.php.net/manual/en/mbstring.installation.php
```

解决方案

```shell
yum install php-mbstring
```

重新启动 Apache

```shell
service httpd restart
```

### 坑3

**页面报 500 错误**。。。

查看 **Apache** 错误日志

```shell
tailf /var/log/httpd/error_log
```

[解决方案](https://www.mantisbt.org/forums/viewtopic.php?f=3&t=25660)

```shell
# 重新下载安装包
curl -LO https://nchc.dl.sourceforge.net/project/mantisbt/mantis-stable/2.24.1/mantisbt-2.24.1.zip
```

### 坑4

输入 http://ip:8080/admin/check/index.php，然后提示 **PHP 不支持的数据库类型**

叕怎么了？

查看文档，Mantis 是支持 mysqli，但是啥问题哈？

研究半天，索性尝试安装了一下 **php-mysqli**，结果可以安装，看来是插件没有安装

```shell
 yum install php-mysqli
```

安装好之后，再试，还是不行。

噢，对了，还需要**重新启动 Apache 服务**

check 成功，安装成功了。

请注意：由于数据库是安装在 Docker 中的，所以数据库的 **hostname** 需要使用 **172.17.0.1** 

### 坑5

安装过程中，会提示无法正常创建 config_inc.php，需要手动创建这个文件，并将页面上展示的配置数据拷贝到这个文件中，或者将 mantis 这个目录权限设置为 apache:apache。

### 坑6

终于跳转到启动页面了，真心不容易，对于我这个对 PHP 没有任何研究的人而言，真的挺难的。

页面会有提示说删除 admin 目录，并且禁用 administrator 账户。

然后我要登录管理员账号，可是 administrator 的密码是什么？

**不是空，也不是 administrator.**

你猜怎么着，是 **root**，从文档中得到的。

## 结语

首先由于我对 PHP 和 Apache 不熟悉，导致坑比较多。其实我没有到详细阅读说明文档，导致遇到许多麻烦事情。所以对于需要安装 Mantis 的小伙伴，若有时间，请先阅读说明文档，若时间不允许，但是自己对整个环境都不熟悉的话，可能需要浪费比较多的时间在安装上面。
