# 日志系统设计

## 什么是日志？

我们使用**日志**记录整个系统的运行情况。包含以下情况：

+ HTTP请求
+ SQL请求
+ 错误
+ 第三方接口
+ ...



## 目的

+ 方便调试，更快的帮我们定位问题



## 使用场景

+ 本地开发
+ 线上



## 等级

+ debug
+ info
+ warn
+ error



## 代码

1. 创建一个utils库来实现我们的日志功能

   ```rust
   cargo new --lib utils
   ```

2. `Cargo.toml`添加所需要的依赖项

   ```rust
   chrono = "0"
   time = "0.3"
   tracing = "0.1"
   tracing-appender = "0.2"
   tracing-subscriber = "0.3"
   ```

3. `utils/Cargo.toml`导入

   ```rust
   [package]
   name = "utils"
   version = "0.1.0"
   edition = "2021"
   
   # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   
   [dependencies]
   chrono = { workspace = true}
   time = { workspace = true }
   tracing = { workspace = true }
   tracing-appender = { workspace = true }
   tracing-subscriber = { workspace = true }
   ```

4. 