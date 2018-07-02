> 本篇目录

<!-- MarkdownTOC -->

- JS 同步与异步
- 常见异步处理
	- 回调函数
	- 事件监听
	- Deferred对象
- Promise对象概念
- Promise对象的用法
	- ES6 Promise对象与jQuery Deferred对象
	- Promise对象例子
- Promise.prototype.then\(\)
- Promise.prototype.catch\(\)
- Promise.prototype.finally\(\)
- Promise异步并发
	- Promise.all\(\)
	- Promise.race\(\)
- Promise.resolve\(\)
- Promise.reject\(\)
- 应用举例
	- 加载图片
- 小结

<!-- /MarkdownTOC -->

## JS 同步与异步

> Javascript语言的执行环境是"单线程"（single thread）。

所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为了解决这个问题，Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。

"同步模式"就是上一段的模式，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；"异步模式"则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。

"异步模式"非常重要。在浏览器端，耗时很长的操作都应该异步执行，避免浏览器失去响应，最好的例子就是Ajax操作。在服务器端，"异步模式"甚至是唯一的模式，因为执行环境是单线程的，如果允许同步执行所有http请求，服务器性能会急剧下降，很快就会失去响应。

## 常见异步处理

在我们日常的开发中，特别是在ES6之前我们常用的解决异步编程的方式主要有以下几种：

- 回调函数
- 事件监听
- Deferred对象

下面我们分别来看一下，假定我们有两个函数`f1`和`f2`，后者等待前者的执行结果再执行。

### 回调函数

```
function f1(callback) {
  setTimeout(function() {
    // f1的任务代码
    callback();
  }, 1000);
}

f1(f2)
```

采用这种方式，我们把同步操作变成了异步操作，f1不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。

回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

### 事件监听

```
f1.on("done", f2);

function f1(callback) {
  setTimeout(function() {
    // f1的任务代码
    f1.trigger("done")
  }, 1000);
}
```

这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

### Deferred对象

Jquery中提供了Deferred对象的概念，Deferred对象是一个延迟对象，意思是函数延迟到某个点才开始执行，改变执行状态的方法有两个（成功：resolve和失败：reject），分别对应两种执行回调（成功回调函数：done和失败回调函数fail）

这也是我在ES6之前最常用的自定义异步方法所使用的方法。

```
function f1(callback) {
  var $dfd = $.Deferred();
  setTineout(function() {
  	if(任务完成) {
  	  dfd.resolve({
  	  	info: "完成"
  	  })
  	} else {
  	  dfd.reject({
  	    info: "失败"
  	  })
  	}
  }, 1000);
  return dfd.promise();
}

f1().then(f2);
```

可以发现，通过上面的方法可以实现链式的异步调用，而且执行过程更加的清晰，每个处理之间也达到了“去耦合”的效果。

## Promise对象概念

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。其实上面的Deferred对象的使用就已经类似于这种处理了。

Promise对象有以下两个特点：

- （1）对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

![图片描述][1]

- （2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise对象提供统一的接口，使得控制异步操作更加容易。

Promise也有一些缺点。首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

## Promise对象的用法

ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。

```
let ajax=function(){
  return new Promise(function(resolve,reject){
    setTimeout(function () {
      resolve()
    }, 1000);
  })
};

ajax().then(function(){
  console.log('promise','resolve');
})
```

resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```
ajax().then(function(){
  console.log('promise','resolve');
}, function(){
  console.log('promise', 'reject')
})
```

Promise 新建后就会立即执行。

```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

### ES6 Promise对象与jQuery Deferred对象

先看例子：

```
【ES6 Promise】
var pro=new Promise(function(resolve){
    resolve(1);
});

//已经resolve了，再设置then回调。
pro.then(function(v){
    alert(v);    //1
});
alert(2);

//还是会已异步方式，发生回调。
//先alert(2)再alert(v);

//而且，以后什么时候注册then，都会异步调用。

【Jquery Deferred】
var defer=$.Deferred();
defer.resolve(1);

//deferred对象已经resolve了
defer.done(function(v){
    alert(v);    //不会执行
});
alert(2);

//只执行alert(2);
//如果需要执行done，就要把注册done回调放到defer.resolve()之前。
```

另外，ES6的Promise没有resolve，reject，notify方法，不能进行状态更改，
只能注册回调。

### Promise对象例子

下面是一个用Promise对象实现的 常见Ajax 操作的例子：

```
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

## Promise.prototype.then()

Promise.prototype.then()方法返回的是一个新的Promise对象，因此可以采用链式写法，即一个then后面再调用另一个then。如果前一个回调函数返回的是一个Promise对象，此时后一个回调函数会等待第一个Promise对象有了结果，才会进一步调用。

![图片描述][2]

关于上图中黄圈1的对value的处理是Promise里面较为复杂的一个地方，value的处理分为两种情况：Promise对象、非Promise对象。

当value 不是Promise类型时，直接将value作为第二个Promise的resolve的参数值即可；当为Promise类型时，promise2的状态、参数完全由value决定，可以认为promsie2完全是value的傀儡，promise2仅仅是连接不同异步的桥梁。

![图片描述][3]

