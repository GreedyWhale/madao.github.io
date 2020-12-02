上一篇总结了一下 useState 原理，除了 useState，React 还提供了其他的 Hook 给开发者使用，这一篇就总结一下其他内置的 Hook，然后就愉快的开始造轮子了！

### 一. useState

useState 返回一个 state 和修改这个 state 的方法 setState，每次使用 setState 去修改 state 的时候，组件会重新渲染

#### 1. 注意事项：

1. 不可局部更新

   ```javascript
   const [user, setUser] = useState({ name: "Allen", age: 17 });

   setUser({ age: 18 }); // 这样会导致user变成{ age: 18 }，丢失name属性
   setUser({ ...user, age: 18 }); // 这样才能保证user的属性不丢失
   ```

2. 如果 state 是一个对象，只有对象的地址（引用）变了，React 才认为这个 state 改变了

   ```javascript
   const [user, setUser] = useState({ name: "Allen", age: 17 });

   user.age = 18;
   setUser(user); // 这样是不会触发重新渲染的
   ```

#### 2. 当参数是函数的时候

useState 和 setState 都可以接受一个函数作为参数：

```javascript
const [n, setN] = useState(() => 0);
setN((n) => n + 1);
```

使用场景：

1. 当 state 的初始值很复杂的时候，可以用 useState 传入一个函数这种写法。
2. 当连续修改 state 的时候，使用 setState 传入一个函数这种写法，例子：

   ```javascript
   const [n, setN] = useState(() => 0);
   setN(n + 1);
   setN(n + 1);
   ```

   上面这种写法最终得到的 n 是 1，并不是 2，原因上一篇有说到，setN 并不会改变 n，所以连续两次 setN(n + 1)的时候，n 的值都为 0，只要改写成这样：

   ```javascript
   const [n, setN] = useState(() => 0);
   setN((n) => n + 1);
   setN((n) => n + 1);
   ```

   这样 n 就会+2，推荐优先使用这一种写法去修改 state

### 二. useReducer

useReducer 像是 useState 的复杂版本，它的语法和 Redux 有些像，使用它也可以代替 Redux，先看看基本的用法：

1. 创建初始数据
   ```javascript
   const initialState = { money: 0 };
   ```
2. 创建 reducer
   ```javascript
   const reducer = (state, action) => {
     switch (action.type) {
       case "take":
         return { ...state, money: state - action.value };
       case "save":
         return { ...state, money: state + action.value };
       default:
         return state;
     }
   };
   ```
3. 使用 useReducer

   ```javascript
   const App = () => {
     const [state, dispatch] = useReducer(reducer, initialState);

     return (
       <div>
         <div>余额：{state.money}</div>
         <button onClick={() => dispatch({ type: "save", value: 100 })}>
           存钱
         </button>
         <button onClick={() => dispatch({ type: "take", value: 10 })}>
           取钱
         </button>
       </div>
     );
   };
   ```

那么什么时候用 useState 什么时候用 useReducer 呢，当数据的关系适合放在一起的时候用 useReducer，比如有一个表单是更新用户信息的，那么这个数据类似这样：

```javascript
const user = {
  name: "",
  age: "",
  gender: "",
};
```

这种情况下对数据有多种修改的需求，比如重置，添加，修改，利用 reducer 可以很好的进行分类操作

#### 如何用 useReducer 代替 Redux

在代替 Redux 之前先简单理解下 Redux 是什么，React 采用的是单项数据流，就是数据只能从一个方向流向另一个方向，而不能反过来，从 props 不能修改这一点就可以看出来，假如子组件想要修改父组件的数据，就只能通过回调函数通知父组件，父组件修改自己的 state，然后在通过 props 传递给子组件，如果有多个组件共用的数据，那么最好的做法就是把这个数据放在最顶层的组件，再通过 props 传递给子组件，当组件越来越多的时候，数据的传递也会变得复杂，Redux 就是为了解决这类问题出现的，它可以更好的、更专业的管理这些数据，至于为什么它更好、更专业，我也不知道，建议先去看看 Flux 的思想。

Redux 中有三个基本的概念：

1. Action

   描述如何修改数据，一般这样定义：

   ```javascript
   const action = {
     type: "ADD_TODO",
     text: "Build my first Redux app",
   };
   ```

   Action 就是一个普通的对象，type 是用来描述如何修改数据的字符串，其他的字段可以任意定义。

