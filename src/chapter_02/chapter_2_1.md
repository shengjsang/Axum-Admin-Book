# 开发环境

## Rust

### 安装

[官方文档](https://www.rust-lang.org/tools/install)

1. MacOS Or Unix OS

   ```shell
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. Other

   查看官方文档

### 换源

1. 在用户目录下创建`.cargo`文件夹，如果有则不用创建

	```shell
	mkdir ~/.cargo	
	```

2. 进入目录后 创建`config`文件

   ```shell
   cd ~/.cargo	
   touch config
   ```

3. 将下面内容粘贴进去，可以使用`vim`编辑config文件

   ```shell
   [source.crates-io]
   registry = "https://github.com/rust-lang/crates.io-index"
   # 指定镜像
   replace-with = 'ustc' # 如：tuna、sjtu、ustc，或者 rustcc
   
   # 注：以下源配置一个即可，无需全部
   
   # 中国科学技术大学
   [source.ustc]
   registry = "git://mirrors.ustc.edu.cn/crates.io-index"
   
   # 上海交通大学
   [source.sjtu]
   registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"
   
   # 清华大学
   [source.tuna]
   registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
   
   # rustcc社区
   [source.rustcc]
   registry = "https://code.aliyun.com/rustcc/crates.io-index.git"
   ```

   
