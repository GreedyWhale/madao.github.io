### 一. 关系型数据库

搜索了一些文章，普遍讲的太晦涩了，还有一方面就是我的知识储备不够，我的数学真的很烂，好在找到了廖雪峰大神的一个教程[关系数据库概述](https://www.liaoxuefeng.com/wiki/1177760294764384/1179613436834240)，我觉得他解释的比较容易懂：

![](/madao.github.io/database/images/articles/node/database2/image1.png)
![](/madao.github.io/database/images/articles/node/database2/image.png)
![](/madao.github.io/database/images/articles/node/database2/image2.png)
![](/madao.github.io/database/images/articles/node/database2/image3.png)

### 二. 关系型数据库的设计标准

在一些更专业的文章里，设计标准也叫范式，根据维基百科上的资料，范式有：

范式如下（从最不规范到规范排序）：

- UNF: 非标准化形式
- 1NF: 第一范式
- 2NF: 第二范式
- 3NF: 第三范式
- EKNF: 主键范式
- BCNF: Boyce–Codd 范式
- 4NF: 第四范式
- ETNF: 关键元组范式
- 5NF: 第五范式
- DKNF: 域键范式
- 6NF: 第六范式

![](/madao.github.io/database/images/articles/node/database2/image4.png)

一般来说数据库的设计满足到第三范式即可。

从一个例子看起：

| 学号 | 学生姓名 | 联系方式            | 班级 | 班主任 | 课程 | 分数 |
| ---- | -------- | ------------------- | ---- | ------ | ---- | ---- |
| 0    | 小桑     | 18xxx/yyy@email.com | A 班 | 罗小虎 | 数学 | 10   |

### 三. 第一范式 1NF

第一范式要求表的每个字段的值都只能是单一值，字段不可再分，且没有重复字段。上面表中可以拆分的字段是联系方式字段，从字段的值就能看出来，联系方式可以拆分成邮箱、手机号甚至是社交应用的 ID

现在将例子中的表改为：

| 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 | 课程 | 分数 |
| ---- | -------- | ------ | ------------- | ---- | ------ | ---- | ---- |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 数学 | 10   |

### 四. 第二范式 2NF

第二范式是在满足第一范式的情况下，要求表中的每个非键属性都和键（主键与候选键）有完全依赖关系

这个定义又牵扯除了一些新的概念

- 非键属性：

  属性指的是表中的每一列，非键属性就是除了作为主键与候选键列之外的所有列，有些文章里描述的是主属性和非主属性。

- 键

  这里说的键指的是关系键，关系键是一个表中的**一个**或**几个属性**，用来标识该表的每一行或与另一个表产生联系。

  关系键又分为：

  1. 超键：能唯一标识表中的一条记录（一行数据）的属性集，比如例子中的：学号、学号 + 姓名、学号 + 手机号等等，这些都是超键
  2. 候选键：在超键的基础上没有多余的属性，称为候选键，比如例子中的超键：学号 + 姓名，当去掉姓名，学号仍然可以标识记录，那么学号就是候选键，但是姓名不行，去掉学号后姓名有可能重复，所以姓名不能成为候选键
  3. 主键：主键是人为选择的，在候选键里选出一个作为主键，一张表只能有一个主键，但是主键不一定由**一个字段**组成，主键可以由多个字段组成，多个字段组成的主键称为**联合主键**，主键选择的原则是：不使用任何业务相关的字段作为主键，所以例子中的学号其实也不适合作为主键，最好是新增一个字段 id，id 一般使用自增整数类型作为值。

- 完全依赖

  完全依赖指的是 y = f(x)，当 x 的值确定了，那么 y 的值就一定确定，那么 y 就可以说是完全依赖 x

  | 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 | 课程 | 分数 |
  | ---- | -------- | ------ | ------------- | ---- | ------ | ---- | ---- |
  | 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 数学 | 10   |
  | 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 物理 | 20   |

  在例子中的完全依赖有：

  - 学号 -> 姓名
  - 学号 -> 电话
  - 学号 -> 邮箱
  - ....
  - (学号, 课程) -> 分数

了解完这些概念后，再看看下面这张表

| 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 | 课程 | 分数 |
| ---- | -------- | ------ | ------------- | ---- | ------ | ---- | ---- |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 数学 | 10   |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 物理 | 20   |
| 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   | 化学 | 100  |
| 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   | 生物 | 90   |

先来找找这张表中的主键，学号可以确定：学生姓名、手机号、邮箱、班级、班主任，但是不能确认课程及分数，分数需要学号 + 课程才能确定，学号 + 课程可以确定这张表上面一条记录的其他属性，所以这张表的主键是学号 + 课程。

其中学生姓名、手机号、邮箱、班级、班主任这几个属性只依赖学号，和课程无关，那么这就不符合第二范式要求的“完全依赖”，这种情况叫部分函数依赖。

不满足第二范式带来的问题：

1. 数据冗余

   从例子中一眼就能看出来，学生信息是有重复的

2. 插入异常

   因为课程和学生的信息是在一起的，主键又是学号 + 课程，如果有一门课程没有学生参加，那么这门课程就无法插入表中

3. 删除异常

   如果某个班级的学生信息都被删除，那班级与班主任的信息也会丢失

4. 更新异常

   数据冗余会导致更新某条记录的时候需要更新多处，当数据越来越多的时候，难免会漏掉一两条

现在将例子中的表转换成符合第二范式的表：

转换前：

| 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 | 课程 | 分数 |
| ---- | -------- | ------ | ------------- | ---- | ------ | ---- | ---- |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 数学 | 10   |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 | 物理 | 20   |
| 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   | 化学 | 100  |
| 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   | 生物 | 90   |

转换后：

- 学生基本信息表

  | 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 |
  | ---- | -------- | ------ | ------------- | ---- | ------ |
  | 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 |
  | 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   |

- 成绩表

  | 学号 | 课程 | 分数 |
  | ---- | ---- | ---- |
  | 0    | 数学 | 10   |
  | 0    | 物理 | 20   |
  | 1    | 化学 | 100  |
  | 1    | 生物 | 90   |

将例子中的一个表拆分成了学生信息表和成绩表，学生信息表的主键是学号，除了学号之外的属性，都完全依赖于学号符合 2NF，成绩表中只有确定了学号和课程才能确定分数，所以也不存在部分依赖，它也符合 2NF

但是仔细观察，2NF 并没有解决上面说的**插入异常**和**删除异常**，班级和班主任信息还是会受到学生信息的影响，这就需要 3NF 了

### 四. 第三范式 3NF

第三范式是在满足第二范式的基础上，不允许非键属性间接依赖于键

| 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 | 班主任 |
| ---- | -------- | ------ | ------------- | ---- | ------ |
| 0    | 小桑     | 18xxx  | yyy@email.com | A 班 | 罗小虎 |
| 1    | 龙卷     | 19xxx  | xxx@email.com | B 班 | 埼玉   |

观察这张表，学号 -> 班级 -> 班主任之间存在间接依赖，意思就是学号确定，学生所在的班级就能确定，班级确定了，这个班的班主任就能确定，所以班主任和学号存在间接依赖的关系（学号 -> 班主任 -> 班级也是一样的），这就不符合 3NF，解决办法就是把间接依赖的相关属性单独建表

- 学生基本信息表

  | 学号 | 学生姓名 | 手机号 | 邮箱          | 班级 id |
  | ---- | -------- | ------ | ------------- | ------- |
  | 0    | 小桑     | 18xxx  | yyy@email.com | 1       |
  | 1    | 龙卷     | 19xxx  | xxx@email.com | 2       |

- 班级表

  | 班级 id | 班级 | 班主任 |
  | ------- | ---- | ------ |
  | 1       | A 班 | 罗小虎 |
  | 2       | B 班 | 埼玉   |

这样就解决了学生数据影响班级数据的问题

当表与表之间有一些关系的时候可以使用关联表建立联系，比如：

可以将书的 id 和作者信息放在一张表里：

- 作者表

  | 作者 id | 姓名 | 作品 id |
  | ------- | ---- | ------- |
  | 1       | 老虚 | 1       |
  | 2       | 诚哥 | 2       |
  | 3       | 麻子 | 3       |

- 作品表

  | 作品 id | 书名     |
  | ------- | -------- |
  | 1       | 马猴烧酒 |
  | 2       | 秒 5     |
  | 3       | CL       |

也可以单独建立关联表

- 作者表

  | 作者 id | 姓名 |
  | ------- | ---- |
  | 1       | 老虚 |
  | 2       | 诚哥 |
  | 3       | 麻子 |

- 作者作品表

  | id  | 作者 id | 作品 id |
  | --- | ------- | ------- |
  | 1   | 1       | 1       |
  | 2   | 2       | 2       |
  | 3   | 3       | 3       |

- 作品表

  | 作品 id | 书名     |
  | ------- | -------- |
  | 1       | 马猴烧酒 |
  | 2       | 秒 5     |
  | 3       | CL       |

### 五. SQL 中的 join

SQL join 用于把来自两个或多个表的行结合起来。它有 7 种用法：

- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL OUTER JOIN
- LEFT JOIN EXCLUDING INNER JOIN
- RIGHT JOIN EXCLUDING INNER JOIN
- FULL OUTER JOIN EXCLUDING INNER JOIN

![](/madao.github.io/database/images/articles/node/database2/image5.png)
[图片来自 - 图解 SQL 里的各种 JOIN](https://zhuanlan.zhihu.com/p/29234064)

他的语法是：

`T1 {[INNER] | {LEFT| RIGHT | FULL}[OUTER]} JOIN T2 ON boolean_expression`

下面实际操作一下：

还是利用 docker 生成一个容器，具体流程参考上一篇

- #### 新建一个数据库

  `CREATE DATABASE IF NOT EXISTS join_demo DEFAULT CHARACTER SET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;`

  `use join_demo;`

- #### 新建 user、book、R_users_books 数据表

  `CREATE TABLE users(id serial, name text);`

  `CREATE TABLE books(id serial, name text);`

  `CREATE TABLE R_users_books(id serial, user_id TINYINT, book_id TINYINT);`

- #### 插入一些数据

  - users

    `insert into users(name) values('allen');`

    `insert into users(name) values('pink');`

  - books

    `insert into books(name) values('Time travel');`

    `insert into books(name) values('idiot');`

  - R_users_books

    `insert into R_users_books(user_id, book_is) values(1,1);`

    `insert into R_users_books(user_id, book_id) values(2,2);`

    现在得到如下的数据表

    - users

      ```
      +----+-------+
      | id | name  |
      +----+-------+
      |  1 | allen |
      |  2 | pink  |
      +----+-------+
      ```

    - books

      ```
      +----+-------------+
      | id | name        |
      +----+-------------+
      |  1 | Time travel |
      |  2 | idiot       |
      +----+-------------+
      ```

    - R_users_books

      ```
      +----+---------+---------+
      | id | user_id | book_id |
      +----+---------+---------+
      |  1 |       1 |       1 |
      |  2 |       2 |       2 |
      +----+---------+---------+
      ```

- #### INNER JOIN

  ```
   select
   users.name as user_name,
   books.name as book_name,
   R_users_books.id as r_id
   from
   (users inner join R_users_books on users.id = R_users_books.user_id)
   inner join books
   on
   R_users_books.book_id = books.id;
  ```

  得到结果：

  ```
  +-----------+-------------+------+
  | user_name | book_name   | r_id |
  +-----------+-------------+------+
  | allen     | Time travel |    1 |
  | pink      | idiot       |    2 |
  +-----------+-------------+------+
  ```

- #### LEFT JOIN

  在 users 里插入一条新数据：

  `insert into users(name) values('Eve');`

  找到 users.id 和 books.id 相等的数据

  ```
  select
  users.name as user_name,
  books.name as book_name
  from
  users left join books on users.id = books.id;
  ```

  结果

  ```
  +-----------+-------------+
  | user_name | book_name   |
  +-----------+-------------+
  | allen     | Time travel |
  | pink      | idiot       |
  | Eve       | NULL        |
  +-----------+-------------+
  ```

  LEFT JOIN 会返回左表中的所有记录，同时右表中关联的数据（符合选择条件的，例子中就是符合 users.id = books.id）也会返回，当某条记录中右表中没有关联数据时会返回 NULL

- #### RIGHT JOIN

  删除一条 users 的记录

  `delete from users where id=1;`

  ```
  select
  users.name as user_name,
  books.name as book_name
  from users full outer join books
  on users.id = books.id;
  ```

  结果：

  ```
  +-----------+-------------+
  | user_name | book_name   |
  +-----------+-------------+
  | NULL      | Time travel |
  | pink      | idiot       |
  +-----------+-------------+
  ```

- ### FULL OUTER JOIN

  MySQL 中不支持 FULL OUTER JOIN，但是可以模拟：

  users 当前的数据

  ```
  +----+-------------+
  | id | name        |
  +----+-------------+
  |  1 | Time travel |
  |  2 | idiot       |
  +----+-------------+
  ```

  books 当前的数据

  ```
  +----+------+
  | id | name |
  +----+------+
  |  2 | pink |
  |  3 | Eve  |
  +----+------+
  ```

  R_users_books 当前的数据

  ```
  +----+---------+---------+
  | id | user_id | book_id |
  +----+---------+---------+
  |  1 |       1 |       1 |
  |  2 |       2 |       2 |
  +----+---------+---------+
  ```

  ```
  select
    R_users_books.id as r_id,
    users.name as user_name,
    users.id as user_id,
    books.name as book_name,
    books.id as book_id
  from (
    select id from users
    union
    select id from books
    union
    select id from R_users_books
  ) as IDs
    left join users on users.id = IDs.id
    left join books on books.id = IDs.id
    left join R_users_books on R_users_books.id = IDs.id;
  ```

  结果：

  ```
  +------+-----------+---------+-------------+---------+
  | r_id | user_name | user_id | book_name   | book_id |
  +------+-----------+---------+-------------+---------+
  |    2 | pink      |       2 | idiot       |       2 |
  | NULL | Eve       |       3 | NULL        |    NULL |
  |    1 | NULL      |    NULL | Time travel |       1 |
  +------+-----------+---------+-------------+---------+
  ```
剩余的集中就不掩饰了，基本上看图片就能明白了

### 六. 事务

事务主要用于处理操作量大，复杂度高的数据，比如要更新某个人员的信息，这个人员的数据关联了很多表，当你更新完某个表之后突然出现一些意外导致无法继续执行后面的更新，这样数据库中的数据就会变得混乱，事务可以很好的避免这种情况，事务可以保证成批的 SQL 语句要么全部执行，要么全部不执行。

数据库关联系统有很多种数据库引擎，事务不是所有引擎都支持的，在MySQL中可以通过：`show engines;` 来查看引擎。

```
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
```

从上表中可以看出 MySQL 默认使用的引擎是InnoDB，这个引擎是支持事务的。
### 七. 索引

索引可以提高数据查询速度，例子：

- 给之前的users添加大量数据

  ```
  delimiter $$
  CREATE PROCEDURE insertData()
  begin
  declare i int default 1;
  while(i<10000) do
  insert into users(name) values('just we');
  set i=i+1;
  end while;
  end $$

  delimiter ;

  call insertData();
  ```

- 查找id为1994的数据，结果是：

  ```
  mysql> select * from users where id=1994;
  +------+---------+
  | id   | name    |
  +------+---------+
  | 1994 | just we |
  +------+---------+
  1 row in set (0.01 sec)
  ```

- 添加索引后查询

  `alter table users add index index_id(id);`

  ```
  mysql> select * from users where id=1994;
  +------+---------+
  | id   | name    |
  +------+---------+
  | 1994 | just we |
  +------+---------+
  1 row in set (0.00 sec)
  ```

  > 索引的优点是提高了查询效率，缺点是在插入、更新和删除记录时，需要同时修改索引，因此，索引越多，插入、更新和删除记录的速度就越慢。 ---- [索引](https://www.liaoxuefeng.com/wiki/1177760294764384/1218728442198976)
