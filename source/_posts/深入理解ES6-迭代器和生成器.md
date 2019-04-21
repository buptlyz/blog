---
title: 深入理解ES6-迭代器(Iterator)和生成器(generator)
date: 2019-04-21 09:47:33
tags: 深入理解ES6
---

用循环语句迭代数据时，必须要初始化一个变量来记录每一次迭代在数据集合中的位置，迭代器的使用可以极大的简化数据操作。

## 什么是迭代器Iterator

迭代器是一种特殊的对象，具有专门的接口，所有迭代器对象都有一个next方法，每次调用都返回一个结果对象。

结果对象有两个属性：一个是`value`，表示下一个将要返回的值；另一个是`done`，是一个布尔值，当没有更多可返回数据时返回true。

迭代器还会保存一个内部指针，用来指向当前集合中值的位置，没调用一次`next()`方法，都会返回下一个可用的值。

如果最后一个值返回后再调用`next()`方法，返回的对象中属性`done`为true，`value`则包含迭代器最终返回的值，这个返回值不是数据集的一部分，与函数的返回值类似，是函数调用过程中最后一次给调用者传递信息的方法，如果没有相关数据则返回`undefined`。

```javascript
// ES5 语法创建一个迭代器
function createIterator(items) {
  var i = 0;
  return {
    next: function() {
      var done = (i >= items.length);
      var value = !done ? items[i++] : undefined;

      return {
        done: done,
        value: value
      };
    }
  };
}

var iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // "{ value: 1, done: false }"
console.log(iterator.next()); // "{ value: 2, done: false }"
console.log(iterator.next()); // "{ value: 3, done: false }"
console.log(iterator.next()); // "{ value: undefined, done: true }"
```

## 什么是生成器Generator

生成器是一种返回迭代器的函数，通过`function`关键字后的星号(`*`)来表示，函数中会用到新的关键字`yield`。星号可以紧挨着`function`关键字，也可以在中间添加一个空格：

```javascript
function *createIterator() {
  yield 1;
  yield 2;
  yield 3;
}
```

`yield`关键字也是ES6的新特性，可以通过它来指定调用迭代器的`next()`方法时的返回值及返回顺序。

生成器函数最有序的部分大概是：每当执行完一条`yield`语句后函数就会自动停止执行，直到再次调用迭代器的`next()`方法才会继续执行下一个`yield`语句。

使用`yield`关键字可以返回任何值或表达式，因此可以通过生成器函数批量的给迭代器添加元素。

```javascript
function *createIterator(items) {
  for(let i = 0, len = items.length; i < len; i++) {
    yield items[i];
  }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // "{ value: 1, done: false }"
console.log(iterator.next()); // "{ value: 2, done: false }"
console.log(iterator.next()); // "{ value: 3, done: false }"
console.log(iterator.next()); // "{ value: undefined, done: true }"
```

### yield的使用限制
yield关键字只可在生成器内部使用，在其它地方使用会导致程序抛出语法错误，即便在生成器内部的函数里使用也是如此。常见案例：

```javascript
function *createIterator(items) {
  items.forEach(function(item) { 
    // 语法错误
    yield item + 1; 
  })
}
```

## 可迭代对象和`for-of`循环

可迭代对象具有`Symbol.iterator`属性，是一种与迭代器密切相关的对象。`Symbol.iterator`通过指定的函数可以返回一个作用于附属对象的迭代器。在ES6中，所有的集合对象（数组、`Set`集合及`Map`集合）和字符串都是可迭代对象，这些对象中都有默认的迭代器。

`for-of`循环每执行一次都会调用可迭代对象的`next()`方法，并将迭代器返回的结果对象的`value`属性存储在一个变量中，循环将持续执行这一过程直到返回对象的`done`属性的值为true。

## 访问默认迭代器

可以通过`Symbol.iterator`来访问对象默认的迭代器。

由于具有`Symbol.iterator`属性的对象都有默认的迭代器，因此可以用它来检测对象是否为可迭代对象：

```javascript
function isIterable(object) {
  return typeof object[Symbol.iterator] === 'function';
}
```

## 创建可迭代对象

默认情况下，开发者定义的对象都是不可迭代对象，但如果给`Symbol.iterator`属性添加一个生成器，则可以将其变为可迭代对象：

```javascript
let collection = {
  items: [],
  *[Symbol.iterator]() {
    for (let item of this.items) {
      yield item;
    }
  }
}

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
  console.log(x);
}

// 输出结果
1
2
3
```

## 内建迭代器

在ES6中有3中类型但集合对象：数组、`Map`集合与`Set`集合。为了更好的访问对象中的内容，这3种对象都内建来三种迭代器：

* entries()
* values()
* keys()

### entries()迭代器(TODO)

### values()迭代器(TODO)

### keys()迭代器(TODO)

## 字符串迭代器

由于方括号操作的是编码单元而非字符，因此无法正确访问双字节字符。由于双字节字符被视作两个独立的编码单元，在使用方括号获取双字节字符时得到的是两个空。

所幸，ES6的目标是全面支持Unicode，并且我们可以通过改变字符串的默认迭代器来解决这个问题，使其操作字符而不是编码单元。

## `NodeList`迭代器

自从ES6添加了默认迭代器后，DOM定义中的`NodeList`类型（定义在HTML标准而不是ES6标准中）也拥有了默认迭代器，其行为与数组的默认迭代器完全一致。所以可以将`NodeList`应用于`for-of`循环及其他支持对象默认迭代器的地方。

## 展开运算符与非数组可迭代对象

由于展开运算符可以作用于任意可迭代对象，因此如果想将可迭代对象转换为数组，这是最简单的方法。你既可以将字符串中的每一个字符（不是编码单元）存入新数组中，也可以将浏览器中的`NodeList`对象中的每一个节点存入新数组中。

```javascript
let set = new Set([1, 2, 3]),
  map = new Map(['name', 'Nicholas'], ['age', 25]),
  arrSet = [...set],
  arrMap = [...map];

console.log(arrSet); // [1, 2, 3]
console.log(arrMap); // ['name', 'Nicholas'], ['age', 25]
```

## 高级迭代器功能

### 给迭代器传递参数(TODO)
