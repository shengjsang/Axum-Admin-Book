# Redis



## 安装Redis

```shell
brew install redis	
```



## 启动Redis

```shell
brew services start redis
```



## 软件连接

![image-20221225163319293](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221225163319293.png)



## Rust集成

### 添加第三方依赖

`Cargo.toml`

```rust
[workspace]
members =[
    "app",
    "configs",
    "utils",
    "service",
    "model",
    "router",
    "migration",
    "api",
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

anyhow = "1"
scru128 = "2.3.0"
md5 = "0.7"
rand = "0.8.5"
captcha_rust = "0.1.3"
thiserror = "1.0.38"
redis = "0.22.0"

```



## Utils

`utils/Cargo.toml`

```rust
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
redis = {workspace = true, features = ["tokio-comp"] }

```

`utils/src/lib.rs`

```rust
pub mod db;
pub mod log;
pub mod redis;
```

`utils/src/redis.rs`

1. Redis连接功能
2. Set功能

```rust
use redis::aio::Connection;
use redis::{Client, RedisResult};
use tracing::info;

pub async fn connect() -> RedisResult<Connection> {
    let client = Client::open("redis://localhost")?;
    let mut con = client.get_tokio_connection().await?;
    let res: String = redis::Cmd::new()
        .arg("Ping")
        .query_async(&mut con)
        .await
        .unwrap();
    info!("Redis Ping {}", res);
    Ok(con)
}

pub async fn set(con: &mut Connection, key: &str, value: &str, expire: u32) -> RedisResult<()> {
    redis::Cmd::new()
        .arg("Set")
        .arg(key)
        .arg(value)
        .query_async(con)
        .await?;

    redis::Cmd::new()
        .arg("Expire")
        .arg(key)
        .arg(expire)
        .query_async(con)
        .await?;

    Ok(())
}

```



## App

```rust
use axum::Router;
use configs::CFG;
use router::api;
use std::net::SocketAddr;
use std::str::FromStr;
use tracing::info;
use utils::{captcha, log, redis};

#[tokio::main]
async fn main() {
    // 初始化日志
    let _guard = log::init();
    info!("Starting");

    // 连接Redis
    let _con = redis::connect().await.unwrap();
    info!("Redis Connect");

 
    let app = Router::new().nest("/v1", api());

    let addr = SocketAddr::from_str(&CFG.server.address).unwrap();
    // 设置端口
    axum::Server::bind(&addr)
        // 服务启动
        .serve(app.into_make_service())
        .await
        .unwrap();
}

```



## git提交

```shell
git add .
git commit -m "集成redis"
```

