---
title: React 学习笔记
date: 2017-09-06 19:20:06
categories:
  - 学无止境
tags:
  - Frontend
  - React
---

这篇文章是我在学习 React 的过程中记录下的笔记，学习主要是跟着官网的文档了解 React 的一些基本用法。我记录下了在这个过程中我遇到过的问题以及我觉得重要的地方。

<!-- more -->

<img src="/images/react.png" class="full-image" alt="React" title="React"/>

## 安装

通过引入脚本文件的方式安装时，需要先引入 React，再引入 ReactDOM。

## JSX

React 中使用了叫做 JSX 的一种 JavaScript 语法扩展，感觉和模板语言很类似，通过 JSX 可以告别传统的通过拼接字符串或者是数据中 push 标签的方式来构建页面，更加的直观、优雅。

在 JSX 中可以直接像书写 HTML 一样写页面，可以任意地使用 JavaScript 表达式，表达式需要用 `{}` 括起来。

推荐在 JSX 代码的外面扩上一个小括号，这样可以防止自动插入分号（简称 ASI）的问题。下面是一个典型的例子：

```JavaScript
function asi(){
  return
    'Test ASI';
}

asi();  // undefined
```

JSX 在编译之后也会转化成 JavaScript 对象，因此可以直接将其赋值给变量或者作为函数的参数和返回值。

React DOM 使用 `camelCase` 驼峰命名来定义属性的名称，而不是使用 HTML 的属性名称。例如：使用 `className` 指代 HTML `class` 属性。

可以使用 JSX 中的点表示法来引用 React 组件，例如：

```JavaScript
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

如果你没有给属性传值，它默认为 true。例如：

```HTML
<MyTextBox autocomplete />

<!--
  不推荐这样使用，因为它会与 ES6 对象简洁表示法 混淆。比如 {foo} 是 {foo: foo} 的简写
-->
<MyTextBox autocomplete={true} />
```

## 元素

元素是构成 React 应用的最小单位。React 元素在写法上看上去像是 HTML 标签，但是不同于浏览器 DOM 元素，实际上只是一个 JavaScript 对象。

React 元素都是不可变的，当元素被渲染之后就无法再改变元素的内容或者属性，更新的唯一方法就是渲染一个新的元素。React 中使用了虚拟 DOM，在重新渲染的时候不会更新全部的元素，而是会先比较前后差异，只更新改变了的部分。

### 条件渲染

React 中的条件渲染核心就是通过控制组件 `render` 的返回值来实现的，而不是像其他框架有 `if` 指令。

render 中 `return null` 可以隐藏组件，阻止组件渲染，这并不会影响该组件生命周期方法的回调。

### 循环渲染

React 可以直接渲染出由 React 元素构成的集合。例如：

```JavaScript
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);

ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('body')
);
```

当使用元素集合时，应当给数组中的每一个元素赋予一个确定的标识，称为 “key”。Keys 可以在 DOM 中的某些元素被增加或删除的时候帮助 React 识别数组中哪些元素发生了变化。

**注意**：
  - key 不会传递给组件，即通过 `props.key` 无法获取到 key。
  - 数组元素中使用的 key 在其兄弟之间应该是独一无二的，但不必全局唯一。

## 组件及属性

组件就像是一个函数，接受一个被成为组件属性的输入，返回一个 React 元素。例如：

```JavaScript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

上面的这个函数就是一个 React 组件，我们也可以使用 ES6 class 来定义组件：

```JavaScript
class Welcome extends React.Component {
  render(){
    return <h1>Hello, {props.name}</h1>;
  }
}
```

定义了组件后，可以直接像使用 HTML 标签一样使用该组件。同时，使用组件时在组件上定义的所有属性会通过一个 `props` 对象传递给该组件。

**注意**：
  - 组件名称必须以大写字母开头
  - 组件的返回值只能有一个根元素
  - 不能修改组件的 props

在我看来，一个组件应该是一个具有特定功能、低耦合、可单独工作的单元。组件应该具有较好的通用性，与上下文的不应该过度的关联。

### 受控组件

值由 React 控制的输入表单元素称为“受控组件”。例如：

```JavaScript
class Input extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      value: ''
    };

    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.setState({
      value: e.target.value.toLocaleUpperCase()
    });
  }

  render() {
    return (
      <input value={this.state.value} onChange={this.handleChange} />
    );
  }
}

ReactDOM.render(
  <Input />,
  document.querySelector('body')
);

```

### 组件组合

组件 JSX 标签内的任何内容都将通过 **children** 属性传入。

可以将一个组件作为另一个组件的属性传入，完全各种组件的组合，例如：

