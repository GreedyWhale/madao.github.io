### 一. Docker

个人觉得学习编程最大的绊脚石就是环境配置了...学习的热情都会被配置环境消磨的一干二净。

Docker 在一定程度上解决了这类问题，简单说 Docker 就是将程序以及程序所需要的依赖（环境）打包在一起，使用者只要运行这个打包好的文件就可以直接运行这个程序而不用担心环境问题。

今天学习的数据库管理系统是 MySQL，就用 Docker 安装它

1. 从官网安装[Docker](https://hub.docker.com/)
2. 安装完成后在命令行输入：`docker --version`，如果得到正确的版本号，证明 Docker 安装成功，[docker 命令](https://docs.docker.com/engine/reference/commandline/docker/)
3. 找到 MySQL 的[image 文件](https://hub.docker.com/_/mysql)（image 文件 的意思可以参考[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)）
4. 按照文档执行：`docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql:tag`

   - --name 用于设置容器名称，所以 some-mysql 可以随意改成自己想要的名字
   - -e 指的是设置环境变量
   - MYSQL_ROOT_PASSWORD 就是你这个容器里 MySQL 的密码，my-secret-pw 也是可以改成你想要的密码的
   - -d 指的是在后台持续运行这个容器
   - mysql:tag 这里的 tag 指的是版本号，需要替换成想要安装的版本号
   - -p 指定端口

### 二. 使用命令行连接数据库

1. 进入容器，`docker exec -it mysql-1 bash`

   通过这个命令，可以进入刚刚创建的 mysql-1 容器：

   ![](/madao.github.io/database/images/articles/node/database/image1.png)

2. 连接 MySQL，`mysql -u root -p`

   输入这个命令会先让你填写密码，注意密码是不会展示在命令行的，所以输入的时候看起来就像没有输入一样，实际上是有的，密码就是创建容器的时候设置的 MYSQL_ROOT_PASSWORD，连接成功后可以看到如下界面：

   ![](/madao.github.io/database/images/articles/node/database/image2.png)

3. 数据库的简单操作

   由于命令行很难写代码，所以只简单的介绍几个语句，具体的增删改查操作在后面介绍

   - show databases;

     查看已有的数据库

   - use xxx;

     使用某个数据库，xxx 指的是数据库名字。

   - show tables;

     展示当前数据库中的所有表

   - select \* from tableName;

     从某个表中获取所有的记录，tableName 指的是表的名字。

   注意这些语句都要以`;`结尾，如果输错了可以输入`ctrl + c`中断重新输入，`ctrl + d`可以退出，比如要退出 MySQL。

### 三. 什么是数据库

> 以一定方式储存在一起、能与多个用户共用、具有尽可能小的冗余度、与应用程序彼此独立的数据集合。 ---维基百科

简单的说就是在计算机上将大量的数据保存成可以进行高效访问的数据集合称为数据库。

常说的 MySQL、PostgreSQL、Oracle、MongoDB、Redis 这些都不是数据库，这些叫数据库管理系统（英语：Database Management System，简称 DBMS）。

### 四、使用 Node.js 连接数据库

#### 1. 先启动之前创建的 mysql-1（如果之前没有 kill 的话，就不需要）

- `docker container ls -a`

  找到所有的容器，包括未启动的

- `docker start mysql-1`

  启动 mysql-1 容器

- `docker ps`

  查看启动的容器列表中是否有 mysql-1

#### 2. 使用 Node 连接 mysql-1

- `yarn add mysql`
- 根据文档创建一个数据库

  index.js

  ```
  const mysql = require('mysql')
   var connection = mysql.createConnection({
   host: 'localhost',
   user: 'root',
   password: '123456'
   });

   connection.connect()

   connection.query('CREATE DATABASE IF NOT EXISTS test_1 DEFAULT CHARACTER SET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;', function (error, results, fields) {
   if (error) throw error;
   console.log('results: ', results);
   console.log('fields: ', fields);
   });

   connection.end()
  ```

  **注意**：注意代码中的字符集`utf8mb4`，在 MySQL 中 utf8 是有问题的，永远不要使用它，具体原因可以搜索一下，有很多文章对这个进行了解释。

  然后执行`node index.js`，如果你用的是 MySQL 8.x 和 mysql 2.x 版本，那么会收到一个报错：

  `Client does not support authentication protocol requested by server; consider upgrading MySQL client`

  具体原因可以[查看这里](https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server)，这里也有解决办法，但是我按照 stackoverflow 中的方法并没有解决，这里记录一下我解决的步骤：

  1. 进入容器 `docker exec -it mysql-1 bash`
  2. 登录 mysql `mysql -u root -p`
  3. `select host, user, plugin, authentication_string from mysql.user;`

     查看所有用户密码的加密方式，可以得到下面结果：

     ![](/madao.github.io/database/images/articles/node/database/image3.png)

  4. 修改 root 用户的加密方式

     `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';`

     `ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';`

  5. 刷新`flush privileges;`
  6. `select host, user, plugin, authentication_string from mysql.user;`

     查看是否修改成功

     ![](/madao.github.io/database/images/articles/node/database/image4.png)

  当修改成功后再去连接就不会报错了。

- 查看是否创建成功：
  在容器中的 mysql 中输入：`show database`，可以看到刚刚创建的 test_1 数据库成功添加了

  ![](/madao.github.io/database/images/articles/node/database/image5.png)
