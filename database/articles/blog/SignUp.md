今天实现注册相关逻辑

### 一. 需求

1. 新增页面用于用户注册
2. 在后端对用户提交的数据进行校验

### 二. 注册页面

- pages/users/signUp.tsx

  ```
  import axios from 'axios';
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

  const SignUp: NextPage = () => {
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
      axios.post('/api/v1/users', {
        username, password,
      }).then((response) => {
        console.log(response.data);
      }, (error) => console.log(error));
    }, [username, password]);

    return (
      <div>
        <h1>注册</h1>
        <form onSubmit={onSubmit}>
          {inputList.map((config) => (
            <TheInput {...config} key={config.id} />
          ))}
          <button type="submit">提交</button>
        </form>
      </div>
    );
  };

  export default SignUp;

  ```

一个简单的注册页面只有两个输入框和提交按钮

### 三. 获取到页面提交的数据

注册相当于创建一个用户，所以使用 POST 方法，POST 方法提交的数据在 Next.js 中通过 request 对象的 body 属性获得

- pages/api/v1/users.tsx

  ```
  import { NextApiHandler } from 'next';

  const users: NextApiHandler = async (request, response) => {
    const { body } = request;
    console.log(body);
    response.status(200);
    response.setHeader('Content-Type', 'application/json');
    response.json({
      message: '请求成功',
      data: null,
    });
  };

  export default users;
  ```

### 三. 重新配置一个打包流程

本来这一步是要做数据校验的逻辑的，但是我遇到一个大坑，就是之前 typeORM 所用的到 ts 代码比如：entity、migration 里面的那些代码仅仅是简单的用 babel 进行了编译，如果这些文件中引用了其他目录（非 src 目录）下的 ts 文件，那么 babel 是不会帮你编译那个文件的，会导致无法进行数据库迁移（Cannot find module 'xxx/xxxx'），所以要为 typeORM 单独配置一个打包流程，我选择 webpack（真香）！

- 安装依赖

  ```
  yarn add webpack webpack-cli webpack-node-externals fork-ts-checker-webpack-plugin babel-loader clean-webpack-plugin -D
  ```

- tools/entries.js

  ```
  // 用于获取src目录下所有ts文件
  const fs = require('fs');
  const { join } = require('path');

  const entries = {};

  const getEntries = (path, prefix = '') => {
    const files = fs.readdirSync(path, {
      withFileTypes: true,
    });
    files.forEach((file) => {
      const currentPath = join(path, file.name);
      file.isFile()
        ? entries[join(prefix, file.name)] = currentPath
        : getEntries(currentPath, join(prefix, file.name));
    });
  };

  getEntries(join(__dirname, '../src'));

  module.exports = entries;
  ```

- webpack.config.js

  ```
  const nodeExternals = require('webpack-node-externals');
  const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const { join } = require('path');
  const entries = require('./tools/entries');

  module.exports = {
    mode: 'production',
    entry: entries,
    target: ['node14.15'],
    output: {
      path: join(__dirname, 'dist'),
      filename: '[name].js',
    },
    module: {
      rules: [
        {
          test: /\.tsx?$/,
          use: { loader: 'babel-loader' },
          exclude: /node_modules/,
        },
      ],
    },
    resolve: {
      extensions: ['.tsx', '.ts', '.js'],
      alias: {
        '~': join(__dirname, './'),
      },
    },
    plugins: [
      new ForkTsCheckerWebpackPlugin(),
      new CleanWebpackPlugin(),
    ],
    externals: [
      // 不需要打包node_modules里的代码
      nodeExternals(),
    ],
    externalsPresets: {
      node: true,
    },
  };
  ```

- package.json

  ```diff
  - "build:orm": "babel -w ./src --out-dir dist --extensions \".ts,.tsx\"",
  + "build:orm": "webpack build --config ./webpack.config.js",
  - "watch:orm": "run-s clean:dist build:orm",
  + "watch:orm": "webpack build --config ./webpack.config.js -w",
  ```

### 四. 验证数据

