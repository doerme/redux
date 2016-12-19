# 异步数据流

不使用 [middleware](Middleware.md) 的话，Redux 的 store 只支持[同步数据流](../basics/DataFlow.md)。 这就是你使用 [`createStore()`](../api/createStore.md) 预想会拿到的。

你可以用 [`applyMiddleware()`](../api/applyMiddleware.md) 来加强 [`createStore()`](../api/createStore.md)。这不是必须的，不过这让你[用比较方便的方式来表达非同步的 action](AsyncActions.md)。

非同步的 middleware 像是 [redux-thunk](https://github.com/gaearon/redux-thunk) 或是 [redux-promise](https://github.com/acdlite/redux-promise) 都包装了 store 的 [`dispatch()`](../api/Store.md#dispatch) 方法，并允许你 dispatch 一些 action 以外的东西，例如：function 或 Promise。你使用的任何 middleware 可以接著解译你 dispatch 的任何东西，并转而传递 action 到下一个链中的 middleware。例如，Promise middleware 可以拦截 Promise，并针对每一个 Promise 非同步的 dispatch 一对的开始/结束 action。

当最后一个在链中的 middleware 要 dispatch action 时，action 必须是个一般对象。这就是[同步的 Redux 数据流](../basics/DataFlow.md)开始的地方。

请查看[非同步范例完整的源码](ExampleRedditAPI.md)。

## 下一步

现在你已经看过一个 middleware 在 Redux 中可以做到什么的范例了，是时候来学习它实际上如何运作，还有你可以如何建立你自己的 middleware。前进到下一个章节有 [Middleware](Middleware.md) 相关的详细内容。
