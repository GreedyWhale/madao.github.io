最近翻看之前写的笔记，发现废话太多，想找一个知识点很麻烦，所以后面改变一下记录风格，之前的笔记也会慢慢重写一遍。

### 一. 需求分析

- 博客系统

  - 用户可以登录，注册，重置密码
  - 用户可以对自己的博客进行增删改查
  - 用户可以对博客进行评论，但无法修改评论
  - 用户可以更新自己的个人信息

- 可用性要求

  要求在 PC 和移动端均可正常使用

- 其他要求

  搜索引擎优化

### 二. 数据库的设计与搭建

根据需求这个博客系统起码需要三张表：

- users（用户表）
- blogs（博客表）
- comments（评论表）

下面来创建它们

#### 1. 使用 docker 创建 PostgreSQL 容器

```
docker run -v "$PWD/blogDatabase":/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=xxxxxx -d --name blog-postgres postgres:latest
```

命令具体组成上篇笔记有说这里就不说了。

查看是否创建成功：`docker ps -a`

![](/madao.github.io/database/images/articles/blog/database_design/image.png)

这里说明下，虽然设置了密码，但是在连接数据库的时候仍然不需要密码就可以连接，这里官方文档有说明，但是我不知道如何去验证：

> Note 1: The PostgreSQL image sets up trust authentication locally so you may notice a password is not required when connecting from localhost (inside the same container). However, a password will be required if connecting from a different host/container.

#### 2. 使用 Next.js 初始化博客项目

- `mkdir diary-of-madao`
- `cd diary-of-madao`
- `yarn create next-app .`
- `git add .`
- `git push -m 'update: 初始化项目'`
- `git push`
- 支持 TypeScript
  1. `yarn add --dev typescript @types/react @types/node`
  2. `yarn tsc --init`
  3. `yarn start`
  4. 将`.js`改为`.tsx`，并消除文件报错
- 支持 scss
  1. `yarn add scss -D`
  2. 将`.css`改为`.scss`
- 提交当前修改到 github（重要）

#### 3. 安装 typeORM

- `yarn add typeorm reflect-metadata pg`
- 在 tsconfig.json 中启用

  ```
  "emitDecoratorMetadata": true,
  "experimentalDecorators": true,
  ```

