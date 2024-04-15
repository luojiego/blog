---
title: Rust 超快安装方法
date: 2021-11-03 21:46:46
tags:
- Rust
---
由于 Rust 安装需要从 github 下载资源，直接安装，速度会非常感人，而且安装到一半还可能会失败，即使使用源码也会面临相同的情况。

感谢中科大，Respect！！！

以下是简单的解决方法：

```bash
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

几乎不出一分钟，就安装成功了。

安装成功，执行个 `cargo install ripgrep` ，依然让人非常无语。

此时只需要添加 `~/.cargo/config` 文件

文件内容如下：

```bash
[source.crates-io]                                                                                                                                                                                              
registry = "https://github.com/rust-lang/crates.io-index"

replace-with = 'tuna'
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
```

这样能就能愉快的  `cargo install/run/build` 了。

注意：ustc 的源已经不能正常使用了，tuna 速度还是不错的
