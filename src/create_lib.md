## 创建相关库

1. 创建`Router`库

   HTTP API 接口的具体实现，主要用来做 HTTP 请求的解包、参数校验、业务逻辑处理、返回。这里的业务逻辑处理应该是轻量级的。
2. 创建`Service`库

   存放应用复杂业务处理代码。
3. 创建`Model`库

   存放和数据库交互的代码。

   ```shell
   cargo new --lib router
   cargo new --lib service
   cargo new --lib model
   ```



4. 更新`Cargo.toml`，将上面的库添加

   ```toml
   [workspace]
   members =[
       "app",
       "configs",
       "utils",
       "service",
       "model",
       "router",
   ]
   
   [workspace.package]
   authors = ["shengj.sang <shengj.sang@icloud.com>"]
   edition = "2021"
   license = "MIT"
   publish = false
   repository = ""
   
   
   [workspace.dependencies]
   # basic
   serde = "1"
   serde_json = "1"
   
   # web
   axum = "0"
   tokio = "1"
   
   # config
   once_cell = "1"
   toml = "0.5"
   
   
   # time
   time = "0.3"
   chrono = "0"
   
   # log
   tracing = "0.1"
   tracing-appender = "0.2"
   tracing-subscriber = "0.3"
   
   ```
