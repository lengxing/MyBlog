> 今天，我们来学习一下ES6中新增的两个数据结构：Set和Map。

## Set

ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是**唯一**的，没有重复的值。

### 创建Set数据结构

```
let list = new Set();

或者

let array = [1, 2, 2, 4, 5];
let list = new Set(array);
```

<!-- more -->

Set实例的创建可以简单通过new方法来实现，同时可以通过传入一个数组格式的参数来实例化。

### size属性

size属性返回Set实例的成员总数

```
let array = [1, 2, 2, 4, 5];
let list = new Set(array);

console.log(list.size);
// 4
```

### add(value)方法

添加某个值，返回 Set 结构本身

```
let list = new Set();
list.add(5);
list.add(7);

console.log('size',list.size);  // 2
```

### delete(value)方法

删除某个值，返回一个布尔值，表示删除是否成功。

```
let arr=['add','delete','clear','has'];
let list=new Set(arr);

console.log('delete',list.delete('add'),list);
// delete true Set(3) {"delete", "clear", "has"}
```

### has(value)方法

返回一个布尔值，表示该值是否为Set的成员。

```
let arr=['add','delete','clear','has'];
let list=new Set(arr);

console.log('has',list.has('has'));
// has true
```

### clear()方法

清除所有成员，没有返回值。

```
let arr=['add','delete','clear','has'];
let list=new Set(arr);

list.clear();
console.log('list',list);
// list Set(0) {}
```

### Set的遍历

Set 结构的实例有四个遍历方法，可以用于遍历成员。

- keys()：返回键名的遍历器
- values()：返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员


```
let arr=['add','delete','clear','has'];
let list=new Set(arr);

for(let key of list.keys()){
  console.log('keys',key);
}

// keys add
// keys delete
// keys clear
// keys has

for(let value of list.values()){
  console.log('value',value);
}

// keys add
// keys delete
// keys clear
// keys has

for(let [key,value] of list.entries()){
  console.log('entries',key,value);
}

// entries add add
// entries delete delete
// entries clear clear
// entries has has

list.forEach(function(item){console.log(item);})
// add
// delete
// clear
// has
```

通过上面的示例结果可知，Set结构的key和value是一致的。

> ※ 需要特别指出的是，Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。

### Set结构的应用

在开始我们已经提到过，Set结构的成员都是唯一的，哪怕我们在传入一个有重复成员的数组时，得到的Set实例也是去重的。是的，Set结构非常重要且常见的应用场景就是：`数组去重`。

首先我们需要知道，Array.from方法可以将 Set 结构转为数组。

> Array.from方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

```
let items = new Set([1, 2, 3, 4, 5]);
let array = Array.from(items);
```

从而我们可以这样来实现一个数组去重的方法

```
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

另外，在ES6中数组的扩展中还有扩展运算符(...)同样可以使用到Set实例

```
let arr = [3, 5, 2, 2, 5, 5];
let unique = [...new Set(arr)];
// [3, 5, 2]
```

小结：通过Set结构可以很方便的来实现数组去重的处理，这在之前我们可能就需要循环判断或者依赖于第三方库来实现了，方便了很多。

## WeakSet

WeakSet 结构与 Set 类似，也是不重复的值的集合。

### WeakSet与Set的区别：

- WeakSet 的成员只能是对象，而不能是其他类型的值。
- 其次，WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。

对于垃圾回收机制这部分大家可以自行查找相关说明，本篇不做论述。

### 创建WeakSet结构

可以通过new命令来创建WeakSet数据结构

```
let ws = new WeakSet();
```

WeakSet 可以接受一个数组或类似数组的对象作为参数。该数组的所有成员，都会自动成为 WeakSet 实例对象的成员，所以数组的成员需要是对象。

```
const a = [[1, 2], [3, 4]];
const ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}
```

### 方法

- add(value)：向 WeakSet 实例添加一个新成员。
- delete(value)：清除 WeakSet 实例的指定成员。
- has(value)：返回一个布尔值，表示某个值是否在 WeakSet 实例之中。

相关使用方法可以参照Set的相关方法来使用。
另外，WeakSet 没有size属性，没有办法遍历它的成员。

> ※WeakSet 不能遍历，是因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在，很可能刚刚遍历结束，成员就取不到了。WeakSet 的一个用处，是储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。

## Map

在ES5中，对象的键值对中键只能使用字符串，这就带来了很大的限制。在ES6中增加了Map的数据结构，它类似于对象，但是键的范围不仅仅是字符串，而是各种类型的值都可以当做键。

### 创建Map数据结构

```
let map = new Map();
// map Map(0) {}
```

作为构造函数，Map 也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。

```
let map = new Map([['a',123],['b',456]]);
console.log('map ',map);
// map  Map(2) {"a" => 123, "b" => 456}
```

### set(key, value)方法

通过set(key, value)方法为Map结构添加成员

```
let map = new Map();
let arr=['123'];

