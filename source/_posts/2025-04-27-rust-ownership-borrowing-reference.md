---
title: Rust 所有权、借用、引用
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-27 20:00:00
---

这一章是 Rust 的灵魂课。
你前面语法都能学得很快，但如果所有权没搞明白，后面基本寸步难行。

先别怕，我尽量说人话。

## 1. 什么叫所有权
Rust 里每个值都有一个“主人”，这个主人就是变量。
同一时间只能有一个主人，主人离开作用域，值就被回收。

看例子：

```rust
fn main() {
    let s = String::from("hello");
    println!("{s}");
} // 这里 s 离开作用域，内存被释放
```

这就是 Rust 不用 GC 也能管好内存的核心原因之一。

## 2. 移动（move）：不是复制，是转让
Rust 默认会在很多场景下“转移所有权”：

```rust
fn main() {
    let s1 = String::from("rust");
    let s2 = s1;
    // println!("{s1}"); // 报错：s1 已经失效
    println!("{s2}");
}
```

`s2 = s1` 不是深拷贝，而是把所有权给了 `s2`。
这样做的目的就是避免重复释放内存，安全第一。

## 3. `clone`：我就要两份
如果你真的要两份独立数据，手动 `clone`：

```rust
fn main() {
    let s1 = String::from("rust");
    let s2 = s1.clone();
    println!("{s1}, {s2}");
}
```

这会复制堆数据，能用，但别无脑乱用，尤其在大对象和高频路径里。

## 4. Copy 类型：复制就像拍照
有些类型赋值是“复制值本身”，不是转移所有权，比如整数、布尔、字符等：

```rust
fn main() {
    let x = 10;
    let y = x;
    println!("{x}, {y}"); // 都能用
}
```

这些类型实现了 `Copy`，所以你会感觉“它怎么不报错”。

## 5. 函数传参也会发生所有权转移
这个是新手高频翻车点：

```rust
fn takes_ownership(s: String) {
    println!("{s}");
}

fn main() {
    let s = String::from("hello");
    takes_ownership(s);
    // println!("{s}"); // 报错，s 的所有权被移走了
}
```

如果你不想把所有权交出去，就用借用。

## 6. 借用（borrowing）：借你看看，不给你拿走
借用就是传引用 `&T`：

```rust
fn len_of(s: &String) -> usize {
    s.len()
}

fn main() {
    let s = String::from("hello rust");
    let n = len_of(&s);
    println!("len = {n}, s = {s}");
}
```

你看，函数能读到值，原变量还能继续用。

## 7. 可变借用：能改，但一次只能一个
Rust 有个铁规则：
1. 同一时刻，要么多个不可变借用 `&T`
2. 要么一个可变借用 `&mut T`
3. 两者不能混着来

例子：

```rust
fn append_world(s: &mut String) {
    s.push_str(" world");
}

fn main() {
    let mut s = String::from("hello");
    append_world(&mut s);
    println!("{s}");
}
```

为什么这么严格？为了避免数据竞争。
编译期先给你拦住，运行时就更稳。

## 8. 悬垂引用（dangling reference）不会出现
C/C++ 里常见的悬垂指针，Rust 编译器直接不让你过。

```rust
// 这段伪代码会被 Rust 拒绝
// let r;
// {
//     let s = String::from("hi");
//     r = &s;
// }
// println!("{r}");
```

因为 `s` 已经销毁了，`r` 指过去就是野引用，Rust 直接报错，干脆利落。

## 9. 字符串切片：最常见的引用类型
`&str` 本质就是“对字符串的一段借用”：

```rust
fn first_word(s: &str) -> &str {
    for (i, &b) in s.as_bytes().iter().enumerate() {
        if b == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}

fn main() {
    let s = String::from("hello rust world");
    let w = first_word(&s);
    println!("{w}");
}
```

这里参数用 `&str` 而不是 `&String`，更通用。
`String` 能传，字符串字面量也能传。

## 10. 结尾
你要记住一句话：
Rust 的所有权不是为了折磨你，是为了让你在编译阶段把内存问题解决掉。

前期会痛一点，后期会非常香。

### 课后小练习
1. 写一个函数，参数是 `&str`，返回最长单词长度。
2. 写一个函数，参数是 `&mut String`，把内容变成大写。
3. 分别用“传所有权”和“传借用”写两个版本，体会调用方有什么区别。

### 下一章
下一篇我们聊 `struct`、`enum`、`Option`、`Result`，开始写更像工程代码的 Rust。
