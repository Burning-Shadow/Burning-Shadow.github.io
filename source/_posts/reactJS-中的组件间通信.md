---
title: reactJS中的组件间通信
date: 2019-07-07 10:13:00
categories: React
tags:
 - props
 - context
 - events
---

由于最近在看`reactJS`，其中的数据为单向传输，有点类似于阉割版的`vue`，所以特地总结一下免得面试的时候搞混了，同时试着找找二者的可借鉴之处

<!--more-->

### 父子组件间通信

#### 父传子

这个就类似于 `vue` 啦。vue 会在父组件中调用子组件之处用`v-bind`绑定要传递给子组件的数据，而子组件则设置`props`项来接收父组件发向自己的东西。其中要注意的就是`props`中的字段一定要和父组件中相应的绑定字段一致，否则还是接收不到的叻~

而 `react` 和 `vue` 的这种传值方式很像，都是通过子组件来接收 `props` 并动态调用。（单向数据流理念）

```javascript
// 父组件
class Parent extends Component {
  constructor() {
    super();
    this.state = {
      value: '',
    }
  }

  handleChange = e => {
    this.value = e.target.value;
  }

  handleClick = () => {
    this.setState({
      value: this.value,
    })
  }

  render() {
    return (
      <div>
        我是parent
        <input onChange={this.handleChange} />
        <div className="button" onClick={this.handleClick}>通知</div>
        <div>
          <Child value={this.state.value} />
        </div> 
      </div>
    );
  }
}
```

```javascript
// 子组件
class Child extends Component {
  render() {
    return (
      <div>
        我是Child，得到传下来的值：{this.props.value}
      </div>
    );
  }
}
```

#### 子传父

由于 `react` 的理念为单向数据流传递，所以我们子组件给父组件传递消息就只能通过另外的“奇技淫巧”

- 在 `vue` 中我们是通过 `$emit` 和 `v-on` 来完成父子组件间的通讯。绑定同一事件名后一旦触发则给父组件传递消息，父组件再调用自身的函数。
- 第二种方式是父组件将子组件的函数

而 `react` 中则是通过将父组件的更改状态的方法通过 `props` 传递给子组件，由子组件直接更改父组件数据：

```javascript
// parent

class Parent extends Component {
  constructor() {
    super();
    this.state = {
      value: '',
    }
  }

  setValue = value => {
    this.setState({
      value,
    })
  }

  render() {
    return (
      <div>
        <div>我是parent, Value是：{this.state.value}</div> 
        <Child setValue={this.setValue} />
      </div>
    );
  }
}
```

```javascript

class Child extends Component {

  handleChange = e => {
    this.value = e.target.value;
  }

  handleClick = () => {
    const { setValue } = this.props;
    setValue(this.value);
  }

  render() {
    return (
      <div>
        我是Child
        <div className="card">
          state 定义在 parent
          <input onChange={this.handleChange} />
          <div className="button" onClick={this.handleClick}>通知</div>
        </div>
      </div>
    );
  }
}
```

如上所示，`react` 中的子组件拿到父组件的方式更改变量，私以为权力过于大，但是仁者见仁智者见智，只读属性通过 `props` 即可完成。

父组件将改变自身状态的函数传递给子组件，而当子组件想要更改父组件的值时，执行父组件的状态更改 `setState` 函数，从而直接更改父组件的 `state`，使得父组件重渲染

### 兄弟组件间通信

#### 状态提升

当年 `vue` 我们实现兄弟组件间通信时有一种方式就是借鉴了 `react` 的“状态提升”。即，将兄弟组件的数据提升至其公共父组件，再由父组件完成分发以达到交流目的

```javascript
// parent

class Container extends Component {
  constructor() {
    super();
    this.state = {
      value: '',
    }
  }

  setValue = value => {
    this.setState({
      value,
    })
  }

  render() {
    return (
      <div>
        <A setValue={this.setValue}/>
        <B value={this.state.value} />
      </div>
    );
  }
}
```

```javascript
// brotherA

class A extends Component {

  handleChange = (e) => {
    this.value = e.target.value;
  }

  handleClick = () => {
    const { setValue } = this.props;
    setValue(this.value);
  }

  render() {
    return (
      <div className="card">
        我是Brother A, <input onChange={this.handleChange} />
        <div className="button" onClick={this.handleClick}>通知</div>
      </div>
    )
  }
}
```

```javascript
// brotherB

const B = props => (
  <div className="card">
    我是Brother B, value是：
    {props.value}
  </div>
);
export default B;
```

原理其实比较简单，只不过需要找到其公共父组件。简单结构是可以满足了。

#### Context

如果是一种很恶心的情况，即，两个组件间需要穿过一层层的父元素才能通信，这要怎么办呢？

此时就可以用到`react`中自带的状态管理器`context`了。`context`设计目的是共享那些对于一个组件树而言是“全局”的数据，比如已登录用户的头像、ID等信息。

为了方便演示我们用一个嵌套层级较多的

```javascript
// 顶级公共组件
class Context extends Component {

  constructor() {
    super();
    this.state = {
      value: '',
    };
  }

  setValue = value => {
    this.setState({
      value,
    })
  }

  getChildContext() { // 必需
    return { 
      value: this.state.value,
      setValue: this.setValue,
    };
  }
  render() {
    return (
      <div>
        <AParent />
        <BParent />
      </div>
    );
  }
}
// 必需
Context.childContextTypes = {
  value: PropTypes.string,
  setValue: PropTypes.func,
};
```

```javascript
// A 的 parent
class AParent extends Component {
  render() {
    return (
      <div className="card">
        <A />
      </div>
    );
  }
}
// A
class A extends Component {

  handleChange = (e) => {
    this.value = e.target.value;
  }

  handleClick = () => {
    const { setValue } = this.context;
    setValue(this.value);
  }

  render() {
    return (
      <div>
        我是parentA 下的 A, <input onChange={this.handleChange} />
        <div className="button" onClick={this.handleClick}>通知</div>
      </div>
    );
  }
}
// 必需
A.contextTypes = {
  setValue: PropTypes.func,
};
```

```javascript
// B 的 parent
class BParent extends Component {
  render() {
    return (
      <div className="card">
        <B />
      </div>
    );
  }
}

// B
class B extends Component {

  render() {
    return (
      <div>
        我是parentB 下的 B, value是：
        {this.context.value}
      </div>
    );
  }
}

B.contextTypes = {
  value: PropTypes.string,
};
```

中间者是`context`公有`container`组件。我们通过`getChildContext`函数定义`context`中的值，注意别忘了`childContextTypes`。这样属于这个`container`组件的子组件通过`this.context`就可以取到定义的值，且起到跟`state`同样的效果。

相当于是官方给你的挂，省去了`props`的传递

### 大型项目

这种时候就直接上库咯。可靠性更高也更加方便。推荐的库有`Redux`和`Mobx`，今后学习了再写吧。



























