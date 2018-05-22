## 1.Number.isFinite(), Number.isNaN()

Number.isFinite()用来检查一个数值是否为有限的（finite），即不是Infinity。

```
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false
```

> ※ 参数类型不是数值，一律返回false

Number.isNaN()用来检查一个值是否为NaN。

```
Number.isNaN(NaN) // true
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true' / 0) // true
Number.isNaN('true' / 'true') // true
```

> ※ 参数类型不是NaN，一律返回false
> ※ NaN === NaN  // false

## 2.Number.parseInt(), Number.parseFloat()

ES6将全局方法parseInt()和parseFloat()移植到了Number对象上面，使用行为保持不变，只是减少了全局方法的存在，增进了语言的模块化。

## 3.Number.isInteger()

Number.isInteger()用来判断一个数值是否为整数。

```
Number.isInteger(25) // true
Number.isInteger(25.1) // false

Number.isInteger(25.0) // true, 整数和浮点数采用的是同样的储存方法,所以被认为相同

Number.isInteger() // false
Number.isInteger(null) // false
Number.isInteger('15') // false
Number.isInteger(true) // false
```

### 双精度问题延伸

> ※ JavaScript的双精度问题：由于 JavaScript 采用 IEEE 754 标准，数值存储为64位双精度格式，数值精度最多可以达到 53 个二进制位（1 个隐藏位与 52 个有效位）。如果数值的精度超过这个限度，第54位及后面的位就会被丢弃，这种情况下，Number.isInteger可能会误判。
```
Number.isInteger(3.0000000000000002) // true
```
> 当然，针对非高精度的操作这个方法是很实用的。
> 引申一下，关于浮点数的计算的精度问题：0.1 + 0.2 === 0.3 // false, 奇葩啊，计算机居然算不对？

其实对于浮点数的四则运算，几乎所有的编程语言都会有类似精度误差的问题，只不过在 C++/C#/Java 这些语言中已经封装好了方法来避免精度的问题，而 JavaScript 是一门弱类型的语言，从设计思想上就没有对浮点数有个严格的数据类型，所以精度误差的问题就显得格外突出。下面就分析下为什么会有这个精度误差，以及怎样修复这个误差。

首先，我们要站在计算机的角度思考 0.1 + 0.2 这个看似小儿科的问题。我们知道，能被计算机读懂的是二进制，而不是十进制，所以我们先把 0.1 和 0.2 转换成二进制看看：

```
0.1 => 0.0001 1001 1001 1001…（无限循环）
0.2 => 0.0011 0011 0011 0011…（无限循环）
```

上面我们发现0.1和0.2转化为二进制之后，变成了一个无限循环的数字，这在现实生活中，无限循环我们可以理解，但计算机是不允许无限循环的，对于无限循环的小数，计算机会进行舍入处理。进行双精度浮点数的小数部分最多支持 52 位，所以两者相加之后得到这么一串 0.0100110011001100110011001100110011001100110011001100 因浮点数小数位的限制而截断的二进制数字，这时候，我们再把它转换为十进制，就成了 0.30000000000000004。从而，就出现了开始时出现的“奇葩”问题。

## 4.安全整数和 Number.isSafeInteger()

JavaScript 能够准确表示的整数范围在-2^53到2^53之间（不含两个端点），超过这个范围，无法精确表示这个值。

```
Math.pow(2, 53) // 9007199254740992

9007199254740992  // 9007199254740992
9007199254740993  // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1
// true
```

具体一些关于安全整数的内容不再详谈，大部分我们的处理过程是不会涉及到这些情况的。

## 5.Math对象的方法扩展

### Math.trunc()

Math.trunc方法用于去除一个数的小数部分，返回整数部分。

```
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0

// 对于非数值，Math.trunc内部使用Number方法将其先转为数值。
Math.trunc('123.456') // 123
Math.trunc(true) //1
Math.trunc(false) // 0
Math.trunc(null) // 0

// 对于空值和无法截取整数的值，返回NaN。
Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN
Math.trunc(undefined) // NaN
```

模拟代码：

```
Math.trunc = Math.trunc || function(x) {
  return x < 0 ? Math.ceil(x) : Math.floor(x);
};
```

> ※ 使用场景：当在处理一些整数位置，但是却又依赖于一些随机数情况时，可以将生成的随机数取整后进行处理

### Math.sign()

Math.sign方法用来判断一个数到底是正数、负数、还是零。对于非数值，会先将其转换为数值。返回值情况：

- 参数为正数，返回+1；
- 参数为负数，返回-1；
- 参数为 0，返回0；
- 参数为-0，返回-0;
- 其他值，返回NaN。

```
Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN

// 如果参数是非数值，会自动转为数值。对于那些无法转为数值的值，会返回NaN。
Math.sign('')  // 0
Math.sign(true)  // +1
Math.sign(false)  // 0
Math.sign(null)  // 0
Math.sign('9')  // +1
Math.sign('foo')  // NaN
Math.sign()  // NaN
Math.sign(undefined)  // NaN
```

模拟代码：

```
Math.sign = Math.sign || function(x) {
  x = +x; // convert to a number
  if (x === 0 || isNaN(x)) {
    return x;
  }
  return x > 0 ? 1 : -1;
};
```

> ※ 使用场景：针对一些返回值情况进行区分处理时。

## 6.结语

> 本次主要针对ES6中数值部分新增的一些方法进行了梳理，只挑选了一些和日常工作相关的内容，其他的一些高精度和数学计算相关的内容没有进行梳理，当有实际应用场景时对应分析处理即可，毕竟目前来看应用场景还是比较少的。针对JavaScript的双精度计算相关的问题只是做了一个基本的分析，一些具体的解决方法后续有机会将会单独整理分享。
