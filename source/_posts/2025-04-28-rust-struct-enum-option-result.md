---
title: Rust struct / enum / Option / Result
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-28 20:00:00
---

前面你已经过了所有权这一关，恭喜，已经不是纯新手了。
这一章我们把 Rust 里最常用的四件套拿下：
`struct`、`enum`、`Option`、`Result`。

这四个学会了，你就能开始写像样的业务代码了。

## 1. `struct`：把相关数据装进一个盒子
你之前可能写过一堆分散变量：

```rust
let name = "zhenyi";
let age = 18;
let active = true;
```

这种写法小项目还行，字段一多就乱。
所以我们用 `struct`：

```rust
struct User {
    name: String,
    age: u8,
    active: bool,
}

fn main() {
    let u = User {
        name: String::from("zhenyi"),
        age: 18,
        active: true,
    };
    println!("{} {} {}", u.name, u.age, u.active);
}
```

### 可变结构体

```rust
let mut u = User {
    name: String::from("zhenyi"),
    age: 18,
    active: true,
};
u.age = 19;
```

结构体实例要改字段，实例本身得是 `mut`。

### 结构体更新语法

```rust
let u1 = User {
    name: String::from("zhenyi"),
    age: 18,
    active: true,
};

let u2 = User {
    name: String::from("new_name"),
    ..u1
};
```

`..u1` 很方便，但要注意所有权移动的问题，尤其是 `String` 这种堆数据字段。

## 2. `enum`：一个值，多种状态
`struct` 是“固定字段集合”，`enum` 是“多种可能之一”。

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

再来个带数据的：

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

一个 `Message` 值只能是其中一种形态，这在建模状态机、协议消息时非常好用。

## 3. `match` + `enum`：天作之合

```rust
fn handle(msg: Message) {
    match msg {
        Message::Quit => println!("退出"),
        Message::Move { x, y } => println!("移动到 ({x}, {y})"),
        Message::Write(text) => println!("文本：{text}"),
        Message::ChangeColor(r, g, b) => println!("颜色：{r},{g},{b}"),
    }
}
```

Rust 强制你把所有分支写全，防止漏处理某个状态。

## 4. `Option<T>`：没有值也算一种正常状态
很多语言里“可能为空”一般用 `null`，然后到处 NPE。
Rust 不走这套，它用 `Option<T>`：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

你有值就 `Some(v)`，没值就 `None`。

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("zhenyi"))
    } else {
        None
    }
}
```

使用时用 `match` 或 `if let`：

```rust
match find_user(1) {
    Some(name) => println!("找到用户：{name}"),
    None => println!("没找到"),
}
```

## 5. `Result<T, E>`：成功 or 失败，明确返回
`Option` 只能表示“有/没有”，不能告诉你“为什么失败”。
这时候用 `Result<T, E>`：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

比如：

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("除数不能为 0"))
    } else {
        Ok(a / b)
    }
}
```

调用：

```rust
match divide(10.0, 2.0) {
    Ok(v) => println!("结果：{v}"),
    Err(e) => println!("错误：{e}"),
}
```

## 6. `?` 运算符：错误传递神器
你在函数里如果返回 `Result`，可以用 `?` 快速向上传递错误：

```rust
use std::fs;

fn read_text(path: &str) -> Result<String, std::io::Error> {
    let content = fs::read_to_string(path)?;
    Ok(content)
}
```

`?` 的意思是：
1. 成功就拿到 `Ok` 里的值继续跑
2. 失败就直接返回 `Err`

写业务代码时这个非常高频。

## 7. 小结
这四件套你可以这样记：

1. `struct`：一条记录有很多字段
2. `enum`：一个值有很多形态
3. `Option`：值可能存在，也可能不存在
4. `Result`：要么成功值，要么错误值

会了它们，你的 Rust 就不是“只会语法”，而是开始有工程味道了。

### 课后小练习
1. 写一个 `struct Book`，包含标题、页数、是否上架。
2. 写一个 `enum Role`：`Admin / User / Guest`，并用 `match` 打印权限说明。
3. 写一个函数把字符串转整数，返回 `Result<i32, String>`。
4. 写一个函数按 id 查询商品名，返回 `Option<String>`。

### 下一章
下一篇我们写一个迷你 CLI 小项目，把前面这些知识串起来。