- utils/signUpValidate.tsx

  ```
  import getConnection from './getConnection';
  import { User } from '~/src/entity/User';

  const validator = async ({ username, password }: {
    username: string; password: string;
  }) => {
    if (!username.trim()) {
      return '用户名不能为空';
    }
    if (!/^[a-zA-Z0-9]{2,15}$/.test(username.trim())) {
      return '用户名格式错误，用户名由2～15个英文字母或数字组成';
    }
    const sameUsers = await (await getConnection()).manager.find(User, {
      username,
    });
    if (sameUsers.length) {
      return '用户名已存在';
    }
    if (!password.trim()) {
      return '密码不能为空';
    }
    if (password.length <= 7) {
      return '密码长度太短，至少需要7位';
    }
    return true;
  };

  module.exports = validator;
  ```

  记录一下：验证这里还可以使用另外一种做法，就是全部判断然后将错误全部返回，这样做的好处是，当一个表单需要填写的字段过多的时候，用户可以一次性知道填错了什么，不然就得一直同过请求来修改填写错误的字段。

### 四. 在数据表中设置唯一索引

因为有一个用户名唯一性检查，这个是需要对数据库进行设置，不能存储相同的用户名

_说明_：之前对 migration 的认识不足，一直以为 migration 只能用于创建数据表，所以 package.json 中命令的名字起的不好，现在将`[x]:table`格式的命令改为`[x]:migration`格式

- `yarn c:migration -n AddUniqueIndexToUsername`

- src/migration/AddUniqueIndexToUsername.ts

  ```
  import { MigrationInterface, QueryRunner, TableIndex } from 'typeorm';

  export class AddUniqueIndexToUsername1615452954116 implements MigrationInterface {
    public async up(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.createIndex('users', new TableIndex({
        name: 'username_unique',
        columnNames: ['username'],
        isUnique: true,
      }));
    }

    public async down(queryRunner: QueryRunner): Promise<void> {
      await queryRunner.dropIndex('users', 'username_unique');
    }
  }
  ```

  - 创建唯一索引参数中的 name 指的是索引的名字

### 五. 实现注册接口

- pages/api/v1/users.tsx

  ```
  import { NextApiHandler } from 'next';

  const users: NextApiHandler = async (request, response) => {}

  export default users;
  ```

  1. 格式化返回数据结构

     一般来说，接口返回的结果都有固定的格式，这里我使用这种格式：

     ```
     {
       code: number,
       message: string
       data: {[key: string]: any},
     }
     ```

     code 是自定义的状态码，因为前端有可能根据错误类型做不同的处理，比如用户名缺失和密码缺失的处理不一样，只是使用 http 状态码是不够用的，所以需要自定义一套状态码

     utils/setResponseData.tsx

     ```
     import { NextApiResponse } from 'next';

     const errorMap: {[key: string ]: string} = {
       200: '请求成功',
       400: '请求异常',
       405: '请求方法错误',
       422: '请求提交的数据错误',
     };
     const customizableMessage: {[key: string ]: string} = {
       400001: '缺失用户名',
       400002: '缺失密码',
     };

     export default function setResponseData(
       this: NextApiResponse,
       httpStatusCode: number,
       data: {[key: string]: any} | null,
       customizableCode?: number,
       message?: string,
     ) {
       this.status(httpStatusCode);
       this.json({
         code: customizableCode || httpStatusCode,
         message: message || (customizableCode ? customizableMessage[customizableCode] : errorMap[httpStatusCode]),
         data,
       });
     }
     ```

  2. 检查请求方法

     pages/api/v1/users.tsx

     ```
     import { NextApiHandler } from 'next';
     import setResponseData from '~/utils/setResponseData';

     const users: NextApiHandler = async (request, response) => {
       if (request.method !== 'POST') {
         response.setHeader('Allow', 'POST');
         return setResponseData.call(response, 405, null);
       }
     }

     export default users;
     ```

  3. 检查请求参数是否有误

     在验证数据之前，先判断一下数据存不存在，就是前端在请求接口的时候是否有携带这些参数，我不确定这样做是不是最佳实践，因为在后面会校验数据，相当于又检查了一遍

     pages/api/v1/users.tsx

     ```
     // 自定义code常量
     import {
       CODE_MISSING_USERNAME,  // 400001
       CODE_MISSING_PASSWORD,  // 400002
     } from '~/utils/constants';

     const checkRequiredParams = (
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

     const users: NextApiHandler = async (request, response) => {
       // 之前代码保持不变
       // ...
       const { passed, missingKey } = checkRequiredParams(request.body, ['username', 'password']);
       if (!passed) {
         return missingKey === 'username'
           ? setResponseData.call(response, 400, null, CODE_MISSING_USERNAME)
           : setResponseData.call(response, 400, null, CODE_MISSING_PASSWORD);
       }
     }
     ```

  4. 验证表单数据

     pages/api/v1/users.tsx

     ```
     import getConnection from '~/utils/getConnection';
     import signUpValidator from '~/utils/signUpValidate';

     const users: NextApiHandler = async (request, response) => {
       // 之前代码保持不变
       // ...
       const { username, password } = request.body;
       const validationResult = await signUpValidator({ username, password });
       if (validationResult === true) {
         const user = new User({ username, passwordDigest: password });
         await (await getConnection()).manager.save(user);
         return setResponseData.call(response, 200, user);
       }
       return setResponseData.call(response, 422, null, undefined, validationResult);
     }
     ```

  5. 完整代码

     ```
     import { NextApiHandler } from 'next';
     import { User } from '~/src/entity/User';
     import getConnection from '~/utils/getConnection';
     import signUpValidator from '~/utils/signUpValidate';
     import setResponseData from '~/utils/setResponseData';
     import {
       CODE_MISSING_USERNAME,
       CODE_MISSING_PASSWORD,
     } from '~/utils/constants';

     const checkRequiredParams = (
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

     const users: NextApiHandler = async (request, response) => {
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
       const validationResult = await signUpValidator({ username, password });
       if (validationResult === true) {
         const user = new User({ username, passwordDigest: password });
         await (await getConnection()).manager.save(user);
         return setResponseData.call(response, 200, user);
       }
       return setResponseData.call(response, 422, null, undefined, validationResult);
     };

     export default users;

     ```

