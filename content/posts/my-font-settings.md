---
title: 我的常用软件字体设置
tags:
  - 字体
  - 设置
date: 2022-12-23 18:28:59
---

由于“霞鹜文楷”这个字体太好看了，不得不想要设置成为，我所有的非常常用的软件字体。
但是这款字体的英文着实不是太漂亮，所以我的常用软件的首选字体是 “Roboto Mono”，这样中文字体会使用“霞鹜文楷”
github 地址：[字体地址](https://github.com/lxgw/LxgwWenKai)
安装比较简单，VS Code 和 Typora 需要机器本身安装这两款字体。
# VS Code 设置
就正常设置 “Font Family” 即可：`'Roboto Mono','霞鹜文楷等宽'`。
<!--more-->
# Logseq 设置
Settings->"Edit custom.css" 添加如下内容
```css
@font-face {
  font-family: "Roboto Mono";
  font-weight: 200 900;
  font-style: normal;
  font-stretch: normal;
  src: url("https://letui-game-res.oss-cn-shenzhen.aliyuncs.com/tmp/RobotoMono.ttf");
}

@font-face {
  font-family: "霞鹜文楷等宽";
  font-weight: 200 900;
  font-style: normal;
  font-stretch: normal;
  src: url("https://letui-game-res.oss-cn-shenzhen.aliyuncs.com/tmp/LXGWWenKaiMono-Regular.ttf");
}

:root {
  --ct-text-size: 20px;
  --ct-line-height: 1.6;
  --ls-font-family: Roboto Mono, "Only Emoji", "霞鹜文楷等宽";
  --ct-page-title-font-family: var(--ls-font-family);
  --ct-code-font-family: Roboto Mono,"霞鹜文楷等宽", monospace;
}
.CodeMirror {
  font-family: Roboto Mono,"霞鹜文楷等宽", monospace;
}
:not(pre)>code {
  font-family: Roboto Mono,"霞鹜文楷等宽", monospace;
}
```
为了统一性，统一使用 `src` 设置字体路径。
上面的配置在手机上会出正常展示。
# Tabby
“设置”->“外观” 下，字体设置为“Roboto Mono Medium”，备用字体设置“霞鹜文楷等宽”。
# Typroa
在 Typora 的主题文件夹下，创建 base.user.css，里面写上如下内容：
```css
body {
    font-family: 'Roboto Mono', "霞鹜文楷等宽";
    --monospace: 'Roboto Mono', "霞鹜文楷等宽"; 
}
```
然后重启 Typora，即可。
# 博客设置
hexo 主题的设置稍微有些复杂。
需要在 themes/next/layout 下的 _layout.njk 中添加如下内容。我添加在了 `head` 标签下。
```css
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/lxgw-wenkai-lite-webfont@1.0.0/style.css" />
  <style>
  body,div.post-body,h1,h2,h3,h4 {
    font-family: "Roboto Mono", "LXGW WenKai LITE", sans-serif;
    font-size: 108%;
  }
```
在 `themes/next` 中的 ”font:“ 的 codes 下 `family` 上设置为 “Roboto Mono” 会将代码设置为 Roboto Mono 字体。
参考网址：[https://zhuanlan.zhihu.com/p/361392320](https://jiapan.me/2022/modify-blog-font/)