2. Reducer

   Reducer 就是一个函数，它会根据 Action 返回新的数据，Reducer 是纯函数，就是返回结果只和参数有关，只会返回一个值，不会修改任何变量：

   ```javascript
   const action = {
     type: "ADD_TODO",
     text: "Build my first Redux app",
   };
   const reducer = (state, action) => {
     switch (action.type) {
       case ADD_TODO:
         return { ...state, task: action.text };
       default:
         return state;
     }
   };
   ```

   reducer 的第一个参数就是当前的数据，需要注意的就是不要改变旧的 state 而是根据 action 返回一个新的 state。

3. Store

   Store 的作用就是把数据和 Reducer 关联起来，store

   ```javascript
   import { createStore } from "redux";
   const action = {
     type: "ADD_TODO",
     text: "Build my first Redux app",
   };
   const reducer = (state, action) => {
     switch (action.type) {
       case ADD_TODO:
         return { ...state, task: action.text };
       default:
         return state;
     }
   };
   const store = createStore(reducer, { task: "初始数据" });
   store.dispatch(action); // 想要修改数据，只能通过store提供的dispatch方法，传入一个action，然后reducer就会根据action进行处理
   ```

了解完 Redux 之后，看下怎么使用 useReducer 代替 redux：

1. 将数据存放至 store
2. 创建 reducer 用于操作数据
3. 使用 useReducer 获取读写数据的 api
4. 使用 React.createContext 创建一个 context
5. 将 useReducer 中获取到的读写 api 传递给 context，并共享给子组件
6. 子组件使用 useContext 获取到读写数据的 api

上面步骤中有两个 api 没有说到，分别是 React.createContext 和 useContext：

- React.createContext： React.createContext 会创建一个 Context，Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法，简单点说就是可以跨组件（跳过中间不需要该数据的组件）传递数据
- useContext：接受一个 context 对象并返回该 context 的当前值，context 的值更新的时候该 hook 会触发重新渲染。

好了，按照这个步骤来实现，先写三个组件

- Grandson.tsx

  ```
  import React from 'react'

  const Grandson = () => {
    const style = {
      color: 'blue',
      borderTop: '1px solid blue'
    }
    return (
      <div style={style}>
        <p>我是孙组件</p>
      </div>
    )
  }

  export default Grandson

  ```

- Child.tsx

  ```
  import React from 'react'
  import Grandson from './Grandson'

  const Child = () => {
    const style = {
      color: 'green',
      borderTop: '1px solid green'
    }
    return (
      <div style={style}>
        <p>我是子组件</p>
        <Grandson />
      </div>
    )
  }

  export default Child

  ```

- Parent.tsx

  ```
  import React from 'react'
  import Child from './Child'

  const Parent = () => {
    const style = {
      color: 'red',
      borderTop: '1px solid red'
    }
    return (
      <div style={style}>
        <p>我是父组件</p>
        <Child />
      </div>
    )
  }

  export default Parent

  ```

- index.tsx

  ```
  import React, { useReducer } from 'react'
  import ReactDom from 'react-dom'
  import Parent from './Parent'

  const App = () => {
    return (
      <div>
        <Parent />
      </div>
    )
  }

  ReactDom.render(<App />, document.querySelector('#app'))
  ```

  效果：

  ![](/madao.github.io/database/images/articles/react/reactHooks/image.png)

1. 将数据存放至 store

   index.tsx

   ```javascript
   // 更好的做法是把它单独的写在一个文件里，方便维护
   const store = {
     user: null,
     skills: [],
     comments: "",
   };
   ```

2. 创建 reducer 用于操作数据

   index.tsx

   ```typescript
   // 更好的做法是把它单独的写在一个文件里，方便维护
   const reducer = (
     state: typeof store,
     action: { type: string; value: any }
   ) => {
     switch (action.type) {
       case "updateUser":
         return { ...state, user: action.value };
       case "updateSkills":
         return { ...state, skills: action.value };
       case "updateComments":
         return { ...state, comments: action.value };
       default:
         return state;
     }
   };
   ```

3. 使用 useReducer 获取读写数据的 api

   index.tsx

   ```diff
   const App = () => {
   +  const [state, dispatch] = useReducer(store, reducer)
     return (
       <div>
         <Parent />
       </div>
     )
   }
   ```

