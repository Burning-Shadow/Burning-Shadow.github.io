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

​		单向数据流的初衷就是让逻辑变得更加清晰，我们写了这些代码的本质上**并非是**想提高运行速度或提升用户体验，（相较于写在 React Components 中的 state 而言这些看似繁复的规范无疑是增加了开发者的学习成本，增加了代码量，同时还增加了读取时间等等），**而是尽可能的**将流程交给我们的**数据管理器**。如此一来发生错误时可以**更精确的定位错误信息**，将错误定位时间压缩到最低。此外可以将我们的逻辑代码从生命周期函数中迁移出来，防止生命周期内布过于臃肿。将其内部的请求、存储等函数方法对象等等交由 Redux 统一管理会使得你的组件中全部为逻辑代码，对于持久化管理也是好处无穷。

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

## 异步

​		在具体介绍之前我们需要先介绍一个名词：**中间件 MiddleWare**。顾名思义是 Redux 的一部分扩展插件，用以辅助存储。

![Redux Data Flow](https://pic.superbed.cn/item/5dbcf376bd461d945aeaeb9d.jpg)

​		我们所常用的 `redux-thunk`、`redux-saga` 等都隶属于 MiddleWare 的范畴。

---

​		我们平日里在组件内请求而来的数据必须写在生命周期中，而这样的写法会使得代码的复用性降低，且无法将 Redux 数据管理的能力发挥到极致，所以为了使得我们可以通过 Redux 请求异步数据后再返回结果我们需要引入 MiddleWare —— `redux-thunk` 来帮助我们进行异步请求的管理。

​		上边我们讲解了对于本地数据存储的 Redux 使用方式，但是对于存在着大量异步请求的 web 端来说异步 Action 的存在同样重要。经过上边的介绍我们得知 Action 本质上就是一个个用于存储数据的 JSON 对象。而异步 Action 与普通 Action 不同之处只有一个：状态

​		在实践中组件发生 action 后，在进入 reducer 前需要完成一个异步任务，只有请求拿到数据之后才能再进入 reducer。而原生的 Redux 是不支持这种操作的，所以自然而然的就可以引入下边介绍的两种中间件：`redux-thunk` 和 `redux-saga`。

​		

### Redux-Thunk

​		这玩意儿跟咱们平时搞的不太一样，主要作用是**将 Action 的返回对象由对象扩展为函数**。它可以像普通 Action 对象一样，由 Store 实例进行

​		同样的，为了实现这一目的我们还需要引入新的中间件：`redux-thunk`。通过 npm 进行安装就可以啦。

安装完毕后我们要改变的其实也并不是很多。在 `store/index.js` 中引入即可

```javascript
// store/index.js

import { createStore, applyMiddleware, compose } from "redux";
import reducer from "./reducers";
import thunk from "redux-thunk";

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__
  ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({})
  : compose;

const enhancer = composeEnhancers(applyMiddleware(thunk));

const store = createStore(reducer, enhancer);

export default store;
```

至于与组件对应的 `actionCreator` 我们也无需担心。异步函数只要这样写就可以啦

```javascript
// store/actionCreators/todoList.js

export const getTodoList = () => {
  return (dispatch) => {
    axios
      .get(
        "https://www.easy-mock.com/mock/5dbc42c0727c0077ea997f43/example/getlist"
      )
      .then(res => {
        // console.log(res);
        const data = res.data;
        const action = getListAction(data);
        dispatch(action);
      })
      .finally(() => {
        console.log("finally");
      })
      .catch(e => {
        console.log(e);
      });
  };
};
```

注意我们传入了一个 `dispatch`，如此一来我们就可以通过 `redux-thunk` 来直接在 `actionCreator` 中更改数据了。

而具体的项目中其实和普通的一模一样。从 `actionCreator` 中引入函数后即可。其后的步骤如同普通的使用方式——在生命周期函数中创立副本；然后 `dispatch(action)`

```javascript
// src/pages/todoList.js

componentDidMount() {
  const action = getTodoList();
  store.dispatch(action)
}
```

### Redux-Saga

​		老规矩，npm下载`npm i redux-saga --save`

​		完成后我们需要在 `store/index.js` 中继续配置

```javascript
// store/index.js

import { createStore, applyMiddleware, compose } from "redux";
import reducer from "./reducers";
import createSagaMiddleware from "redux-saga";
import mySagas from "./sagas";

// 创建 saga 中间件
const sagaMiddleware = createSagaMiddleware();

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__
  ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({})
  : compose;

const enhancer = composeEnhancers(applyMiddleware(sagaMiddleware));

const store = createStore(reducer, enhancer);
sagaMiddleware.run(mySagas);

export default store;
```

​		细心的 boy 会发现我们会多引入一个 `sagas.js` 文件。这玩意儿得咱自己写.

​		同时还有这么一句：`sagaMiddleware.run(mySagas)` emmmmm... 非要执行一下？喔原来 `saga.js` 是一个 `generator`

```javascript
// store/saga.js

function* mySaga(){}
export default mySaga;
```

​		`generator` 函数本质是一个生成器，而 `saga.js` 中我们正需要她的特性。让我们完善一下我们刚才的 `saga.js`

```javascript
// store/saga.js

import { takeEvery, put } from 'redux-saga/effects';
import { GET_MY_LIST } from './actionTypes';
import { getListAction } from './actionCreators';
import axios from 'axios';

function* mySaga() {
  yield takeEvery(GET_MY_LIST, getList)
}

function* getList() {
  const res = yield axios.get("https://www.easy-mock.com/mock/5dbc42c0727c0077ea997f43/example/getlist");
  
  const action = getListAction(res.data);
  yield put(action);	// 使用 saga 特定 API put。一旦 action 改变就将变化传入 Store 中
}

export default mySaga;
```

​		其中，`takeEvery` 可以理解为一个**监听器**。每次我们将对应的 action 传入时就是事件的投入，而他则会**将异步请求返回的结果统一暴露给外界**——也就是监听它的组件 `store/index.js`。

​		而上面的 `getActionList` 也是和 `redux-thunk` 中的 `getTodoList` 异步函数不同，它就是一个普通的 action。

```javascript
// actionCreator.js

export const getMyListAction = () => ({
  type: GET_MY_LIST
})
```

​		而生命周期中的操作也很简单，在对应生命周期中写入即可

```javascript
componentDidMount() {
  const action = getMyListAction();
  store.dispatch(action);
}
```

​		相信大家看到这里已经明白，`react-saga` 不同于 `react-thunk` 直接使用异步函数的形式，而是通过 `generator` 对 `action` 状态进行监听，一旦发生变化则立刻通知 `store` 进行更新数据。颇有一种 `Don't call me, i'll call you` 的异步风采。而其实现刨去观察者-监听者模式之外更多的是利用 ES6 的 `generator` 函数进行异步的回调处理。

### 最后说两句

​		上面介绍的两种异步 Action 的 MiddleWare 使用方式事实上并无好坏之区分，适用场景不同，使用方法不同罢了，据说 `redux-saga` 更适用于大型项目。



## React-Redux

上述讲了这么多事实上