### 六. 处理敏感信息

前面有说过密码不能明文存储，加密有在前端加密的，有在后端加密的，这里暂定在后端加密，后面正式开始做的时候会在前后端都进行加密。

typeORM 提供一个装饰器`@BeforeInsert`，可以在存入数据库之前执行一些任务

- 安装依赖

  ```
  yarn add crypto-js lodash
  yarn add @types/crypto-js @types/lodash -D
  ```

- src/entity/User.ts

  ```
  import SHA3 from 'crypto-js/sha3';
  @Entity('users')
  export class User {
    // 之前代码保持不变
    @BeforeInsert()
    passwordEncryption() {
      this.passwordDigest = SHA3(this.passwordDigest, {
        outputLength: 256,
      }).toString();
    }
  }
  ```

- 实现 toJSON 方法

  因为注册成功需要返回用户信息，一般不会将密码也返回，所以需要实现一个 toJSON 将密码过滤

  [toJSON](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#tojson_%E6%96%B9%E6%B3%95)

  - src/entity/User.ts

    ```
    import { omit } from 'lodash';
    export class User {
      // 之前代码保持不变
      toJSON() {
        return omit(this, ['passwordDigest']);
      }
    }
    ```

### 七. 将注册结果显示在页面上

- pages/users/signUp.tsx

  ```
  const SignUp: NextPage = () => {
    // 之前代码保持不变
    const [signUpResults, setSignUpResults] = useState('');

    const onSubmit = useCallback((event: React.FormEvent<HTMLFormElement>) => {
      event.preventDefault();
      axios.post('/api/v1/users', { username, password }).then((response) => {
        setSignUpResults(response.data.message);
      }, (error: AxiosError) => {
        setSignUpResults(error.response ? error.response.data.message : error.message);
      });
    }, [username, password]);

    return (
      <div>
        {/* 之前代码保持不变 */}
        <p>{signUpResults}</p>
      </div>
    )
  }
  ```

这样简单的注册流程就完成了

[GitHub](https://github.com/GreedyWhale/diary-of-madao/tree/b0f927208ff51e4025f06cb8a9c350ab6a690372)