- 提交当前修改到 github（重要）
- `yarn typeorm init --database postgres`
- 保留`ormconfig.json`文件，其他全部回退至上一次提交
- 在`.gitignore`中添加`ormconfig.json`文件，避免 ormconfig.json`文件被提交到 github 上
- 修改 ormconfig.json 文件内容
  1. username：创建容器时的 POSTGRES_USER
  2. password：创建容器时的 POSTGRES_PASSWORD
  3. synchronize：false

#### 4. 统一 ts 文件的执行方式

1. `yarn add @babel/cli @babel/core @babel/preset-typescript @babel/plugin-proposal-decorators -D`
2. 在项目根目录新建`.babelrc`
   ```
   {
     "presets": ["next/babel", "@babel/preset-typescript"],
     "plugins": [
       ["@babel/plugin-proposal-decorators", { "legacy": true }]
     ]
   }
   ```
3. package.json 中添加命令

   `·"build:orm": "babel -w ./src --out-dir dist --extensions \".ts,.tsx\""`

4. 清除 babel 转译后文件目录

   tools/clearBabelDist.js

   ```
   const fse = require('fs-extra')
   const { join } = require('path')

   try {
     fse.emptyDirSync(join(__dirname, '../dist'))
   } catch(error) {
     console.error(error)
   }
   ```

5. 安装依赖 `yarn add fs-extra npm-run-all -D`
6. package.json 中添加命令

   ```
   "watch:orm": "run-s clean:dist build:orm"
   ```

7. 修改`ormconfig.json`

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

#### 5. 创建数据库

一般来说，项目都需要三个数据库，分别对应 development（开发环境），test（测试环境），production（正式环境）

1. 创建

   ```
   CREATE DATABASE development ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
   ```

   ```
   CREATE DATABASE test ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
   ```

   ```
   CREATE DATABASE production ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
   ```

2. 查看是否创建成功：`\l`
3. 修改`ormconfig.json`的 database 为：development

### 6. 创建表

1. 在 package.json 中添加命令：

   ```
   "c:table": "typeorm migration:create"
   ```

2. 创建 users 表

   ```
   yarn c:table -n CreateUsers
   ```

3. 修改`src/migration/[timestamp]-CreateUsers.ts`

   users 表应有的字段为：

   - id
   - username
   - passwordDigest：用户密码摘要
   - createdAt：创建这条数据的时间
   - updatedAt：最近一次修改这条数据的时间

   **注意：用户的密码一定不能明文存储**

   ```
   import {MigrationInterface, QueryRunner, Table} from "typeorm";

   export class CreateUsers1615020850775 implements MigrationInterface {

     public async up(queryRunner: QueryRunner): Promise<void> {
       await queryRunner.createTable(new Table({
         name: 'users',
         columns: [
           { name: 'id', isPrimary: true, type: 'bigint', generationStrategy: 'increment', isNullable: false, isGenerated: true },
           { name: 'username', type: 'varchar', isNullable: false },
           { name: 'passwordDigest', type: 'varchar', isNullable: false },
           { name: 'createdAt', type:  'timestamp', isNullable: false, default: 'now()' },
           { name: 'updatedAt', type:  'timestamp', isNullable: false, default: 'now()' }
         ]
       }))
     }

     public async down(queryRunner: QueryRunner): Promise<void> {
       await queryRunner.dropTable('users')
     }

   }
   ```

   now()是 PostgreSQL 内置的时间函数，可以获取当前操作数据的时间

4. 将 users 表迁移到数据库

   - package.json

     ```
     "m:table": "typeorm migration:run"
     ```

   在终端执行`yarn m:table`

5. 查看是否创建成功

   在 docker 容器的终端中输入： `\d users`

6. 同样的步骤创建 blogs、comments 表，注意都要有 createdAt 和 updatedAt 字段

   - blogs 表应有的字段有：

     - id
     - title：博客标题
     - content：博客内容
     - authorId：作者 id
     - createdAt：数据创建时间
     - updatedAt：数据更新时间

   - comments 表应有的字段有：

     - id
     - userId：提交评论的用户
     - blogId：评论的文章 id
     - content：评论内容
     - createdAt：数据创建时间
     - updatedAt：数据更新时间

7. 修改一下数据类型

   做完上面步骤之后我去查了一下 PostgreSQL 的数据类型，发现`bigint`的范围过于大了，这个项目基本不会出现这么大的数，所以修改一下: `users, blogs, comments`这三个表 id 字段的数据类型

   1. `yarn c:table -n ModifyIdDataType`
   2. `src/migration/ModifyIdDataType`

      ```
      import {MigrationInterface, QueryRunner} from "typeorm";

      export class ModifyIdDataType1615171068066 implements MigrationInterface {

        public async up(queryRunner: QueryRunner): Promise<void> {
          await queryRunner.query('ALTER TABLE users ALTER COLUMN id TYPE integer;')
          await queryRunner.query('ALTER TABLE blogs ALTER COLUMN id TYPE integer;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN id TYPE integer;')
          await queryRunner.query('ALTER TABLE blogs ALTER COLUMN "authorId" TYPE integer;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN "blogId" TYPE integer;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN "userId" TYPE integer;')
        }

        public async down(queryRunner: QueryRunner): Promise<void> {
          await queryRunner.query('ALTER TABLE users ALTER COLUMN id TYPE bigint;')
          await queryRunner.query('ALTER TABLE blogs ALTER COLUMN id TYPE bigint;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN id TYPE bigint;')
          await queryRunner.query('ALTER TABLE blogs ALTER COLUMN "authorId" TYPE bigint;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN "blogId" TYPE bigint;')
          await queryRunner.query('ALTER TABLE comments ALTER COLUMN "userId" TYPE bigint;')
        }

      }
      ```

      **注意**：PostgreSQL 中如果表名或者列名中有大小写，请用`""`包把表名或者列名起来，否则 PostgreSQL 会自动转换成小写

      [How can I specify column name casing in TypeORM migrations](https://stackoverflow.com/questions/52931663/how-can-i-specify-column-name-casing-in-typeorm-migrations)

   3. `yarn m:table`

### 7. 数据关联

- users

  1. 一个用户可以有多个博客
  2. 一个用户可以写多条评论

- blogs

  1. 一篇博客属于一个用户
  2. 一篇博客可以有多条评论

- comments

  1. 一条评论属于一篇博客
  2. 一条评论属于一个用户

- 实现

  1. 在 package.json 中新增命令：`"c:entity": "typeorm entity:create"`
  2. 使用`c:entity`命令，创建三个实体：User，Blog，Comment

     ```
     yarn c:entity -n User
     yarn c:entity -n Blog
     yarn c:entity -n Comment
     ```

  3. users 和 blogs

     - src/entity/User.ts

       ```
       import {Column, Entity, OneToMany, PrimaryGeneratedColumn} from "typeorm";
       import { Blog } from './Blog'

       @Entity('users')
       export class User {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('varchar')
         username: string;

         @Column('varchar')
         passwordDigest: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @OneToMany(() => Blog, Blog => Blog.author)
         blogs: Blog[];

         constructor(data: Partial<User>){
          data && Object.assign(this, data)
         }
       }
       ```

     - src/entity/Blog.ts

       ```
       @Entity('blogs')
       export class Blog {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('varchar')
         title: string;

         @Column('text')
         content: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @ManyToOne(() => User, user => user.blogs)
         author: User;

         constructor(data: Partial<Blog>){
          data && Object.assign(this, data)
         }
       }
       ```

  4. users 和 comments

     - src/entity/User.ts

       ```
       import {Column, Entity, OneToMany, PrimaryGeneratedColumn} from "typeorm";
       import { Blog } from './Blog'
       import { Comment } from "./Comment";

       @Entity('users')
       export class User {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('varchar')
         username: string;

         @Column('varchar')
         passwordDigest: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @OneToMany(() => Blog, Blog => Blog.author)
         blogs: Blog[];

         @OneToMany(() => Comment, comment => comment.user)
         comments: Comment[];

         constructor(data: Partial<User>){
          data && Object.assign(this, data)
         }
       }
       ```

     - src/entity/Comment.ts

       ```
       import {Column, Entity, ManyToOne, PrimaryGeneratedColumn} from "typeorm";
       import { User } from "./User";

       @Entity('comments')
       export class Comment {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('text')
         content: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @ManyToOne(() => User, user => user.comments)
         user: User;

         constructor(data: Partial<Comment>){
          data && Object.assign(this, data)
         }
       }
       ```

  5. blogs 和 comments

     - src/entity/Blog.ts

       ```
       import {Column, Entity, ManyToOne, OneToMany, PrimaryGeneratedColumn} from "typeorm";
       import { Comment } from "./Comment";
       import { User } from "./User";

       @Entity('blogs')
       export class Blog {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('varchar')
         title: string;

         @Column('text')
         content: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @ManyToOne(() => User, user => user.blogs)
         author: User;

         @OneToMany(() => Comment, comment => comment.blog)
         comments: Comment[];

         constructor(data: Partial<Blog>){
          data && Object.assign(this, data)
         }
       }
       ```

     - src/entity/Comment.ts

       ```
       import {Column, Entity, ManyToOne, PrimaryGeneratedColumn} from "typeorm";
       import { Blog } from "./Blog";
       import { User } from "./User";

       @Entity('comments')
       export class Comment {
         @PrimaryGeneratedColumn('increment')
         id: number;

         @Column('text')
         content: string;

         @Column('timestamp')
         createdAt: string;

         @Column('timestamp')
         updatedAt: string;

         @ManyToOne(() => User, user => user.comments)
         user: User;

         @ManyToOne(() => Blog, blog => blog.comments)
         blog: Blog;

         constructor(data: Partial<Comment>){
          data && Object.assign(this, data)
         }
       }
       ```

### 8. 使用 seed 添加数据

- src/seed.ts

  ```
  import "reflect-metadata";
  import {createConnection} from "typeorm";
  import { User } from './entity/User'
  import { Blog } from './entity/Blog'
  import { Comment } from './entity/Comment'

  createConnection().then(async connection => {
    const user = new User({
      username: 'MADAO',
      passwordDigest: Math.random().toString(16)
    })
    const blog = new Blog({
      title: '我的第一篇博客',
      content: '写点什么好呢？',
      author: user
    })
    const comment = new Comment({
      content: '我也不知道写点什么好',
      user,
      blog
    })
    await connection.manager.save(user)
    await connection.manager.save(blog)
    await connection.manager.save(comment)
  }).catch(error => console.log(error))
  ```

  结果：

  ![](/madao.github.io/database/images/articles/blog/database_design/image1.png)

  这就是关联的好处，用户 id，博客 id，评论 id 这些写好的关联会自动的进行同步，图片中的 id 不是从 1 开始，是因为我测试了好几次，把之前的数据都删除了导致的。
