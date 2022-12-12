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




## sea_orm

+ [SeaORM 是一种关系型 ORM，可帮助您在熟悉动态语言的情况下使用 Rust 构建 Web 服务。](https://docs.rs/sea-orm/latest/sea_orm/#seaorm-is-a-relational-orm-to-help-you-build-web-services-in-rust-with-the-familiarity-of-dynamic-languages)
+ 详细介绍使用可以点击上面的链接学习使用

`Cargo.toml`

添加`sea_orm`

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

# sea-orm
sea-orm = "0"

```

+ 在后面的不同的库使用`sea_orm`必须要添加`feature`说明使用的是哪个数据库和异步运行时，后面添加时再说明

  ```toml
  sea-orm = { version = "^0", features = [ <DATABASE_DRIVER>, <ASYNC_RUNTIME>, "macros" ] }
  ```



`migration`

+ 使用SeaORM自带的一个迁移工具，对我们的数据库进行版本控制

1. 创建迁移

   1. 先安装工具

   ```shell
   cargo install sea-orm-cli
   ```

   2. 初始化

   ```shell
   sea-orm-cli migrate init
   ```

   ![image-20221212152457706](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221212152457706.png)

   可以看到我们多了一个迁移目录

   ![image-20221212152541600](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221212152541600.png)

2. 配置

   `Cargo.toml`

   ```toml
   [workspace]
   members =[
       "app",
       "configs",
       "utils",
       "service",
       "model",
       "router",
       "migration",
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
   
   # sea-orm
   sea-orm = "0"
   
   ```

3. `migration/Cargo.toml`

   将`features`的两个注释去掉，我们这边数据库是使用的`postgres`

   ```toml
   [package]
   name = "migration"
   version = "0.1.0"
   edition = "2021"
   publish = false
   
   [lib]
   name = "migration"
   path = "src/lib.rs"
   
   [dependencies]
   async-std = { version = "^1", features = ["attributes", "tokio1"] }
   
   [dependencies.sea-orm-migration]
   version = "^0.10.0"
   features = [
     # Enable at least one `ASYNC_RUNTIME` and `DATABASE_DRIVER` feature if you want to run migration via CLI.
     # View the list of supported features at https://www.sea-ql.org/SeaORM/docs/install-and-config/database-and-async-runtime.
     # e.g.
      "runtime-tokio-rustls",  # `ASYNC_RUNTIME` feature
      "sqlx-postgres",         # `DATABASE_DRIVER` feature
   ]
   
   ```

4. git提交

   ```shell
   git add .
   git commit -m "新增数据库版本控制"
   ```

   

5. 数据库连接

   `utils/Cargo.toml`

   + 添加所需要的库

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
     # db
     sea-orm = { workspace = true,  features = ["macros", "runtime-tokio-rustls", "with-chrono","sqlx-postgres"] }
     tokio = {workspace = true}
     once_cell = {workspace = true}
     
     ```

   `utils/src/db.rs`

   + 创建数据库初始化连接代码

     ```rust
     use std::time::Duration;
     use sea_orm::{entity::prelude::DatabaseConnection, ConnectOptions, Database};
     use tokio::sync::OnceCell;
     use configs::CFG;
     
     //  异步初始化数据库
     pub static DB: OnceCell<DatabaseConnection> = OnceCell::const_new();
     
     pub async fn init() -> DatabaseConnection {
         let mut opt = ConnectOptions::new(CFG.database.url.to_owned());
         opt.max_connections(1000)
             .min_connections(5)
             .connect_timeout(Duration::from_secs(10))
             .idle_timeout(Duration::from_secs(10))
             .sqlx_logging(true);
         let db = Database::connect(opt).await.expect("数据库连接失败");
         db
     }
     
     ```

   `Config.toml`

   + 新增数据库连接信息

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
     
     [database]
     url = "postgres://postgres:123456@localhost/ava"
     ```

   `configs/src/config.rs`

   + 更新config.rs

     ```rust
     use serde::Deserialize;
     
     /// 配置文件
     #[derive(Debug, Deserialize)]
     pub struct Configs {
         /// 程序配置
         pub server: Server,
         /// 日志配置
         pub log: Log,
         pub database: Database,
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
     
     
     /// 数据库配置
     #[derive(Debug, Deserialize)]
     pub struct Database {
         /// `url` 数据库连接
         pub url: String,
     
     }
     
     ```

   `app/Cargo.toml`

    + 添加所需库

      ```rust
      [package]
      name = "app"
      version = "0.1.0"
      edition = "2021"
      
      # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
      
      [dependencies]
      utils = {path = "../utils"}
      configs = {path = "../configs"}
      migration = {path = "../migration"}
      axum = { workspace = true }
      serde_json = { workspace = true }
      tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
      tracing = {workspace = true}
      
      ```

      

   `app/src/main.rs`

   + 初始化连接

     ```rust
     use std::net::SocketAddr;
     use std::str::FromStr;
     use axum::{routing::get, Router};
     use tracing::{ info};
     use configs::CFG;
     use migration::{Migrator, MigratorTrait};
     use utils::db::{DB, init};
     use utils::log;
     
     #[tokio::main]
     async fn main() {
         // 初始化日志
         let _guard = log::init();
         info!("Starting");
     
         // 初始化数据库
         let db = DB.get_or_init(init).await;
         // 设置数据库迁移
         Migrator::up(db, None).await.unwrap();
     
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

   6. git提交

      ```shell
      git add .
      git commit -m "数据库初始化"
      ```

      

   