map.set(arr,456);
console.log('map',map);
// map Map(1) {Array(1) => 456}
```

### get(key)方法

通过get(key)方法获取Map结构中对应key的value值

```
let map = new Map();
let arr=['123'];

map.set(arr,456);
console.log('map',map,map.get(arr));
// map Map(1) {Array(1) => 456} 456
```

### size属性、delete(key)方法、has(key)方法和clear()方法

Ma数据结构的`size`属性和`delete`、`has`以及`clear`方法，可以参考Set数据结构的对应方法来使用。唯一的区别是`delete`和`has`方法的参数是对应的`key`值。

```
let map = new Map([['a',123],['b',456]]);
console.log('size',map.size);
// 2
console.log('has',map.has('a'));
// has true
console.log('delete',map.delete('a'),map);
// delete true Map(1) {"b" => 456}
console.log('clear',map.clear(),map);
// clear undefined Map(0) {}
```

### Map的遍历

Map结构的遍历基本和Set结构是一样的，具体可以参照上面Set的遍历，此处就不再一一来说了。

## WeakMap

WeakMap结构与Map结构类似，也是用于生成键值对的集合。相关创建方法基本和Map结构是一致的。

### WeakMap和Map的区别

- WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名。
- WeakMap的键名所指向的对象，不计入垃圾回收机制。

> ※WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。

### 方法

WeakMap只有四个方法：

- get(key)
- set(key, value)
- has(key)
- delete(key)

除了上面四个方法，WeakMap既不能遍历，也没有长度`size`属性，同时还不能清空`clear`。

----------

> 数据结构的操作主要集中在增删查改四个方面，下面我们就从这四个方面分别简单将Set/Map和Array/Object来做下对比：

## Set、Map和Array的对比

```
let set = new Set();
let map = new Map();
let array = [];
let item = {
  t: 5
}

// 增
set.add(item);
map.set("t", 5);
array.push({
  t: 5
});

console.log("set-map-array", set, map, array)
```
![图片描述][1]

```
// 查
let setExist = set.has(item);
let mapExist = map.has("t");
let arrayExist = array.find(item=>item.t);
console.log("setExist-mapExist-arrayExist", setExist, mapExist, arrayExist)
```
![图片描述][2]

```
// 改

set.forEach(item=>item.t?item.t=10: "");
map.set("t", 10);
array.forEach(item=>item.t?item.t=10: "");
console.log("set-map-array-modify", set, map, array)
```
![图片描述][3]

```
// 删
set.delete(item);
map.delete("t");
array.splice(array.findIndex(item=>item.t));
console.log("set-map-array-delete", set, map, array)
```
![图片描述][4]

## Set、Map和Object的对比

```
let set = new Set();
let map = new Map();
let obj = {};
let item = {
  t: 5
}

// 增
set.add(item);
map.set("t", 5);
obj["t"] = 5

console.log("set-map-obj", set, map, obj)
```

![图片描述][5]

```
// 查
let setExist = set.has(item);
let mapExist = map.has("t");
let objExist = "t" in obj;
console.log("setExist-mapExist-objExist", setExist, mapExist, objExist)
```

![图片描述][6]
```
// 改

set.forEach(item=>item.t?item.t=10: "");
map.set("t", 10);
obj["t"] = 10;
console.log("set-map-obj-modify", set, map, obj)
```

![图片描述][7]
```
// 删
set.delete(item);
map.delete("t");
delete obj["t"];
console.log("set-map-obj-delete", set, map, obj)
```
![图片描述][8]

## 总结

> 通过上面简单的代码示例我们这次学习了ES6中新增的Set和Map数据结构，并且针对Array和Object数据结构进行了简单的对比分析。不难看出，在使用的过程中明显的Set和Map比我们之前经常使用的Array和Object是有明显的便捷优势的。这也给了我们一个建议，后续的开发过程中，我们可以根据场景需要来通过Set和Map来替代Array和Object来使用。

  [1]: https://segmentfault.com/img/bVbcjKi?w=622&h=169
  [2]: https://segmentfault.com/img/bVbcjKt?w=517&h=59
  [3]: https://segmentfault.com/img/bVbcjKv?w=760&h=167
  [4]: https://segmentfault.com/img/bVbcjKx?w=520&h=24
  [5]: https://segmentfault.com/img/bVbcjNf?w=667&h=163
  [6]: https://segmentfault.com/img/bVbcjNt?w=511&h=23
  [7]: https://segmentfault.com/img/bVbcjNv?w=751&h=167
  [8]: https://segmentfault.com/img/bVbcjNB?w=504&h=25