```
Promise.prototype.then = function(onFulfilled, onRejected) {
  return new Promise(function(resolve, reject) { //此处的Promise标注为promise2
    handle({
      onFulfilled: onFulfilled,
      onRejected: onRejected,
      resolve: resolve,
      reject: reject
    })
  });
}

function handle(deferred) {
  var handleFn;
  if (state === 'fulfilled') {
    handleFn = deferred.onFulfilled;
  } else if (state === 'rejected') {
    handleFn = deferred.onRejected;
  }
  var ret = handleFn(value);
  deferred.resolve(ret); //注意，此时的resolve是promise2的resolve
}

function resolve(val) {
  if (val && typeof val.then === 'function') {
    val.then(resolve); // if val为promise对象或类promise对象时，promise2的状态完全由val决定
    return;
  }
  if (callback) { // callback为指定的回调函数
    callback(val);
  }
}
```

关于then的使用

```
let ajax = function() {
  return new Promise(function(resolve, reject) {
      setTimeout(function() {
          resolve()
      }, 1000);
  })
};

ajax()
  .then(function() {
    return new Promise(function(resolve, reject) {
      setTimeout(function() {
          resolve()
      }, 2000);
    });
  })
  .then(function() {
    console.log('all done');
  })
```
## Promise.prototype.catch()

Promise.prototype.catch方法是Promise.prototype.then(null, rejection)的别名，用于指定发生错误时的回调函数。

```
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```

使用Promise对象的catch方法可以捕获异步调用链中callback的异常，Promise对象的catch方法返回的也是一个Promise对象，因此，在catch方法后还可以继续写异步调用方法。这是一个非常强大的能力。

```
const someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};

someAsyncThing().then(function() {
  return someOtherAsyncThing();
}).catch(function(error) {
  console.log('oh no', error);
  // 下面一行会报错，因为 y 没有声明
  y + 2;
}).catch(function(error) {
  console.log('carry on', error);
});
// oh no [ReferenceError: x is not defined]
// carry on [ReferenceError: y is not defined]
```

上面代码中，第二个catch方法用来捕获前一个catch方法抛出的错误。

一般来说，不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用catch方法。

```
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

## Promise.prototype.finally()

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。

```
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

finally方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是fulfilled还是rejected。这表明，finally方法里面的操作，应该是与状态无关的，不依赖于 Promise 的执行结果。

## Promise异步并发

如果几个异步调用有关联，但它们不是顺序式的，是可以同时进行的，我们很直观地会希望它们能够并发执行(这里要注意区分“并发”和“并行”的概念，不要搞混)。

### Promise.all()

Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
```
let p = Promise.all([p1, p2, p3]);
```
Promise.all方法接受一个数组作为参数，p1、p2、p3都是Promise对象实例。

p的状态由p1、p2、p3决定，分两种情况：

- （1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

- （2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

一个具体的例子：

```
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

### Promise.race()

Promise.race()也是将多个Promise实例包装成一个新的Promise实例：

```
let p = Promise.race([p1,p2,p3]);
```

上述代码中，p1、p2、p3只要有一个实例率先改变状态，p的状态就会跟着改变，那个率先改变的Promise实例的返回值，就传递给p的返回值。如果Promise.all方法和Promise.race方法的参数不是Promise实例，就会先调用下面讲到的Promise.resolve方法，将参数转为Promise实例，再进一步处理。

```
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);

p
.then(console.log)
.catch(console.error);
```
上面代码中，如果 5 秒之内fetch方法无法返回结果，变量p的状态就会变为rejected，从而触发catch方法指定的回调函数。

## Promise.resolve()

有时候需将现有对象转换成Promise对象，可以使用Promise.resolve()。

如果Promise.resolve方法的参数，不是具有then方法的对象（又称thenable对象），则返回一个新的Promise对象，且它的状态为fulfilled。

如果Promise.resolve方法的参数是一个Promise对象的实例，则会被原封不动地返回。

```
var p = Promise.resolve('Hello');

p.then(function (s){
    console.log(s)
});
// Hello
```

## Promise.reject()

Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为rejected。Promise.reject方法的参数reason，会被传递给实例的回调函数。

```
var p = Promise.reject('Wrong');

p.then(null, function (s){
    console.log(s)
});
// Wrong
```

## 应用举例

### 加载图片

```
let preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    const image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
```

## 小结

在JS开发过程中，异步编程是非常常见和非常重要的处理方式，而ES6中将Promise对象列为标准，方便了我们的开发使用，使得我们不再需要依赖高耦合的方式或者第三方库来完成这一目标。

本篇参照了一些相关文章的内容，具体可见参考部分。

----------
相关参考：
- [JavaScript 异步编程 与异步式I/O](https://blog.csdn.net/youhan26/article/details/47057559)
- [Promise对象](http://es6.ruanyifeng.com/#docs/promise)
- [谈谈异步编程](https://www.cnblogs.com/bigbrother1984/p/4140685.html)


  [1]: https://segmentfault.com/img/bVbcZmh?w=551&h=426
  [2]: https://segmentfault.com/img/bVbcZpV?w=1279&h=426
  [3]: https://segmentfault.com/img/bVbcZqj?w=1091&h=510
