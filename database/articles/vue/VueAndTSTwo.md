这次记录一下 axios 在 vue + typescript 项目中的遇到的问题：

一般项目都会自己封装请求方法，便于重试，请求错误处理，验证等等，因为我写的是个人项目所以封装的不是很全面，就不贴代码了，主要是在封装好之后，想要绑定到 vue 的原型上的时候遇到了一些问题。

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image.png)

首先封装好之后，在 index.ts 文件中导出：

```
export default {
  install: function (Vue: any, Option: any) {
    Object.defineProperty(Vue.prototype, '$ajax', {value: Axios})
  }
}
```

然后在 mian.ts 中导入

```
import Axios from "./utils/request/"

Vue.use(Axios)
```

这样就可以在组件中使用`this.$ajax`来发出一个请求。

完成上面的步骤回到组件中使用的时候碰到了一个红线提示：

`Property '$ajax' does not exist on type 'Components'.`

提示中的 Components 就是组件的名字。

在网上搜索了一下，解决方案是在`shims-vue.d.ts`中添加以下的代码：

```
import Vue from 'vue';
import { AxiosInstance } from 'axios';
declare module 'vue/types/vue' {
  interface Vue {
    $ajax: AxiosInstance
  }
}
```

可是添加了之后还是有问题，然后我再网上看了一个大神的回答，居然是要添加了之后**重启 vscode**。重启之后果然是 ok 了。但是又出现一个问题，那就是在 main.ts 文件中

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image1.png)

又出现了这样的红线提示。

好吧，查看官方文档发现这样一句话：

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image2.png)

难道是要自己新建一个声明文件吗？

尝试自己新建一个声明文件：

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image3.png)

然后添加下面的代码，并把 shims-vue.d.ts 中添加的代码删除：

```
import Vue from 'vue';
import { AxiosInstance } from 'axios';
declare module 'vue/types/vue' {
  interface Vue {
    $ajax: AxiosInstance
  }
}
```

然后重启 vscode。世界和平了，所有的红线都没了。

补充：

在封装 axios 的时候，假如用了自定义的 axios 实例，给它配置一些官方没有的配置，比如重试次数，重试时间间隔等等，也是会有红线提示的。

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image4.png)

这个就和上面的问题一样，那么照猫画虎，先去看 axios 中 config 的声明文件在哪里：

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image5.png)

在这个文件中找到了 AxiosRequestConfig：

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image6.png)

可以看到所有的配置项都在里面了。按照之前的写法，在 shims-requset-property.d.ts 文件中加入下面的代码：

```
import { AxiosInstance, AxiosRequestConfig } from 'axios';

declare module 'axios/' {
  interface AxiosRequestConfig {
    retry?: number;
    retryDelay?: number;
    __retryCount?: number;
  }
}
```

只要有新的配置都可以在这里面声明。

最后，再记一个 rap2 使用过程的问题，就是 mock.js 中的方法是要在初始值这一栏使用`@方法名`使用，我一直在规则那一栏使用，搞了半天才发现。rap2 的文档页太不友好了。

![](/caisr.github.io/database/images/articles/vue/vue_and_ts_two/image7.png)
