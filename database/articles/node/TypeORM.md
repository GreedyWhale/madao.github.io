### 一. TypeORM

> TypeORM - Amazing ORM for TypeScript and JavaScript (ES7, ES6, ES5). Supports MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, WebSQL databases. Works in NodeJS, Browser, Ionic, Cordova and Electron platforms.
> ----[TypeORM 官网](https://typeorm.io/#/)

上面的描述来自 TypeORM 官网，翻译过来的意思是：

> 用于 TypeScript 和 JavaScript 的 ORM，支持 MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, WebSQL databases。适用于 NodeJS，浏览器，Ionic，Cordova 和 Electron 这些平台。

所以 ORM 是什么？

ORM 是 Object Relational Mapping 的缩写，翻译过来是**对象关系映射**。

> ORM 就是通过实例对象的语法，完成关系型数据库的操作的技术，是"对象-关系映射"（Object/Relational Mapping） 的缩写。
> ----[ORM 实例教程](http://www.ruanyifeng.com/blog/2019/02/orm-tutorial.html)

个人理解：把数据库变成对象，然后以操作对象的方式操作数据库。

### 二. 在 Docker 中启动 PostgreSQL

PostgreSQL 是一个免费开源的数据库管理系统。

在[Docker Hub postgres](https://github.com/docker-library/docs/blob/master/postgres/README.md)中可以找到在
Docker 中启动 PostgreSQL 的命令

注意这是在 Macos 环境下：

```
docker run -v "$PWD/TypeORM-demo":/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=admin -e POSTGRES_HOST_AUTH_METHOD=trust -d --name postgres-demo postgres:latest
```

- `docker run -v 本地目录:容器目录`

  这句命令的意思是将指定本机的目录挂载到容器中，这样做的好处是可以让数据持久化，你在指定的本地目录里面新增文件可以同步到容器目录下，同样的在容器里面新增文件也可以同步到本地目录下，当容器被删除挂载的目录及目录里面的文件不会删除。

  这种挂载的目录一般称为数据卷，简单来说就是容器与主机之间共享的目录或者文件

- `-p 5432:5432`

  这句命令是设置端口映射，当有人访问你本机的这个端口，这个请求会自动的映射到容器的这个端口上

- `-e POSTGRES_USER=admin`

  设置环境变量用户名

- `-e POSTGRES_HOST_AUTH_METHOD=trust`

  这个用于设置客户端的身份认证方式，trust 表示无条件的允许连接，这种方式允许任何人使用任何数据库账号连接 PostgreSQL 服务而不需要密码认证

- `-d --name postgres-demo postgres:latest`

  `-d`表示在后台运行并返回容器号

  `--name postgres-demo`用于设置容器名

  `postgres:latest`则是 postgres 的版本，latest 表示最新版

执行完上面的命令之后，在终端输入`docker ps -a`，就可以看到刚刚创建的容器：

![](/madao.github.io/database/images/articles/node/typeORM/image.png)

注意 status 那一项，up 表示该容器正在运行

### 三. 验证 PostgreSQL

1. `docker exec -it 容器id bash`

   进入容器的 bash shell

2. `psql -U admin -W`

   admin 就是之前创建容器时候的 POSTGRES_USER 值，这里不用输入密码，因为创建的时候也设置了 POSTGRES_HOST_AUTH_METHOD 的值为 trust

   `-W`表示强制提示输入密码，即使没有设置密码，因为没设置密码直接回车即可

   参考文档[psql](https://www.postgresql.org/docs/13/app-psql.html)

   如果都成功了，现在应该得到如下界面：

   ![](/madao.github.io/database/images/articles/node/typeORM/image1.png)

3. 使用 pg 的命令

   - `\l`: 展示现有的数据库
   - `\c`: 链接某个数据库
   - `\dt`: 展示数据库中的所有数据表

4. 使用 SQL 语句创建三个数据库 development，test，production

   `CREATE DATABASE development ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';`

   创建命令可以[查阅文档](https://www.postgresql.org/docs/9.2/sql-createdatabase.html)

### 四. 安装 TypeORM

目前还没有项目，之前写 Next.js 笔记时创建的项目被我删除了，所以这边重新创建一个。

具体过程可以参考 Next.js 那一篇笔记

1. 在[官网](https://typeorm.io/#/)找到 Installation 章节，按照文档说的先安装依赖:

   `yarn add typeorm reflect-metadata pg`

   `yarn add @types/node -D`

2. 在你项目中的`tsconfig.json`中添加：

   ```
   "emitDecoratorMetadata": true,
   "experimentalDecorators": true,
   ```

3. 使用 typeorm 连接 PostgreSQL

   **重要**：做到这一步的时候需要先`git commit`一下，因为后面的命令会修改你现有的配置。

   在当前项目目录下(如果不在项目目录下需要加--name 参数，name 指项目文件夹)的终端输入：

   `yarn typeorm init --database postgres`

   输入上面的命令之后，会改变以下文件：

   ![](/madao.github.io/database/images/articles/node/typeORM/image2.png)

   src 目录下的 index.ts 和 User.ts 以及 ormconfig.json 是 typeORM 创建的

   接下来一个一个还原....

### 五. 还原项目配置

1. `.gitignore`

   typeORM 的初始化命令居然还会修改`.gitignore`文件，这个文件就看个人需求，可以选择自己配置的，也可以选择它帮你配置的，或者两个结合也行

   这里我选择两个结合

2. `ormconfig.json`

   这个文件是 typeORM 的配置文件，需要坐下修改：

   - username：改为在 docker 中创建 PostgreSQL 容器时的用户名，这里是`admin`
   - password：改为空字符串，因为在之前创建容器的时候没有设置 password
   - database：改为 development，这个数据库是上一节中使用 SQL 语句创建的
   - synchronize：改为 false，这个选项在文档上有说明，不要在生产环境开启，会造成数据丢失。如果 synchronize 为 true，那么 typeORM 会在连接数据库的时候根据 entity 目录修改数据库，比如 entity 目录下有一个 User.ts 文件，那么 typeORM 就会帮你创建一个 User 数据表，如果你的数据库之前已经存在 User 数据表了，它会覆盖掉之前的表，所以会造成数据丢失。

   其他的暂时不用改

3. package.json

   注意：这里如果选择还原，一定要手动的安装 pg

   `yarn add pg`

   这个文件建议直接还原，原因如下：

   1. 帮你装了 ts-node，ts-node 可以让 node 直接执行 ts 文件，但是我这里是使用了 Next.js 搭建项目，而 Next.js 是通过 babel 将 ts 转译成 js 执行的，如果它们两各自采用不同的方式执行 ts 文件，在将来的某一天如果出现冲突改起来那就很痛苦了，由于 Next.js 内置了这个功能，改 Next.js 的话不太现实，那么只能将 typeORM 执行 ts 文件的方式修改成 babel 的方式了
   2. 它还降级了一些依赖的版本，这点看个人吧，我不太喜欢这种行为

4. tsconfig.json

   这个文件也直接还原，使用项目原有的即可，但是要确保这两项被添加进去

   ```
   "emitDecoratorMetadata": true,
   "experimentalDecorators": true,
   ```

5. 删除 src 目录下的 entity 和 migration 目录

   以减少一切的干扰，从头开始学习

最后记得 commit

### 六. 统一 ts 文件的执行方式

1. 安装@babel/cli，@babel/core，@babel/preset-typescript

   `yarn add @babel/cli @babel/core @babel/preset-typescript -D`

2. 找到 Next.js 文档中关于 babel 的配置[Customizing Babel Config](https://www.nextjs.cn/docs/advanced-features/customizing-babel-config)

   在项目中新建`.babelrc`

   ```
   {
     "presets": ["next/babel"],
     "plugins": []
   }
   ```

3. 在 babel 的官网上找到如何编译 typescript，根据文档，在`.babelrc`中添加：

   ```
   {
     "presets": ["next/babel", "@babel/preset-typescript"],
     "plugins": []
   }
   ```

   这里还要做一件事，就是要安装`yarn add @babel/plugin-proposal-decorators -D`，因为 typeORM 用到了装饰器语法，所以最终的`.babelrc`内容是：

   ```
    {
      "presets": ["next/babel", "@babel/preset-typescript"],
      "plugins": [
        ["@babel/plugin-proposal-decorators", { "legacy": true }]
      ]
    }
   ```

4. 在 package.json 中添加命令

   ```
   "build:typeorm": "babel -w ./src --out-dir dist --extensions \".ts,.tsx\""
   ```

   这里加了`-w`，为的是监听 src 目录下的文件变化，只要有变化就重新编译。

   接下来还要修改 ormconfig.json 中的`entities`，`migrations`，`subscribers`，把它们改为：

   ```
   "entities": [
       "dist/entity/**/*.js"
   ],
   "migrations": [
       "dist/migration/**/*.js"
   ],
   "subscribers": [
       "dist/subscriber/**/*.js"
   ],
   ```

5. 修改 src/index.ts 文件内容

   ```
   import "reflect-metadata";
   import {createConnection} from "typeorm";

   createConnection().then(async connection => {

       console.log("成功连接development");
       console.log("connection:", connection)

   }).catch(error => console.log(error));
   ```

   使用 node 执行它：`node dist/index.js`

   如果没问题的话你的终端应该会显示：成功连接 development 文案

### 七. 创建表

到这里我有点疑惑，方方老师的课是使用 Migration 来创建表的，可是我看文档总感觉 entity 才是用来创建表的，留个坑，后面掌握了再来填坑，先安装课程教的记笔记。

Migration 是迁移的意思，就是要把对数据库的修改同步到数据库去。

通过创建一个 users 表来看看如何使用

1. yarn typeorm migration:create -n CreateUser

   输完这个命令后，会得到`src/migration/[timestamp]-CreateUser.ts`这个文件

   文件名中的数字是时间戳

2. 迁移和还原

   CreateUser.ts 里面有写好两个函数，分别是 up 和 down，up 中写迁移的逻辑，down 里写还原的逻辑

   ```
   import {MigrationInterface, QueryRunner, Table} from "typeorm";

   export class CreateUser1614236876244 implements MigrationInterface {

     public async up(queryRunner: QueryRunner): Promise<void> {
       queryRunner.createTable(new Table({
         name: 'users',
         columns: [
           { name: 'id', isGenerated: true, generationStrategy: 'increment', type: 'bigint', isPrimary: true },
           { name: 'name', type: 'text' },
           { name: 'age', type: 'smallint' },
           { name: 'address', type: 'text' }
         ]
       }), true)
     }

     public async down(queryRunner: QueryRunner): Promise<void> {
       queryRunner.dropTable('users', true)
     }

   }
   ```

   这里的还原很简单，就直接删除 up 中创建的表即可

   写好 migration 代码之后，输入：

   - `yarn typeorm migration:run`：进行迁移
   - `yarn typeorm migration:revert`: 进行还原

   这里我还有一个疑问就是，看文档说对数据库的操作都需要连接成功才可以后续的操作，但是我执行迁移的时候，并没有调用它的 api`createConnections`连接数据库，猜测这里 typeORM 是自动使用配置好的`ormconfig.json`进行了连接。

### 八. 数据的添加和修改

上一节成功的创建了一个 users 表，现在来操作一下这个表

使用 entity 来操作数据表，entity 是实体的意思，实体是一个映射到数据库表（或使用 MongoDB 时的集合）的类。

1. 创建一个 entity

   `yarn typeorm entity:create -n User`

   输入了该命令会在 src\entity 目录下创建一个 User.ts

2. 根据表结构定义类

   ```
   import {Entity, Column, PrimaryGeneratedColumn} from "typeorm";

   @Entity('users')
   export class User {
     @PrimaryGeneratedColumn('increment')
     id: number;
     @Column('text')
     name: string;
     @Column('smallint')
     age: number;
     @Column('text')
     address: string;
   }

   ```

   这里有可能 ts 会报错，说要给每个属性一个初始值，这是因为在 tsconfig 里开启了`strict`，`strict`会默认开启`strictPropertyInitialization`，解决办法就是把`strictPropertyInitialization`改为 false

   接下来就要使用它，typeORM 提供了两种方式来操作实体[EntityManager](https://typeorm.io/#/working-with-entity-manager)和[Repository](https://typeorm.io/#/working-with-repository)

   先来看 EntityManager

3. EntityManager

   [文档](https://typeorm.io/#/entity-manager-api)

   src/index.ts

   ```
   import "reflect-metadata";
   import {createConnection} from "typeorm";
   import {User} from './entity/User'

   createConnection().then(async connection => {
       const users = await connection.manager.find(User)
       console.log(users)
       connection.close()
   }).catch(error => console.log(error));
   ```

   执行通过 babel 转译后的 js 文件，得到的结果是`[]`，一个空数组。

   现在添加一点数据

   ```
   import "reflect-metadata";
   import {createConnection} from "typeorm";
   import {User} from './entity/User'

   createConnection().then(async connection => {
       const users = await connection.manager.find(User)
       console.log(users)
       // 添加数据
       const usersEntity = new User()
       usersEntity.name = 'allen'
       usersEntity.age = 20
       usersEntity.address = 'BeiJing'
       await connection.manager.save(usersEntity)
       // 查看最新内容
       const newUsers = await connection.manager.find(User)
       console.log(newUsers)
       connection.close()
   }).catch(error => console.log(error));
   ```

   结果：

   ![](/madao.github.io/database/images/articles/node/typeORM/image3.png)

   这时候你再去 docker 容器中的数据库查看，会有一条相同的数据的

4. Repository

   直接看代码

   ```
   import "reflect-metadata";
   import {createConnection} from "typeorm";
   import {User} from './entity/User'

   createConnection().then(async connection => {
       const usersRepository = connection.getRepository(User)
       const users = await usersRepository.find()
       console.log(users)
       const usersEntity = new User()
       usersEntity.name = 'Jack'
       usersEntity.age = 40
       usersEntity.address = 'NanJing'
       await usersRepository.save(usersEntity)
       const newUsers = await usersRepository.find()
       console.log(newUsers)
       connection.close()
   }).catch(error => console.log(error));
   ```

   从代码中可以看出 repository 和 manager 的区别是：repository 会先构造一张表的对象，然后这个对象就只操作这张表了，manager 则更自由一点，它可以根据实体操作任何表，而不需要先构造数据表的对象。

### 九. Seed 填充数据

在填充数据的之前先优化一下 users 数据表的实体 User.ts，上面代码在构造数据的时候都是先 new 一下，然后`xxx.xx = xxx...`这样写十分的麻烦，这些属性完全可以在 new 的时候就传进去：

User.ts

```
import {Entity, Column, PrimaryGeneratedColumn} from "typeorm";

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('increment')
  id: number;
  @Column('text')
  name: string;
  @Column('smallint')
  age: number;
  @Column('text')
  address: string;

  constructor(user: Partial<User>) {
    user && Object.assign(this, user)
  }
}
```

然后再来填充数据

src/index.ts

```
import "reflect-metadata";
import {createConnection} from "typeorm";
import {User} from './entity/User'

createConnection().then(async connection => {
    const usersRepository = connection.getRepository(User)
    const users = await usersRepository.find()
    if (!users.length) {
        const data = Array.from({ length: 10 }).map((v, i) => {
            return new User({
                name: `robot-${i}`,
                age: parseInt((Math.random() * 99 + 1) as unknown as string, 10),
                address: `district-${i}`
            })
        })
        await usersRepository.save(data)
    }
    connection.close()
}).catch(error => console.log(error));
```

![](/madao.github.io/database/images/articles/node/typeORM/image4.png)

这样在表里就快速填充了一些数据，填充数据场景一般是开发时需要，生产环境不会这样做
