## Redux Thunk 解决了什么问题

在 Redux 中，reducer 必须是一系列的纯函数，不能包含任何副作用或者异步逻辑—，而在实际业务开发过程中，这些异步逻辑往往是必须的。Redux Thunk 就是提供了一个在使用 Redux 过程中编写这些异步逻辑地方。

## Thunk 的概念

“Thunk”并不是 Redux 中的新概念，而是一个编程术语，意为“一些执行延迟工作的代码”，例如我们有一个加法操作： `let x = 1 + 2`,这个操作运行时会立即执行，如果我们需要**延迟执行**这个操作,那么可以将其封装为一个函数： `let res = () => 1 + 2` ,这样这段加法操作就能延迟到需要的时候，通过调用 res 函数再执行。

因此 res 函数就是一个 Thunk！

所以结合上述概念，我们可以得出编写一个Redux中Thunk，就是编写一个“可以延迟执行 dispach 操作的函数”：

```JS
const thunkFunction = (dispatch, getState) => {
  // 这里可以编写一些异步逻辑，请求等

  // 最终再执行dispatch
  dispatch({type:'xxx',payload:'xxx'})
}

// dispatch 一个 Thunk
store.dispatch(thunkFunction)
```

## Redux Thunk 实现方式解析

上文中已明了Thunk的编写和使用：一个执行一些异步操作，完成后再执行dispatch的函数，该函数可以直接传入 dispatch方法。

我们可以发现，Redux 提供的dispatch本身是只能接受一个纯 Object 对象的，因此要想支持 dispatch 一个Thunk，重点便是要编写一个 Redux middleware 拦截 dispach 的 action。


Redux Thunk 的核心代码如下：

```JS
function createThunkMiddleware<
  State = any,
  BasicAction extends Action = AnyAction,
  ExtraThunkArg = undefined,
>(extraArgument?: ExtraThunkArg) {
  
  const middleware: ThunkMiddleware<State, BasicAction, ExtraThunkArg> =
    ({ dispatch, getState }) =>
    next =>
    action => {
      if (typeof action === 'function') {
        return action(dispatch, getState, extraArgument)
      }
      return next(action)
    }
  return middleware
}

export const thunk = createThunkMiddleware()

export const withExtraArgument = createThunkMiddleware


```

和正常编写中间件函数（middleware）一样，Redux Thunk最终返回了一个三层的嵌套函数。

- 第一层函数接受 dispatch 和 getState。
- 第二层函数接受 next，表示下一个中间件或最终的 reducer。
- 第三层函数接受 action，表示当前的动作。

在第三层函数中首先检查 action 是否是一个函数（Thunk）。如果是函数，则调用该函数，并传入 dispatch、getState 和 extraArgument

如果 action 不是函数，则将其传递给下一个中间件或最终的 reducer。


最后默认导出了不带任何参数的thunk middleware，以及一个支持传入自定义参数的 thunk middleware 生成函数。

## 使用方式

默认不带参数的 thunk middleware

```JS

import { createStore, applyMiddleware } from 'redux'
import { thunk } from 'redux-thunk'
import rootReducer from './reducers/index'

const store = createStore(rootReducer, applyMiddleware(thunk))

```

带参数的 thunk middleware

```JS

import { createStore, applyMiddleware } from 'redux'
import { thunk } from 'redux-thunk'
import rootReducer from './reducers/index'
import { myCustomApiService } from './api'
const thunk = withExtraArgument(myCustomApiService)
const store = createStore(rootReducer, applyMiddleware(thunk))

const myThunk  = (dispatch, getState, myCustomApiService) => {

   myCustomApiService().then(res=>{

    dispatch({type:'xxx',payload:res})

   })
}

```