4. 使用 React.createContext 创建一个 context

   index.tsx

   ```typescript
   // 更好的做法是把它单独的写在一个文件里，方便维护
   export const context = React.createContext({
     state: store,
     dispatch: (value: any) => {},
   });
   ```

5. 将 useReducer 中获取到的读写 api 传递给 context，并共享给子组件

   index.tsx

   ```typescript
   const App = () => {
     const [state, dispatch] = useReducer(reducer, store);
     return (
       <context.Provider value={{ state, dispatch }}>
         <Parent />
       </context.Provider>
     );
   };
   ```

   index.tsx 现在的代码

   ```typescript
   import React, { useReducer } from "react";
   import ReactDom from "react-dom";
   import Parent from "./Parent";

   const store = {
     user: null,
     skills: [],
     comments: "",
   };
   const reducer = (
     state: typeof store,
     action: { type: string; value: any }
   ) => {
     switch (action.type) {
       case "updateUser":
         return { ...state, user: action.value };
       case "updateSkills":
         return { ...state, skills: action.value };
       case "updateComments":
         return { ...state, comments: action.value };
       default:
         return state;
     }
   };

   export const context = React.createContext({
     state: store,
     dispatch: (value: any) => {},
   });

   const App = () => {
     const [state, dispatch] = useReducer(reducer, store);
     return (
       <context.Provider value={{ state, dispatch }}>
         <Parent />
       </context.Provider>
     );
   };
   ReactDom.render(<App />, document.querySelector("#app"));
   ```

6. 子组件使用 useContext 获取到读写数据的 api

   Parent.tsx

   ```diff
   + import React, { useContext } from 'react'
   import Child from './Child'
   + import { context } from './index'

   const Parent = () => {
     const style = {
       color: 'red',
       borderTop: '1px solid red'
     }
   + const { state, dispatch } = useContext(context)
     return (
       <div style={style}>
         <p>我是父组件</p>
         <Child />
       </div>
     )
   }

   export default Parent
   ```

   其他组件同上，都是引入同一个 content，然后通过 useContext 获取到数据和修改数据的方法

   现在组件就可以通过 useContext 返回的 state 获取到最新的数据，通过 dispatch 更新数据，举个例子：

   Grandson.tsx

   ```typescript
   import React, { useContext } from "react";
   import { context } from "./index";

   const Grandson = () => {
     const style = {
       color: "blue",
       borderTop: "1px solid blue",
     };
     const action = {
       type: "updateUser",
       value: {
         name: "自来也",
         age: "54",
         height: "191.2cm",
         weight: "88kg",
       },
     };
     const { state, dispatch } = useContext(context);
     return (
       <div style={style}>
         <button onClick={() => dispatch(action)}>更新user</button>
         {state.user && (
           <ul>
             {/*
               (xxx as any)这种写法非常不好，这里写只是为了写例子方便
               或者说在TypeScript中尽量少用 any
             */}
             <li>姓名：{(state.user as any).name}</li>
             <li>年龄：{(state.user as any).age}</li>
             <li>身高：{(state.user as any).height}</li>
             <li>体重：{(state.user as any).weight}</li>
           </ul>
         )}
         <p>我是孙组件</p>
       </div>
     );
   };

   export default Grandson;
   ```

   parent.tsx

   ```typescript
   import React, { useContext } from "react";
   import Child from "./Child";
   import { context } from "./index";

   const Parent = () => {
     const style = {
       color: "red",
       borderTop: "1px solid red",
     };
     const { state } = useContext(context);
     return (
       <div style={style}>
         {state.user && (
           <ul>
             {/*
               (xxx as any)这中写法非常不好，这里写只是为了写例子方便
               或者说在TypeScript中尽量少用 any
             */}
             <li>我也可以获得user的信息</li>
             <li>姓名：{(state.user as any).name}</li>
             <li>年龄：{(state.user as any).age}</li>
             <li>身高：{(state.user as any).height}</li>
             <li>体重：{(state.user as any).weight}</li>
           </ul>
         )}
         <p>我是父组件</p>
         <Child />
       </div>
     );
   };

   export default Parent;
   ```

   效果：

   ![](/madao.github.io/database/images/articles/react/reactHooks/image1.png)

### 三. useContext

useContext 在 useReducer 中已经用过了，就简单说下如何使用把：

