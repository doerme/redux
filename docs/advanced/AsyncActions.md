# Async Action

在[基础教学](../basics/README.md)中，我们建立了一个简单的 todo 应用程序。它完全是同步的。每次 action 被 dispatch，state 都会立刻被更新。

在这份教学中，我们将会建立一个不同而且非同步的应用程序。它将会使用 Reddit API 针对选择的 subreddit 来显示现在的头条新闻。如何让非同步与 Redux 数据流结合呢？

## Action

当你请求一个非同步 API，有两个关键的时刻：你开始请求的的时候，以及当你收到回应 (或是超时) 的时候。

这两个时刻通常都可以要求改变应用程序的 state；要做到这一点，你需要 dispatch 会被 reducer 同步处理的一般 action。通常，针对任何一个 API 请求你会需要 dispatch 至少三个不同种类的 action：

* **一个告知 reducer 请求开始的 action。**

  reducer 可以通过打开 state 里的 `isFetching` flag 来处理这个 action。这样 UI 就知道是时候显示一个 spinner 了。

* **一个告知 reducer 请求成功完成的 action。**

  reducer 可以通过把新的数据合并到它们管理的 state 里并重置 `isFetching` 属性来处理这个 action。UI 将会把 spinner 隐藏，并显示抓回来的数据。

* **一个告知 reducer 请求失败的 action。**

  Reducers 可以通过重置 `isFetching` 属性来处理这个 action。此外，有些 reducer 也会想要储存错误讯息，这样 UI 就可以显示它。

你可以在 actions 里使用一个专用的 `status` 属性：

```js
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

或者你也可以为它们定义不同的 types：

```js
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

