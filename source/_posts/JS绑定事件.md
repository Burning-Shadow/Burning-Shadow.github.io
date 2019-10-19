---
title: JS绑定事件
date: 2019-09-24 11:41:07
categories: Tips
tags:
 - 绑定事件
---

​		本文用以记录一些 JS 绑定事件及相应的事件对象，以供查阅

<!--more-->

## 绑定事件

### 触摸类

- onTouchCancel
- onTouchEnd
- onTouchMove
- onTouchStart

### 键盘类

- onKeyDown
- onKeyUp
- onKeyPress

### 剪切

- onCopy
- onCut
- onPaste

### 表单

- onChange
- onInput
- onSubmit

### 焦点

- onFocus
- onBlur

### UI元素

- onScroll

### 滚动

- onWheel

### 鼠标

#### 点击/焦点事件

- onClick
- onContextMenu：右键（上下文菜单），一般用来设置右键自定义事件显示自己希望显示的东西 
- onDoubleClick
- onMouseDown
- onMouseEnter
- onMouseLeave
- onMouseMove
- onMouseOut
- onMouseOver
- onMouseUp

#### 拖拽事件

- onDrop
- onDrag
- onDragEnd
- onDragEnter
- onDragExit
- onDragLeave
- onDragOver
- onDragStart

## 事件对象

### 通用属性

- boolean bubbles：事件是否可冒泡
- boolean cancelable：事件是否可以取消
- DOMEventTarget currentTarget：与 target 类似，但是事件是可以冒泡的
- boolean defaultPrevented：事件是否禁止了默认行为
- number eventPhase：事件所处的阶段
- boolean isTrusted：事件是否可信（用户操作的为可信，通过 JS 脚本触发的为不可信事件）
- DOMEvent nativeEvent：获取原生的、浏览器封装的事件对象
- void preventDefault()：对应 boolean defaultPrevented，调用此函数就会禁止默认行为
- void stopPropagation()：对应 boolean bubbles，调用此函数会禁止冒泡
- DOMEventTarget target：reactJS 会将 target 抽象封装，通过此 API 获取到的并非原生的 target
- number timeStamp：事件触发的时间
- string type：事件的类型

### 剪切事件

- DOMDataTransfer clipboardData：剪切、复制数据的值

### 键盘事件

- boolean altKey：是否按下 alt 键
- Number charCode：获取按下键值的<i style="color: red">字符编码</i>
- boolean ctrlKey：是否按下 ctrl 键
- function getModifierState(key)：是否按下辅助按键
- String key：按下的键
- Number keyCode：有些按键对应的并不是字符，故需要拿到他的 <i style="color: red">keyCode</i> 进行判断
- String locale：本地化字符串
- Number location：位置
- boolean metaKey：windows 系统下的 win 键
- boolean repeat：按键是否重复
- boolean shiftKey：是否按下 shift
- Number which：经过通用化的 charCode 和 keyCode

### 焦点

- DOMEventTarget relatedTarget：焦点源头 / 去处

### 鼠标事件

- boolean altKey
- Number button
- Number buttons
- Number clientX：参照系为**浏览器窗口**左上角 
- Number clientY：参照系为**浏览器窗口**左上角 
- boolean ctrlKey
- function getModifierState(key)
- boolean metaKey
- Number pageX：参照系为 **HTML 页面**左上角
- Number pageY：参照系为 **HTML 页面**左上角
- DOMEventTarget relatedTarget
- Number screenX：参照系为**显示器**左上角
- Number screenY：参照系为**显示器**左上角
- boolean shiftKey

### 触摸事件

- boolean altKey
- DOMTouchList changedTouches：
- boolean ctrlKey
- function getModifierState(key)
- boolean metaKey
- boolean shiftKey
- DOMTouchList targetTouches
- DOMTouchList touches

### UI 元素

- Number detail：鼠标滚动的距离
- DOMAbstractView view：鼠标滚动的视窗

### 鼠标滚轮滚动

- Number deltaMode：滚动的距离
- Number deltaX
- Number deltaY
- Number deltaZ

 