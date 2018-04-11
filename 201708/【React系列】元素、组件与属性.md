### 元素（Elements）

  元素（Elements）是React应用中最小的构建部件，用于描述能够在屏幕上看到的内容，如： 
    ```
    
    const element = <h1> Hello World ! </h1>
    
    ```
  元素是构成组件的“材料”。
    
#### 渲染元素到DOM

  要渲染一个React元素到DOM上面需要有一个根节点，之后把他们传递给ReactDOM.render()方法。
    ```
    <div id="root"></div>
    
    const element = <h1>Hello World!</h1>;
    ReactDOM.render(
        element,
        document.getElementById("root")
    );
    ```
#### 更新已经渲染的元素
  React元素是不可突变（immutable）的。一旦创建了一个元素，就不能更改其子元素或者属性。只能创建一个新的元素，通过传入ReactDOM.render()方法进行更新。
  另外，React DOM在更新的时候，会将元素及其子元素与之前版本逐一逐级对比，只更新对应有变化的，并不会整体进行刷新。

### 组件（Components）
  组件是整个UI实现中划分出来的一个个独立可复用的小部件，每个部件间互不干涉，可以单独设计。
  组件可以通过接收属性（Props)输入从而返回对应的React元素，用以描述显示的内容。
#### 组件的分类
  - 函数式组件
    ```
    function Welcome(props) {
        return <h1> Hi, {props.name} ! </h1>;
    }
    ```

  - 类组件
    ```
    class Welcome extends React.Component {
      render() {
        return <h1>Hello, {this.props.name}</h1>;
      }
    }
    ```
#### 渲染组件

  以函数式组件为例：
  
    ```
    function Welcome(props) {
      return <h1>Hello, {props.name}</h1>;
    } 
    const element = <Welcome name="Sara" />;
    ReactDOM.render(
      element,
      document.getElementById('root')
    );
    ```    
  当 React 遇到一个代表用户定义组件的元素时，它将 JSX 属性以一个单独对象的形式传递给相应的组件。 我们将其称为 "props" 对象。简单顺序如下：

- 我们调用了 ReactDOM.render() 方法并向其中传入了 <Welcome name="Sara" /> 元素。
- React 调用 Welcome 组件，并向其中传入了 {name: 'Sara'} 作为 props 对象。
- Welcome 组件返回 `<h1>Hello, Sara</h1>`。
- React DOM 迅速更新 DOM ，使其显示为 `<h1>Hello, Sara</h1>`。

> ❤ 组件名称总是以大写字母开始。

 另外，组件间可以进行引用嵌套。组件可以在它们的输出中引用其它组件。

> ❤ 组件必须返回一个单独的根元素。这就是为什么我们添加一个 <div> 来包含所有 <Welcome /> 元素的原因。

### 属性（Props）

#### Props是只读的

> ❤ 所有 React 组件都必须是纯函数，并禁止修改其自身 props 。
