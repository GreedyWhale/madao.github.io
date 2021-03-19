今天实现博客的增删改查功能，这个功能实现完成后，整个博客系统的主要逻辑也就完成了，那就可以正式做起来了，后面剩余的就是部署相关的流程，这个等我正式做完这个系统再写。

### 一. 实现创建博客页面

- pages/blogs/editor.tsx

  ```
  import React, { useRef } from 'react';
  import { NextPage } from 'next';

  const BlogEditor:NextPage = () => {
    const input = useRef<HTMLInputElement>(null);
    const textarea = useRef<HTMLTextAreaElement>(null);
    return (
      <div>
        <form>
          <label htmlFor="title">
            标题
            <input
              type="text"
              placeholder="请输入标题"
              name="title"
              ref={input}
            />
          </label>
          <br />
          <textarea ref={textarea} />
          <br />
          <button type="submit">提交</button>
        </form>
      </div>
    );
  };

  export default BlogEditor;
  ```

### 二. 实现 blogs 接口

- 检查登录

  对博客的编辑都必须登录

  - pages/api/v1/blogs.tsx

    ```
    import { NextApiHandler, NextApiRequest, NextApiResponse } from 'next';
    import withSession from '~/utils/withSession';

    const blogs: NextApiHandler = async (request, response) => {
      if (!request.session.get('currentUser')) {
        response.redirect(301, '/users/signIn');
        response.end();
        return;
      }
    };
    ```

    **注意**: 接口这里的判断，如果是你发送 AJAX 后发现没有登录，使用`response.redirect`并不会让浏览器自动重定向至登录页面，而是会让浏览器再次以相同的请求方法请求一次`/users/signIn`，除非直接在浏览器的地址栏输入`api/v1/blogs`，这样才会让浏览器自动重定向至`/users/signIn`，所以登录检查需要在对应的页面再检查一次。

  - pages/blogs/editor.tsx

    ```
    // 之前代码保持不变
    import { GetServerSideProps, NextPage } from 'next';
    import withSession from '~/utils/withSession';

    export const getServerSideProps: GetServerSideProps = withSession(async (context) => {
      const userId = context.req.session.get('currentUser') || null;
      if (!userId) {
        context.res.writeHead(307, { Location: '/users/signIn' });
        context.res.end();
      }
      return { props: { } };
    });
    ```

- 判断请求方法

  1. POST：创建
  2. GET：获取
  3. DELETE：删除
  4. PUT：更新

  - pages/api/v1/blogs.tsx

    ```
    import { NextApiHandler, NextApiRequest, NextApiResponse } from 'next';

    const blogs: NextApiHandler = async (request, response) => {
      const { method } = request;
      switch (method) {
        case 'POST':
          return setResponseData.call(response, 200, null);
        case 'GET':
          return setResponseData.call(response, 200, null);
        case 'DELETE':
          return setResponseData.call(response, 200, null);
        case 'PUT':
          return setResponseData.call(response, 200, null);
        default:
          response.setHeader('Allow', 'POST GET DELETE PUT');
          return setResponseData.call(response, 405, null);
      }
    };

    export default createBlog;
    ```

