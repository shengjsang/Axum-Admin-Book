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




## Postgres

### 安装

[官方网站](https://www.postgresql.org/)

1. 根据自己的电脑系统[官方下载](https://www.postgresql.org/download/)合适的版本，并安装，以及设置数据库和用户名密码

   ![image-20221206113025981](https://repo-1256831547.cos.ap-shanghai.myqcloud.com/image-20221206113025981.png)

2. MacOS操作系统还可以使用`homebrew`来安装

   ```shell
   # 安装
   brew install postgresql
   
   # 查看版本
   psql -V
   
   # 初始化数据库
   initdb /usr/local/var/postgres -E utf8
   
   # 数据库写入配置文件(zsh)
   vim ~/.zshrc # 如果是bash vim ~/.bash_profile
   
   # 将下面一行代码粘贴进~/.zshrc
   export PGDATA=/usr/local/var/postgres
   
   # 保存退出然后终端输入下行命令使配置生效
   source ~/.zshrc
   
   # 启动数据库
   pg_ctl start
   
   # 关闭数据库
   pg_ctl stop
   ```

### 配置

1. 终端命令

   ```postgresql
   # 创建用户
   $ createuser username -P
   
   # 创建数据库
   $ createdb dbname -O username -E UTF8 -e
   
   # 删除数据库
   $ dropdb -U username dbname
   
   -O username  拥有者（owner）
   -E UTF8  数据库的编码（encoding）
   -e 显示执行操作的命令
   
   # 终端上查看显示已创建的列表
   $ psql -l
   
   # 连接数据库
   $ psql -U username -d dbname -h 127.0.0.1
   ```

2. 数据库命令

   ```postgresql
   # 查看数据库用户列表
   > \du
   
   # 创建数据库用户
   > create user user1 with password '123456';
   
   # 修改用户密码
   > alter user user1 with password 'XXXXXX';
   
   # 删除数据库用户
   > drop user user1;
   
   # 查看数据库列表:
   > \l (list的意思)
   
   # 创建数据库
   > create database db1;
   
   # 删除数据库
   > drop database db1;
   
   # 选择数据库 
   > \c dbname (choose的意思)
   
   # 查看数据库信息
   > \d (database list的意思)
   
   ```



## 其他安装

基本开发环境只需要`Rust`和`Postgres`，其他开发所需要的安装我们后续在开发教程中使用的时候再安装。



**参考：**

1. [Mac下PostgreSQL的安装与简单使用](https://pengshiyu.blog.csdn.net/article/details/113808724?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-113808724-blog-124277853.pc_relevant_vip_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-113808724-blog-124277853.pc_relevant_vip_default&utm_relevant_index=2)



