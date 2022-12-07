# 集成Axum

## Axum介绍

+ axum 是一个专注于人体工程学和模块化的 Web 应用程序框架。

+ 一些高级功能介绍和使用可以查看[官方文档](https://docs.rs/axum/latest/axum/)

+ 后续我们用到的功能边用边介绍



## 代码开发

1. 添加`axum`

   `Cargo.toml`

   ```toml
   [workspace]
   members = ["app"]
   
   [workspace.package]
   authors = ["shengj.sang <shengj.sang@icloud.com>"]
   edition = "2021"
   license = "MIT"
   publish = false
   repository = ""
   
   
   [workspace.dependencies]
   axum = "0"
   serde = "1"
   serde_json = "1"
   tokio = "1"
   
   ```

2. `app`集成`axum`

   `app/Cargo.toml`

   ```toml
   [package]
   name = "app"
   version = "0.1.0"
   edition = "2021"
   
   # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   
   [dependencies]
   axum = { workspace = true }
   serde_json = { workspace = true }
   tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
   
   ```

3. 修改`main.rs`

   1. Axum**官方示例**，最简单的启动http服务

   ```rust
   use axum::{
       routing::get,
       Router,
   };
   
   // axum 是和tokio与hyper组合使用的
   // 目前最受欢迎的异步运行时框架，功能强大，还提供了异步所需的各种工具(例如 tracing )、网络协议框架(例如 HTTP，gRPC )等等
   #[tokio::main]
   async fn main() {
     
       let app = Router::new().route("/", get(|| async { "Hello, World!" }));
   
       // run it with hyper on localhost:3000
       axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
           .serve(app.into_make_service())
           .await
           .unwrap();
   }
   ```

   2. 测试

   ![](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221207190132008.png)

   + 发送**get**请求，成功的返回了`Hello,World!`

4. git提交

   ```shell
   git add .
   git commit -m "集成axum"
   ```



## 其他说明

1. **Cargo.toml**里面的**dependencies**

   + 从`crates.io`上下载依赖包，只需要一个包名和版本号
   + 后续还可以直接引用存放本地的依赖包以及github上的依赖包

   

   
