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

+ 返回对应的响应来给前端开发者进行后续处理

+ 格式为：

  ```rust
  {
  	"code":
  	"data":
  	"msg":
  }
  ```

### Model

`model/Cargo.toml`

```
[package]
name = "model"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
sea-orm = {workspace = true}
serde = {workspace = true}
serde_json = {workspace = true}
axum = {workspace = true}
```

`model/response/mod.rs`

规范返回响应，并创建几个快速返回正确错误的响应方法

```rust
use std::fmt::Debug;

use axum::body;
use axum::body::Full;
use axum::http::StatusCode;
use axum::response::{IntoResponse, Response};
use serde::Serialize;

#[derive(Debug, Serialize, Default)]
pub struct Res<T> {
    pub code: Option<u32>,
    pub data: Option<T>,
    pub msg: Option<String>,
}

impl<T> IntoResponse for Res<T>
where
    T: Serialize + Send + Sync + Debug + 'static,
{
    fn into_response(self) -> Response {
        let res = Self {
            code: self.code,
            data: self.data,
            msg: self.msg,
        };

        let res = match serde_json::to_string(&res) {
            Ok(res) => res,
            Err(e) => {
                return Response::builder()
                    .status(StatusCode::INTERNAL_SERVER_ERROR)
                    .body(body::boxed(Full::from(e.to_string())))
                    .unwrap()
            }
        };
        res.into_response()
    }
}

impl<T: Serialize> Res<T> {
    pub fn new(code: u32, data: T, msg: String) -> Self {
        Self {
            code: Some(code),
            data: Some(data),
            msg: Some(msg),
        }
    }

    pub fn ok() -> Self {
        Self {
            code: Some(200),
            data: None,
            msg: None,
        }
    }

    pub fn ok_with_data(data: T) -> Self {
        Self {
            code: Some(200),
            data: Some(data),
            msg: Some("Success".to_string()),
        }
    }

    pub fn ok_with_msg(msg: String) -> Self {
        Self {
            code: Some(200),
            data: None,
            msg: Some(msg),
        }
    }

    pub fn error(code: u32) -> Self {
        Self {
            code: Some(code),
            data: None,
            msg: None,
        }
    }

    pub fn error_with_data(code: u32, data: T) -> Self {
        Self {
            code: Some(code),
            data: Some(data),
            msg: Some("Error".to_string()),
        }
    }

    pub fn error_with_msg(code: u32, msg: String) -> Self {
        Self {
            code: Some(code),
            data: None,
            msg: Some(msg),
        }
    }
}

```



### API

`api/user/mod.rs`

```rust
use axum::Json;
use model::response::Res;
use model::user::request::CreateReq;

use service::user::register;
use utils::db::{init, DB};

pub async fn create(Json(req): Json<CreateReq>) -> Res<String> {
    let db = DB.get_or_init(init).await;
    let res = register(db, req).await;

    match res {
        Ok(x) => Res::ok_with_msg(x),
        Err(e) => Res::error_with_msg(500, e.to_string()),
    }
}

```



## 测试

![image-20221222164125466](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221222164125466.png)

![image-20221222164141843](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221222164141843.png)

+ 数据库将`username`,`email`,`phone`这几个键设置成唯一键，不能出现重复的值。
+ 依次测试新用户注册，重复注册，用户名重复注册，邮件重复注册，手机重复注册等请求



### git提交

```shell
git add .
git commit -m "优化用户注册"
```

