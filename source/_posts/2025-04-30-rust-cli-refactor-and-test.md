---
title: Rust CLI 项目模块化重构 + 测试实战
tags: [rust]
categories: [rust]
layout: post
date: 2025-04-30 20:00:00
---

上一章我们把迷你待办 CLI 跑起来了。
这一章不加新功能，先做两件更工程化的事：

1. 模块化重构（代码不再一坨在 `main.rs`）
2. 加测试（以后改代码不慌）

这一步非常关键，很多人写到这就开始“技术债爆炸”，我们今天把坑先填了。

## 1. 重构目标
先定规则，不然重构会越改越乱：

1. `main.rs` 只做参数入口，不放业务细节
2. 数据结构、存储、命令逻辑分离
3. 让核心逻辑可以单测，不依赖命令行输入

## 2. 建议目录结构
把项目整理成这样：

```txt
src/
├─ main.rs
├─ app.rs
├─ model.rs
├─ store.rs
└─ commands.rs
```

每个文件只干一件事，别贪。

## 3. 代码拆分示例
下面给你一套可直接参考的拆法。

### `src/model.rs`

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Task {
    pub done: bool,
    pub content: String,
}
```

### `src/store.rs`

```rust
use crate::model::Task;
use std::fs;
use std::io;
use std::path::Path;

pub fn load_tasks(path: &str) -> io::Result<Vec<Task>> {
    if !Path::new(path).exists() {
        return Ok(Vec::new());
    }

    let content = fs::read_to_string(path)?;
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

pub fn save_tasks(path: &str, tasks: &[Task]) -> io::Result<()> {
    let mut lines = Vec::new();
    for t in tasks {
        let flag = if t.done { "1" } else { "0" };
        lines.push(format!("{}|{}", flag, t.content));
    }
    fs::write(path, lines.join("\n"))?;
    Ok(())
}
```

### `src/commands.rs`

```rust
use crate::model::Task;
use crate::store::{load_tasks, save_tasks};

pub fn add(path: &str, content: String) -> Result<(), String> {
    let mut tasks = load_tasks(path).map_err(|e| e.to_string())?;
    tasks.push(Task {
        done: false,
        content,
    });
    save_tasks(path, &tasks).map_err(|e| e.to_string())
}

pub fn list(path: &str) -> Result<Vec<Task>, String> {
    load_tasks(path).map_err(|e| e.to_string())
}

pub fn done(path: &str, idx: usize) -> Result<(), String> {
    if idx == 0 {
        return Err(String::from("序号从 1 开始"));
    }

    let mut tasks = load_tasks(path).map_err(|e| e.to_string())?;
    if idx > tasks.len() {
        return Err(format!("序号超出范围，当前共 {} 条任务", tasks.len()));
    }

    tasks[idx - 1].done = true;
    save_tasks(path, &tasks).map_err(|e| e.to_string())
}
```

### `src/app.rs`

```rust
use crate::commands;

pub const DATA_FILE: &str = "todo.txt";

pub fn run(args: &[String]) -> Result<String, String> {
    if args.len() < 2 {
        return Ok(help_text());
    }

    match args[1].as_str() {
        "add" => {
            if args.len() < 3 {
                return Err(String::from("add 命令缺少任务内容"));
            }
            let content = args[2..].join(" ");
            commands::add(DATA_FILE, content)?;
            Ok(String::from("已新增任务"))
        }
        "list" => {
            let tasks = commands::list(DATA_FILE)?;
            if tasks.is_empty() {
                return Ok(String::from("当前没有任务"));
            }
            let mut out = String::new();
            for (i, t) in tasks.iter().enumerate() {
                let mark = if t.done { "[x]" } else { "[ ]" };
                out.push_str(&format!("{} {} {}\n", i + 1, mark, t.content));
            }
            Ok(out.trim_end().to_string())
        }
        "done" => {
            if args.len() < 3 {
                return Err(String::from("done 命令缺少序号"));
            }
            let idx: usize = args[2]
                .parse()
                .map_err(|_| String::from("序号必须是数字"))?;
            commands::done(DATA_FILE, idx)?;
            Ok(format!("任务 {} 已完成", idx))
        }
        _ => Ok(help_text()),
    }
}

fn help_text() -> String {
    [
        "todo - 迷你待办 CLI",
        "用法:",
        "  todo add \"任务内容\"",
        "  todo list",
        "  todo done <序号>",
    ]
    .join("\n")
}
```

### `src/main.rs`

```rust
mod app;
mod commands;
mod model;
mod store;

use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    match app::run(&args) {
        Ok(text) => println!("{text}"),
        Err(err) => eprintln!("错误: {err}"),
    }
}
```

这样一拆，后面你加 `remove`、`clear` 都很顺手。

## 4. 测试怎么下手
重点：先测“核心逻辑”，别急着测 `main`。
因为 `main` 通常只是胶水代码。

我们重点测两个地方：
1. `store` 的读写
2. `commands` 的行为

## 5. 写第一个测试：存取一致
在 `src/store.rs` 里加测试模块：

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::time::{SystemTime, UNIX_EPOCH};

    fn temp_file() -> String {
        let ts = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_nanos();
        format!("todo_test_{}.txt", ts)
    }

    #[test]
    fn save_then_load_should_keep_data() {
        let path = temp_file();
        let data = vec![
            Task { done: false, content: "a".into() },
            Task { done: true, content: "b".into() },
        ];

        save_tasks(&path, &data).unwrap();
        let loaded = load_tasks(&path).unwrap();

        assert_eq!(loaded, data);

        std::fs::remove_file(path).ok();
    }
}
```

这条用例很核心，能保证你文件格式没写炸。

## 6. 再加命令测试：完成任务逻辑
在 `src/commands.rs` 里：

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::time::{SystemTime, UNIX_EPOCH};

    fn temp_file() -> String {
        let ts = SystemTime::now()
            .duration_since(UNIX_EPOCH)
            .unwrap()
            .as_nanos();
        format!("todo_cmd_test_{}.txt", ts)
    }

    #[test]
    fn done_should_mark_target_task() {
        let path = temp_file();
        add(&path, "task1".into()).unwrap();
        add(&path, "task2".into()).unwrap();

        done(&path, 2).unwrap();
        let tasks = list(&path).unwrap();

        assert!(!tasks[0].done);
        assert!(tasks[1].done);

        std::fs::remove_file(path).ok();
    }
}
```

这条测试能防止你以后改索引逻辑时出现“错标完成”的回归 bug。

## 7. 运行测试

```shell
cargo test
```

如果你看到 `test result: ok`，说明这轮重构至少没把核心功能改坏。

## 8. 实战建议（非常重要）
很多人写测试写崩溃，是因为一上来就追求“全覆盖”。
正确姿势是：

1. 先测关键路径（读写、索引、错误处理）
2. 每加一个命令加一两个测试
3. 修过的 bug 必须补回归测试

测试不是 KPI，是你的保险丝。

## 9. 结尾
到这一步，你的 CLI 就从“能跑”进化到“可维护”了。
以后功能再多，你也不会一改就心慌。

这就是模块化 + 测试的价值。

### 课后小练习
1. 给 `done` 增加“重复完成提示”，并写测试。
2. 增加 `remove` 命令，同时补 3 条测试（正常、越界、空列表）。
3. 把测试里的临时文件逻辑抽成公共函数，减少重复代码。

### 下一章
下一篇我们把 CLI 再升级一版：接入 `clap` 做专业命令行参数解析。
