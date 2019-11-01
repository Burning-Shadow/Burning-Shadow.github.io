---
title: Redux初体验
categories: 状态管理库
date: 2019-10-30 10:24:01
tags:
 - 状态管理库
 - redux
---

​		鉴于单向数据流的理念逐渐深入人心，人们对 `react` 的设计理念也逐步提高了认同度。但是由于其祖孙间通讯过程不直观等问题，在大型项目中我们需要一个第三方的更稳定版本的 “`event bus`”。在诸多候选者中易于操作且基于单向数据流的 `redux` 成功取代 `Flux` 成为脱颖而出的佼佼者。

<!--more-->

## 注意事项

- `redux` 本身由 ES2015 编写，故其可以支持任何现代浏览器，无需 Babel 或模块打包器进行处理
- 使用 `react` 进行开发时你需要 `react绑定库` 和 `开发者工具`  —— `react-redux` & `redux-devtools`
- `Redux` 生态下的很多包都不提供 UMD 文件，故为了提升开发体验，官方推荐使用像 Webpack 和 Browserify 这样的 CommonJS 模块打包器。

## 特点

​		React 最基本的逻辑就是将内容（state）以对象的形式存储在一个单一的 store 中，只有 action 可以对 state 进行更改。

![Redux Flow](https://pic.superbed.cn/item/5db93767bd461d945a59da84.jpg)

​		如图，我们可以将此理解为一个图书管理系统。我们所使用状态管理器（Redux）的组件可以理解为**借书者**，Store 则是**图书管理员**。我们借书的过程中必须借助 Action Creators 进行“沟通”，将我们借书的请求分派（dispatch）给**图书管理员**（Store），而后由图书管理员对此请求进行梳理，去**图书馆**（Reducers）中进行图书的取借，完成后将新状态返还给我们的**借书者**（React Components）

- Store 只能有一个，用于存储整个应用的状态（数据）。
- React Components 改变 state的唯一方法是通过调用 store 的 dispatch 方法，触发一个 action。此 action 被**相应的 reducer** 处理，完成 state 的更新。
- **reducer 必须为纯函数**。
- 组件间的值传递需要 dispatch action 给 store，而非直接通知。
- 组件更新时通过 subscribe store 中的 state 来刷新自己的视图。

## 为什么这么复杂

​		单向数据流的初衷就是让逻辑变得更加清晰，我们写了这些代码的本质上**并非是**想提高运行速度或提升用户体验，（相较于写在 React Components 中的 state 而言这些看似繁复的规范无疑是增加了开发者的学习成本，增加了代码量，同时还增加了读取时间等等），**而是尽可能的**将流程交给我们的**数据管理器**。如此一来发生错误时可以**更精确的定位错误信息**，将错误定位时间压缩到最低。

## 基础篇

​		介绍的话还是从基础的使用讲起吧。此部分分为三块：Action、Reducer、Store

### Action

​		Action ，顾名思义：`行动、活动、功能`。其在 Redux 中扮演着非常重要的角色。主要是用来己撸我们的状态。如果忘记前边那张图的 boy 可以翻到前边看一下。

​		不过需要大家伙儿注意的一点是：**action 只是一个个的；用于描述存储值的对象！我们需要的是通过 actionCreators 创建相应的 action！**

​		举个例子，action 的样子如下

```json
{
  type: "change_ipnut_value",
  value: inputValue
}
```

​		而 actionCreators 则是这样的

```javascript
export const getInputChangeAction = value => ({
  type: "change_ipnut_value",
  value: inputValue
});
```

​		我们在组件内通过 actionCreators 创建相应的 action 后，需要在同样的位置由 store 进行 dispatch(action) 操作

### Reducers

​		还是那张老图，我们通过 `store.dispatch(action)` 操作**通知** Store 后，Store 再将相应的消息通知给 Reducers 进行处理。Reducers 接收之前的老数据，并将更改后的 数据返还给 Store 进行更改。

```javascript
const defaultState = {
  inputValue: "",
  list: ["1", "2", "3", "4", "5"]
};

export default (state = defaultState, action) => {
  // // state 为上一次所保存的数据（value），action 为用户所传递过来的描述（type）
  // // console.log("state = ", state);
  // // console.log("action = ", action);

  const newState = JSON.parse(JSON.stringify(state));

  switch (action.type) {
    case CHANGE_INPUT_VALUE:
      newState.inputValue = action.value;
      break;
    case ADD_TODO_ITEM:
      newState.list.push(newState.inputValue);
      newState.inputValue = "";
      break;
    case DELETE_TODO_ITEM:
      newState.list.splice(action.index, 1);
      break;
    default:
  }
  return newState || state;
};
```

​		如上例，我们要做的最多的就是将 reducer 变成一个 [纯函数]( https://juejin.im/post/5b1a251e6fb9a01e83146ddf ) ： **相同的输入，永远会得到相同的输出** 。而通过 ES6 语法令 state 拥有默认值无疑也是更加便捷的操作。

#### 拆分

​		Reducer 虽好用但是在系统足够复杂之后我们不得不对其进行**拆分**了。同样的层级同样的 `switch...case` 使得虽然可以完成判断但是不同的组件间不同 State 的分层已经不再明显。基础的方法嘛……函数化呗。将我们相应的组件以函数的形式独立出去，再引入进一个汇总所用的 reducer 中。时不时很简单？

​		才疏学浅所以就用官方的例子咯。`todos` 和 `visibilityFilter` 是两个相互独立的更新系统，所以我们将其单独拆分成两个系统（`visibilityFilter` 相对结构较为简单所以就不对其进行拆分啦）。这也是组件化的一部分，使得其逻辑更为清晰。

```javascript
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    case TOGGLE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

​		如此虽然说简化了 `reducers/index` 的代码量，但是稳定性、复用性等等都与编码者的编码习惯有关。算是项目中的一个不稳定因素。所以我们可以使用官方提供的 API

#### combineReducers

​		这东西物如其名，就是将不同模块对应的 Reducer 结合到一起。我们将 `reducer/index` 中的代码改成这样

```javascript
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

​		上边的写法和下边的完全等价

```javascript
export default function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

​		而具体的诸如 `visibilityFilter` 之类的 Reducer 中则照着原本的样子进行编写。如此就可以完成模块的拆分咯。相较于老版本的写法官网的 API 使得 index 更为整洁，不同的是在对应组件中需要获取 `state` 必须通过 `store.getState().visibilityFilter ` / `store.getState().todos` 等方式获取，以此完成组件对数据的订阅。

### Store

​		上边说了那么多，对 Store 只是提了一下而已。事实上 Store 是将上述内容所联系起来的对象

>- 它有以下几个方法：
>
>  -  提供 [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 方法获取 state； 
>
>  -  提供 [`dispatch(action)`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法更新 state； 
>
>  -  通过 [`subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 注册监听器; 
>
>  -  通过 [`subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 返回的函数注销监听器。 
>
>    ​																									——Redux官方文档

​		相应的，为了其后期的调试及使用，我们需要在 chrome 上安装插件 `Redux Devtools`。同时，需要在相应的存储文件夹对应的文件中（一般是index，毕竟是用来关联 Reducers 和 actionCreators 的）进行如下设置：

```javascript
import { createStore } from "redux";
import reducer from "./reducers";

const store = createStore(
  reducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);

export default store;
```

​		最后，不要忘记把此文件暴露给具体应用到 Redux 的组件哦。

## 提升篇

### 异步Action

​		上边我们讲解了对于本地数据存储的 Redux 使用方式，但是对于存在着大量异步请求的 web 端来说异步 Action 的存在同样重要。经过上边的介绍我们得知 Action 本质上就是一个个用于存储数据的 JSON 对象。而异步 Action 与普通 Action 不同之处只有一个：状态

​		就像是 ES6 中的 Promise，除去 `pending` 状态外还有 `fulfilled` 和 `reject` 两种状态分别代表着执行成功与否，相应的，异步 Action 也存在着如上的情景，所以我们通常也会使用一个特殊字段来记录请求的状态：`status`.

​		举个例子，一条后台返还的数据通常会有状态码 `status` 和数据 `data`，所以一条异步 Action 也应该具备以下三个字段（当然，以下是个人习惯，大家选自己适合的就好啦）

```json
{ 
  type: 'FETCH_POSTS',
  status: 0,
  response: { ... } 
}
```

#### Redux Thunk

​		这玩意儿跟咱们平时搞的不太一样，主要作用是**将 Action 的返回对象由对象扩展为函数**。它可以像普通 Action 对象一样，由 Store 实例进行

​		同样的，为了实现这一目的我们还需要引入新的中间件：`redux-thunk`。通过 npm 进行安装就可以啦。