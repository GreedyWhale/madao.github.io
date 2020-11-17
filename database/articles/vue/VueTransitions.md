虽然经常用 Vue 做提供的 transition 做动画效果，但是每次都要去官方文档看好几遍才能写，所以总结下加深理解：

### 过渡的类名

Vue 提供了以下类名用于实现动画：

- v-enter
- v-enter-active
- v-enter-to
- v-leave
- v-leave-active
- v-leave-to

之前我对这些类名添加在元素身上的时机，理解是有误的导致写动画效果的时候，基本靠猜，下面说下我自己的理解:

一个完整的过渡效果流程：

1.  元素未插入之前：

    - 元素未插入之前在 v-enter 和 v-enter-active 这两个类名就会添加到元素上。

2.  元素插入后的下一帧：

    - 元素插入后的下一帧，v-enter 这个类名就会被移除同时 v-enter-to 会被添加到元素上，如果 v-enter 中写了样式，如果过渡的时间足够长，你就会看到元素是以 v-enter 中定义的样式开始过渡的，可以理解 v-enter 是用于定义元素过渡开始的初始样式。

3.  元素过渡效果结束

    - 元素过渡效果结束，v-enter-active 和 v-enter-to 被移除，因为 v-enter-active 一直从元素的过渡开始直到过渡结束才会被移除，所以 v-enter-active 中常常用来定义元素的过渡效果。v-enter-active 和 v-enter-to 不同的是 v-enter-to 是元素被插入后的下一帧才被添加，v-enter-active 在元素插入前就存在了。（我试着将过渡时间写在 v-enter-to 中，效果看起来没什么区别）

    - 元素过渡效果结束之后，由于 v-enter-to 被移除了，所以元素会变为自身的样式，这也是我老是理解错的一点，我之前的理解是这样：

      `v-enter -> v-enter-active -> v-enter-to（这里定义了元素过渡结束后的样式）`

      这个理解是错的，其实是这样：

      `v-enter, v-enter-active -> v-enter-active, v-enter-to -> 元素本身的样式`

      如果定义了 v-enter-to，v-enter-to 中的样式和元素本身的样式不一样，而且元素本身样式中没有过渡效果相关的 css，在过渡结束后元素的样式会突然从 v-enter-to 中的样式变成元素本身的样式，也可以将 v-enter-to 中的样式写的和元素本身一样，那么过渡结束后的变化就不会很突兀。一般的过渡都是从某个样式过渡到元素本身的样式，所以，v-enter-to 一般是不写的。

4.  元素被移除的过渡开始：

    - 元素被移除的过渡开始会添加 v-leave，v-leave-active， v-leave-to 类名，过渡开始的下一帧会 v-leave 被移除，按照我对 v-enter 的立即，v-leave 应该也是元素移除过渡开始的初始样式，但是可能移除过渡开始的速度太快了，我自己测试的时候并没有看到定义在 v-leave 中的样式生效。v-leave-active 和 v-leave-to 都会在过渡效果完成之后移除。离开的过渡流程大概是这样：

      `v-leave, v-leave-active, v-leave-to -> v-leave-active, v-leave-to -> 元素被移除`

综合上面得出的结论：

1. v-enter, v-leave-to 中的 css 一般相同，一个是进入时过渡（动画）的初始样式，一个是离开过渡（动画）结束时的样式。
2. v-enter-active ，v-leave-active 中的 css 一般相同，一般都是用于定义过渡（动画）的过程时间，延迟和曲线函数。当然离开的过渡（动画）的过程时间，延迟和曲线函数和进入的可以是不同的。
3. v-enter-to，v-leave 一般不写，不写的原因是：按照一般的过渡效果（动画），进入的最后状态就是元素本身的样式，离开的最初状态也是元素本身的样式，所以是没有必要写的。（v-leave 我自己是没有测试出效果。。。）
