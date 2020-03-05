---
title: React生命周期之——父子组件触发顺序
categories: React
date: 2020-03-03 22:21:02
tags:
 - React
---

​		React 生命周期一直是大家学习中最基础的知识点，但是涉及到父子组件层级嵌套时就需要特别注意。父组件的操作也可能会影响到子组件从而触发子组件的重渲染，所以为了尽量避免多余的 `re-render` 我们今天就来详细的探讨一下会触发子组件重渲染的机制。

<!--more-->

### 代码

​		生命周期方面的知识在此就不再赘述，我们直接来看其渲染。首先贴代码

```javascript
import React from "react";

class SubComp extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isUpdate: false
    };
  }

  UNSAFE_componentWillReceiveProps(nextProps) {
    console.log(this.props.order + " ChildComponent WillRecieveProps");
  }
  UNSAFE_componentWillMount(){
    console.log(this.props.order + " ChildComponent WillMounted");
  }
  componentDidMount() {
    console.log(this.props.order + " ChildComponent Mounted");
  }
  UNSAFE_componentWillUpdate() {
    console.log(this.props.order + " ChildComponent WillUpdate");
  }
  componentDidUpdate() {
    console.log(this.props.order + " ChildComponent DidUpdate");
  }

  render() {
    console.log(this.props.order + " ChildComponent Render");
    /*接受一个从父元素传来的props.order*/
    return (
      <button
        onClick={() => {
          this.setState({ isUpdate: true });
        }}
      >
        {this.props.order + " button updated? : " + this.state.isUpdate}
        <br />
      </button>
    );
  }
}

//父组件
class SuperComp extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isChanged: "false"
    };
  }

  UNSAFE_componentWillUpdate() {
    console.log("SuperComponent WillUpdate");
  }
  componentDidUpdate() {
    console.log("SuperComponent DidUpdate");
  }
  UNSAFE_componentWillMount(){
    console.log("SuperComponent WillMounted");
  }
  componentDidMount() {
    console.log("SuperComponent Mounted");
  }

  render() {
    //渲染三个子组建，并提供一个按钮，当按钮被点击时强行update
    console.log("SuperComponent Render");
    return (
      <div>
        <SubComp order="1st" />
        <br />
        <SubComp order="2nd" />
        <br />
        <SubComp order="3rd" />
        <br />
        <button
          onClick={() => {
            this.setState({
              isChanged: "true"
            });
          }}
        >
          Change Sup Data
        </button>
        <div>{this.state.isChanged}</div>
      </div>
    );
  }
}

export default SuperComp;
```

​		以上代码罗列出了一些相关钩子函数，那么我们就按时段解析引起子组件更新的条件

### 首次渲染

​		首次渲染时控制台打印结果

![首次渲染](https://pic.downk.cc/item/5e607afa98271cb2b869d8e4.png)

由于父组件是顶级组件故无需写 `constructor` 和 `super` 。所以最初的打印结果为此。可见**首次渲染时钩子执行顺序**为：

- Father Constructor
- Father ComponentWillMount
- Father Render
  - Child1 Constructor
  - Child1 ComponentWillMount
  - Child1 Render
  - Child2 Constructor
  - Child2 ComponentWillMount
  - Child2 Render
  - 。。。。
  - Child1 ComponentDidMount
  - Child2 ComponentDidMount
  - 。。。。
- Father ComponentWillMount

### 子组件状态改变

![子组件状态改变](https://pic.downk.cc/item/5e607b5f98271cb2b869f94e.png)

我们点击第一个 `childComponent` 时会触发其点击事件，从而引起子组件生命周期的一系列变化。但是过程之中并不会影响到父组件，所以不用担心

### 父组件状态改变

![父组件状态改变](https://pic.downk.cc/item/5e607cbb98271cb2b86acc80.png)

在例子中我特地设定了一个与子组件无关的变量：`isChanged` 。之所以设置它是因为希望证实在父组件被更改后子组件的变化。而图示也很好的说明了这一点。一旦父组件状态变更那么触发的钩子函数如下

- Father WillUpdate
- Father Render
  - Child1 WillReceiveProps
  - Child1 WillUpdate
  - Child1 Render
  - Child2 WillReceiveProps
  - Child2 WillUpdate
  - Child2 Render
  - 。。。。
  - Child1 DidUpdate
  - Child2 DidUpdate
- Father DidUpdate

**可见，父元素任意状态的更改都会让子组件随之更新**，而为了解决这个问题，React 也同样提出了 `shouldComponentUpdate` ，将重渲染的条件交由开发者来设立，以此减少 `re-render` 次数，优化性能。

### 怎么办？

在生命周期钩子函数中想必大家还记得那个名为 `shouldComponentUpdate` 的钩子嘛，为了最大程度上保证程序的正确执行，所有的 `shouldComponentUpdate` 都默认返回为 `true` 。即无论何时均重新渲染。所以为了优化逻辑我们往往会手动编辑该函数

```javascript
shouldComponentUpdate(nextProps, nextState){
    if(nextState.isChanged == this.state.isChanged){
        return false
    }
    return true
}
```

如此一来只有父组件的相应状态更改之后才会对子组件进行重新渲染（此处只是举一个例子，代码中 `isChanged` 与子组件并无联系，故实战中直接返回 `false` 即可）

![shouldComponentUpdate](https://pic.downk.cc/item/5e60bd9898271cb2b8990d18.png)

如此执行结果就如上图，完美解决重渲染问题咯~

### 其他导致的重渲染

​		上述方法虽然有用，但是由于数据类型我们将其限定为了字符串所以仍然有所局限。简而言之，JS 中基本类型的数据存放于栈内存中，可以直接进行比较，而引用类型则类似于 C 中的 “指针”，只保存引用，比较时即使两个对象的类型完全一样亦会导致 “===” 失效，因为**二者比较的判断方式是识别双方引用的是否为同一块内存地址**。



​		判断机制说完了，那么接下来说问题：如果将上述的 `isChanged` 转换为引用类型的数据，那么其变更时仍会触发子组件的 `re-render` 机制！所以在判断时更要求各位在相应的方面做到细致咯~。建议去了解一下**深拷贝**以及 `immutable.js`。当然，如果你足够懒那么也可以使用 `PureComponent`。当然这就是另一个故事啦~



​		最后附上[源码地址]( https://github.com/Burning-Shadow/life_circle-test )。本人使用了 CRA 进行架构，所以下载后需要安装相应环境才可以跑得起来哦

