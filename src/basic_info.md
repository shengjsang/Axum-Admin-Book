# Rust

官方对`Rust`语言的定义是**一种使每个人都能 构建可靠、高效软件的语言。**

它深受程序员的喜爱，因为它的高性能和可靠性。

它还融合了现代开发人员工具：

+ `Cargo`是包含依赖项管理的构建工具，它使添加、编译和管理依赖项变得轻松，且在整个生态中保持一致。
+ `Rustfmt` 确保开发人员之间的编码风格一致。



# Axum

`Axum`是`tokio`官方出的一个优秀的web开发框架。它集成了`tokio`和`hyper`一起使用，而且充分利用了中间件、服务和实用程序的生态系统[`tower`](https://crates.io/crates/tower)，[`tower-http`](https://crates.io/crates/tower-http)。

官方列举了一些高级功能：

+ 使用**无宏** API 将请求路由到处理程序。
+ 可以使用**Extractor**声明式的解析**requests**。
+ 简单直观的处理错误，只要错误实现了`IntoResponse`
+ 生成response时使用最少的样板代码
+ 充分利用了中间件、服务和实用程序的生态系统[`tower`](https://crates.io/crates/tower)，[`tower-http`](https://crates.io/crates/tower-http)。





