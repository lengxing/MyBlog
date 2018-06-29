> 本篇目录：

<!-- MarkdownTOC -->

- Proxy
	- Proxy基本概念
	- 常见的Proxy拦截操作
		- get
		- set
		- has
		- deleteProperty
		- ownKeys
	- Proxy.revocable\(\)
	- Proxy的this问题
- Reflect
	- 设计目的
	- 相关方法
- Proxy和Reflect实现观察者模式
- 小结

<!-- /MarkdownTOC -->

<!--more-->

> 猪八戒去高老庄找高翠兰，结果高小姐是孙悟空变的,在这个场景中，对于猪八戒来说，孙悟空可以算是高小姐的一个代理，在长相上来说，他们是一致的。猪八戒只能访问到被孙悟空假扮的高小姐，却见不到真正的高小姐。

# Proxy

## Proxy基本概念

在上面的场景中，孙悟空就类似于我们今天要讲的ES6中的Proxy，它是一种“代理”，或者可以称之为“拦截”。外界在对一个对象进行访问的时候，都先必须通过这层拦截，才能进行访问。而这个拦截的过程中可以对外界的访问进行过滤和改写。

我们先来看一个例子：

```
let obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
```

在上面这个例子中，我们通过Proxy对一个空对象进行了拦截，重新定义了对象属性的读取(`get`)和设置(`set`)行为。

ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

```
let proxy = new Proxy(target, handler);
```
其中，new Proxy()表示生成一个Proxy实例，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。

> 注意，要使得Proxy起作用，必须针对Proxy实例（上例是proxy对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。

如果handler没有设置任何拦截，那就等同于直接通向原对象。

```
let target = {};
let handler = {};
let proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
```

## 常见的Proxy拦截操作

Proxy支持的拦截操作有：

- get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
- set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
- has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
- deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
- ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
- getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
- preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
- getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
- isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
- setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
- construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。

下面我们就几个常见的拦截进行举例说明。

默认一个对象：

```
let obj={
  time:'2017-03-11',
  name:'net',
  _r:123
};
```

### get

```
// 拦截对象属性的读取
get(target,key){
  return target[key].replace('2017','2018')
}
```

### set

```
// 拦截对象设置属性
set(target,key,value){
  if(key==='name'){
    return target[key]=value;
  }else{
    return target[key];
  }
}
```

### has

```
// 拦截key in object操作
has(target,key){
  if(key==='name'){
    return target[key]
  }else{
    return false;
  }
}
```

### deleteProperty

```
// 拦截delete
deleteProperty(target,key){
  if(key.indexOf('_')>-1){
    delete target[key];
    return true;
  }else{
    return target[key]
  }
}
```

### ownKeys

```
// 拦截Object.keys,Object.getOwnPropertySymbols,Object.getOwnPropertyNames
ownKeys(target){
  return Object.keys(target).filter(item=>item!='time')
}
```

## Proxy.revocable()

Proxy.revocable方法返回一个可以取消的Proxy实例

```
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以取消Proxy实例。上面代码中，当执行revoke函数之后，再访问Proxy实例，就会抛出一个错误。

Proxy.revocable的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

## Proxy的this问题

虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理。

```
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```

所以在一些情况下，有些对象的属性只能通过正确的this才能拿到时，由于this指向的变化，导致Proxy无法代理目标对象。

```
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

proxy.getDate();
// TypeError: this is not a Date object.
```

# Reflect

为什么我们要一起来说Reflect呢？Reflect对象与Proxy对象一样，也是 ES6 为了操作对象而提供的新 API。

## 设计目的

（1） 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。

（2） 修改某些Object方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。

```
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```

（3） 让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。

```
// 老写法
'assign' in Object // true

// 新写法
Reflect.has(Object, 'assign') // true
```

（4）Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为。

```
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target,name, value, receiver);
    if (success) {
      log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

## 相关方法

Reflect与ES5的Object有点类似，包含了对象语言内部的方法，Reflect也有13种方法，与proxy中的方法一一对应。

> Proxy相当于去修改设置对象的属性行为，而Reflect则是获取对象的这些行为。

相关使用大家参照Object和Proxy的使用即可，不再一一赘述。

# Proxy和Reflect实现观察者模式

> 观察者模式（Observer mode）指的是函数自动观察数据对象，一旦对象有变化，函数就会自动执行。

在我们之前的开发过程中，我们如果想要实现观察者模式的话，我们可能需要进行事件绑定和触发来实现。

```
Event.listen('changeName', name => console.log(name))

Event.trigger('changeName', name )
```
但是在ES6中，我们可以通过使用Proxy和Reflect来实现这个目的

```
//添加观察者
const queuedObservers = new Set();
const observe = fn => queuedObservers.add(fn);

//proxy 的set 方法
function set(target, key, value, receiver) {
    const result = Reflect.set(target, key, value, receiver);
    queuedObservers.forEach(observer => observer());
    return result;
}
//创建proxy代理
const observable = obj => new Proxy(obj, {set});
//被观察的 对象
const person = observable({
  name: '张三',
  age: 20
});

function print() {
    console.log(`${person.name}, ${person.age}`)
}
function print2() {
    console.log(`我是二号观察者：${person.name}, ${person.age}`)
}
//添加观察者
observe(print);
observe(print2);
person.name = '李四';
// 输出
// 李四, 20
// 我是二号观察者：李四, 20
```

# 小结

Proxy和Reflect都是ES6中针对对象新增的方法，Proxy修改设置对象的属性行为，而Reflect则是获取对象的这些行为。
