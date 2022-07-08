# JSX语法
__用法__：用JSX语法编写值或表达式，为元素element赋值。

__语法特征__：1.语法形式类似HTML，采用标签嵌套，标签可以为HTML5原生标签，也可以为自定义标签。但标签属性的写法与HTML5不同，采用驼峰命名的方式。
> <标签 属性名 = 值/{JS代码}>内容\[<\结束标签>\]

2.JSX表达式中可使用大括号嵌套JS代码。

3.多行JSX代码建议使用括号包裹。

# 元素 element
本质上是要在某DOM节点上呈现的内容。元素代表特定时刻的UI，创建后无法更改，更新UI的唯一方式是创建新的元素重新渲染。

__用法__ ：使用JSX语法定义并赋值元素，使用ReactDOM.render函数渲染到DOM节点。
```js
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

# 组件

__用法__ :定义函数/类组件，组件可作为JSX对象中的标签定义元素。元素中设置的属性可通过 pros.属性名 传入组件。

组件可以嵌套（函数返回值中的JSX中使用其他组件作为标签）。
__组件定义方式__ :
1. 函数组件（返回值为一个JSX对象）
```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
2. 类组件：继承React.Component，类内部定义rander函数，rander函数返回值为一个JSX对象。

    将函数组件返回值设置为null和将类组件的rander函数返回值设为null都可以阻止组件渲染但不影响其生命周期函数。

    类组件可以通过构造函数定义state对象，state相当于一个存储类组件私有变量的对象，props相当于一个存储传入参数的对象。

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};  //设置state对象
  }

  componentDidMount() {               //生命周期函数，组件挂载后执行。
    this.timerID = setInterval(
      () => this.tick(),
      1000                            //每1000ms执行一次
    );
  }

  componentWillUnmount() {            //生命周期函数，组件将被注销时执行
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
/*执行流程大致为 ：渲染Clock组件到根节点，this.state初始化 
                -> 执行componentDidMount(），调用tick，更新state，重新渲染组件
                -> 执行componentWillUnmount()，清除定时器
*/
```
__state__：（1）不能直接给this.state.变量名 赋值来更新state；需要使用setstate({})函数，函数参数为一个对象，对象中包含需要更新的键值对。（setstate是异步执行的）
```js
// Wrong React可能会把多个setState调用合并，
        //可能会造成连续的某几次累加，使用的setState是同一个值，从而结果也是同一个值。
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
（2）state的更新会被合并，其实就是可以使用setstate单独修改变量，其他变量不受影响。

（3）数据是向下流动的，其实就是state可以在类组件内部作为JSX的属性值传递到子组件中，子组件通过props接收。
## 受控组件

对于一些\<input>、 \<textarea> 和 \<select>等表单元素会根据用户的输入更新value等值。以value为例，React中通过使用setState函数将event.target.value赋值给value，再在组件内部使用this.state.value调用实现对value的控制，也可以通过state设置value初始值。简而言之就是对用户输入值的控制。

在受控组件上指定 value 的 prop 会阻止用户更改输入。如果你指定了 value，但输入仍可编辑，则可能是你意外地将value 设置为 undefined 或 null。

```js
ReactDOM.render(<input value="hi" />, mountNode); //用户不可编辑

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode); //用户可编辑
}, 1000);
```

# 事件处理
> <标签 事件名 = {事件处理函数名}> 内容 <\结束标签>

其中事件名采用小驼峰式命名，事件处理函数往往声明为类的方法，由于类中的方法默认不会绑定this，所以需要在构造函数中绑定this。

或者在回调中使用箭头函数
> <标签 事件名 = {箭头函数}> 内容 <\结束标签>

或者在回调中直接绑定
> <标签 事件名 = {this.函数名.bind(this,其他参数)}> 内容 <\结束标签>

向事件处理程序传递参数

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
//e 是一个合成事件，e 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递。
///事件处理函数不能通过返回 false 的方式阻止默认行为。你必须显式的使用 e.preventDefault。
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
//而通过 bind 的方式，事件对象以及更多的参数将会被隐式的进行传递。
```

# Key

- key 帮助 React 识别哪些元素改变了，应当给数组中的每一个元素赋予一个确定的标识。通常，我们使用数据中的 id 来作为元素的 key，不建议使用索引来用作 key 值。

- 元素的 key 只有放在就近的数组上下文中才有意义，也就是放在组成表单的最外层组件（这样才帮助React完整的识别元素变化）。

- 数组元素中使用的 key 在其兄弟节点之间应该是独一无二的。但不需要是全局唯一的。当生成两个不同的数组时，可以使用相同的 key 值。

# Hooks
在函数组件中使用state和生命周期函数。
其他自定义Hooks：https://ahooks.js.org/zh-CN/hooks/use-request/index
## useState
相当于函数组件中的state
```js

import React, { useState } from 'react'; //首先进行引入

[ver, setVer] = useState(0);
// ver为定义的变量
// setVar为更改变量值的函数
// useState传入参数作为变量的初始值

// 函数组件内部可以直接使用var和setVar
```


### useEffect
useEffect 会在每次渲染后都执行，相当于函数组件中的componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

__用法__ ：（1）effect接收一个函数为参数，React 会保存传递的函数，在执行 DOM 变化（挂载、更新、卸载）后调用它。（2） effect 传入的函数参数返回值为一个函数，React 将会在执行清除操作时调用返回值的函数：
- 可以在 effect 中直接访问 state 变量（或其他 props）

# pureComponent
当使用component时，父组件的state或prop更新时，无论子组件的state、prop是否更新，都会触发子组件的更新，这会形成很多没必要的render，浪费很多性能；pureComponent的优点在于：pureComponent在shouldComponentUpdate只进行浅层的比较，只要外层对象没变化，就不会触发render,减少了不必要的render，
