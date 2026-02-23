---
title: Rust 迷你 CLI 小项目实战
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-29 20:00:00
---

前面语法你学了不少，这一篇我们不聊概念了，直接干一个能跑的。
目标：做一个迷你待办 CLI 工具，支持新增、查看、完成任务。

别怕，项目很小，但很像真实开发流程。

## 1. 先定需求
我们要的命令长这样：

```shell
todo add "学习 Rust"
todo list
todo done 1
```

功能就 3 个：
1. `add`：新增任务
2. `list`：查看任务
3. `done`：按序号标记完成

数据先用本地文本文件存，简单直接。

## 2. 创建项目

```shell
cargo new todo-cli
cd todo-cli
```

项目结构先保持默认就行。

## 3. 核心代码（`src/main.rs`）
先给完整代码，你先跑起来，后面我再拆开讲。

```rust
use std::env;
use std::fs;
use std::io;
use std::path::Path;

const DATA_FILE: &str = "todo.txt";

#[derive(Debug, Clone)]
struct Task {
    done: bool,
    content: String,
}

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        print_help();
        return;
    }

    let result = match args[1].as_str() {
        "add" => cmd_add(&args),
        "list" => cmd_list(),
        "done" => cmd_done(&args),
        _ => {
            print_help();
            Ok(())
        }
    };

    if let Err(e) = result {
        eprintln!("错误: {e}");
    }
}

fn print_help() {
    println!("todo - 迷你待办 CLI");
    println!("用法:");
    println!("  todo add \"任务内容\"");
    println!("  todo list");
    println!("  todo done <序号>");
}

fn cmd_add(args: &[String]) -> Result<(), String> {
    if args.len() < 3 {
        return Err(String::from("add 命令缺少任务内容"));
    }

    let content = args[2..].join(" ");
    let mut tasks = load_tasks().map_err(|e| e.to_string())?;
    tasks.push(Task {
        done: false,
        content,
    });
    save_tasks(&tasks).map_err(|e| e.to_string())?;
    println!("已新增任务");
    Ok(())
}

fn cmd_list() -> Result<(), String> {
    let tasks = load_tasks().map_err(|e| e.to_string())?;
    if tasks.is_empty() {
        println!("当前没有任务");
        return Ok(());
    }

    for (i, t) in tasks.iter().enumerate() {
        let mark = if t.done { "[x]" } else { "[ ]" };
        println!("{} {} {}", i + 1, mark, t.content);
    }

    Ok(())
}

fn cmd_done(args: &[String]) -> Result<(), String> {
    if args.len() < 3 {
        return Err(String::from("done 命令缺少序号"));
    }

    let idx: usize = args[2]
        .parse()
        .map_err(|_| String::from("序号必须是数字"))?;

    if idx == 0 {
        return Err(String::from("序号从 1 开始"));
    }

    let mut tasks = load_tasks().map_err(|e| e.to_string())?;
    if idx > tasks.len() {
        return Err(format!("序号超出范围，当前共 {} 条任务", tasks.len()));
    }

    tasks[idx - 1].done = true;
    save_tasks(&tasks).map_err(|e| e.to_string())?;
    println!("任务 {} 已完成", idx);
    Ok(())
}

fn load_tasks() -> io::Result<Vec<Task>> {
    if !Path::new(DATA_FILE).exists() {
        return Ok(Vec::new());
    }

    let content = fs::read_to_string(DATA_FILE)?;
    let mut tasks = Vec::new();

    for line in content.lines() {
        if line.trim().is_empty() {
            continue;
        }
        let done = line.starts_with("1|");
        let text = line.get(2..).unwrap_or("").to_string();
        tasks.push(Task {
            done,
            content: text,
        });
    }

    Ok(tasks)
}

fn save_tasks(tasks: &[Task]) -> io::Result<()> {
    let mut lines = Vec::new();
    for t in tasks {
        let flag = if t.done { "1" } else { "0" };
        lines.push(format!("{}|{}", flag, t.content));
    }
    fs::write(DATA_FILE, lines.join("\n"))?;
    Ok(())
}
```

## 4. 代码拆解（重点看思路）
你会发现我们用到了前几章几乎所有知识点：

1. `struct Task`：把任务数据组织起来
2. `Result<(), String>`：命令执行结果可成功可失败
3. `match`：根据子命令走不同分支
4. 借用 `&[String]`：函数读参数不抢所有权

再看存储格式：

```txt
0|学习 Rust
1|写一篇博客
```

`0` 表示未完成，`1` 表示已完成，后面跟任务内容。
很土，但够稳定，先把功能跑通比花哨重要。

## 5. 运行试试

```shell
cargo run -- add "学习 Rust"
cargo run -- add "写一个小工具"
cargo run -- list
cargo run -- done 1
cargo run -- list
```

如果你看到任务状态从 `[ ]` 变 `[x]`，就说明项目成功了。

## 6. 你可以怎么升级这个项目
第一版能用，接下来慢慢加功能：

1. 增加 `remove <序号>` 删除任务
2. 增加 `clear` 清空已完成任务
3. 用 `clap` 做更专业的参数解析
4. 把文本存储改成 JSON
5. 加单元测试，至少测 `load/save` 和参数错误处理

## 7. 结尾
你看，Rust 入门最关键不是“看懂了”，是“跑起来了”。
这个迷你 CLI 虽然小，但已经有一点工程味道了。

到这里你应该已经能自己做第二个、第三个小工具了。

### 课后小练习
1. 自己加上 `remove` 命令。
2. 给 `done` 增加重复完成提示（比如已完成就不重复写文件）。
3. 把错误信息统一封装成一个 `enum AppError`。

### 下一章
下一篇我们把这个 CLI 重构成多文件模块，并补上测试。
