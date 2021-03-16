继续完善博客系统，今天要实现的功能是登录

### 一. 创建登录页面

- pages/users/signIn.tsx

  ```
  import { NextPage } from 'next';
  import React, { useCallback, useState } from 'react';

  const TheInput: React.FC<{
    inputConfigs: {
      type: string;
      placeholder: string;
      name: string;
    }
    value: string;
    labelText: string;
    onChange: React.FormEventHandler<HTMLInputElement>
  }> = (props) => {
    const {
      inputConfigs, labelText, value, onChange,
    } = props;
    return (
      <label htmlFor={inputConfigs.name}>
        {labelText}
        <input {...inputConfigs} value={value} onChange={onChange} />
      </label>
    );
  };

  const SignIp: NextPage = () => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');

    const inputList = [
      {
        id: 1,
        inputConfigs: { type: 'text', placeholder: '请输入用户名', name: 'username' },
        labelText: '用户名：',
        value: username,
        onChange: (event: React.FormEvent<HTMLInputElement>) => setUsername(event.currentTarget.value),
      },
      {
        id: 2,
        inputConfigs: { type: 'password', placeholder: '请输入密码', name: 'password' },
        labelText: '密码：',
        value: password,
        onChange: (event: React.FormEvent<HTMLInputElement>) => setPassword(event.currentTarget.value),
      },
    ];

    const onSubmit = useCallback((event: React.FormEvent<HTMLFormElement>) => {
      event.preventDefault();
    }, [username, password]);

    return (
      <div>
        <h1>登录</h1>
        <form onSubmit={onSubmit}>
          {inputList.map((config) => (
            <TheInput {...config} key={config.id} />
          ))}
          <button type="submit">提交</button>
        </form>
      </div>
    );
  };

  export default SignIp;
  ```

  代码和注册页面一样，只要改下组件名即可。

### 二. 实现登录接口

登录接口命名为 sessions，意思是登录接口其实是创建了一个会话或者是在线的用户

在实现之前先优化一下代码

1. 做注册的时候创建了一个`checkRequiredParams`方法，这个方法是用来检查请求参数是否有缺失，在登录的时候也需要做这个验证，所以把`checkRequiredParams`方法单独存放在一个文件中便于复用

   - utils/validateRequestMetadata.ts

     ```
     export const checkRequiredParams = (
       targetObject: {[key: string]: any},
       requiredKeys: string[],
     ) => {
       const targetObjectKeys = Object.keys(targetObject);
       let missingKey = '';
       const passed = requiredKeys.every((key) => {
         if (targetObjectKeys.indexOf(key) === -1) {
           missingKey = key;
           return false;
         }
         return true;
       });
       return {
         passed, missingKey,
       };
     };
     ```

     拿出来之后记得在`pages/api/signUp.tsx`中引入

2. 和注册一样，登录也需要做数据验证，做注册的时候有封装一个 signUpValidate 方法，对于这个方法存放的位置并不好，如果了解过`MVC（Model–View–Controller）`模式，这个验证其实可以看作 Controller 的部分，pages 就是 View，src/entity 就是 Model（个人理解）

   - 重命名 src 文件夹为 model，记得要修改代码中相关路径及`ormconfig.json`中的路径
   - 创建 controller 目录，并重构`signUpValidate.tsx`为`SignUp.ts`

     - controller/SignUp.ts

       ```
       import getConnection from '~/utils/getConnection';
       import { User } from '~/model/entity/User';

       class SignUpController {
         public username: string;

         public password: string;

         public async validator() {
           if (!this.username.trim()) {
             return '用户名不能为空';
           }
           if (!/^[a-zA-Z0-9]{2,15}$/.test(this.username.trim())) {
             return '用户名格式错误，用户名由2～15个英文字母或数字组成';
           }
           const sameUsers = await (await getConnection()).manager.find(User, {
             username: this.username,
           });
           if (sameUsers.length) {
             return '用户名已存在';
           }
           if (!this.password.trim()) {
             return '密码不能为空';
           }
           if (this.password.length < 7) {
             return '密码长度太短，至少需要7位';
           }
           return true;
         }

         constructor(data: { username: string, password: string }) {
           Object.assign(this, data);
         }
       }

       export default SignUpController;
       ```

- controller/SignIn.ts

  实现操作登录数据的 controller

  ```
  import SHA3 from 'crypto-js/sha3';
  import getConnection from '~/utils/getConnection';
  import { User } from '~/model/entity/User';

  class SignInController {
    public username: string;

    public password: string;

    public user: User;

    public async validator() {
      if (!this.username.trim()) {
        return '请填写用户名';
      }
      const user = await (await getConnection()).manager.findOne(User, {
        username: this.username,
      });
      if (!user) {
        return '用户名不存在';
      }
      const passwordDigest = SHA3(this.password, { outputLength: 256 }).toString();
      if (passwordDigest !== user.passwordDigest) {
        return '密码与用户名不匹配';
      }
      this.user = user;
      return true;
    }

    constructor(data: { username: string, password: string }) {
      Object.assign(this, data);
    }
  }

  export default SignInController;
  ```

