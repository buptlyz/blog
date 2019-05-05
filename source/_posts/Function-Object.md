---
title: Function && Object
date: 2019-05-05 10:32:31
tags:
---

## [`Function`][]

`Function`构造函数可以用来创建`Function`对象，直接调用`Function`构造函数可以动态创建函数，但有安全性和与`eval`相同的性能问题。不同于`eval`的是，`Function`构造函数只能在全局作用域创建函数。

每个JavaScript函数实际上都是一个`Function`对象。

```javascript
(function() {}).constructor === Function; // true
```

### Properties

* `Function.prototype.caller`
* `Function.prototype.length`
* `Function.prototype.name`
* `Function.prototype.displayName`
* `Function.prototype.constructor`

### Methods

* `Function.prototype.apply()`
* `Function.prototype.bind()`
* `Function.prototype.call()`
* `Function.prototype.isGenerator()`
* `Function.prototype.toSource()`
* `Function.prototype.toString()`

## [`Object`][]

* `Object.assign()`
* `Object.create()`
* `Object.defineProperty()`
* `Object.defineProperties()`
* `Object.entries()`
* `Object.entries()`
* `Object.freeze()`
* `Object.fromEntries()`
* `Object.getOwnPropertyDescriptor()`
* `Object.getOwnPropertyDescriptors()`

### `Object.assgin()`

只复制`enumerable`和`own`的properties。在source上使用`[[Get]]`，在target上使用`[[Set]]`，因此会调用`getters`和`setters`方法。

不适用于包含`getters`的source的merge。

```javascript
var foo = {
  get bar() {
    return 1;
  },
  set bar(val) {
    this.bar = val;
  },
  set baz(val) {
    this.baz = val;
  }
}
var obj = {};

Object.assign(obj, foo);
console.log(obj); // {bar: 1, baz: undefined}
```

如果需要copy属性的定义，如enumerability，需要使用`Object.getOwnPropertyDescriptor()`和`Object.defineProperty()`。

`String`和`Symbol`类型的属性都会被复制。

`Object.assign`不接收参数`null`和`undefined`，会报错。原始类型会被包装成对象。

```javascript
var v1 = 'abc';
var obj = Object.assign({}, v1);

console.log(obj); // {"0": "a", "1": "b", "2": "c"}
```

如果assign过程中报错，之前的改变会保留在target上：

```javascript
var target = Object.defineProperty({}, 'foo', {
  value: 1,
  writable: false
}); // target.foo 是read-only的属性

Object.assign(target, {bar: 2}, {foo2: 3, foo: 3, foo3: 3}, {baz: 4});
// TypeError: "foo" is read-only
// 在尝试 assign target.foo的时候报错

console.log(target); // {bar: 2, foo2: 3, foo: 1}
// foo assign的时候报错，foo3和baz都没有assign
```

polyfill

```javascript
if (typeof Object.assign !== 'function') {
  // Must be writable: true, enumerable: false, configurable: true
  Object.defineProperty(Object, 'assign', {
    value: function assign(target, varArgs) { // .length of function is 2
      'use strict';
      if (target === null || target === undefined) { // TypeError if undefined or null
        throw new TypeError('Cannot convert undefined or null to object');
      }

      var to = Object(target);

      for (var index = 1; index < arguments.length; index++) {
        var nextSource = arguments[index];

        if (nextSource !== null && nextSource !== undefined) { // Skip if undefined or null
          for (var nextKey in nextSource) {
            // Avoid bugs when hasOwnProperty is shadowed
            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
              to[nextKey] = nextSource[nextKey];
            }
          }
        }
      }

      return to;
    },
    writable: true,
    configurable: true
  })
}
```

### `Object.create()`

创建一个新对象，并将传入的对象参数作为新对象的原型。

例子：

```javascript
// Shape
function Shape() {
  this.x = 0;
  this.y = 0;
}

// method
Shape.prototype.move = function(x, y) {
  this.x += x;
  this.y += y;
  console.info('Shape moved. ');
}

// Rectangle
function Rectangle() {
  Shape.call(this); // call super constructor
}

// extends
Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rect = new Rectangle();

console.log('Is rect an instance of Rectangle?', rect instanceof Rectangle); // true
console.log('Is rect an instance of Shape?', rect instanceof Shape); // true
rect.move(1, 2); // Shape moved.
```

继承多个对象：

```javascript
function MyClass() {
  SuperClass.call(this);
  OtherSuperClass.call(this);
}

// inherit one
MyClass.prototype = Object.create(SuperClass.prototype);
// mixin another
Object.assign(MyClass.prototype, OtherSuperClass.prototype);
// re-assign constructor
MyClass.prototype.constructor = MyClass;

MyClass.prototype.myMethod = function() {
  // do something
}
```

使用`Object.create()`的`propertiesObject`参数：

```javascript
var o;

o = Object.create(null); // null 作为原型

o = {};
// 等于
o = Object.create(Object.prototype);

o = Object.create(Object.prototype, { // 创建一个带有示例属性的对象
  foo: { // 普通属性
    writable: true,
    configurable: true,
    value: 'hello'
  },
  bar: { // getter-setter 属性
    configurable: false,
    get: function() {return 10;},
    set: function(val) {
      console.log("Setting 'o.bar' to", value);
    }
  }
});

function Constructor() {}

o = new Constructor();
// 等于
o = Object.create(Constructor.prototype);
// 这里不会调用constructor内部的初始化代码，如果有的话，如this.x = 1

o = Object.create({}, {p: {value: 42}});
// 创建一个对象，原型是一个新的空的对象，并给它添加属性'p'

// 默认属性是不可写、枚举、配置的
o.p = 24;
o.p; // 42

o.q = 12;
for (var prop in o) {
  console.log(prop);
}
// 'q'

delete o.p; // false

var o2;

o2 = Object.create({}, {
  p: {
    value: 42,
    writable: true,
    enumerable: true,
    configurable: true
  }
});
// 等于
o2 = Object.create({p: 42});
```

继承自`null`的对象：

```javascript
var oco = Object.create({}); // 普通对象
var ocn = Object.create(null); // null 对象

console.log(oco); // {}
console.log(ocn); // {}

oco.p = 1; // {p: 1}
ocn.p = 0; // {p: 0}

console.log('oco is ' + oco); // oco is [object Object]
console.log('ocn is ' + ocn); // throws error: Cannot convert object to primitive value

alert(oco); // [object Object]
alert(ocn); // throws error: Cannot convert object to primitive value

oco.toString(); // [object Object]
ocn.toString(); // throws error: ocn.toString is not a function
// 同样，valueOf、hasOwnProperty 都会报错

oco.constructor; // Object() { [native code] }
ocn.constructor; // undefined
```

```javascript
var ocn = Object.create(null);

ocn.toString = toString;
// 或者
Object.setPrototypeOf(ocn, Object.prototype);

ocn.toString(); // [object Object]
ocn.constructor(); // Object() { [native code] }
```

polyfill

```javascript
if (typeof Object.create !== 'function') {
  Object.create = function(proto, propertiesObject) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
      throw new TypeError('Object prototype may only be an Object: ' + proto);
    } else if (proto === null) {
      throw new Error("This browser's implementation of Object.create is a shim and doesn't support 'null' as the first argument.");
    }

    if (typeof propertiesObject !== 'undefined') {
      throw new Error("This browser's implementation of Object.create is a shim and doesn't support a second argument.");
    }

    function F() {}
    F.prototype = proto;

    return new F();
  }
}
```

### `Object.defineProperty()`

[`Function`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
[`Object`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
