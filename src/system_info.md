# 获取服务器状态信息

## Cargo.toml

+ 添加依赖 `sysinfo`

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
captcha = "0.0.9"
axum-macros = "0.3.0"
headers = "0.3.8"
sysinfo = "0.26.8"

```



## Service

`service/Cargo.toml`

```toml
[package]
name = "service"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
utils = {path = "../utils"}
model = {path = "../model"}
sea-orm = {workspace = true}
tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
anyhow = {workspace = true}
serde_json = {workspace = true}
serde = {workspace = true}
scru128 = {workspace = true }
thiserror = {workspace = true}
headers = {workspace = true}
sysinfo = {workspace = true}

```



`service/src/lib.rs`

```rust
pub mod system;
pub mod user;
```



`service/src/system/mod.rs`

```rust
pub mod info;
```



`service/src/system/info.rs`

```rust
use anyhow::Result;
use model::system::info::Info;
use sysinfo::{DiskExt, System, SystemExt};

const BYTE_GB: u64 = 1024 * 1024 * 1024;

pub fn get() -> Result<Info> {
    let mut sys = System::new_all();

    sys.refresh_all();

    Ok(Info {
        total_disk: sys.disks()[0].total_space() / BYTE_GB,
        unused_disk: sys.disks()[0].available_space() / BYTE_GB,
        total_memory: sys.total_memory() / BYTE_GB,
        unused_memory: sys.available_memory() / BYTE_GB,
        system_type: sys.name(),
        cpu_number: sys.cpus().len(),
    })
}

```



## Api

`api/src/lib.rs`

```rust
pub mod common;
pub mod system;
pub mod user;
```



`api/src/system/mod.rs`

```rust
pub mod info;
```



`api/src/system/info.rs`

```rust
use model::response::Res;
use model::system::info::Info;

pub async fn get_info() -> Res<Info> {
    match service::system::info::get() {
        Ok(info) => Res::ok_with_data(info),
        Err(_) => Res::error(500),
    }
}

```



## Model

`model/src/lib.rs`

```rust
pub mod common;
pub mod entity;
pub mod response;
pub mod system;
pub mod user;
```



`model/src/system/mod.rs`

```rust
pub mod info;
```



`model/src/system/info.rs`

```rust
use serde::Serialize;

#[derive(Debug, Serialize, Default)]
pub struct Info {
    pub total_disk: u64,
    pub unused_disk: u64,
    pub total_memory: u64,
    pub unused_memory: u64,
    pub system_type: Option<String>,
    pub cpu_number: usize,
}

```



## Router

`router/src/lib.rs`

```rust
use api::common::{show_captcha, test};
use api::system::info::get_info;
use api::user::{create, login};
use axum::routing::{get, post};
use axum::Router;

pub fn api() -> Router {
    // 嵌套path 方便我们对不同的功能进行细分和管理
    Router::new()
        .nest("/user", user_api()) //  /user/register
        .nest("/common", captcha_api())
        .nest("/system", system_api())
}

fn user_api() -> Router {
    Router::new()
        .route("/register", post(create)) // 注册
        .route("/login", post(login))
}

fn captcha_api() -> Router {
    Router::new()
        .route("/show-captcha", post(show_captcha))
        .route("/test", get(test))
}

fn system_api() -> Router {
    Router::new().route("/info", get(get_info))
}

```

