>在上一篇中写过，组件可以分为函数式组件和类组件，并且更新组件的方法也给出了通过传入ReactDOM.render()方法进行更新。但是这种方式并不能很好地进行封装成独立功能的组件，一些操作会由外部进行控制。而我们理想中的组件应该是一个功能独立的个体，只是不同场合不同的数据才会出现不同。
而这就就关联到了我们这次的主题---状态（State）

## 状态（State）

### 什么是状态

状态（state） 和 属性（props） 类似，都是一个组件所需要的一些数据集合，但是它是私有的，并且由组件本身完全控制，可以认为它是组件的“**私有属性（或者是局部属性）**”。

### 函数式组件转化为类组件

如果想要使用状态（State）的话，则需要我们在构建组件的时候是要以类组件为形式的。
针对直接以类组件形式构造的组件不需要变化，那么针对函数式组件该如何转化呢？

- 创建一个继承自 React.Component 类的 ES6 class 同名类。
- 添加一个名为 render() 的空方法。
- 把原函数中的所有内容移至 render() 中。
- 在 render() 方法中使用 this.props 替代 props。
- 删除保留的空函数声明。

例如：

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
上面示例就是一个简单的类组件Clock。

### 类组件中添加State

 1) 替换 render() 方法中的 this.props.date 为 this.state.date：

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
 2) 添加一个 类构造函数(class constructor) 初始化 this.state:
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
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
```

> ❤ 注意我们如何将 props 传递给基础构造函数：

```
constructor(props) {
  super(props); ***
  this.state = {date: new Date()};
}
```

 3) 移除 <Clock /> 元素中的 date 属性：
```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
通过上面的步骤就可以实现添加State到类组件了，最终结果：

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
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
```

### 如何修改State

State的初始化和常规的赋值形式是一样的，但是如果我们需要更新相关数据的话，则需要通过setState方法进行修改，如：

```
this.setState({
  date: new Date()
});
```

### 正确地使用 State(状态)

关于如何正确使用State以及setState() 有三个方面需要注意：

#### 不要直接修改 state(状态)
正如上面提到的如何修改State那样，在初始化后，如果修改的话只能通过setState方法进行修改。
#### state(状态) 更新可能是异步的
React 为了优化性能，有可能会将多个 setState() 调用合并为一次更新。

因为 this.props 和 this.state 可能是异步更新的，你不能依赖他们的值计算下一个state(状态)。
如：
```
// 错误
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
要弥补这个问题，使用另一种 setState() 的形式，它接受一个函数而不是一个对象。这个函数将接收前一个状态作为第一个参数，应用更新时的 props 作为第二个参数：
```
// 正确
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```
#### state(状态)更新会被合并
当你调用 setState()， React 将合并你提供的对象到当前的状态中。所以当State是一个多键值的结构时，可以单独更新其中的一个，此时会进行“差分”更新，不会影响其他的属性值。
## 生命周期方法

### 什么是生命周期

> 生命周期就是指一个对象的生老病死。

而组件的生命周期则是这个组件从创建到销毁的一个过程。在这个过程中会有不同的阶段，从而会产生一些对应的生命周期函数来供我们使用，以便能够进行一些渲染、更新等处理。

可以简单分为几个阶段：

+ 初始化阶段
    - getDefaultProps:获取实例的默认属性(即使没有生成实例，组件的第一个实例被初始化CreateClass的时候调用，只调用一次,)
    - getInitialState:获取每个实例的初始化状态（每个实例自己维护）
    - componentWillMount：组件即将被装载、渲染到页面上（render之前最好一次修改状态的机会）
    - render:组件在这里生成虚拟的DOM节点（只能访问this.props和this.state；只有一个顶层组件，也就是说render返回值值职能是一个组件；不允许修改状态和DOM输出）
    - componentDidMount:组件真正在被装载之后，可以修改DOM
+ 运行中状态
    - componentWillReceiveProps:组件将要接收到属性的时候调用（赶在父组件修改真正发生之前,可以修改属性和状态）
    - shouldComponentUpdate:组件接受到新属性或者新状态的时候（可以返回false，接收数据后不更新，阻止render调用，后面的函数不会被继续执行了）
    - componentWillUpdate:不能修改属性和状态
    - componentDidUpdate:可以修改DOM
+ 销毁阶段
    - componentWillUnmount:开发者需要来销毁（组件真正删除之前调用，比如计时器和事件监听器）
    
    
## 数据向下流动

无论作为父组件还是子组件，它都无法获悉一个组件是否有状态，同时也不需要关心另一个组件是定义为函数组件还是类组件。

这就是 state(状态) 经常被称为 本地状态 或 封装状态的原因。 它不能被拥有并设置它的组件 以外的任何组件访问。

一个组件可以选择将 state(状态) 向下传递，作为其子组件的 props(属性)：
```
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```
这通常称为一个“从上到下”，或者“单向”的数据流。任何 state(状态) 始终由某个特定组件所有，并且从该 state(状态) 导出的任何数据 或 UI 只能影响树中 “下方” 的组件。

如果把组件树想像为 props(属性) 的瀑布，所有组件的 state(状态) 就如同一个额外的水源汇入主流，且只能随着主流的方向向下流动。

## 结语

在 React 应用中，一个组件是否是有状态或者无状态的，被认为是组件的一个实现细节，随着时间推移可能发生改变。你可以在有状态的组件中使用无状态组件，反之亦然。

