> 最近在学习一些第三方代码，发现里面出现了Symbol字段，由于之前ES6系列梳理有个小暂停，所以本篇开始针对Symbol进行一下学习。

## JavaScript 数据类型

在ES6之前，我们所知道的JavaScript 数据类型有：

- Null
- Undefined
- 布尔值（Boolean）
- 字符串（String）
- 数值（Number）
- 对象（Object）
- 数组（Array）

## Symbol引入

<!-- more -->

在我们常规的开发过程中，特别是在多人协作过程中，总是不可避免的会出现互相冲突覆盖的情况。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法（mixin 模式），新方法的名字就有可能与现有方法产生冲突。此时我们可能会想，如果能有一个不被覆盖，自己独一无二的属性字段该多好啊。从而ES6中引入了Symbol这一个数据类型，来解决这种冲突问题。

Symbol()函数会返回symbol类型的值，该类型具有静态属性和静态方法。它的静态属性会暴露几个内建的成员对象；它的静态方法会暴露全局的symbol注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法："new Symbol()"。

每个从Symbol()返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

### 语法结构

![图片描述][1]

### 基本使用

直接使用Symbol()创建新的symbol类型，并用一个字符串（可省略）作为其描述。

```
let sym1 = Symbol();
let sym2 = Symbol('foo');
let sym3 = Symbol('foo');
```

上面的代码创建了三个新的symbol类型。Symbol函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。注意，Symbol("foo") 不会强制字符串 “foo” 成为一个 symbol类型。它每次都会创建一个新的 symbol类型：

```
Symbol("foo") === Symbol("foo"); // false
```

当使用new运算符的语法时，会抛出TypeError错误，这是因为生成的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

```
let sym = new Symbol(); // TypeError
```

如果 Symbol 的参数是一个对象，就会调用该对象的toString方法，将其转为字符串，然后才生成一个 Symbol 值。

```
const obj = {
  toString() {
    return 'abc';
  }
};
const sym = Symbol(obj);
sym // Symbol(abc)
```

> 注意，Symbol函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的。

```
// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2 // false

// 有参数的情况
let s1 = Symbol('foo');
let s2 = Symbol('foo');

s1 === s2 // false
```

Symbol 值不能与其他类型的值进行运算，会报错。

```
let sym = Symbol('My symbol');

"your symbol is " + sym
// TypeError: can't convert symbol to string
`your symbol is ${sym}`
// TypeError: can't convert symbol to string
```

但是，Symbol值可以显式转为字符串，或是布尔值，但是不能是数值。

```
let sym = Symbol('My symbol');

String(sym) // 'Symbol(My symbol)'
sym.toString() // 'Symbol(My symbol)'

Boolean(sym) // true
!sym  // false

if (sym) {
  // ...
}

Number(sym) // TypeError
sym + 2 // TypeError
```

## 作为属性名的Symbol

在前面我们也说到了，Symbol的出现就是解决覆盖冲突问题而产生的，它的主要作用也是被用作对象的属性名来使用，因为它是独一无二的，这样就不会出现同名的属性，也就不会出现覆盖的情况了。这对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖。

```
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```

> 注意，Symbol 值作为对象属性名时，不能用点运算符。

```
const mySymbol = Symbol();
const a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```

上面代码中，因为点运算符后面总是字符串，所以不会读取mySymbol作为标识名所指代的那个值，导致a的属性名实际上是一个字符串，而不是一个 Symbol 值。同理，在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。

```
let s = Symbol();

let obj = {
  [s]: function (arg) { ... }
};

obj[s](123);
```

Symbol 类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

```
const log = {};

log.levels = {
  DEBUG: Symbol('debug'),
  INFO: Symbol('info'),
  WARN: Symbol('warn')
};
console.log(log.levels.DEBUG, 'debug message');
console.log(log.levels.INFO, 'info message');

// 另一个例子
const COLOR_RED    = Symbol();
const COLOR_GREEN  = Symbol();

function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default:
      throw new Error('Undefined color');
    }
}
```

常量使用 Symbol 值最大的好处，就是其他任何值都不可能有相同的值了，因此可以保证上面的switch语句会按设计的方式工作。

### 实例：消除魔术字符串

> 魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。

```
function getArea(shape, options) {
  let area = 0;

  switch (shape) {
    case 'Triangle': // 魔术字符串
      area = .5 * options.width * options.height;
      break;
    /* ... more code ... */
  }

  return area;
}

getArea('Triangle', { width: 100, height: 100 }); // 魔术字符串
```

上面这类处理我们可能会感觉到很熟悉，这也是日常开发过程中非常常见的处理方式。而上面的字符串`Triangle`就是一个魔术字符串。

而消除魔术字符串的方法就是可以通过使用Symbol来把它变成一个常量

```
const shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  let area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```

这时我们可以发现`shapeType.triangle`的具体值我们并不关心，但它恰恰又能满足我们的需求。

> 还有一点需要注意，Symbol 值作为属性名时，该属性还是公开属性，不是私有属性。

## getOwnPropertySymbols遍历

Symbol 作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。

```
const obj = {};
let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```

## Reflect.ownKeys

Reflect.ownKeys方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。

```
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};

Reflect.ownKeys(obj)
//  ["enum", "nonEnum", Symbol(my_key)]
```

## Symbol.for()，Symbol.keyFor()

在上面使用`Symbol()`函数的语法，不会在我们整个代码库中创建出一个可用的全局的Symbol类型的变量，但是有时候我们希望能够重新使用同一个Symbol值。要创建跨文件可用的symbol，甚至跨域（每个都有它自己的全局作用域），我们可以使用`Symbol.for()` 方法和 `Symbol.keyFor()`方法从全局的Symbol注册表中创建和获取Symbol。

```
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');

s1 === s2 // true
```

Symbol()和Symbol.for()的区别是：`前者会被登记在全局环境中供搜索，后者不会。Symbol.for()不会每次调用就返回一个新的 Symbol 类型的值，而是会先检查给定的key是否已经存在，如果不存在才会新建一个值。比如，如果你调用Symbol.for("cat")30 次，每次都会返回同一个 Symbol 值，但是调用Symbol("cat")30 次，会返回 30 个不同的 Symbol 值`。

```
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```

Symbol.keyFor方法返回一个已登记的 Symbol 类型值的key。

```
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo", 已登记的Symbol值

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined, 未登记的Symbol值
```

> Symbol.for为 Symbol 值登记的名字，是全局环境的，可以在不同的 iframe 或 service worker 中取到同一个值

```
iframe = document.createElement('iframe');
iframe.src = String(window.location);
document.body.appendChild(iframe);

iframe.contentWindow.Symbol.for('foo') === Symbol.for('foo')
// true
```

## 小结

本次主要针对ES6中新增的数据类型Symbol进行了梳理，关于内置的Symbol值的相关内容，在后续使用到的过程中再进行分享，基本的一些使用大家可以参照[ECMAScript 6入门中Symbol中的讲解](http://es6.ruanyifeng.com/#docs/symbol)。


  [1]: https://segmentfault.com/img/bVbcb22?w=1086&h=282
