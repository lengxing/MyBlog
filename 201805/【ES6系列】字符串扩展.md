## 1.字符串的遍历器

ES6为字符串添加了遍历器接口，使得字符串可以被`for...of`循环遍历

```
for(let code of "foo") {
  console.log(code);
}
// "f"
// "o"
// "o"
```

## 2.includes(), startsWith(), endsWith()

传统JS中字符串中只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6中提供了三种新方法：
+ includes(): 返回布尔值，表示是否找到了参数字符串
+ startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部
+ endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部

```
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```
这三个方法都支持第二个参数，表示开始搜索的位置。

```
let s = 'Hello world!';

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```
上面代码中使用第二个参数n时，endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束。

## 3.repeat()

`repeat`方法返回一个新字符串，表示将原字符串重复`n`次

```
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""

'na'.repeat(2.9) // "nana" 参数是小数时会被取整
'na'.repeat(-0.9) // ""  参数是 0 到-1 之间的小数，则等同于 0

'na'.repeat(Infinity)
// RangeError
'na'.repeat(-1)
// RangeError

'na'.repeat(NaN) // ""  参数NaN等同于 0

// repeat的参数是字符串，则会先转换成数字
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```

## 4.padStart(), padEnd()

ES2017 引入了字符串补全长度的功能。如果某个字符串不够指定长度，会在头部或尾部补全。padStart()用于头部补全，padEnd()用于尾部补全。

```
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'
```

padStart和padEnd一共接受两个参数，第一个参数用来指定字符串的最小长度，第二个参数是用来补全的字符串

如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串。

```
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
```

如果用来补全的字符串与原字符串，两者的长度之和超过了指定的最小长度，则会截去超出位数的补全字符串。

```
'abc'.padStart(10, '0123456789')
// '0123456abc'
'abc'.padEnd(10, '0123456789')
// 'abc0123456'
```

如果省略第二个参数，默认使用空格补全长度。

```
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```

padStart的常见用途是为数值补全指定位数。

```
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```

另一个用途是提示字符串格式

```
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

这在我们统一处理一些固定格式的字符串或者生成一些固定格式的字符串时是很有帮助的。

## 5.模板字符串

我们在处理一些模板的操作时，往往会依赖于字符串拼接的方式来作为模板字符串来使用

```
$('#target').append(
  'There are <b>' + basket.count + '</b> ' +
  'items in your basket, ' +
  '<em>' + basket.onSale +
  '</em> are on sale!'
);
```

ES6中引入了模板字符串来简化解决这种问题

```
$('#target').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```

> 模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

```
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

上面代码中，所有模板字符串的空格和换行，都是被保留的，比如<ul>标签前面会有一个换行。如果你不想要这个换行，可以使用trim方法消除它。

```
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```

在上面的例子可以看出，模板字符串中嵌入变量，需要将变量名写在`${}`之中

```
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 传统写法为
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```

另外，大括号内部还可以使用表达式，运算,调用函数以及对象属性引用等

```
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字符串支持嵌套。

```
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;

const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```

## 6.标签模板

> 模板字符串的功能，不仅仅是上面这些。它可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。这被称为“标签模板”功能（tagged template）。

```
alert`123`
// 等同于
alert(123)
```

标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。
但是，如果模板字符里面有变量，就不是简单的调用了，而是会将模板字符串先处理成多个参数，再调用函数。

```
let a = 5;
let b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

上面代码中，模板字符串前面有一个标识名tag，它是一个函数。整个表达式的返回值，就是tag函数处理模板字符串后的返回值。

tag函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推。

tag函数的其他参数，都是模板字符串各个变量被替换后的值。由于本例中，模板字符串含有两个变量，因此tag会接受到value1和value2两个参数。

`tag`函数所有参数的实际值：
- 第一个参数：['Hello ', ' world ', '']
- 第二个参数: 15
- 第三个参数：50

一个简单的`tag`函数的写法：

```
let a = 5;
let b = 10;

function tag(s, v1, v2) {
  console.log(s[0]);
  console.log(s[1]);
  console.log(s[2]);
  console.log(v1);
  console.log(v2);

  return "OK";
}

tag`Hello ${ a + b } world ${ a * b}`;
// "Hello "
// " world "
// ""
// 15
// 50
// "OK"
```

### 标签模板应用一：过滤HTML字符串

“标签模板”的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容，xss攻击，常见的是在一些移动应用页面嵌套进入一恶意的代码，如微信红包等。

```
let message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}

let sender = '<script>alert("abc")</script>'; // 恶意代码
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;

message
// <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
```

### 标签模板应用二：多语言转换（国际化处理）

```
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "欢迎访问xxx，您是第xxxx位访问者！"
```