```JavaScript
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

组件可以接受任意元素，包括基本数据类型、React 元素或函数。

如果要在组件之间复用 UI 无关的功能，我们建议将其提取到单独的 JavaScript 模块中。这样可以在不对组件进行扩展的前提下导入并使用该函数、对象或类。

### 属性类型检查

1. 使用 Flow 或者 TypeScript这样的 JavsScript 扩展来对整个应用程序进行类型检查。
2. 使用 `prop-types` 库。

## 生命周期

### 装载

- constructor：构造函数，在创建组件时调用一次。
- componentWillMount：组件被挂载在页面之前调用。
- componentDidMount：组件挂载在页面之后调用。

### 更新

- componentWillReceiveProps：组件接收到新属性前调用。
- shouldComponentUpdate：状态改变时调用，默认返回 true ，需要重新渲染。可以再这里做判断返回 false 阻止渲染。
- componentWillUpdate：shouldComponentUpdate 返回 rue 或者调用forceUpdate之后，在渲染前被立即调用。初始化渲染时不会调用。
- componentDidUpdate：在更新发生后立即被调用，不会在初始化渲染时调用。

### 卸载

- componentWillUnmount：当一个组件从 DOM 中被移除时，该方法被调用。一般在 componentDidMount 里面注册的事件需要在这里删除。

## 状态

状态与属性十分相似，但是状态是私有的，完全由组件自己控制。

对于组件中不需要输出到页面上的属性，不要将属性添加到 props 或者 state 上面，而是直接作为组件的属性添加到 `this` 上。

**注意**：
  - 不要直接的修改状态，而是使用 `setState` 方法
  - 状态更新可能是异步的
  - 调用 `setState()` 时，React 会将对象合并到当前状态。
  - 向下的单项数据流，属性只能由父组件到子组件。这点和 Polymer 不一样。

### 状态提升

在React中，状态分享是通过将 state 数据提升至离需要这些数据的组件最近的父组件来完成的。这就是所谓的状态提升。

通过状态提升，可以进行子组件间的通信。我们可以把子组件看作是一个 HTML 元素，这就类似于前面提到的受控组件：由父组件进行状态管理，将属性通过 props 传递给需要通信的子组件，这样子组件就可以拥有一致的数据，因为属性通过 props 传递给子组件，子组件是不能直接进行修改的，所以同时传递一个处理函数给相应的子组件。

## 事件

React 组件上绑定的写法与原生写法类似，只是事件名称使用驼峰命名，并将一个函数传入，而原生写法使用的是事件名小写并传入一个字符串。如下：

```HTML
// 原生写法
<button onclick="doClick()">
  Click Me!
</button>

// React 写法
<button onClick={doClick}>
  Click Me!
</button>
```

React 在事件处理中阻止浏览器默认行为不能使用 `return false`，只能使用 `e.preventDefault()`。

使用 ES6 class 语法来定义一个组件的时候，事件处理器会成为类的一个方法。绑定事件时需要注意 this 的指向。推荐在构造函数中绑定 this 或使用属性初始化器语法。如下：

```JavaScript
class ToggleButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isToggleOn: true
    };

    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    /*
     * 状态的更新可能是异步的，因此最好不要直接使用当前状态来计算下一个状态。
     * setState 支持传入一个函数，函数中会将上次状态作为参数传入，推荐使用此种方式。
     *
     * this.setState({
     *    isToggleOn: !this.state.isToggleOn
     * });
     */

    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      /*
       * 绑定事件时，注意 this 的指向，需要绑定 this：
       *    this.handClick.bind(this) 或 (e) => this.handleClick(e)
       *
       * 使用这种方式在 ToggleButton 渲染的时候都会创建一个新的函数，如果这个函数作为属性传入子组件，那么组件可能会进行额外的重新渲染。
       * 因此，推荐在构造函数中绑定 this 或使用属性初始化器语法
       */
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'On' : 'Off'}
      </button>
    );
  }
}

ReactDOM.render(
  <ToggleButton />,
  document.querySelector('body')
);
```

属性初始化器语法如下：

```JavaScript
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

## 其他

### Ref

通过 Ref 我们可以直接操作 DOM 元素或者子组件。例如：

```JavaScript
class AutoFocusTextInput extends React.Component {
  componentDidMount() {
    this.textInput.focus();
  }

  render() {
    return (
      <CustomTextInput
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

给 HTML 元素添加 ref 属性时，ref 回调接收了底层的 DOM 元素作为参数；给组件添加 ref 属性时，参数是组件的实例，**仅对 class 声明的 CustomTextInput 有效**。

想要在父组件内直接操作子组件的 DOM 元素，可以在子组件上添加 ref 属性，然后父组件把函数作为参数传入，例如：

```JavaScript
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

#### 非受控组件

 在受控组件中，表单数据由 React 组件处理。如果让表单数据由 DOM 处理时，替代方案为使用非受控组件。受控组件通过 ref 从 DOM 获取表单值；通过 defaultValue 属性指定默认值，而不是 value，checkbox 和 radio 则是 defaultChecked。
