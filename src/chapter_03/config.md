# 设计配置信息

## 为何要设计？

`app/src/main.rs`

```rust
axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
```

我们可以看出端口是写死在代码里的，后续的开发中还有很多信息，比如

+ 数据库连接信息
+ Redis连接信息
+ Log的级别
+ 第三方的一些key和secret
+ ...



## 代码

1. 创建一个`config`库，将我们的配置信息写入

   ```rust
   cargo new --lib configs
   ```

   创建后项目目录结构为：

   ![image-20221208095404481](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221208095404481.png)

2. 添加到`worksapce.members`，并添加config所需要的新依赖项

   + `once_cell`
   + `toml`

   

   `Cargo.toml`

   ```toml
   [workspace]
   members =[
       "app",
       "configs"
   ]
   
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
   once_cell = "1"
   toml = "0.5"
   
   ```

3. 将所需依赖项添加到`configs/Cargo.toml`

   ```toml
   [package]
   name = "configs"
   version = "0.1.0"
   edition = "2021"
   
   # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
   
   [dependencies]
   once_cell = {workspace = true}
   serde = {workspace = true, features = ["derive"]}
   toml = {workspace = true}
   
   ```

   

4. 在根目录创建`Config.toml`

   ```toml
   [server]
   name = "Axum-Admin"
   # 服务器IP:端口
   address = "0.0.0.0:3000"
   
   ```

   后续用到其他配置信息的时候再增加

5. 在`configs/src/`下面创建`config.rs`，用来反序列化，将`toml`转化为里面的`Configs`结构体的值

   ```rust
   use serde::Deserialize;
   
   /// 配置文件
   #[derive(Debug, Deserialize)]
   pub struct Configs {
       /// 程序配置
       pub server: Server,
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
   
   ```

6. 将`configs/src/lib.rs`内容替换以下内容

   1. `once_cell` 

      用来存储堆上的信息，并且具有最多只能赋值一次的特性

      `Lazy`：在第一次使用的时候产生一个动态的全局静态变量

   2. `toml`

      实现了一个[TOML](https://github.com/toml-lang/toml) v0.5.0 兼容的解析器，主要支持[`serde`](https://serde.rs/)在 Rust 中编码/解码各种类型的库。

   ```rust
   use std::{fs::File, io::Read};
   use once_cell::sync::Lazy;
   use crate::config::Configs;
   mod config;
   
   
   // 设置配置文件路径
   const CFG_FILE: &str = "Config.toml";
   // 在第一次访问的时候初始化值 静态的 之后可以一直使用
   pub static CFG: Lazy<Configs> = Lazy::new(Configs::init);
   
   impl Configs {
       pub fn init() -> Self {
           let mut file = match File::open(CFG_FILE) {
               Ok(f) => f,
               Err(e) => panic!("找不到配置文件：{}，错误信息：{}", CFG_FILE, e),
           };
           let mut config_info = String::new();
           // 读取内容
           file.read_to_string(&mut config_info).expect("读取配置文件内容错误");
           // toml -> Configs
           toml::from_str(&config_info).expect("解析配置文件错误")
       }
   }
   
   ```

7. 在`app/src/main.rs`中使用

   1. 先将configs库添加到`app/Cargo.toml`

      ```toml
      [package]
      name = "app"
      version = "0.1.0"
      edition = "2021"
      
      # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
      
      [dependencies]
      configs = {path = "../configs"}
      axum = { workspace = true }
      serde_json = { workspace = true }
      tokio = {workspace = true,default-features =false,  features = ["rt-multi-thread", "macros", "parking_lot", "signal"]}
      ```

   2. app/src/main.rs直接使用，因为它是全局静态变量

      ```rust
      use std::net::SocketAddr;
      use std::str::FromStr;
      use axum::{routing::get, Router};
      use configs::CFG;
      
      #[tokio::main]
      async fn main() {
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

8. 运行程序并测试

   ```rust
   cargo run --bin app
   ```

   ![image-20221208172413166](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221208172413166.png)

9. git提交

   ```shell
   git add .
   git commit -m "添加config配置文件"
   ```

   
