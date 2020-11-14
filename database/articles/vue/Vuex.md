很早就听说过 vuex，但是一直没用过。正好做到了登录这一块，趁着这个机会掌握它。

### 一. 对 Vuex 的理解

> Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。 ---引用自 Vuex 官网

我个人的理解是这样的，就拿登录来说，登录是一种状态，在网站中有可能登录后的好多页面和未登录的状态都是不同的，把每个页面想象成组件，也就是说多个组件都要共享这个登录状态，如果是采用传参的方式来共享，会变得很繁琐，就像是多个函数嵌套一样，要一层一层的将参数传递下去，而且不易维护，这时候就需要将这种多个组件共享的状态或是数据抽取出来，单独存放在一个地方，只要需要这些状态或者数据的组件，都去这个地方找就好了，那么实现这个功能的东西就叫做 Vuex。

### 二. Vuex 中的核心概念

- #### state

  state 就是数据源，也就是说所有共享的数据都存放在这个地方，例子：

  使用 vue-cli 搭建的项目，会有一个 store.js 的文件

  ![](/caisr.github.io/database/images/articles/vue/vuex/image.png)

  ```
  import Vue from "vue";
  import Vuex from "vuex";

  Vue.use(Vuex);

  export default new Vuex.Store({
      state: {
          isLogin: false  // state里面就可以放共享的数据
      },
      mutations: {},
      actions: {}
  });

  ```

  其他组件如果要获取 state 里面的数据，可以通过 this.\$store.state.xxx 获得

  ```
  export default {
      name: "HelloWorld",
      props: {
          msg: String
      },
      mounted() {
          console.log(this.$store.state.isLogin)
      },
  };
  ```

  - #### mapState

    假如一个组件需要获取多个共享数据，例子：

    ```
    // store.js

    export default new Vuex.Store({
        state: {
            isLogin: false,
            name: "admin",
            password: 123456
        },
        mutations: {},
        actions: {}
    });
    ```

    ```
    // HelloWorld.vue

    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        computed: {
            isLogin() {
                return this.$store.state.isLogin
            },
            name() {
                return this.$store.state.name
            },
            passsword() {
                return this.$store.state.password
            }
        },
    };
    ```

    例子中是一个一个的写出来，3 个还好。假如有 10 个写起来就很麻烦了。

    mapState 可以解决这个问题：

    上面的例子用 mapState 写就是这样的：

    ```
    // HelloWorld.vue
    import { mapState } from 'vuex';
    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        computed: mapState(['isLogin', 'name', 'password'])
    };
    ```

    mapState 的参数如果是数组的话，那数组的每一项就是 state 中数据的 key 值，按照官网的说法是：映射 this.isLogin 为 store.state.isLogin。

    但是 computed 不可能只有 state 里的属性，还有组件自身的属性，这时候就要使用对象展开运算符了：

    ```
    // HelloWorld.vue
    import { mapState } from 'vuex';
    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        computed: {
            greetings() {
                return `${this.name}！你好`
            },
            ...mapState(['isLogin', 'name', 'password'])
        }
    };
    ```

    结果：

    ![](/caisr.github.io/database/images/articles/vue/vuex/image1.png)

    mapState 还可以传入一个对象，比如上面例子可以改写成这样

    ```
    // HelloWorld.vue
    import { mapState } from 'vuex';
    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        computed: {
            ...mapState({
                isLogin: state => state.isLogin,
                name: state => state.isLogin,
                password: 'password',
                greetings(state) {
                    return `${state.name}！你好`
                }
            })
        }
    };
    ```

    结果：

    ![](/caisr.github.io/database/images/articles/vue/vuex/image2.png)

    这两种写法都可获得 state 中对应 key 值得数据

    ```
    isLogin: state => state.isLogin
    password: 'password'
    ```

    依赖 state 中的计算属性可以使用例子中函数的方式，

    ```
    greetings(state) {
        return `${state.name}！你好`
    }
    ```

    注意不要使用箭头函数，为了保证 this 指向正确（没看过源码，不知道调用方式，所以不能确定 this 的指向，但是官方文档说不要使用箭头函数，按照文档的来）

  - #### Getter

    官方文档上有一句话我觉得已经解释的很清楚了：

    > 可以认为是 store 的计算属性 ---引用自 Vuex 官网

    直接看例子：

    ```
    // store.js
    export default new Vuex.Store({
        state: {
            todeList: [
                { task: "追番", isDone: false },
                { task: "玩游戏", isDone: false },
                { task: "睡觉", isDone: true }
            ]
        },
        mutations: {},
        actions: {}
    });
    ```

    现在需要获得 todeList 中已完成的事项数量。如果只有一个组件需要，那么直接在组件内部 filter 就好了，如果是多个组件都要，那又出现了重复代码的情况，这种情况下可以使用 getters 解决：

    ```
    // store.js
    export default new Vuex.Store({
        state: {
            todeList: [
                { task: "追番", isDone: false },
                { task: "玩游戏", isDone: false },
                { task: "睡觉", isDone: true }
            ]
        },
        getters: {
            doneList: state => {
                return state.todeList.filter(task => task.isDone);
            }
        }
        mutations: {},
        actions: {}
    });
    ```

    ```
    // HelloWorld.vue

    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        computed: {
            doneAmount() {
                return this.$store.getters.doneList.length;
            }
        }
    };
    ```

    每当 state 中的数据变化，getter 中依赖它的数据就会重新计算，在各个组件中也就不用重复的去写那些方法了，直接获取即可。

    getters 中的 getter 还有几种其他的写法：

    - getter 可以引用其他 getter。上面的例子可以改成这样：

      ```
      // store.js
      export default new Vuex.Store({
          state: {
              todeList: [
                  { task: "追番", isDone: false },
                  { task: "玩游戏", isDone: false },
                  { task: "睡觉", isDone: true }
              ]
          },
          getters: {
              doneList: state => {
                  return state.todeList.filter(task => task.isDone);
              },
              doneAmount: (state, getters) => {
                  return getters.doneList.length;
              }
          },
          mutations: {},
          actions: {}
      });
      ```

      ```
      // HelloWorld.vue
      export default {
          name: "HelloWorld",
          props: {
              msg: String
          },
          computed: {
              doneAmount() {
                  return this.$store.getters.doneAmount;
              }
          }
      };
      ```

      这样改完之后组件里直接用 doneAmount 就行了，getters 中的 doneAmount 会根据 doneList 的改变而重新计算出数量。

    - getter 可以返回一个函数，使用的时候就可以给 getter 传参数了

      ```
      // store.js
      export default new Vuex.Store({
          state: {
              todeList: [
                  { task: "追番", isDone: false },
                  { task: "玩游戏", isDone: false },
                  { task: "睡觉", isDone: true }
              ]
          },
          getters: {
              getTask: state => taskName => {
                  return state.todeList.find(taskItem => taskName === taskItem.task);
              }
          },
          mutations: {},
          actions: {}
      });
      ```

      ```
      // HelloWorld.vue
      export default {
          name: "HelloWorld",
          props: {
              msg: String
          },
          computed: {
              task() {
                  return this.$store.getters.getTask('玩游戏')
              }
          }
      };
      ```

      ![](/caisr.github.io/database/images/articles/vue/vuex/image3.png)

    同样，getter 也有辅助的函数 mapGetters，它和 mapState 差不多，这里就不继续说了。

  - #### Mutation

    想要改变 state 中的数据，在 vuex 中唯一的办法就是提交一个 mutation，直接看看它的使用方式吧，例子:

    - 不带参数

      ```
      // store.js
      export default new Vuex.Store({
          state: {
              num: 0
          },
          getters: {},
          mutations: {
              addOne(state) {
                  state.num++
              }
          },
          actions: {}
      });
      ```

      ```
      // HelloWorld.vue
      export default {
          name: "HelloWorld",
          props: {
              msg: String
          },
          mounted() {
              this.$store.commit('addOne')
              console.log(this.num)
          },
          computed: {
              num() {
                  return this.$store.state.num
              }
          }
      };
      ```

    - 带参数
      ```
      // store.js
      export default new Vuex.Store({
          state: {
              num: 0
          },
          getters: {},
          mutations: {
              add(state, n) {
                  state.num += n
              },
              multiplication(state, payload) {
                  state.num *= payload.num;
              }
          },
          actions: {}
      });
      ```
      ```
      // HelloWorld.vue
      export default {
          name: "HelloWorld",
          props: {
              msg: String
          },
          mounted() {
              this.$store.commit('add', 10)
              this.$store.commit('multiplication', {
                  num: 10
              })
              this.$store.commit({
                  type: 'multiplication',
                  num: 10
              })
              console.log(this.num)
          },
          computed: {
              num() {
                  return this.$store.state.num
              }
          }
      };
      ```
      带参数方式中如果参数是对象，有两种写法，一种是把 mutation 的名字作为第一个参数，第二种是第一个参数是对象，对象中有个 type 属性，type 属性的值就是 mutation 的名字。

    mutation 也有辅助函数 mapMutations，上面例子可以写成下面这样：

    ```
    // HelloWorld.vue
    import { mapMutations } from "vuex";
    export default {
        name: "HelloWorld",
        props: {
            msg: String
        },
        mounted() {
            this.add(10)
            this.multiplication({
                num: 100
            })
        },
        methods: {
            ...mapMutations([
                'add',
                'multiplication'
            ])
        },
        computed: {
            num() {
                return this.$store.state.num
            }
        }
    };
    ```

    mutation 还需要注意的两点有：

    - mutation 事件名建议使用常量代替（官方推荐）：

      ```
      // 常量也建议全部存放在单独的文件中
      const MUTATION_ADD = 'MUTATION_ADD'


      mutations: {
          [MUTATION_ADD](state, n) {
              state.num += n
          }
      }
      ```

    - mutation 必须是同步函数，原因参考[官网](https://vuex.vuejs.org/zh/guide/mutations.html)。

  - #### Action

    由于 mutation 只能是同步函数，如果有异步的操作，那么就可以用 action。例子：

    ```
    export default new Vuex.Store({
        state: {
            num: 1
        },
        getters: {},
        mutations: {
            add(state, payload) {
                state.num += payload.num;
            }
        },
        actions: {
            add(context, payload) {
                setTimeout(() => {
                    context.commit("add", payload);
                }, 1000);
            }
        }
    });
    ```

    action 就好像吧 mutation 包了一层一样，在 action 中就可以使用各种异步了。
    组件中使用 action 用的是这样的方式:

    ```
    this.$store.dispatch('xxx')
    this.$store.dispatch('xxx', {key: value})
    this.$store.dispatch({
        type: 'xxx',
        key: value
    })
    ```

    action 还可以这样用：

    ```
    actions: {
        actionOne() {
            return new Promise(reslove => {
                console.log("呆死ki");
                reslove();
            });
        },
        add(context, payload) {
            context.dispatch("actionOne").then(() => {
                setTimeout(() => {
                    context.commit("add", payload);
                }, 1000);
            });
        }
    }
    ```

    它的辅助函数叫 mapActions，用法和 mapMutations 类似。

  最后剩 Module，目前还没用过，所以理解可能会有问题。这里就不说了。

  总结下就是：

  vuex 中：

  - store：作为一个容器，里面包含了 state，mutation 等等，就像一个 vue 的实例一样。
  - state：store 中的数据源。
  - getter：store 中的计算属性
  - mutation：修改 state 的唯一方法就是提交一个 mutation，mutation 只能是同步函数
  - action：由于 mutation 只能是同步函数，那么有异步的操作就要用到 action，action 就好像将 mutation 包裹了一层一样。
  - module: 如果 store 特别的庞大，可以使用 module 将它分割成模块。
