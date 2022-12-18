# 创建项目

1. 创建项目目录

   ```shell
   mkdir Axum-Admin 
   cd Axum-Admin 
   touch Cargo.toml
   ```

2. 使用编辑器打开目录，可以使用`vscode`，个人用的是`clion`

3. 版本控制

   ```shell
   git init .
   git add .
   git commit -m "创建项目目录"
   ```

4. 添加`.gitignore`

   ```shell
   touch .gitignore
   ```

   `.gitignore`

   ```shell
   # 后续有增加再持续更新
   # IDE #
   .idea/
   .vscode/
   
   # System #
   .DS_Store
   
   # Rust #
   target/
   *.log
   
   # Web #
   node_modules/
   ```

5. git提交

   ```shell
   git add .
   git commit -m "新增 .gitignore"
   ```

6. 创建一个Rust的可执行程序

   1. 将`app`添加到项目**根目录**下的`Cargo.toml`   

   ```toml
   [workspace]
   members = ["app"]
   
   [workspace.package]
   authors = ["name <email>"]
   edition = "2021"
   license = "MIT"
   publish = false
   repository = ""
   ```

   2. 创建项目

   ```shell
   cargo new --bin app
   ```

7. 运行项目，终端会打印`Hello,world!`

   ```shell
   cargo run --bin app	
   ```

   ![image-20221206171400109](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221206171400109.png)

8. 更改rust版本为`nightly`

   ```shell
   rustup toolchain install nightly
   rustup override set nightly
   ```

9. git提交

   ```shell
   git add .
   git commit -m "新增app"
   ```
