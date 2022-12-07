# 集成Axum

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
   ```

3. 修改`main.rs`

   ```rust
   use axum::{
       routing::get,
       Router,
   };
   
   #[tokio::main]
   async fn main() {
       // build our application with a single route
       let app = Router::new().route("/", get(|| async { "Hello, World!" }));
   
       // run it with hyper on localhost:3000
       axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
           .serve(app.into_make_service())
           .await
           .unwrap();
   }
   ```

   