- 实现创建博客逻辑

  - pages/api/v1/blogs.tsx

    ```
    const blogs: NextApiHandler = async (request, response) => {
      switch (method) {
        case 'POST':
          return createBlog(request, response);
        // 之前代码保持不变
      }
    }
    ```

    - 判断请求参数

      - pages/api/v1/blogs.tsx

        ```
        import { NextApiHandler, NextApiRequest, NextApiResponse } from 'next';
        import {
          CODE_MISSING_BLOG_CONTENT,
          CODE_MISSING_BLOG_TITLE,
          CODE_MISSING_USER_ID,
        } from '~/utils/constants';

        const createBlog = (request: NextApiRequest, response: NextApiResponse) => {
          const { passed, missingKey } = checkRequiredParams(request.body, ['title', 'content', 'userId']);
          if (!passed) {
            const errorCode: {[key: string]: number} = {
              title: CODE_MISSING_BLOG_TITLE,
              content: CODE_MISSING_BLOG_CONTENT,
              userId: CODE_MISSING_USER_ID,
            };
            return setResponseData.call(response, 400, null, errorCode[missingKey]);
          }
        }
        // 之前代码保持不变
        ```

      - utils/constants.tsx

        ```
        /** 缺失博客标题 */
        export const CODE_MISSING_BLOG_TITLE = 400003;
        /** 缺失博客内容 */
        export const CODE_MISSING_BLOG_CONTENT = 400004;
        /** 缺失用户id */
        export const CODE_MISSING_USER_ID = 400005;
        ```

      - utils/setResponseData.tsx

        ```
        // 之前代码保持不变
        const customizableMessage: {[key: string ]: string} = {
          400001: '缺失用户名',
          400002: '缺失密码',
          400003: '缺失博客标题',
          400004: '缺失博客内容',
          400005: '缺失用户ID',
        };
        ```

    - 验证提交数据

      - controller/Blog.ts

        ```
        import getConnection from '~/utils/getConnection';
        import { User } from '~/model/entity/User';
        import { Blog } from '~/model/entity/Blog';

        class BlogController {
          public title: string;

          public content: string;

          public authorId: string;

          private author: User;

          public async validator() {
            if (!this.title) {
              return '博客标题不能为空';
            }
            if (!this.content) {
              return '博客内容不能为空';
            }
            if (!this.authorId) {
              return '用户Id为空';
            }
            const author = await (await getConnection())
              .manager
              .findOne(User, { id: parseInt(this.authorId, 10) });
            if (!author) {
              return '用户信息不匹配';
            }
            this.author = author;
            return true;
          }

          constructor(data: { title: string, content: string, authorId: string }) {
            Object.assign(this, data);
          }
        }

        export default BlogController;
        ```

      - pages/api/v1/blogs.tsx

        ```
        import BlogController from '~/controller/Blog';

        const createBlog = async (request: NextApiRequest, response: NextApiResponse) => {
           // 之前代码保持不变
          const { title, content, authorId } = request.body;
          const blogController = new BlogController({ title, content, authorId });
          const validationResult = await blogController.validator();
          if (validationResult === true) {
            const isSucceed = await blogController.create();
            return isSucceed
              ? setResponseData.call(response, 200, null)
              : setResponseData.call(response, 500, null, CODE_BLOG_CREATE_ERROR);
          }
          return setResponseData.call(response, 422, null, undefined, validationResult);
        };
        ```

    - 创建博客

      - controller/Blog.ts

        ```
        class BlogController {
          // 之前代码保持不变
          public async create() {
            const blog = new Blog({
              title: this.title,
              content: this.content,
              author: this.author,
            });
            const result = await (await getConnection())
              .manager
              .save(blog);

            return result;
          }
        }

        export default BlogController;
        ```

      - utils/constants.tsx
        ```
        // 之前代码保持不变
        /** 博客创建失败 */
        export const CODE_BLOG_CREATE_ERROR = 500001;
        ```
      - utils/setResponseData.tsx
        ```
        // 之前代码保持不变
        const customizableMessage: {[key: string ]: string} = {
          400001: '缺失用户名',
          400002: '缺失密码',
          400003: '缺失博客标题',
          400004: '缺失博客内容',
          400005: '缺失用户ID',
          500001: '博客创建失败',
        };
        ```
      - pages/api/v1/blogs.tsx

        ```
        import {
          CODE_MISSING_BLOG_CONTENT,
          CODE_MISSING_BLOG_TITLE,
          CODE_MISSING_USER_ID,
          CODE_BLOG_CREATE_ERROR,
        } from '~/utils/constants';

        const createBlog = async (request: NextApiRequest, response: NextApiResponse) => {
          // 之前代码保持不变
          if (validationResult === true) {
            const result = await blogController.create();
            return result
              ? setResponseData.call(response, 200, result)
              : setResponseData.call(response, 500, null, CODE_BLOG_CREATE_ERROR);
          }
        }
        ```

      - model/entity/Blog.ts
        ```
        import { omit } from 'lodash';
        // 之前代码保持不变
        @Entity('blogs')
        export class Blog {
          toJSON() {
            return omit(this, 'author');
          }
        }
        ```
      - pages/blogs/editor.tsx

        ```
        import React, {
          FormEventHandler,
          useCallback,
          useRef,
        } from 'react';
        import { GetServerSideProps, NextPage, InferGetServerSidePropsType } from 'next';
        import axios from 'axios';
        import withSession from '~/utils/withSession';

        const BlogEditor: NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = (props) => {
          const { authorId } = props;
          const input = useRef<HTMLInputElement>(null);
          const textarea = useRef<HTMLTextAreaElement>(null);
          const onSubmit: FormEventHandler<HTMLFormElement> = useCallback((e) => {
            e.preventDefault();
            axios.post('/api/v1/blogs', {
              title: input.current?.value,
              content: textarea.current?.value,
              authorId,
            }).then((res) => console.log(res), (err) => console.log(err));
          }, [input, textarea, authorId]);
          return (
            <div>
              <form onSubmit={onSubmit}>
                <label htmlFor="title">
                  标题
                  <input
                    type="text"
                    placeholder="请输入标题"
                    name="title"
                    ref={input}
                  />
                </label>
                <br />
                <textarea ref={textarea} />
                <br />
                <button type="submit">提交</button>
              </form>
            </div>
          );
        };

        export const getServerSideProps: GetServerSideProps = withSession(async (context) => {
          const userId = context.req.session.get('currentUser') || null;
          if (!userId) {
            context.res.writeHead(307, { Location: '/users/signIn' });
            context.res.end();
          }
          return { props: { authorId: userId } };
        });

        export default BlogEditor;
        ```

[完整代码](https://github.com/GreedyWhale/diary-of-madao/tree/56c04a9bb1adc744798dd53016a428abaca39d02)

本来想把删除、更新、查找的实现过程都写上去，但是发现逻辑差不多，只是在操作数据库那里换成对的方法就可以了，所以这里就不记录了，而且发现上面写的也有些啰嗦，等我完全实现完成后再重新修改吧。
