# 受控组件

## 表单

HTML 表单用于搜集不同类型的用户输入。

### <form> 元素
`<form>` 元素定义 HTML 表单：

```
<form>
  form elements
</form>
```
### 表单元素

表单元素指的是不同类型的 input 元素、复选框、单选按钮、提交按钮等等。
当用户提交表单时浏览器会打开一个新页面，如果你希望 React 中保持这个行为，也可以工作。但是多数情况下，用一个处理表单提交并访问用户输入到表单中的数据的 JavaScript 函数也很方便。实现这一点的标准方法是使用一种称为“受控组件(controlled components)”的技术。

## 受控组件(Controlled Components)

在 HTML 中，表单元素如 `<input>`，`<textarea>` 和 `<select>` 表单元素通常保持自己的状态，并根据用户输入进行更新。而在 React 中，可变状态一般保存在组件的 state(状态) 属性中，并且只能通过 setState() 更新。
我们可以通过使 React 的 state 成为 “单一数据源原则” 来结合这两个形式。然后渲染表单的 React 组件也可以控制在用户输入之后的行为。这种形式，其值由 React 控制的输入表单元素称为“受控组件”。
一个常规的受控组件的形式例子：


```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          /*① 设置表单元素的value属性之后，其显示值将由this.state.value决定，以满足React状态的同一数据理念。*/
          /*② 每次键盘敲击之后会执行handleChange方法以更新React状态，显示值也将随着用户的输入改变。*/
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

由于 value 属性设置在我们的表单元素上，显示的值总是 this.state.value，以满足 state 状态的同一数据理念。由于 handleChange 在每次敲击键盘时运行，以更新 React state(状态)，显示的值将更新为用户的输入。

对于受控组件来说，每一次 state(状态) 变化都会伴有相关联的处理函数。这使得可以直接修改或验证用户的输入。比如，如果我们希望强制 name 的输入都是大写字母，可以这样来写 handleChange 方法：

```
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

### 受控表单的表单元素

上面例子中是以input为例来写的，其实其他的表单元素基本形式是差不多的。

#### textare

```
<textarea value={this.state.value} onChange={this.handleChange} />
```

#### select 

```
<select value={this.state.value} onChange={this.handleChange}>
……
</select>
```

### 处理多个输入元素

当您需要处理多个受控的 表单元素时，您可以为每个元素添加一个 name 属性，并且让处理函数根据 event.target.name 的值来选择要做什么。

由于 setState() 自动将部分状态合并到当前状态，所以我们只需要调用更改的部分即可。

## 受控组件的替代方案

有时使用受控组件有些乏味，因为你需要为每一个可更改的数据提供事件处理器，并通过 React 组件管理所有输入状态。当你将已经存在的代码转换为 React 时，或将 React 应用程序与非 React 库集成时，这可能变得特别烦人。在这些情况下，您可能需要使用不受控的组件，用于实现输入表单的替代技术。

# 不受控组件

在大多数情况下，我们推荐使用受控组件来实现表单。在受控组件中，表单数据由 React 组件负责处理。另外一个选择是不受控组件，其表单数据由 DOM 元素本身处理。

要编写一个未控制组件，你可以使用一个 ref 来从 DOM 获得 表单值，而不是为每个状态更新编写一个事件处理程序。

例如，在不受控组件中，以下代码接受一个单独的名字 :

```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

# 受控组件与不受控组件的选取

因为不受控组件的数据来源是 DOM 元素，当使用不受控组件时很容易实现 React 代码与非 React 代码的集成。如果你希望的是快速开发、不要求代码质量，不受控组件可以一定程度上减少代码量。否则。你应该使用受控组件。
