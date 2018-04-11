>本篇我们来认识一下react中的列表（Lists）和键（Keys）。首先让我们回顾一下在 JavaScript 中如何转换列表。

我们知道，在JavaScript中map() 方法返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。map() 方法按照原始数组元素顺序依次处理元素。如：
```
/*返回一个数组，数组中元素为原始数组的平方根*/
var numbers = [4, 9, 16, 25];

function myFunction() {
    x = document.getElementById("demo")
    x.innerHTML = numbers.map(Math.sqrt);
}
```
在 React 中，转换数组为 元素列表 的方式，和上述方法基本相同。

## 列表（Lists）

### 多组件渲染

可以创建元素集合，并用一对大括号 {} 在 JSX 中直接将其引用即可。

下面，我们用 JavaScript 的 map() 函数将 numbers 数组循环处理。对于每一项，我们返回一个 <li> 元素。最终，我们将结果元素数组分配给 listItems：
```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```
把整个 listItems 数组包含到一个 <ul> 元素，并渲染到 DOM：
```
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```
这样就可以创建出一个列表元素了，包含5个li元素的列表。

### 基本列表组件

上面的例子只是简单的元素列表的创建过程，而在React中，往往我们使用的是组件列表的创建，我们可以简单改造上面的例子。
```
function ListItem(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <ListItem numbers={numbers} />,
  document.getElementById('root')
);
```
这样就转化成了基本的React的组件形式了。
> ❤当运行上述代码的时候，将会收到一个警告：![图片描述][1]这个警告在非 min 版本的就可以看到。当创建元素列表时，“key” 是一个你需要包含的特殊字符串属性，下面会有说明。


  [1]: https://segmentfault.com/img/bVT4IZ?w=560&h=77

## 键（Keys）

上面的例子中出现的警告告诉我们数组中的每一个元素都需要一个“key”（键）来进行标识。键(Keys) 帮助 React 标识哪个项被修改、添加或者移除了。有点类似于元素的唯一标识，相信会比较好理解。

挑选 key 最好的方式是使用一个在它的**同辈元素**中不重复的标识字符串。多数情况你可以使用数据中的 IDs 作为 keys：
```
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

当要渲染的列表项中没有稳定的 IDs 时，你可以使用数据项的索引值作为 key 的最后选择：
```
const todoItems = todos.map((todo, index) =>
  // 只是当列表元素没有稳定的ids时使用索引index
  <li key={index}>
    {todo.text}
  </li>
);
```
> ❤如果列表项可能被重新排序时，我们不建议使用索引作为 keys，因为这导致一定的性能问题，会很慢。

### 使用 keys 提取组件

keys 只在数组的上下文中存在意义。例如，如果你提取 一个 ListItem 组件，应该把 key 放置在数组处理的 <ListItem /> 元素中，不能放在 ListItem 组件自身中的 <li> 根元素上。

**例子：错误的 key 用法**
```
function ListItem(props) {
  const value = props.value;
  return (
    // 错误！不需要在这里指定 key：
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 错误！key 应该在这里指定：
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**例子：正确的 key 用法**
```
function ListItem(props) {
  // 正确！这里不需要指定 key ：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在这里被指定
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
> ❤一个好的经验准则是元素中调用 map() 需要 keys 。因为列表都会通过map()方法进行实现。

### keys 在同辈元素中必须是唯一的

在数组中使用的 keys 必须在它们的**同辈之间唯一**。然而它们并不需要全局唯一。我们可以在操作两个不同数组的时候使用相同的 keys。也就是说，在一个列表中选取keys时，这个属性需要在数组中的元素中是唯一的，不会重复，具有**唯一性**。这也和我们常规见到的key（键值、主键）的概念是一致的。

### keys的内部性

键是React的一个内部映射，但其不会传递给组件的内部。如果你需要在组件中使用相同的值，可以明确使用一个不同名字的 prop 传入。
```
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```
如上面的例子，在组件Post中可以通过props.id获取值，但是无法获取props.key的值。

## 在 JSX 中嵌入 map()

JSX允许在大括号中嵌入任何表达式，因此可以 内联 map()：
```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```
有时这可以产生清晰的代码，但是这个风格也可能被滥用。就像在 JavaScript 中，是否有必要提取一个变量以提高程序的可读性，这取决于你。但是记住，如果 map() 体中有太多嵌套，可能是提取组件的好时机。总之要在适当的时机使用组件，这样就能使你的结构更加的清晰。
