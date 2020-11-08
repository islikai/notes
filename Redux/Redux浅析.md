# Redux

> Redux 由 [Flux](http://facebook.github.io/flux/docs/overview/) 演变而来，但受 [Elm](https://elm-lang.org/docs) 的启发，避开了 Flux 的复杂性。

Redux 是 JavaScript 状态容器，提供可预测化的状态管理。



**1）为什么需要状态管理？**

随着 SPA开发日趋复杂，需要管理的 state （状态）也越来越多。 这些 state 可能包括服务器响应、缓存数据、本地生成尚未持久化到服务器的数据，也包括 UI 状态，如激活的路由，被选中的标签等等。

管理不断变化的 state 非常困难。如果一个 model 的变化会引起另一个 model 变化，那么当 view 变化时，就可能引起对应 model 以及另一个 model 的变化，依次地，可能会引起另一个 view 的变化。这就会导致**state 在什么时候，由于什么原因，如何变化已然不受控制。**

而Redux就是来解决这个问题的。



**2）什么场景需要使用状态管理?**

多交互、多数据源；

> 我们目前开发组件来说需要状态管理的场景有
>
> - 需要共享的组件状态
>
> - 需要在任何地方都可以拿到的状态
>
> - 组件需要改变全局状态
>
> - 组件需要改变另一个组件的状态



**3) Redux三大原则**

- 单一数据源

  整个应用的 state被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store中，做统一管理，便于调试与维护。

  我们可以把Redux的状态管理理解成一个全局对象，那么这个全局对象是唯一的，所有的状态都在全局对象store下进行统一”配置”，这样做也是为了做统一管理，便于调试与维护。

- State是只读的

  唯一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。

  这样做的好处是当我们去修改状态时，Redux会记录这个动作是什么类型的、具体完成了什么功能等（更新、传播过程），在调试阶段可以为开发者提供完整的数据流路径。

- Reducer必须是一个纯函数
  Reducer用来描述action如何改变state，接收旧的state和action，返回新的state。

  - why 纯函数？

    > 状态更新是可以测的，所以Reducer内部的执行操作必须是无副作用

  - why 返回新的state？

    > Redux是对新旧state直接用浅比较（==）来进行比较的，如果直接修改state对象，内存地址是没有变化的，因此Redux将不会响应我们的更新；
    > 可以提高性能，避免深层的对象递归比较；

  <img src = 'https://icon.qiantucdn.com/20201108/63ea3c3b4436d8e308deac18b62656302'     /> 

**4) 使用**

```react
import { createStore } from 'redux'

function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/incremented':
      return { value: state.value + 1 }
    case 'counter/decremented':
      return { value: state.value - 1 }
    default:
      return state
  }
}

// 创建一个保存应用程序状态的Redux存储
// API { subscribe, dispatch, getState }.
let store = createStore(counterReducer)

// 添加订阅
store.subscribe(() => console.log(store.getState()))

// 改变内部状态的唯一方法是dispatch一个action
store.dispatch({ type: 'counter/incremented' })
// {value: 1}
store.dispatch({ type: 'counter/incremented' })
// {value: 2}
store.dispatch({ type: 'counter/decremented' })
// {value: 1}
```



**5）Redux实现刨析** 

[code](https://stackblitz.com/edit/react-mbuhes?file=src%2Findex.js)

- createStore

- reducer

- dispatch

- middleware

  middleware就是“中间件”，是对 dispatch 的扩展，允许我们在某个流程的执行中间插入我们自定义的一段代码。

  > 特性：
  >
  > 1. 执行时机。middleware的代码执行时机是由框架确定的，我们只能定义代码，但无法改变代码运行的时机。比如Redux的middleware是在dispatch一个action，和action到达reducer之间调用。
  > 2. middleware允许链式调用。可以注册多个middleware，框架会按照顺序依次调用middleware。
  > 3. 如果注册了middleware，那么在之后每次dispatch的时候都会把所有middleware执行一遍。



### redux-thunk

> Thunk [middleware](https://redux.js.org/advanced/middleware) for Redux.

[redux源码](https://github.com/reduxjs/redux-thunk/blob/master/src/index.js)

```react
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {
  	// 如果是function类型，就调用这个function
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

