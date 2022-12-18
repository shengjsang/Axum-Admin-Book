# 优化用户注册



## 处理用户请求

上一小节，我们是将创建用户的值是写死的，而不是利用用户请求，我们现在讲读取用户请求并处理。



### 添加所需要的依赖

1. 验证码
2. 随机数
3. UUID

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

```



### Router

+ 将`register`的请求方法改为`post`

```rust
use api::user::create;
use axum::routing::{get, post};
use axum::Router;

pub fn api() -> Router {
    // 嵌套path 方便我们对不同的功能进行细分和管理
    Router::new().nest("/user", user_api()) //  /user/register
}

fn user_api() -> Router {
    Router::new().route("/register", post(create)) // 注册
}

```



### Model

`model/Cargo.toml`

+ 添加所需要使用的依赖

```toml
[package]
name = "model"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
sea-orm = {workspace = true}
serde = {workspace = true}
serde_json = {workspace = true}

```

`model/src/user/request/mod.rs`

+ 用户注册请求模型

```rust
use serde::Deserialize;
#[derive(Deserialize, Debug)]
pub struct CreateReq {
    pub username: String,
    pub password: String,
    pub email: String,
    pub phone: String,
}
```

`model/src/user/mod.rs`

```rust
pub mod request;
```

`model/src/lib.rs`

```rust
pub mod entity;
pub mod user;
```



### Api

`api/src/user/mod.rs`

+ `Json<CreateReq>`  **提取并解析请求**

```rust
use axum::Json;
use model::user::request::CreateReq;
use serde_json::{json, Value};
use service::user::register;
use utils::db::{init, DB};

pub async fn create(Json(req): Json<CreateReq>) -> Json<Value> {
    let db = DB.get_or_init(init).await;
    let res = register(db, req).await;

    match res {
        Ok(_x) => Json(json!({"data": 200 })),
        Err(_e) => Json(json!({"data": 404 })),
    }
}

```



### Service

`service/Cargo.toml`

+ 添加要使用的依赖

```
[package]
name = "service"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
model = {path = "../model"}
sea-orm = {workspace = true}
tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
anyhow = {workspace = true}
serde_json = {workspace = true}
serde = {workspace = true}
scru128 = {workspace = true }

```

`service/user/mod.rs`

+ `scru128::new_string()` 创建一个新的用户id
+ `user::Entity::insert(user).exec(db).await?` 插入数据库

```rust
use anyhow::Result;
use model::entity::user;
use model::user::request::CreateReq;
use sea_orm::prelude::Json;
use sea_orm::ActiveValue::Set;
use sea_orm::DatabaseConnection;
use sea_orm::EntityTrait;
use std::process::id;

pub async fn register(db: &DatabaseConnection, req: CreateReq) -> Result<String> {
    let id = scru128::new_string();
    let user = user::ActiveModel {
        id: Set(id),
        username: Set(req.username),
        phone: Set(req.phone),
        email: Set(req.email),
        password: Set(req.password),
    };

    user::Entity::insert(user).exec(db).await?;

    Ok("用户添加成功".to_string())
}

```



## 测试

1. 启动程序

2. 发送请求

   + 方法改为`post`
   + 去掉`id`

   ![image-20221218133726357](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221218133726357.png)

3. 查看数据库

   ![image-20221218133804074](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221218133804074.png)

4. 成功

5. git提交

   ```shell
   git add .
   git commit -m "读取解析用户请求"
   ```
   



## 处理用户响应

1. 返回对应的响应来告知用户操作是否成功，哪里出错了







