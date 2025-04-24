---
title: Rust 学习入门
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-25 23:00:00
---


## 学习Rust之前要安装的软件

工欲善其事必先利其器，战斗之前我们要先配备好武器。

 #### 一. 安装Rust环境
     
可以选择在[Rust官网](https://www.rust-lang.org/zh-CN/install.html)
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
下载安装Rust环境。
国内的用户可以选择[Rust字节跳动镜像](https://rsproxy.cn/)
```shell
curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh
```
下载安装Rust环境。

在安装Rust环境时，请选择`Install Rustup`，安装完成后，在命令行输入`rustc --version`，如果安装成功，则会显示Rust的版本号。

！！！ 对于Windows用户， 在安装Rust环境之前，请先安装[Git](https://git-scm.com/download/win)和[Msvc工具链](https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/)

！！！ 对于Linux用户， 在安装Rust环境之前，请先安装[Git](https://git-scm.com/download/linux)和[GCC](https://gcc.gnu.org/install/)
```shell
sudo apt-get install gcc git
```

#### 镜像换源(可选)
我相信大多数使用过Linux的用户都对镜像源不陌生，但是对于Rust来说，镜像源的配置也不是难，以字节跳动镜像为例，在Linux: `~/.cargo/config`, Windows: `{用户目录(通常在C盘的user目录下你的电脑用户名)}\.cargo\config`文件中添加以下内容：
```shell
[source.crates-io]
replace-with = 'rsproxy-sparse'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true
```

#### 二. 安装自己熟悉的代码编辑器
目前rust的编辑器有很多，包括RustRover、VSCode、Visual Studio Code、Atom、Sublime Text、IntelliJ IDEA等，
这里推荐一个[RustRover](https://www.jetbrains.com/rust/download/#section=windows)，是一个专业Rust的代码编辑器，且对于个人开发者来说是免费的，这是一个不错的选择



#### 三.  熟悉基础语法
Rust 的语法与 C++ 和 Python 有一定相似之处，但也有其独特的地方。可以从以下几点入手：

1. 变量与数据类型：了解 let 关键字、不可变性（immutability）、基本数据类型（如 i32, f64）。
2. 所有权与借用：这是 Rust 最核心的概念之一，理解内存管理机制是掌握 Rust 的关键`(ps: 这一点在Web或部分开发中是完全可以忽略的，直接无脑clone就行。但是对于一些需要考虑内存管理的场景，比如游戏开发，它还是非常有用的)`。
3. 控制流：学习 if 表达式、loop 循环、for 循环、while 循环、match 匹配、 if let匹配、while let匹配、let ... else ... 模式等等。
4. 函数：了解函数定义`我相信你已经学习过其他语言了，这什么什么定义就是屁话，一句话表述就是我给你什么什么东西，你还我什么什么东西`、参数`这就是我给你的什么什么东西`、返回值`这就是你还我的什么什么东西`、函数调用`这算我在那里怎么怎么你`、闭包`这相当于在自己家里能做一件什么什么事情，然后并不需要给它写成函数，而是直接写在函数体里，然后调用这个闭包，然后这个闭包就可以执行了`、async关键字。

如果上述1234点都掌握好，那么Rust的基础语法你就学会了

这个时候可能就有朋友问了，`诶，博主博主，我现在还不会写代码，怎么办勒？`

那我问你，我让你去写一个`命令行工具`你会写吗？嗯？回答我
当然是不会啦

所以说接下来我们的目的就是开始做一个命令行工具 
### 下一章