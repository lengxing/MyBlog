>解构赋值（destructuring assignment）语法是一个Javascript表达式，这种语法能够更方便的提取出 Object 或者 Array 中的数据。这种语法可以在接受提取的数据的地方使用，比如一个表达式的左边。有明确的语法模式来告诉我们如何使用这种语法提取需要的数据值。

## 数组的解构赋值

> 在ES6中允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

在之前，我们为变量赋值，只能是单一的直接进行赋值

```
var a = 1;
var b = 2;
var c = 3;
```

在ES6中通过解构，我们可以一次性对多个变量进行赋值。

```
let [a, b, c] = [1, 2, 3];
```

### 变量声明并赋值时的解构

```
let foo = ["one", "two", "three"];

let [one, two, three] = foo;
console.log(one); // "one"
console.log(two); // "two"
console.log(three); // "three"
```

### 变量先声明后赋值时的解构

```
let a, b;

[a, b] = [1, 2];
console.log(a); // 1
console.log(b); // 2
```

#### 不完全解构

即等号左边的模式只匹配一部分等号右边的数组，这种情况下依然能够解构成功。

```
let [x, y] = [1, 2, 3];
x // 1
y // 2
let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
c // 4
```

> **※ 如果等号右边不是数组，则解构将会报错**

### 默认值

为了防止从数组中取出一个值为undefined的对象，解构赋值允许指定默认值

```
let a, b;

[a=5, b=7] = [1];
console.log(a); // 1
console.log(b); // 7

[x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

> **※注意，ES6内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。**

```
let [x = 1] = [undefined];
x // 1
var [x = 1] = [null];
x // null
```

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

```
function f() {
  return 2;
}

let [x = f()] = [1]

x // 1
```

### 交换变量

没有解构赋值的情况下，交换两个变量需要一个临时变量。在一个解构表达式中可以直接交换两个变量的值。

```
let a = 1;
let b = 3;

[a, b] = [b, a];
console.log(a); // 3
console.log(b); // 1
```

### 解析一个从函数返回的数组

从一个函数返回一个数组是十分常见的情况.。解构使得处理返回值为数组时更加方便。
在下面例子中，[1, 2] 作为函数的 f() 的输出值，可以使用解构用一句话完成解析。

```
function f() {
  return [1, 2];
}

let a, b; 
[a, b] = f(); 
console.log(a); // 1
console.log(b); // 2
```

### 忽略某些返回值

当在处理一些数组时，其中一些值不是我们需要的，我们可以进行忽略。

```
function f() {
  return [1, 2, 3];
}

let [a, , b] = f();
console.log(a); // 1
console.log(b); // 3
```

### 将剩余数组赋值给一个变量

可以通过rest参数将一个数组后面剩余的一部分作为数组赋值给一个变量

```
let [a, ...b] = [1, 2, 3];
console.log(a); // 1
console.log(b); // [2, 3]
```

### 用正则表达式匹配提取值

用正则表达式方法exec()匹配字符串会返回一个数组，该数组第一个值是完全匹配正则表达式的字符串，然后的值是匹配正则表达式括号内内容部分。解构赋值允许你轻易地提取出需要的部分，忽略完全匹配的字符串——如果不需要的话。

```
let url = "https://developer.mozilla.org/en-US/Web/JavaScript";

let parsedURL = /^(\w+)\:\/\/([^\/]+)\/(.*)$/.exec(url);
console.log(parsedURL); // ["https://developer.mozilla.org/en-US/Web/JavaScript", "https", "developer.mozilla.org", "en-US/Web/JavaScript"]

let [, protocol, fullhost, fullpath] = parsedURL;

console.log(protocol); // "https"
```

## 对象的解构赋值

解构不仅可以用于数组，还可以用于对象。

### 基本赋值

```
let o = {p: 42, q: true};
let {p, q} = o;

console.log(p); // 42
console.log(q); // true
```

### 无声明赋值

```
let a, b;

({a, b} = {a: 1, b: 2});
```

> ※ 赋值语句周围的( .. ) 是使用对象字面解构赋值时不需要声明的语法。{a, b} = {a: 1, b: 2}不是有效的独立语法，因为左边的{a, b}被认为是一个块而不是对象字面量。然而，({a, b} = {a: 1, b: 2})是有效的，正如 var {a, b} = {a: 1, b: 2}

> 注意：你的( .. ) 表达式需要一个分号在它前面，否则它也许会被当成上一行中的函数来执行。

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

```
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```

### 给新的变量名赋值

可以从一个对象中提取变量并赋值给和对象属性名不同的新的变量名。

```
let o = {p: 42, q: true};
let {p: foo, q: bar} = o;
 
console.log(foo); // 42 
console.log(bar); // true
```
这实际上说明，对象的解构赋值是下面形式的简写

```
let { p: foo, q: bar } = { p: 42, q: true };
```

也就是说，对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。

```
let { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
```

foo是匹配的模式，baz才是变量。真正被赋值的是变量baz，而不是模式foo。

### 默认值

变量可以先赋予默认值。当要提取的对象没有对应的属性，变量就被赋予默认值。

```
let {a = 10, b = 5} = {a: 3};

console.log(a); // 3
console.log(b); // 5
```

#### 给新的变量命名并提供默认值

一个属性可以是
 - 1）从一个对象解构，并分配给一个不同名称的变量，
 - 2）分配一个默认值，以防未解构的值是undefined。

```
let {a:aa = 10, b:bb = 5} = {a: 3};

console.log(aa); // 3
console.log(bb); // 5
```

### 嵌套结构对象的解构

```
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```

注意，这时p是模式，不是变量，因此不会被赋值。如果p也要作为变量赋值，可以写成下面这样。

```
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]
```

### 对象方法赋值

对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。

```
let { log, sin, cos } = Math;
```

上面代码将Math对象的对数、正弦、余弦三个方法，赋值到对应的变量上，使用起来就会方便很多。

## 字符串的解构赋值

前面讲了数组可以进行解构赋值，同样的，我们可以将字符串理解成特殊的数组，所以字符串也可以进行解构赋值

```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。

```
let {length : len} = 'hello';
len // 5
```

## 数值和布尔值的解构赋值

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。对于undefined和null无法转为对象，则它们进行解构赋值，都会报错。

```
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true

let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 函数参数的解构赋值

```
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

上面代码中，函数add的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量x和y。对于函数内部的代码来说，它们能感受到的参数就是x和y。

函数参数的解构也可以使用默认值。

```
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

> 结语：本次主要针对ES6新增的解构赋值的知识进行了回顾，从上面的一些代码段中不难发现，解构赋值能够将我们原本需要相对复杂处理的逻辑简单化，如交换赋值等。更多的使用方法也在等着我们慢慢的去发掘与应用。
