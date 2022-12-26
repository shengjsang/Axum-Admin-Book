# 验证码

+ 生成一个`captcha_id`将生成的验证码作为它的值存储到`redis`中，并设置过期时间
+ 生成一个随机验证码，并将其编码成`base64`
+ 将编码后的`captcha_img`和`captcha_id`返回给前端



## 添加依赖

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

```



## Utils

`utils/Cargo.toml`

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
redis = {workspace = true, features = ["tokio-comp"] }
anyhow = {workspace = true}
captcha = {workspace = true}
rand = {workspace = true}

```



`utils/src/lib.rs`

```rust
pub mod captcha;
pub mod db;
pub mod log;
pub mod redis;

```



`utils/src/captcha`

+ 设置id的长度，随机内容

+ 设置验证码长度
+ 将`验证码id`和`验证码`保存到redis
+ 设置混淆点
+ 宽高
+ 将验证码图片转为base64返回给前端

```rust
use crate::redis::{connect, set};
use anyhow::Result;
use captcha::filters::Noise;
use captcha::Captcha;
use rand::Rng;

pub async fn new() -> Result<()> {
    const CHARSET: &[u8] = b"ABCDEFGHIJKLMNOPQRSTUVWXYZ\
                            abcdefghijklmnopqrstuvwxyz\
                            0123456789";
    const CAPTCHA_ID_LEN: usize = 16;
    let mut rng = rand::thread_rng();

    let captcha_id: String = (0..CAPTCHA_ID_LEN)
        .map(|_| {
            let idx = rng.gen_range(0..CHARSET.len());
            CHARSET[idx] as char
        })
        .collect();

    let mut captcha = Captcha::new();
    let captcha_code = captcha.add_chars(6).chars_as_string();
    let _captcha_img = captcha
        .apply_filter(Noise::new(0.5))
        .view(220, 80)
        .as_base64()
        .expect("captcha  create failed");
    let mut con = connect().await.unwrap();
    set(
        &mut con,
        captcha_id.as_str(),
        captcha_code.as_str(),
        60 * 15,
    )
    .await
    .unwrap();
    Ok(())
}

```



## 测试

`app/src/main.rs`

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

    let _ = captcha::new().await.unwrap();
    info!("Captcha Generated");
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



![image-20221226161053467](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221226161053467.png)

![image-20221226165405489](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221226165405489.png)
