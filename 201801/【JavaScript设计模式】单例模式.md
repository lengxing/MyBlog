## 单例模式

> 又被称为单体模式，是只允许实例化一次的对象类。有时我们也用一个对象来规划一个命名空间，井井有条的管理对象上面的属性和方法。

  传统的面向对象语言中单例模式的实现，均是单例对象从“类”中创建而来，在以类为中心的语言中，这是很常见的做法。如果需要某个对象，就必须先定义一个类，对象总是从类中创建而来。但是在JavaScript中却是并不需要这样做。

    单例模式的核心是确保只有一个实例，并提供全局访问。

全局变量不是单例模式，但是在JavaScript中，我们经常会把全局变量当成单例来使用。如浏览器中的window对象。再比如：
var a = {}; 
当用这种方式创建对象a 时，对象a 确实是独一无二的。如果a 变量被声明在全局作用域下，
则我们可以在代码中的任何位置使用这个变量，全局变量提供给全局访问是理所当然的。这样就满足了单例模式的两个条件。
但是全局变量存在很多问题，它很容易造成命名空间污染。在大中型项目中，如果不加以限制和管理，程序中可能存在很多这样的变量。JavaScript 中的变量也很容易被不小心覆盖，相信每个JavaScript 程序员都曾经历过变量冲突的痛苦，就像上面的对象var a = {};，随时有可能被
别人覆盖。

## 命名空间

适当地使用命名空间，并不会杜绝全局变量，但可以减少全局变量的数量。 
最简单的方法依然是用对象字面量的方式：

```  
var namespace1 = { 
    a: function(){ 
        alert (1); 
    }, 
    b: function(){ 
        alert (2); 
    } 
};
```
 
把a 和b 都定义为namespace1 的属性，这样可以减少变量和全局作用域打交道的机会。另外
我们还可以动态地创建命名空间，代码如下（引自Object-Oriented JavaScrtipt 一书）： 

```
var MyApp = {}; 
 
MyApp.namespace = function( name ){ 
    var parts = name.split( '.' ); 
    var current = MyApp; 
    for ( var i in parts ){ 
        if ( !current[ parts[ i ] ] ){ 
            current[ parts[ i ] ] = {}; 
        } 
        current = current[ parts[ i ] ]; 
    } 
}; 
 
MyApp.namespace( 'event' ); 
MyApp.namespace( 'dom.style' ); 
 
console.dir( MyApp ); 

// 上述代码等价于： 

 var MyApp = { 
     event: {}, 
     dom: { 
         style: {} 
     } 
 };
```

## 模块管理

其实在JavaScript中单例模式除了定义命名空间外，还有一个作用，就是通过单例模式来管理代码库的各个模块。这样我们就可以通过单例模式来创建一个自己的小型代码方法库，进而来规范我们自己代码库的各个模块的功能和方法。比如：

```
var OurLib = {
    // 公共模块部分
	Util: {
		util_method1: function(){},
		util_method2: function(){}
		// ...
	},
    // 工具模块部分
	Tool: {
		tool_method1: function(){},
		tool_method2: function(){}
		// ...
	},
    // ajax模块部分
	Ajax: {
		get: function(){},
		post: function()
		// ...
	},
	others: {
		// ...
	}
}
```

## 维护静态变量

我们知道的是JavaScript中并没有static这类关键字（暂不考虑es6等新语法），所以定义任何变量理论上都是可以更改的，所以在JavaScript中实现创建静态变量又是很重要的。根据静态变量只能访问不能修改并且创建后就能使用这一特点，可以如此来实现：能访问的变量定义的方式有很多，比如定义在全局空间中，或者定义一个函数内部，并定义一个特权方法来进行访问等。但是既然不能修改，定义在全局空间里面就不行了，我们可以将变量放在一个函数内部，通过特权方法进行访问，并且不提供赋值变量的方法只提供获取变量的方法，就可以了。由于放在了函数内部，如果想要能够供外界访问，我们需要让创建的函数执行一遍。此时我们创建的对象内保存静态变量通过取值器访问，最后将这个对象作为一个单例放在全局空间里面作为静态变量单例对象供他人使用。如：

```
var Conf = (function(){
	// 私有变量
	var conf = {
		MAX_NUM: 100,
		MIN_NUM: 1,
		COUNT: 1000
	}

	// 返回取值器对象
	return {
		// 取值器方法
		get: function(name) {
			return conf[name] ? conf[name] : null;
		}
	}
})();
```
这样我们就可以来维护我们的静态变量了。

## 惰性单例

有时候对于单例对象我们需要延迟创建，所以在单例中还存在一种延迟创建的形式，也被称为“惰性创建”。例子如下：

```
// 惰性载入单例
var LazySingle = (function(){
	// 单例实例引用
	var _instance = null;
	// 单例
	function Single() {
		/* 这里定义私有属性和方法 */
		return {
			publicMethod: function(){},
			publicProperty: '1.0'
		}
	}
	// 获取单例对象接口
	return function(){
		// 如果未创建单例，将创建单例
		if(!_instance) {
			_instance = Single();
		}
		// 返回单例
		return _instance;
	}
})();
```
## 总结

通过上面对于单例模式在JavaScript中的一些提现形式，我们对于单例模式的使用有了一个基本的了解，同样也理解了为什么很多代码库都提供一个变量（当然还有一些还提供了简单的别名，如jQuery的$别名，underscore的_别名等）来管理每个模块的方式。这也为我们以后来设计我们自己的公共代码库提供了参考。但是同样的，也能发现，单例模式在JavaScript中是有别于其他的面向对象语言中的实现方式的。

最后，欢迎大家拍砖，但是请轻拍:)