无论要选择使用单一一个 action type 与 flag，或是选择多种 action type，这都取决于你。这是一个你需要跟你的团队一起决定的惯例。多种的 type 出现错误的空间更小，不过如果你使用像是 [redux-actions](https://github.com/acdlite/redux-actions) 之类的 helper library 来产生 action creator 和 reducer 的话，这不会是个问题。

无论你选择怎样的惯例，请在整个应用程序中贯彻下去。
在这份教学中，我们将会使用不同的 type。

## 同步的 Action Creator

让我们开始定义几个在范例应用程序中需要的同步 action type 和 action creator。在这里，使用者可以选择显示一个 subreddit：

#### `actions.js`

```js
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```

他们也可以按下「刷新」按钮来更新它：

```js
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```

有一些 action 是通过使用者互动来控制。我们也会有其他种类通过网路请求控制的 action。我们之后将会看到要如何 dispatch 它们，但现在，我们只想要定义它们。

当是时候针对 subreddit 抓取 posts 时，我们会 dispatch 一个 `REQUEST_POSTS` action：

```js
export const REQUEST_POSTS = 'REQUEST_POSTS'

export function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```

把 `SELECT_SUBREDDIT` 和 `INVALIDATE_SUBREDDIT` 分开对它来说是非常重要的。当它们可能一个发生在另一个之后，而随著应用程序变得更复杂，你可能会想要针对使用者的动作独立的抓取一些数据 (举例来说，预先抓取最有人气的 subreddits，或是在一段时间之后刷新旧的数据)。你可能还需要对应 route 的改变去抓取数据，所以在初期就把抓取数据跟 一些特定的 UI 事件耦合在一起不是很明智。

最后，当网路请求传回来时，我们会 dispatch `RECEIVE_POSTS`：

```js
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

export function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}
```

这些就是我们现在需要知道的。可以随著网路请求一并 dispatch 这些 action 的特别机制将会在之后讨论。

>##### 关于错误处理的附注

>在一个真实的应用程序中，你也会想要在请求失败时 dispatch 一个 action。我们不会在这份教学中实践错误处理，不过 [real world example](../introduction/Examples.md#real-world) 展示了其中一个可行的方法。

## 设计 State 的形状

就像在基础教学中一样，你会需要在仓促的开始实践之前，先[设计应用程序 state 的形状](../basics/Reducers.md#designing-the-state-shape)。有了非同步的代码，会有更多 state 要关照，所以我们需要把它完整思考过一遍。

这个部分常常让初学者困惑，因为要用什么资讯来描述一个非同步的应用程序的 state，还有要如何在单一一个 tree 上组织它不是一开始就很明显。

我们会先从最常见的使用案例开始：清单。网页应用程序时常会显示一些东西的清单。例如：post 的清单、朋友的清单。 你会需要弄清楚你的应用程序可以显示什么类型的清单。你想要把它们分别储存在 state 里，因为这样你可以快取它们而且只在需要时才重新抓取数据。

这就是我们的「Reddit 头条新闻」应用程序的 state 形状可能会看起来的样子：

```js
{
  selectedSubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```

这里有几个重要的点：

* 我们分别的储存每一个 subreddit 的资讯，所以我们可以快取每一个 subreddit。当使用者第二次在它们之间切换，将会即时更新，而且除非我们想要不然我们不需要重新抓取数据。不要担心这些东西全部都会在记忆体里：除非你正在处理数以万计的项目，并且你的使用者不太关闭 tab，不然你完全不需要用任何方式清除他们。

* 针对每一个项目清单，你会想储存一个 `isFetching` 属性来显示 spinner，`didInvalidate` 让你可以在数据已经过时的时候再去触发更新它，`lastUpdated` 让你知道最后一次抓取数据的时间，还有 `items` 它们自己。在一个真实的应用程序中，你也会想要储存 pagination state 像是 `fetchedPageCount` 和 `nextPageUrl`。

>##### 关于嵌套 Entity 的附注

>在这个范例中，我们把收到的项目跟 pagination 资讯储存在一起。但是，如果你有嵌套且互相参考的 entity，或是如果你让使用者可以编辑项目，那这个方法不会运作得很好。试想如果使用者想要去编辑一个抓回来的 post，但是这个 post 被复制到 state tree 中的好几个地方。实践这个将会非常痛苦。

>如果你有嵌套的 entity，或是如果你让使用者可以编辑接收到的项目，你应该把它们分别保存在 state 中，就像它是一个数据库。在 pagination 资讯中，你只会通过它们的 IDs 来参考它们。这使你能让它们始终保持更新到最新状态。[real world example](../introduction/Examples.md#real-world) 展示了这个方法，并使用了 [normalizr](https://github.com/gaearon/normalizr) 来正规化嵌套的 API 回应。用这个方法，你的 state 可能会看起来像这样：

>```js
> {
>   selectedSubreddit: 'frontend',
>   entities: {
>     users: {
>       2: {
>         id: 2,
>         name: 'Andrew'
>       }
>     },
>     posts: {
>       42: {
>         id: 42,
>         title: 'Confusion about Flux and Relay',
>         author: 2
>       },
>       100: {
>         id: 100,
>         title: 'Creating a Simple Application Using React JS and Flux Architecture',
>         author: 2
>       }
>     }
>   },
>   postsBySubreddit: {
>     frontend: {
>       isFetching: true,
>       didInvalidate: false,
>       items: []
>     },
>     reactjs: {
>       isFetching: false,
>       didInvalidate: false,
>       lastUpdated: 1439478405547,
>       items: [ 42, 100 ]
>     }
>   }
> }
>```

>在这份教学中，我们不会把 entity 正规化，不过针对一个更动态的应用程序，你应该考虑这样做。

## 处理 Actions

在走进把 dispatch action 和网路请求结合的细节之前，我们将会编写 reducer 给我们上面定义的 action。

>##### 关于 Reducer Composition 的附注

> 这里，我们假设你已经了解通过 [`combineReducers()`](../api/combineReducers.md) 来做 reducer composition，它被描述在[基础教学](../basics/README.md)的[拆分 Reducer](../basics/Reducers.md#splitting-reducers) 章节中。如果你不了解，请[先阅读它](../basics/Reducers.md#splitting-reducers)。

#### `reducers.js`

```js
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedSubreddit(state = 'reactjs', action) {
  switch (action.type) {
    case SELECT_SUBREDDIT:
      return action.subreddit
    default:
      return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: []
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = {}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedSubreddit
})

export default rootReducer
```

在这份代码中，有两个有趣的部分：

* 我们使用 ES6 computed property 语法，所以我们可以把 `state[action.subreddit]` 和 `Object.assign()` 更新成一个更简洁的方式。这样：

  ```js
  return Object.assign({}, state, {
    [action.subreddit]: posts(state[action.subreddit], action)
  })
  ```
  等同于：

  ```js
  let nextState = {}
  nextState[action.subreddit] = posts(state[action.subreddit], action)
  return Object.assign({}, state, nextState)
  ```
* 我们把 `posts(state, action)` 抽出来管理具体的 post 清单的 state。这只是 [reducer composition](../basics/Reducers.md#splitting-reducers)！这是我们选择用来把 reducer 拆分成更小的 reducers 的方式，而在这个案例中，我们把在对象中更新项目的工作委派给 `posts` reducer。[real world example](../introduction/Examples.md#real-world) 更进一步，展示了如何建立一个 reducer factory 来参数化 pagination reducer。

请记得 reducer 只是些 function，所以你可以尽你所能舒适的使用 functional composition 和 higher-order function。

## 非同步的 Action Creator

最后，我们要如何一起使用我们[之前定义的](#synchronous-action-creators)同步的 action creator 和网路请求呢？用 Redux 要做到这个的标准方式是使用 [Redux Thunk middleware](https://github.com/gaearon/redux-thunk)。它属于一个独立的套件，叫做 `redux-thunk`。我们[晚点](Middleware.md)会解释 middleware 一般来说是如何运作的；现在，你只有一件重要的事必需知道：通过使用这个特定的 middleware，action creator 可以回传一个 function 来取代 action 对象。这样的话，function creator 就变成一个 [thunk](https://en.wikipedia.org/wiki/Thunk)。

当一个 action creator 回传一个 function 的时候，这个 function 将会被 Redux Thunk middleware 执行。这个 function 不需要是 pure 的；因此它被允许一些有 side effect 的动作，包括执行非同步的 API 请求。这个 function 也可以 dispatch action—像是那些我们之前定义的同步 action。

我们仍然可以把 这些特别的 thunk action creator 定义在我们的 `actions.js` 档案中：

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

// 迎接我们的第一个 thunk action creator！
// 虽然它里面不一样，不过你可以就像其他的 action creator 一般使用它：
// store.dispatch(fetchPosts('reactjs'))

export function fetchPosts(subreddit) {

  // Thunk middleware 知道如何去处理 function。
  // 它把 dispatch method 作为参数传递给 function，
  // 因此让它可以自己 dispatch action。

  return function (dispatch) {

    // 第一个 dispatch：更新应用程序 state 以告知
    // API 请求开始了。

    dispatch(requestPosts(subreddit))

    // 被 thunk middleware 请求的 function 可以回传一个值，
    // 那会被传递作为 dispatch method 的回传值。

    // 在这个案例中，我们回传一个 promise 以等待。
    // 这不是 thunk middleware 所必须的，不过这样对我们来说很方便。

    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // 我们可以 dispatch 许多次！
        // 在这里，我们用 API 请求的结果来更新应用程序的 state。

        dispatch(receivePosts(subreddit, json))
      )

      // 在一个真实世界中的应用程序，你也会想要
      // 捕捉任何网路请求中的错误。
  }
}
```

>##### 关于 `fetch` 的附注

>在范例中，我们使用 [`fetch` API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API)。它是一个用来建立网路请求的新 API，取代 `XMLHttpRequest` 最常见的需求。因为大部份的浏览器还没有原生的支持它，我们建议你使用 [`isomorphic-fetch`](https://github.com/matthew-andrews/isomorphic-fetch) library：

>```js
// 在每一个你使用 `fetch` 的文件中加入这段
>import fetch from 'isomorphic-fetch'
>```

>内部机制中，它在客户端上会使用 [`whatwg-fetch` polyfill](https://github.com/github/fetch)，而在服务器上会使用 [`node-fetch`](https://github.com/bitinn/node-fetch)，所以如果你把应用程序改变成 [universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9) 的，不需要改变任何的 API 请求。

>要注意，所有的 `fetch` polyfill 都假设已经有一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) polyfill。要确保你有一个 Promise polyfill 的最简单的方式，是在你的进入点任何其他的代码执行之前启用 Babel 的 ES6 polyfill：

>```js
>// 在你的应用程序任何其他的代码之前做一次这个
>import 'babel-polyfill'
>```

我们要如何把 Redux Thunk middleware 加进 dispatch 机制里？我们使用 Redux 里的 [`applyMiddleware()`](../api/applyMiddleware.md) store enhancer，如下所示：

#### `index.js`

```js
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // 让我们来 dispatch() function
    loggerMiddleware // 巧妙的 middleware，用来 log action
  )
)

store.dispatch(selectSubreddit('reactjs'))
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```

有关 thunk 的好处是，它们可以 dispatch 其他 thunk 的结果：

#### `actions.js`

```js
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

export const RECEIVE_POSTS = 'RECEIVE_POSTS'
function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {
  return dispatch => {
    dispatch(requestPosts(subreddit))
    return fetch(`http://www.reddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json => dispatch(receivePosts(subreddit, json)))
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {

  // 记住，function 也会收到 getState()，
  // 它让你选择下一个要 dispatch 什么。

  // 如果被快取的值已经是可用的话，
  // 这对于避免网路请求很有用。

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // 从 thunk Dispatch 一个 thunk！
      return dispatch(fetchPosts(subreddit))
    } else {
      // 让请求的代码知道没有东西要等待了。
      return Promise.resolve()
    }
  }
}
```

这让我们可以渐渐的编写更复杂的非同步控制流程，而使用的代码却可以保持几乎一样：

#### `index.js`

```js
store.dispatch(fetchPostsIfNeeded('reactjs')).then(() =>
  console.log(store.getState())
)
```

>##### 关于服务器 Render 的附注

>Async action creator 对服务器 render 特别方便。你可以建立一个 store，dispatch 一个单一的 async action creator，它会 dispatch 其他的 async action creator 来为整个应用程序抓取数据，并在 Promise 回传并完成之后才 render。接著你 render 之前需要的 state 将必须被 hydrate 到你的 store。

[Thunk middleware](https://github.com/gaearon/redux-thunk) 不是在 Redux 中协调非同步 action 的唯一方式：
- 你可以使用 [redux-promise](https://github.com/acdlite/redux-promise) 或是 [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) 来 dispatch Promise 取代 function。
- 你可以使用 [redux-observable](https://github.com/redux-observable/redux-observable) 来 dispatch Observables。
- 你可以使用 [redux-saga](https://github.com/yelouafi/redux-saga/) middleware 来建置更复杂的非同步 action。
- 你甚至可以编写一个客制化的 middleware 来描述你的 API 请求，像是 [real world example](../introduction/Examples.md#real-world) 做的那样。

你可以自由地尝试几个选项，选择一个你喜欢的惯例，并遵守它，无论有没有使用 middleware。

## 连结到 UI

Dispatch async action 跟 dispatch 同步的 action 没有什么不同，所以我们不会详细讨论这个。查看[搭配 React 运用](../basics/UsageWithReact.md)了解有关结合 Redux 与 React component 的介绍。查看[范例：Reddit API](ExampleRedditAPI.md)来取得在这个范例中讨论的完整原始码。

## 下一步

查看[非同步数据流](AsyncFlow.md)回顾一下 async action 如何融入 Redux 数据流。