1. 使用 React.createContext 创建 Context 对象
2. 使用 Context.Provider 圈定可以使用 Context 对象的值的范围，只有被 Context.Provider 包裹的组件才能使用对应的 Context
3. 在组件中将 Context 对象传入 useContext 用于获取 Context 对象的值，每当 Context 对象更新的时候，就会触发重新渲染

### 四. useEffect

> useEffect 接收一个包含命令式、且可能有副作用代码的函数。

```
let a = 1
const fn = (n) => {
  a += 1
  document.title = 'xxx'
  return n + 1
}
```

例子中的函数修改了外部变量，而且修改了 dom，这就是具有副作用代码的函数，因为在改变当前执行环境中的任何东西都有可能造成一些预期以外的影响。

useEffect 是一个很有用的 hook，它可以：

1. 作为 componentDidMount 使用，例子：
   ```
   useEffect(() => {
     console.log('我只在组件第一次渲染完成后执行')
   }, [])
   ```
2. 作为 componentDidUpdate 使用，例子：
   ```
   useEffect(() => {
     console.log('我会在组件每次重新渲染后执行')
   })
   const [n, setN] = useState(0)
   useEffect(() => {
     console.log('只有n的值发生变化，我才会执行')
     console.log('初次渲染也会执行')
   }, [n])
   ```
3. 作为 componentWillUnmount 使用
   ```
   useEffect(() => {
     const timer = setInterval(() => {
       console.log(1)
     }, 1000)
     return () => {
       clearInterval(timer)
       console.log('只有组件卸载时我才会执行')
     }
   }, [])
   ```

可以同时存在多个 useEffect，它们会按照书写的顺序依次执行。

### 五. useLayoutEffect

useLayoutEffect 用法和 useEffect 相同，它们之间的区别是：
useLayoutEffect 会在浏览器渲染之前执行，useEffect 会在浏览器渲染之后执行，useLayoutEffect 会阻塞浏览器的渲染，导致页面呈现给用户的时间变长，所以推荐使用 useEffect，如果你的具有副作用的函数操作了 DOM，那么建议放在 useLayoutEffect 中，不过一般 DOM 的操作都交给 React 去做。

### 六. useMemo

React 默认存在多余的渲染，比如：

```
import React, { useState } from 'react'
import ReactDom from 'react-dom'

const Child = (props: { m: number; }) => {
  console.log('Child 被执行了')
  return (
    <div>m: {props.m}</div>
  )
}
const App = () => {
  const [n, setN] = useState(0)
  const [m, setM] = useState(0)
  console.log('App 被执行了')
  return (
    <div>
      <div>n: {n}</div>
      <Child m={m}/>
      <button onClick={() => setN(n => n + 1)}>n + 1</button>
      <button onClick={() => setM(m => m + 1)}>m + 1</button>
    </div>
  )
}
ReactDom.render(<App />, document.querySelector('#app'))
```

结果：
![](/madao.github.io/database/images/articles/react/reactHooks/image2.png)

组件 Child 只依赖 m，但是 n 更新的时候，它也重新执行了，使用 React.memo 就可以避免这种情况，现在将 Child 用 React.memo 包起来

```
const Child = React.memo((props: { m: number; }) => {
  console.log('Child 被执行了')
  return (
    <div>m: {props.m}</div>
  )
})
```

结果：
![](/madao.github.io/database/images/articles/react/reactHooks/image3.png)

但是 React.memo 有一个问题，当如果 Child 组件的 props 中有一个函数，那么 React.memo 就失效了：

```
const Child = React.memo((props: {
  m: number;
  sayM: () => void;
 }) => {
  console.log('Child 被执行了')
  return (
    <div onClick={() => props.sayM()}>m: {props.m}</div>
  )
})
const App = () => {
  const [n, setN] = useState(0)
  const [m, setM] = useState(0)
  const sayM = () => console.log(m)
  console.log('App 被执行了')
  return (
    <div>
      <div>n: {n}</div>
      <Child m={m} sayM={sayM} />
      <button onClick={() => setN(n => n + 1)}>n + 1</button>
      <button onClick={() => setM(m => m + 1)}>m + 1</button>
    </div>
  )
}
```

![](/madao.github.io/database/images/articles/react/reactHooks/image4.png)

