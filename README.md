# <a href='http://redux.js.org'><img src='https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67' height='60'></a>

Redux 是个给 JavaScript 应用程式所使用的可预测 state 容器（如果你正在寻找一个 WordPress 框架，请查看 [Redux Framework](https://reduxframework.com/)）。

他帮助你撰写行为一致的应用程式，可以在不同的环境下执行 (客户端、伺服器、原生应用程式)，并且易于测试。在这之上，它提供一个很棒的开发体验，例如[把程式码即时编辑与时间旅行除错器结合](https://github.com/gaearon/redux-devtools)。

你可以使用 Redux 结合 [React](https://facebook.github.io/react/)，或结合其他任何的 view library。
它非常小 (2kB ，包含依赖套件)。

[![build status](https://img.shields.io/travis/reactjs/redux/master.svg?style=flat-square)](https://travis-ci.org/reactjs/redux)
[![npm version](https://img.shields.io/npm/v/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![npm downloads](https://img.shields.io/npm/dm/redux.svg?style=flat-square)](https://www.npmjs.com/package/redux)
[![redux channel on discord](https://img.shields.io/badge/discord-%23redux%20%40%20reactiflux-61dafb.svg?style=flat-square)](https://discord.gg/0ZcbPKXt5bZ6au5t)
[![#rackt on freenode](https://img.shields.io/badge/irc-%23rackt%20%40%20freenode-61DAFB.svg?style=flat-square)](https://webchat.freenode.net/)
[![Changelog #187](https://img.shields.io/badge/changelog-%23187-lightgrey.svg?style=flat-square)](https://changelog.com/187)

>**新的东西！从 Redux 的作者学习它：
>[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) (三十部免费影片)**

### 推荐

>[「我爱那些你在 Redux 做的东西」](https://twitter.com/jingc/status/616608251463909376)
>Jing Chen，Flux 作者

>[「我在 FB 的内部 JS 讨论群组寻求对 Redux 的评论，并获得了普遍的好评。真的做得非常棒。」](https://twitter.com/fisherwebdev/status/616286955693682688)
>Bill Fisher，Flux 文件的作者

>[「这很酷，你借由完全不做 Flux 来发明了一个更好的 Flux。」](https://twitter.com/andrestaltz/status/616271392930201604)
>André Staltz，Cycle 作者

### 开发经验

我在准备我的 React Europe 演讲 [「Hot Reloading 与时间旅行」](https://www.youtube.com/watch?v=xsSnOQynTHs) 的时候撰写了 Redux。我那时的目标是建立一个 state 管理 library，它只有最少的 API，但却拥有完全可预测的行为，所以它可以实现 logging、hot reloading、时间旅行、universal 应用程式、记录和重播，而不需要开发者任何其他的代价。

### 受到的影响

Redux 从 [Flux](http://facebook.github.io/flux/) 的概念发展而来，不过借由从 [Elm](https://github.com/evancz/elm-architecture-tutorial/) 获取线索来避免它的复杂度。
不管你以前有没有用过它们，只需要花几分钟就能入门 Redux。

### 安装

安装稳定版本：

```
npm install --save redux
```

这里假设你是使用 [npm](https://www.npmjs.com/) 作为你的套件管理器。
若不是的话，你可以[在 npmcdn 取得这些档案](https://npmcdn.com/redux/)并下载它们，或是将套件管理器指向它们。

最常见的是人们将 Redux 作为 [CommonJS](http://webpack.github.io/docs/commonjs.html) 模组中的一个 collection 使用。当你在 [Webpack](http://webpack.github.io)、[Browserify](http://browserify.org/) 或 Node 环境中 import `redux` 时就能取得此模组。若你愿意冒风险使用 [Rollup](http://rollupjs.org)，我们也同样支援它。

如果你不想使用模组 bundler 也没关系。`redux` npm 套件的 [`dist` 资料夹](https://npmcdn.com/redux/dist/)包含了已编译之 production 与 development 的 [UMD](https://github.com/umdjs/umd) build。你可以不透过 bundler 直接使用它们，也因此它们与许多热门的 JavaScript 模组 loader 及环境相容。举个例子，你可以将一个 UMD build 作为 [`<script>` 标签](https://npmcdn.com/redux/dist/redux.js)放入网页中，或[透过 Bower 进行安装](https://github.com/reactjs/redux/pull/1181#issuecomment-167361975)。UMD build 让 Redux 能够作为 `window.Redux` 全域变数进行使用。

Redux 的原始码由 ES2015 撰写而成，但是我们预先编译了 CommonJS 及 UMD build 两种 ES5 版本，让它们可以运作于[任何现代的浏览器](http://caniuse.com/#feat=es5)。你不必使用 Babel 或模组 bundler 即可[开始使用 Redux](https://github.com/reactjs/redux/blob/master/examples/counter-vanilla/index.html)。

#### 补充性套件

大多数情况，你也会需要 [React 的绑定](https://github.com/reactjs/react-redux)和[开发者工具](https://github.com/gaearon/redux-devtools)。

```
npm install --save react-redux
npm install --save-dev redux-devtools
```

请注意，这些套件不同于 Redux 自身，许多 Redux 生态系中的套件并不提供 UMD build，所以我们建议使用像是 [Webpack](http://webpack.github.io) 或 [Browserify](http://browserify.org/) 的 CommonJS 模组 bundler，以取得最舒适的开发体验。

### 程式码片段

你的应用程式的完整 state 被以一个 object tree 的形式储存在单一一个的 *store* 里面。
改变 state tree 的唯一方式是去发送一个 *action*，action 是一个描述发生什么事的物件。
要指定 actions 要如何转换 state tree 的话，你必须撰写 pure *reducers*。

就这样！

```js
import { createStore } from 'redux'

/**
 * 这是一个 reducer，一个有 (state, action) => state signature 的 pure function。
 * 它描述一个 action 如何把 state 转换成下一个 state。
 *
 * state 的形状取决于你：它可以是基本类型、一个阵列、一个物件，
 * 或甚至是一个 Immutable.js 资料结构。唯一重要的部分是你
 * 不应该改变 state 物件，而是当 state 变化时回传一个新的物件。
 *
 * 在这个范例中，我们使用一个 `switch` 陈述句和字串，不过你可以使用一个 helper，
 * 来遵照一个不同的惯例 (例如 function maps)，如果它对你的专案有意义。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// 建立一个 Redux store 来掌管你的应用程式的 state。
// 它的 API 是 { subscribe, dispatch, getState }。
let store = createStore(counter)

// 你可以手动的去订阅更新，或是使用跟你的 view layer 之间的绑定。
// 通常你会使用一个 view 绑定 library（例如：React Redux），而不是直接 subscribe()。
// 然而也可以很方便的将目前状态储存在 localStorage。
store.subscribe(() =>
  console.log(store.getState())
)

// 变更内部 state 的唯一方法是 dispatch 一个 action。
// actions 可以被 serialized、logged 或是储存并在之后重播。
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

你必须指定你想要随著被称作 *actions* 的一般物件而发生的变更，而不是直接改变 state。接著你会写一个被称作 *reducer* 的特别 function，来决定每个 action 如何转变整个应用程式的 state。

如果你以前使用 Flux，那你需要了解一个重要的差异。Redux 没有 Dispatcher 也不支援多个 stores。反而是只有一个唯一的 store 和一个唯一的 root reducing function。当你的应用程式变大时，你会把 root reducer 拆分成比较小的独立 reducers 来在 state tree 的不同部分上操作，而不是添加 stores。 这就像在 React 应用程式中只有一个 root component，但是他是由许多小的 components 组合而成。

这个架构用于一个计数器应用程式可能看似有点矫枉过正，不过这个模式的美妙之处就在于它如何扩展到大型且模杂的应用程式。它也启用了非常强大的开发工具，因为它可以追踪每一次的变更和造成变更的 action。你可以记录使用者的 sessions 并借由重播每个 action 来重现它们。

### 从 Redux 的作者学习它

[Getting Started with Redux](https://egghead.io/series/getting-started-with-redux) 是一个由 30 部 Dan Abramov 讲述的影片组成的影片课程，他是 Redux 的作者。它被设计用来补充文件的「基础」部分，而带来有关 immutability、测试、Redux 最佳实践、与搭配 React 使用 Redux 的额外洞悉。**这个课程是免费的，而且是永远的。**

>[「在 egghead.io 上面由 @dan_abramov 出品的伟大课程 - 不只是展示了如何使用 #redux 给你看，它也展示了如何以及为什么打造 redux！」](https://twitter.com/sandrinodm/status/670548531422326785)
>Sandrino Di Mattia

>[「钻研过 @dan_abramov 'Getting Started with Redux' - 它借由影片不知道简化了多少观念非常惊人。」](https://twitter.com/chrisdhanaraj/status/670328025553219584)
>Chris Dhanaraj

>[「@eggheadio 上面的这系列 @dan_abramov 出品的 Redux 影片非常的惊人！」](https://twitter.com/eddiezane/status/670333133242408960)
>Eddie Zaneski

>[「从名字炒作而来。而留下了坚若磐石的基础。(感谢，@dan_abramov 和 @eggheadio 做的伟大的事！)」](https://twitter.com/danott/status/669909126554607617)
>Dan

>[「这系列 @dan_abramov 出品的 Redux 影片反复的让我留下深刻印象 - 打算做一些认真的重构」](https://twitter.com/gelatindesign/status/669658358643892224)
>Laurence Roberts

所以，你还在等什么？

#### [观看 30 部免费影片！](https://egghead.io/series/getting-started-with-redux)

如果你喜欢我的课程，请考虑借由[购买订阅](https://egghead.io/pricing)来支持 Egghead。订阅者可以存取我的每一个影片中的范例的原始码，以及无数的其他主题的进阶课程，包括深入 JavaScript、React、Angular、和更多其他的。许多的 [Egghead 讲师](https://egghead.io/instructors) 也是开源 library 的作者，所以购买订阅是一个感谢他们目前所做的事的好方式。

### 文件

* [介绍](http://redux.js.org/docs/introduction/index.html)
* [基础](http://redux.js.org/docs/basics/index.html)
* [进阶](http://redux.js.org/docs/advanced/index.html)
* [Recipes](http://redux.js.org/docs/recipes/index.html)
* [疑难排解](http://redux.js.org/docs/Troubleshooting.html)
* [术语表](http://redux.js.org/docs/Glossary.html)
* [API 参考](http://redux.js.org/docs/api/index.html)

想要输出成 PDF、ePub 和 MOBI 以方便离线阅读的话，关于如何产生它们的说明，请参阅：[paulkogel/redux-offline-docs](https://github.com/paulkogel/redux-offline-docs)。

### 范例

* [Counter Vanilla](http://redux.js.org/docs/introduction/Examples.html#counter-vanilla) ([原始码](https://github.com/reactjs/redux/tree/master/examples/counter-vanilla))
* [Counter](http://redux.js.org/docs/introduction/Examples.html#counter) ([原始码](https://github.com/reactjs/redux/tree/master/examples/counter))
* [Todos](http://redux.js.org/docs/introduction/Examples.html#todos) ([原始码](https://github.com/reactjs/redux/tree/master/examples/todos))
* [Todos with Undo](http://redux.js.org/docs/introduction/Examples.html#todos-with-undo) ([原始码](https://github.com/reactjs/redux/tree/master/examples/todos-with-undo))
* [TodoMVC](http://redux.js.org/docs/introduction/Examples.html#todomvc) ([原始码](https://github.com/reactjs/redux/tree/master/examples/todomvc))
* [Shopping Cart](http://redux.js.org/docs/introduction/Examples.html#shopping-cart) ([原始码](https://github.com/reactjs/redux/tree/master/examples/shopping-cart))
* [Tree View](http://redux.js.org/docs/introduction/Examples.html#tree-view) ([原始码](https://github.com/reactjs/redux/tree/master/examples/tree-view))
* [Async](http://redux.js.org/docs/introduction/Examples.html#async) ([原始码](https://github.com/reactjs/redux/tree/master/examples/async))
* [Universal](http://redux.js.org/docs/introduction/Examples.html#universal) ([原始码](https://github.com/reactjs/redux/tree/master/examples/universal))
* [Real World](http://redux.js.org/docs/introduction/Examples.html#real-world) ([原始码](https://github.com/reactjs/redux/tree/master/examples/real-world))

如果你不熟悉 NPM 生态系并在让专案运作起来时遇到了困难，或是你不确定要在哪里贴上上面的程式码片段，请查看 [simplest-redux-example](https://github.com/jackielii/simplest-redux-example)，它把 Redux 和 React、Browserify 结合在一起。

### 讨论

加入 [Reactiflux](http://www.reactiflux.com) Discord 社群的 [#redux](https://discord.gg/0ZcbPKXt5bZ6au5t) 频道。

### 致谢

* [Elm 架构](https://github.com/evancz/elm-architecture-tutorial) 关于如何用 reducers 来更新 state 的伟大介绍；
* [Turning the database inside-out](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) 启发我的心；
* [Developing ClojureScript with Figwheel](http://www.youtube.com/watch?v=j-kj2qwJa_E) 说服我，让我重新评估这应该「可行」；
* [Webpack](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) 的 Hot Module Replacement；
* [Flummox](https://github.com/acdlite/flummox) 教我如何不使用 boilerplate 和 singletons 来达成 Flux；
* [disto](https://github.com/threepointone/disto) 证明了 Stores 是 hot reloadable 的概念；
* [NuclearJS](https://github.com/optimizely/nuclear-js) 证明这个架构可以有很好的效能；
* [Om](https://github.com/omcljs/om) 推广单一原子化 state 的想法；
* [Cycle](https://github.com/cyclejs/cycle-core) 展示 function 往往是最好的工具；
* [React](https://github.com/facebook/react) 实际的创新。

特别感谢 [Jamie Paton](http://jdpaton.github.io) 它移交了 `redux` NPM 套件名称给我们。

### Logo

你可以在 [Github 上](https://github.com/reactjs/redux/tree/master/logo) 找到官方的 logo。

### 变更日志

这个专案依照 [Semantic Versioning](http://semver.org/)。
每一个释出版本都会伴随它的迁移说明，被记录在 Github [Releases](https://github.com/reactjs/redux/releases) 页面上。

### 赞助者

在 Redux 的工作是[由社群出资](https://www.patreon.com/reactdx)。
遇到一些卓越的公司使这可以成真：

* [Webflow](https://github.com/webflow)
* [Ximedes](https://www.ximedes.com/)

[查看完整的 Redux 赞助者清单。](PATRONS.md)

### License

MIT
