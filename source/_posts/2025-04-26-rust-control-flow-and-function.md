---
title: Rust 控制流与函数
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-26 20:00:00
---

上一章我们把变量和数据类型搞定了。
这一章开始让程序“动起来”，不然一直 `let let let` 也挺无聊的。

## 1. `if` 表达式
Rust 的 `if` 不是语句，是表达式，也就是说它能直接返回值。

```rust
let score = 85;
let level = if score >= 60 { "及格" } else { "挂科" };
println!("{level}");
```

注意点：
1. 条件必须是 `bool`，不能拿数字硬凑。
2. `if` 和 `else` 返回类型要一致。

## 2. 三种循环：`loop`、`while`、`for`
### `loop`：无限循环，配合 `break`

```rust
let mut n = 0;
let result = loop {
    n += 1;
    if n == 10 {
        break n * 2;
    }
};
println!("{result}"); // 20
```

看见没，`break` 还能带返回值，这个很实用。

### `while`：条件循环

```rust
let mut n = 3;
while n != 0 {
    println!("{n}");
    n -= 1;
}
println!("发射！");
```

### `for`：最常用，最安全

```rust
let arr = [10, 20, 30];
for x in arr {
    println!("{x}");
}
```

如果你要遍历区间：

```rust
for i in 1..=5 {
    println!("{i}");
}
```

`1..=5` 表示包含 5，`1..5` 表示不包含 5。

## 3. `match`：Rust 招牌动作
`match` 可以理解成“超级加强版 switch”。

```rust
let coin = "btc";

match coin {
    "btc" => println!("比特币"),
    "eth" => println!("以太坊"),
    _ => println!("其他币"),
}
```

每个分支都要覆盖到，不允许漏判断，这就是 Rust 稳的地方。

你还能这样写范围匹配：

```rust
let score = 92;
match score {
    90..=100 => println!("A"),
    80..=89 => println!("B"),
    60..=79 => println!("C"),
    _ => println!("D"),
}
```

## 4. `if let`：只关心一种情况
有时候你只想匹配一种模式，`match` 写全套就有点重。

```rust
let maybe_name = Some("zhenyi");

if let Some(name) = maybe_name {
    println!("hello {name}");
}
```

干净利落，够用了。

## 5. 函数：输入什么，输出什么
Rust 函数长这样：

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

最后一行没有分号，表示“把这个值返回出去”。
你要是写成 `a + b;`，那就变成返回 `()` 了，新手很容易在这踩坑。

调用：

```rust
let sum = add(3, 7);
println!("{sum}");
```

## 6. 一个完整小例子
写个简单函数，判断一个数字等级：

```rust
fn level(score: u8) -> &'static str {
    match score {
        90..=100 => "优秀",
        60..=89 => "合格",
        _ => "继续努力",
    }
}

fn main() {
    let s = 77;
    println!("成绩：{}，评级：{}", s, level(s));
}
```

## 7. 结尾
到这里，Rust 的基础控制流和函数你已经摸清主线了。
接下来能做什么？写个小命令行项目，哪怕只支持两三个命令，也比光看概念强。

### 课后小练习
1. 用 `for` 循环打印 1 到 100 的偶数。
2. 写一个函数 `max(a, b)` 返回较大值。
3. 用 `match` 把 1-7 映射成星期几。

### 下一章
下一篇我们开始上 Rust 核心概念：所有权、借用、引用。
