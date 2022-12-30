## 优化redis和captcha

1. 将redis的配置信息写入配置文件中，以后修改redis的链接直接从配置文件中修改
2. 将captcha设置的一些参数写入配置文件中，同上方便以后修改



### 配置文件

`Config.toml`

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

[redis]
url = "redis://localhost"


[captcha]
length = 6
noise = 0.5
width = 220
height = 80

```



### Configs

`configs/src/config.rs`

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
    pub redis: Redis,
    pub captcha: Captcha,
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

/// Redis配置
#[derive(Debug, Deserialize)]
pub struct Redis {
    /// `url` redis连接
    pub url: String,
}

/// 验证码配置
#[derive(Debug, Deserialize)]
pub struct Captcha {
    /// [`length`] 验证码长度
    pub length: u32,
    pub noise: f32,
    pub width: u32,
    pub height: u32,
}

```



### Utils

`utils/src/captcha.rs`

```rust
use crate::rand::Random;
use crate::redis::{connect, set};
use anyhow::Result;
use captcha::filters::Noise;
use captcha::Captcha;
use configs::CFG;

pub async fn new() -> Result<(String, String)> {
    let (captcha_id, captcha_code, captcha_img) = generate();
    let mut con = connect().await?;
    set(
        &mut con,
        captcha_id.as_str(),
        captcha_code.as_str(),
        60 * 15,
    )
    .await?;

    Ok((captcha_id, captcha_img))
}

pub fn generate() -> (String, String, String) {
    let captcha_id = Random::new(12).generate();

    let mut captcha = Captcha::new();
    let captcha_code = captcha.add_chars(CFG.captcha.length).chars_as_string();
    let captcha_img = captcha
        .apply_filter(Noise::new(CFG.captcha.noise))
        .view(CFG.captcha.width, CFG.captcha.height)
        .as_base64()
        .expect("captcha  create failed");

    (captcha_id, captcha_code, captcha_img)
}

```



`utils/src/redis.rs`

```rust
use configs::CFG;
use redis::aio::Connection;
use redis::{Client, RedisResult};
use tracing::info;

pub async fn connect() -> RedisResult<Connection> {
    let client = Client::open(CFG.redis.url.as_str())?;
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

