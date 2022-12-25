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

