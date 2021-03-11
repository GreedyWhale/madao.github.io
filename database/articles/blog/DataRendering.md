### 一. 目标

结合上一篇笔记将数据库中的数据渲染在页面上

### 二. 在 Next.js 中连接数据库

- utils/getConnection.tsx

  ```
  import 'reflect-metadata';
  import { ConnectionManager } from 'typeorm';
  import ormConfig from 'ormconfig.json';
  import { User } from 'src/entity/User';
  import { Blog } from 'src/entity/Blog';
  import { Comment } from 'src/entity/Comment';

  const getConnection = async () => {
    const connectionManager = new ConnectionManager();
    const currentConnection = connectionManager.has('default') && connectionManager.get('default');
    if (currentConnection && currentConnection.isConnected) {
      return currentConnection;
    }
    // @ts-ignore
    const connection = connectionManager.create({
      ...ormConfig,
      entities: [User, Blog, Comment],
    });
    await connection.connect();
    return connection;
  };

  export default getConnection;
  ```

  **注意**：

  1. 在 Next.js 中开发环境中热更新会导致重复创建连接，需要先判断一下当前是否已经有连接上的连接
  2. 当成功连接数据库后，获取数据表的时候有可能会报`No metadata for "xxxx" was found`这个错，解决方法参考[No metadata for "User" was found](https://github.com/typeorm/typeorm/issues/1327#issuecomment-455791578)

### 二. 获取数据

- utils/hooks/useBlogs.tsx

  ```
  import getConnection from 'utils/getConnection';
  import { Blog } from 'src/entity/Blog';

  export default async function useBlogs() {
    const connection = await getConnection();
    const blogs = await connection.manager.find<Blog>('blogs');
    return JSON.parse(JSON.stringify(blogs));
  }

  ```

  **注意**：获取到的数据中有 Date，undefined 类型会导致 getServerSideProps 函数返回值不是一个可序列化的对象[getserversideprops-server-side-rendering](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering)而报错，所以需要提前对数据进行 JSON 序列化

### 三. 展示博客列表

- pages/index.tsx

  ```
  import React from 'react';
  import type { NextPage } from 'next';
  import Link from 'next/link';

  const Home: NextPage = () => (
    <div>
      这是我的博客首页
      <Link href="/blogs/index">博客列表</Link>
    </div>
  );

  export default Home;
  ```

- pages/blogs/index.tsx

  ```
  import React from 'react';
  import type { GetServerSideProps, NextPage, InferGetServerSidePropsType } from 'next';
  import Link from 'next/link';
  import { Blog } from 'src/entity/Blog';
  import useBlogs from 'utils/hooks/useBlog';

  const BlogList: NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = (props) => {
    const { blogs } = props;
    return (
      <>
        <div>文章列表</div>
        <ul>
          { blogs && blogs.map((blog) => (
            <li key={blog.id}>
              <Link href={`/blogs/${blog.id}`}>{blog.title}</Link>
            </li>
          )) }
        </ul>
      </>
    );
  };

  export const getServerSideProps: GetServerSideProps<{
    blogs: Blog[]
  }> = async () => {
    const blogs = await useBlogs();
    return {
      props: {
        blogs,
      },
    };
  };

  export default BlogList;
  ```

### 四. 展示博客详情

- utils/hooks/useBlogs.tsx

  由于要在 useBlogs 中加新功能，所以要修改一下 useBlogs

  ```
  import getConnection from 'utils/getConnection';
  import { Blog } from 'src/entity/Blog';

  const getBlogDetails = async (id: number) => {
    const connection = await getConnection();
    const blog = await connection
      .getRepository(Blog)
      .createQueryBuilder('blog')
      .where('blog.id = :id', { id })
      .getOne();
    return JSON.parse(JSON.stringify(blog));
  };

  const getBlogs = async (): Promise<Blog[]> => {
    const connection = await getConnection();
    const blogs = await connection.getRepository(Blog).find();

    return JSON.parse(JSON.stringify(blogs));
  };

  export default function useBlogs() {
    return {
      getBlogs,
      getBlogDetails,
    };
  }

  ```

  然后在`pages/blogs/index.tsx`中修改：

  ```
  export const getServerSideProps: GetServerSideProps<{
    blogs: Blog[]
  }> = async () => {
    const { getBlogs } = useBlogs();
    const blogs = await getBlogs();
    return {
      props: { blogs },
    };
  };
  ```

- pages/blogs/[id].tsx

  ```
  import React from 'react';
  import type { NextPage, GetServerSideProps, InferGetServerSidePropsType } from 'next';
  import useBlogs from 'utils/hooks/useBlog';

  const BlogDetails:NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = (props) => {
    const { blog } = props;
    return (
      <div>{blog.content}</div>
    );
  };

  export const getServerSideProps: GetServerSideProps = async (context) => {
    const { params } = context;
    const { getBlogDetails } = useBlogs();
    const blog = params
      ? await getBlogDetails(parseInt((params.id as unknown as string), 10))
      : { content: '文章不存在' };
    return {
      props: {
        blog,
      },
    };
  };

  export default BlogDetails;
  ```

  这样文章列表和文章详情就可以通过Next.js的服务端渲染的方式呈现给用户，数据也是直接从数据库读取的。

  [GitHub](https://github.com/GreedyWhale/diary-of-madao/tree/810d1b1a2e3cd34a94ad3eb791b31fa23c38696a)