- pages/api/sessions

  ```
  import { NextApiHandler } from 'next';
  import setResponseData from '~/utils/setResponseData';
  import SignInController from '~/controller/SignIn';
  import { checkRequiredParams } from '~/utils/validateRequestMetadata';
  import {
    CODE_MISSING_USERNAME,
    CODE_MISSING_PASSWORD,
  } from '~/utils/constants';

  const sessions: NextApiHandler = async (request, response) => {
    if (request.method !== 'POST') {
      response.setHeader('Allow', 'POST');
      return setResponseData.call(response, 405, null);
    }
    const { passed, missingKey } = checkRequiredParams(request.body, ['username', 'password']);
    if (!passed) {
      return missingKey === 'username'
        ? setResponseData.call(response, 400, null, CODE_MISSING_USERNAME)
        : setResponseData.call(response, 400, null, CODE_MISSING_PASSWORD);
    }
    const { username, password } = request.body;
    const signInController = new SignInController({ username, password });
    const validationResult = await signInController.validator();
    if (validationResult === true) {
      return setResponseData.call(response, 200, signInController.user);
    }
    return setResponseData.call(response, 422, null, undefined, validationResult);
  };

  export default sessions;
  ```

### 三. 补全登录页面提交方法

- pages/user/signIn.tsx

  ```
  import axios, { AxiosError } from 'axios';
  // 原有代码保持不变
  const onSubmit = useCallback((event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    axios.post('/api/v1/sessions', { username, password })
      .then((res) => {
        console.log(res);
      }, (error: AxiosError) => {
        console.log(error);
      });
  }, [username, password]);
  ```

  虽然现在登录接口可以请求成功，但是并不算登录成功，因为服务端仍然不知道用户已经登录了，使用`cookie & session`机制可以让服务端记住用户的登录状态。

### 四. cookie & session

> HTTP 是无状态的：在同一个连接中，两个执行成功的请求之间是没有关系的。这就带来了一个问题，用户没有办法在同一个网站中进行连续的交互，比如在一个电商网站里，用户把某个商品加入到购物车，切换一个页面后再次添加了商品，这两次添加商品的请求之间没有关联，浏览器无法知道用户最终选择了哪些商品。而使用 HTTP 的头部扩展，HTTP Cookies 就可以解决这个问题。把 Cookies 添加到头部中，创建一个会话让每次请求都能共享相同的上下文信息，达成相同的状态。

上面是 MDN 上的描述，把为什么用 cookie 说的非常清楚

cookie：是服务端放在浏览器上的一段数据，这个数据会随着之后的请求带回给服务端

session：是服务端自己存储的一段数据，可以存在内存、文件甚至是数据库里面，session 使用请求中携带的 cookie 来识别请求。

上面就是一些基本概念，很不幸的是 Next.js 没有内置对`cookie&session`机制的支持[文档](https://nextjs.org/docs/authentication)，需要安装第三方库

- 安装依赖

  ```
  yarn add next-iron-session
  ```

- 扩充 NextApiRequest

  因为 next-iron-session 会给 Next.js 中的 request 对象添加一个 session 的属性，需要让 TypeScript 也知道

  types/addonNextApiRequest.d.ts

  ```
  import { Session } from 'next-iron-session';

  declare module 'next' {
    export interface NextApiRequest {
      session: Session
    }
  }
  ```

- utils/withSession.tsx

  ```
  import { Handler, withIronSession } from 'next-iron-session';

  export default function withSession(handler: Handler) {
    return withIronSession(handler, {
      cookieName: 'diary-of-madao',
      password: process.env.COOKIE_PASSWORD!,
      cookieOptions: {
        secure: process.env.NODE_ENV === 'production',
      },
    });
  }
  ```
  主要说下 withIronSession 中的参数

  - cookieName：cookie 是以`key=value;key1=value1`这种形式进行传输和储存的，cookieName 就是设置这个 key 的
  - password：加密 cookie 的密钥，最少 32 位，注意代码里使用了环境变量，这也是文档推荐的写法，环境变量需要你只存在你的电脑上[Environment Variables](https://nextjs.org/docs/basic-features/environment-variables)
  - secure：如果为 true，只有在 https 协议下才设置 cookie

- 改写 pages/api/sessions.tsx

  ```
  import withSession from '~/utils/withSession';
  // 之前代码不变

  const sessions: NextApiHandler = async (request, response) => {
    if (validationResult === true) {
      request.session.set('currentUser', signInController.user.id);
      await request.session.save();
      return setResponseData.call(response, 200, signInController.user);
    }
  }

  export default withSession(sessions);
  ```

- 创建.env.local

  ```
  COOKIE_PASSWORD=xxxxxxx
  ```

  加密的密钥可以去这里[password-generator](https://1password.com/password-generator/)生成一些无规则的字符串

  注意这个文件需要放在`.gitignore`里面，不能提交到 GitHub 上去。

### 五. 在页面上判断用户是否登录

- pages/users/signIn.tsx

  ```
  import { GetServerSideProps, NextPage, InferGetServerSidePropsType } from 'next';
  import withSession from '~/utils/withSession';

  // 之前代码保持不变

  const SignIp: NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = (props) => {
    // 之前代码保持不变
    const { userId } = props;
    return (
      <div>
        {/* 之前代码保持不变 */}
        <p>{userId}</p>
      </div>
    );
  }

  export const getServerSideProps: GetServerSideProps<{
    userId: number
  }> = withSession(async (context) => {
    // currentUser是在登录成功的时候存入session的，这里可以通过同样的key拿到存储的信息，之前存的是userId，所以这里拿到的也是userId
    const userId = context.req.session.get('currentUser') || null;
    return {
      props: {
        userId,
      },
    };
  });
  ```