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



## 库

1. `tracing`

   + 一个有**范围**的、**结构化**的日志和诊断系统

   + 异步编程中使用传统的日志是存在一些问题的，因为异步任务执行起来没有确定的顺序，输出的时候没有确定的逻辑顺序。

   + `tracing`引入了`span`概念，一个span代表了从开始到结束的一个时间段，在此期间的信息都可以记录。

   + 5个日志级别

     + TRACE
     + DEBUG
     + INFO
     + WARN 
     + ERROR

     

   

2. `tracing-subscriber`
   + `tracing`本身不会记录日志，我们可以使用`tracing-subscriber`来收集`event!()`或者其他记录日志宏发出的日志。
   + 只需要在可执行程序中初始化`subscriber`，其他地方都可以直接使用日志相关宏来发出日志即可
3. `tracing_appender`
   1. 更改日志输出的一些默认设置
      1. 输出位置
      2. 颜色
      3. 日期
      4. 显示信息
      5. ...
   2. 使用`tracing_appender`时，要求必须在`main()`函数中使用`Guard`，否则`guard`被丢弃将不会写入任何信息到文件



## 代码

1. 创建`utils`库

   ```shell
   cargo new --lib utils
   ```

2. `Cargo.toml`

   ```rust
   [workspace]
   members =[
       "app",
       "configs",
       "utils"
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

   

3. `configs/config.rs`

   + 设计日志的相关配置信息

   ```rust
   use serde::Deserialize;
   
   /// 配置文件
   #[derive(Debug, Deserialize)]
   pub struct Configs {
       /// 程序配置
       pub server: Server,
       /// 日志配置
       pub log: Log,
   }
   
   
   /// server 配置文件
   #[derive(Debug, Deserialize)]
   pub struct Server {
       /// 服务器名称
       pub name: String,
       /// 服务器(IP地址:端口)
       /// `0.0.0.0:3000`
       pub address: String,
   }
   
   /// 日志配置
   #[derive(Debug, Deserialize)]
   pub struct Log {
       /// `log_level` 日志输出等级
       pub log_level: String,
       /// `dir` 日志输出文件夹
       pub dir: String,
       /// `prefix` 日志输出文件前缀名
       pub prefix: String,
   }
   ```

4. `Config.toml`

   ```toml
   [server]
   name = "Axum-Admin"
   # 服务器IP:端口
   address = "0.0.0.0:3000"
   
   [log]
   dir = "logs"
   prefix = "log"
   # 日志级别  TRACE DEBUG  INFO  WARN ERROR
   log_level = "DEBUG"
   
   ```

5. `utils/Cargo.toml`

   ```toml
   [package]
   name = "utils"
   version = "0.1.0"
   edition = "2021"
   
   # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   
   [dependencies]
   configs = {path = "../configs"}
   # time
   chrono = { workspace = true}
   time = { workspace = true }
   # log
   tracing = { workspace = true }
   tracing-appender = { workspace = true }
   tracing-subscriber = {workspace = true,default-features =false,  features = ["json", "env-filter", "local-time", "registry"]}
   
   ```

6. `utils/log.rs`

   ```rust
   use chrono::Local;
   use tracing::Level;
   use tracing_appender::non_blocking::WorkerGuard;
   use tracing_subscriber::fmt::format::{Compact, Format, Writer};
   use tracing_subscriber::layer::SubscriberExt;
   use tracing_subscriber::{EnvFilter, fmt, Registry};
   use tracing_subscriber::fmt::time::FormatTime;
   use configs::CFG;
   
   pub fn init() -> WorkerGuard {
   
       // 设置日志为每天轮换 - 存放目录 - 日志文件名前缀
       let file_appender = tracing_appender::rolling::daily(&CFG.log.dir, &CFG.log.prefix);
   
       let (non_blocking, file_guard) = tracing_appender::non_blocking(file_appender);
   
       // 从Config.toml 读取设置的日志显示级别
       let log_level = get_level();
   
       let logger = Registry::default()
           .with(EnvFilter::from_default_env().add_directive(log_level.into()))
           .with(fmt::Layer::default().with_writer(non_blocking).event_format(formats()));
   
   
       // 收集日志设置全局默认值 返回是否初始化成功
       tracing::subscriber::set_global_default(logger).unwrap();
   
       // 必须返回guard到main()函数 不然日志文件配置失败
       file_guard
   }
   
   // 设置日志打印的格式
   pub fn formats() -> Format<Compact, LocalTimer> {
       fmt::format()
           .with_ansi(false)
           .with_level(true)
           .with_target(true)
           .with_thread_ids(true)
           .with_timer(LocalTimer)
           .compact()
   }
   
   
   //  读取配置文件中的日志记录级别
   pub fn get_level() -> Level {
       match CFG.log.log_level.as_str() {
           "TRACE" => Level::TRACE,
           "DEBUG" => Level::DEBUG,
           "INFO" => Level::INFO,
           "WARN" => Level::WARN,
           "ERROR" => Level::ERROR,
           _ => Level::INFO,
       }
   }
   
   // 设置日志日期格式
   pub struct LocalTimer;
   
   impl FormatTime for LocalTimer {
       fn format_time(&self, w: &mut Writer<'_>) -> std::fmt::Result {
           write!(w, "{}", Local::now().format("%FT %T"))
       }
   }
   
   ```

7. `utils/lib.rs`

   ```rust
   pub mod log;
   ```

8. `app/Cargo.toml`

   ```rust
   [package]
   name = "app"
   version = "0.1.0"
   edition = "2021"
   
   # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   
   [dependencies]
   utils = {path = "../utils"}
   configs = {path = "../configs"}
   axum = { workspace = true }
   serde_json = { workspace = true }
   tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
   tracing = {workspace = true}
   
   ```

9. `app/main.rs`

   ```rust
   use std::net::SocketAddr;
   use std::str::FromStr;
   use axum::{routing::get, Router};
   use tracing::{ info};
   use configs::CFG;
   use utils::log;
   
   #[tokio::main]
   async fn main() {
       // 初始化日志
       let _guard = log::init();
       info!("Starting");
       // route函数来设置路由，两个参数 一个路由路径 一个handler函数
       // handler函数就是一个异步函数来处理程序逻辑，从请求中提取解析作为参数，并返回响应，响应要实现IntoResponse
       let app = Router::new().route("/", get(|| async { "Hello, World!" }));
   
       let addr =SocketAddr::from_str(&CFG.server.address).unwrap();
       // 设置端口
       axum::Server::bind(&addr)
           // 服务启动
           .serve(app.into_make_service())
           .await
           .unwrap();
   }
   
   ```

10. git提交

   ```shell
   git add .
   git commit -m "新增log"
   ```

   