原因是，App 重新执行的时候，`const sayM = () => console.log(m)`这句代码也会重新执行，那么会生成一个新的 sayM，新的 sayM 和旧 sayM 功能一样，但是引用却不一样，所以 React 会认为 Child 组件的 props 变了，所以 Child 也重新执行了。

那么为了解决 props 中有函数导致的 React.memo 失效问题，useMemo 就出现了，它的用法是这样的：

```
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

和 useEffect 有点像，只有 useMemo 第二个参数的数组中的值发生变化，才会执行 useMemo 第一个参数中的函数，那么例子中的 sayM 就可以写成这样：

```
const sayM = useMemo(() => {
  return () => console.log(m)
}, [m])
```

这样每次渲染只要 m 的值不变，useMemo 就不会重新计算，sayM 就不会变，Child 也就不存在多余的渲染了

### 七. useCallback

useMemo 中的例子是这样的：

```
const sayM = useMemo(() => {
  return () => console.log(m)
}, [m])
```

它可以使用 useCallback 进行简化：

```
const sayM = useCallback(() => console.log(m), [m])
```

`useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`

### 八. useRef

> useRef 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

上面解释中的保持不变指的这个对象的引用保持不变（内存地址），也就是从组件到销毁，该 ref 对象都是同一个。

注意的一点就是，如果你修改了 ref 对象的`.current` 属性值，组件是不会触发重新渲染的

#### React.forwardRef

这里简单说一下`React.forwardRef`, 在 React 中 ref 对象无法作为 props 进行传递，例子：

```
const Child = (props) => {
  return (
    <div>Child</div>
  )
}
const App = () => {
  const ref = useRef(null)
  return (
    <div>
      <Child ref={ref}>
    </div>
  )
}
```

这样进行传递会得到一个报错：
`Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()`

只需要将 Child 这样修改：

```
const Child = React.forwardRef((props, ref) => {
  return (
    <div ref={ref}>Child</div>
  )
})
```

这样就不会报错了，为什么需要传递 ref，因为这样父组件就可以通过 ref 获取到子组件的 DOM 对象的引用了

### 九. useImperativeHandle

这个 hook 还得搭配 useRef 和 React.forwardRef 来说，还是上面的例子：

```
const Child = React.forwardRef((props, ref) => {
  return (
    <div ref={ref}>Child</div>
  )
})

const App = () => {
  const ref = useRef(null)
  useEffect(() => {
    console.log(ref)
  }, [])
  return (
    <div>
      <Child ref={ref}>
    </div>
  )
}
```

![](/madao.github.io/database/images/articles/react/reactHooks/image5.png)

这样得到的 ref 对象的 current 属性就是 Child 组件的 DOM 对象的引用，假如不想返回 Child 组件的 DOM 对象的引用，那么就可以使用 useImperativeHandle，很奇怪吧，不知道这个 hook 为什么叫这个名字：

```
const Child = React.forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    hello: () => console.log('hello')
  }))
  return (
    <div ref={ref}>Child</div>
  )
})

const App = () => {
  const ref = useRef(null)
  useEffect(() => {
    console.log(ref)
  }, [])
  return (
    <div>
      <Child ref={ref}>
    </div>
  )
}
```

![](/madao.github.io/database/images/articles/react/reactHooks/image6.png)

### 十. 自定义 hook

自定义 hook，将为了完成某个需求所需要的 hook 封装起来，看一个例子就明白了：

myHook.js

```
const myHook = () => {
  const [user, setUser] = useState(null)
  useEffect(() => {
    getUser().then(res => setUser(res))
  }, [])
  return { user, setUser }
}
const getUser = () => {
  return new Promise(resolve => {
    setTimeout(() => resolve({
      name: 'allen',
      age: 18
    }), 1000)
  })
}

export default myHook
```

App.jsx

```
import myHook from './myHook'

const App = () => {
  const { user, setUser } = myHook()
  return (
    <div>
      {user && (
        <div>{user.name}</div>
        <div>{user.age}</div>
      )}
    </div>
  )
}
```

例子中将获取用户的逻辑封装起来，这样 App 组件的可读性就变得非常高，而且 myHook 的作用也很单一就是获取用户信息，这样复用性也非常高。

### 十一. stale closures

过时的闭包

[Be Aware of Stale Closures when Using React Hooks](https://dmitripavlutin.com/react-hooks-stale-closures/)