下面引用Jack Hsu给出的一个完整的多语言转换[Demo](https://jaysoo.ca/2014/03/20/i18n-with-es2015-template-literals/)

```
// Matches optional type annotations in i18n strings.
// e.g. i18n`This is a number ${x}:n(2)` formats x as number
//      with two fractional digits.
const typeInfoRegex = /^:([a-z])(\((.+)\))?/;

let I18n = {
  use({locale, defaultCurrency, messageBundle}) {
    I18n.locale = locale;
    I18n.defaultCurrency = defaultCurrency;
    I18n.messageBundle = messageBundle;
    return I18n.translate;
  },

  translate(strings, ...values) {
    let translationKey = I18n._buildKey(strings);
    let translationString = I18n.messageBundle[translationKey];

    if (translationString) {
      let typeInfoForValues = strings.slice(1).map(I18n._extractTypeInfo);
      let localizedValues = values.map((v, i) => I18n._localize(v, typeInfoForValues[i]));
      return I18n._buildMessage(translationString, ...localizedValues);
    }

    return 'Error: translation missing!';
  },

  _localizers: {
    s /*string*/: v => v.toLocaleString(I18n.locale),
    c /*currency*/: (v, currency) => (
      v.toLocaleString(I18n.locale, {
        style: 'currency',
        currency: currency || I18n.defaultCurrency
      })
    ),
    n /*number*/: (v, fractionalDigits) => (
      v.toLocaleString(I18n.locale, {
        minimumFractionDigits: fractionalDigits,
        maximumFractionDigits: fractionalDigits
      })
    )
  },

  _extractTypeInfo(str) {
    let match = typeInfoRegex.exec(str);
    if (match) {
      return {type: match[1], options: match[3]};
    } else {
      return {type: 's', options: ''};
    }
  },

  _localize(value, {type, options}) {
    return I18n._localizers[type](value, options);
  },

  // e.g. I18n._buildKey(['', ' has ', ':c in the']) == '{0} has {1} in the bank'
  _buildKey(strings) {
    let stripType = s => s.replace(typeInfoRegex, '');
    let lastPartialKey = stripType(strings[strings.length - 1]);
    let prependPartialKey = (memo, curr, i) => `${stripType(curr)}{${i}}${memo}`;

    return strings.slice(0, -1).reduceRight(prependPartialKey, lastPartialKey);
  },

  // e.g. I18n._formatStrings('{0} {1}!', 'hello', 'world') == 'hello world!'
  _buildMessage(str, ...values) {
    return str.replace(/{(\d)}/g, (_, index) => values[Number(index)]);
  }
};

// Usage
let messageBundle_fr = {
  'Hello {0}, you have {1} in your bank account.': 'Bonjour {0}, vous avez {1} dans votre compte bancaire.'
};

let messageBundle_de = {
'Hello {0}, you have {1} in your bank account.': 'Hallo {0}, Sie haben {1} auf Ihrem Bankkonto.'
};

let messageBundle_zh_Hant = {
  'Hello {0}, you have {1} in your bank account.': '你好{0}，你有{1}在您的銀行帳戶。'
};

let name = 'Bob';
let amount = 1234.56;
let i18n;

i18n = I18n.use({locale: 'fr-CA', defaultCurrency: 'CAD', messageBundle: messageBundle_fr});
console.log(i18n `Hello ${name}, you have ${amount}:c in your bank account.`);

i18n = I18n.use({locale: 'de-DE', defaultCurrency: 'EUR', messageBundle: messageBundle_de});
console.log(i18n `Hello ${name}, you have ${amount}:c in your bank account.`);

i18n = I18n.use({locale: 'zh-Hant-CN', defaultCurrency: 'CNY', messageBundle: messageBundle_zh_Hant});
console.log(i18n `Hello ${name}, you have ${amount}:c in your bank account.`);
```

## 7.String.raw()

String.raw方法，往往用来充当模板字符串的处理函数，返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，对应于替换变量后的模板字符串。

```
String.raw`Hi\n${2+3}!`;
// 返回 "Hi\\n5!"

String.raw`Hi\u000A!`;
// 返回 "Hi\\u000A!"
```

如果原字符串的斜杠已经转义，那么String.raw会进行再次转义。

```
String.raw`Hi\\n`
// 返回 "Hi\\\\n"
```

String.raw方法可以作为处理模板字符串的基本方法，它会将所有变量替换，而且对斜杠进行转义，方便下一步作为字符串来使用。

String.raw方法也可以作为正常的函数使用。这时，它的第一个参数，应该是一个具有raw属性的对象，且raw属性的值应该是一个数组。

```
String.raw({ raw: 'test' }, 0, 1, 2);
// 't0e1s2t'

// 等同于
String.raw({ raw: ['t','e','s','t'] }, 0, 1, 2);
```

作为函数，String.raw的代码实现基本如下。

```
String.raw = function (strings, ...values) {
  let output = '';
  let index;
  for (index = 0; index < values.length; index++) {
    output += strings.raw[index] + values[index];
  }

  output += strings.raw[index]
  return output;
}
```

## 8.总结

> 本次主要针对ES6中字符串方面的扩展进行了梳理，其中比较实用的，当属标签模板的应用：屏蔽非法注入以及多语言转换，都是前端开发过程中比较常见的问题。其他的一些新增方法也为我们提供了便利的处理。
