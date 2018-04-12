## 箭头函数

在之前ES5的版本中，我们定义一个函数的形式如下：
```
function a() {
  // do something……
}
```
但是在ES6中，则新增了箭头函数的方式，ES6中允许使用“箭头”（`=>`）来定义函数。
```
() => {
  // do something……
}
```
其中`()`中代表的是参数部门，`{}`中是函数体部分。
如果箭头函数不需要参数或者需要多个参数时，需要使用一个`()`来包裹，当只有一个参数时，可以省略`()`。
```
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};

var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```
当我们在函数中返回一个对象时，之前的写法是：
```
var f = function(id) {
  return {
    id: id,
    name: "name"
  }
}
```
此时如果改为箭头函数的方式来写时，需要注意的是，由于在ES6中`{}`代表了一个代码块，所以在返回一个对象的时候，需要在对象`{}`外面使用括号包起来：
```
let f = id => ({id: id, name: "name"})
```
如果箭头函数只有一行语句，且不需要返回值，可以采用下面的写法，就不用写大括号了。
```
let fn = () => void doesNotReturn();
```
不难发现，ES6中的箭头函数相对ES5中的函数的写法更加的简洁方便
```
const isEven = n => n % 2 == 0;
const square = n => n * n;
```
如上面只是简单地两行，就定义了两个常用的工具函数。按照之前的写法则会书写多行。特别是在针对回调函数的时候，更能够起到简化的作用：
```
// ES5的写法
var evens = [1,2,3,4,5];
var odds = evens.map(function(v){
  return v + 1
})

// ES6的写法

let evens = [1,2,3,4,5];
let odds = evens.map(v => v + 1)
```

### 箭头函数的this的绑定
箭头函数和普通函数的另一个区别，在于this的绑定。例如：

```
var factory = function() {
  this.a = "a";
  this.b = "b";
  this.c = {
    a : "a+",
    b : function() {
      return this.a
    }
  }
}
console.log(new factory().c.b()) // "a+"
```
通过上面的代码执行可以看出，得到的结果是“a+”，从而可以看出`this.a`指向的是`c.a`。有人曾总结过`this的指向是该函数被调用的对象`。此处b是c调用的，所以指向c中的a。现在修改为ES6的箭头函数的写法：
```
let factory = function() {
  this.a = "a";
  this.b = "b";
  this.c = {
    a : "a+",
    b : () => {
      return this.a
    }
  }
}
console.log(new factory().c.b()) // "a"
```
执行上面的代码发现，输出的结果是`“a”`，这又是为什么呢？请看下图所示
![图片描述][1] 

### 注意点
箭头函数有几个使用注意点
- （1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。这个我们在上面的例子中也已经见到过了，再举一个比较常见的例子：
在涉及到setTimeout和setInterval时，我们之前的方法是在调用前将函数体内的this通过赋值给一个变量的方法，使得能够在setTimeout和setInterval的回调函数中使用函数体的this
```
function foo() {
  this.id = "111"; 
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```
但是在ES6中在使用箭头函数之后则不再需要进行赋值替换而能直接使用函数体中的this
```
function foo() {
  this.id = "111"; 
  setTimeout(()=>{
  	console.log(this.id);
  }, 100);
}
```
this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。正是因为它没有this，所以也就不能用作构造函数。

- （2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
- （3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替（下面会讲解rest参数）。
- （4）不可以使用yield命令，因此箭头函数不能用作 Generator 函数，这部分在后面具体遇到时再进行说明。

## 函数的默认参数

对于带有参数的函数，我们往往在一些情况下需要给出一个默认值的设置，当不传入参数时，也能确保函数能够正常执行。在ES5中，我们的写法往往如下：
```
function f(x, y, z) {
  if(y === undefined) {
    y = 7;
  }
  if(z === undefined) {
    z = 5;
  }
  return x + y + z
}
console.log(f(1,3));
```
在ES6中允许为函数的参数设置默认值，即直接写在参数定义的后面。
```
function f(x, y = 7, z = 5) {
  return x + y + z
}
console.log(f(1,3));
```
这段代码和上面的执行结果是一致的，可以看出函数默认值的写法使得函数定义更加的简洁清晰。
ES6 的写法还有两个好处：
- 首先，阅读代码的人，可以立刻意识到哪些参数是可以省略的，不用查看函数体或文档；
- 其次，有利于将来的代码优化，即使未来的版本在对外接口中，彻底拿掉这个参数，也不会导致以前的代码无法运行。

### 应用
另外，在上面的处理中，我们并没有针对x的赋值进行验证，这样当在没有给出x的值的时候，函数会导致报错。这个时候，我们可以来写一个非空验证的方法，来给函数添加校验。
```
function checkParameter() {
    throw new Error('can\'t be empty')
}
function f(x = checkParameter(), y = 7, z = 5) {
  return x + y + z
}
console.log(f(1,3));
try{
  f()
} catch(e) {
  console.log(e)
} finally {
  
}
```
这样给x设置默认值的方式来执行校验函数，就可以确认x的赋值的有效性了。这也是函数默认值的一种使用技巧。

## rest参数
上面针对参数默认值做了说明，下面我们考虑一下一个概念，`可变参数`，比如我们现在来做一个求和运算的函数，但是是几个数的求和是不一定的，按照ES5中的处理，我们的写法：

```
function sum() {
  var array = Array.prototype.slice.call(arguments);
  // arguments对象不是数组，而是一个类似数组的对象。所以为了使用数组的方法，必须使用Array.prototype.slice.call先将其转为数组。
  var sum = 0;
  array.forEach(function(item){
	sum += item * 1
  })
  return sum;
}

sum(1,2,3,4)
```
主要是通过arguments来获取函数的参数列表来获取参数。但是在ES6中却并不需要这么麻烦，ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用arguments对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。
```
function sum(...a) {
  var sum = 0;
  a.forEach(item => sum+= item*1)
  return sum;
}
sum(1,2,3,4)
```
通过ES6的rest参数，能够更加简洁的来实现不定参数的函数的实现。rest参数是一个真正的数组，数组特有的方法都可以使用，完全比上面通过arguments的方法更加自然、简洁。

`注:rest参数之后不能再有其他参数（即只能是最后一个参数）。`

## 小结

>本次主要针对ES6中对于函数部分的相关扩展内容作了梳理，并没有非常的全面，而是选择其中比较常用比较重要的箭头函数、函数默认值和rest参数这几部分进行了说明。更加细致的体会还是需要在后续的不断使用中进行加强和熟悉。

  [1]: https://segmentfault.com/img/bV8jWc?w=447&h=282
