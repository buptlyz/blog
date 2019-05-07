---
title: Function && Object
date: 2019-05-05 10:32:31
tags:
---

## [`Function`][]

* [`AsyncFunction`][]
* [`GeneratorFunction`][]
* [`ArrowFunctions`][]

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

* `Object.prototype.hasOwnProperty()`
* `Object.prototype.isPrototypeOf()`
* `Object.prototype.propertyIsEnumerable()`
* `Object.prototype.toSource()`
* `Object.prototype.toString()`
* `Object.prototype.toLocaleString()`
* `Object.prototype.unwatch()`
* `Object.prototype.valueOf()`
* `Object.prototype.watch()`

### Object literal notation vs JSON

* `JSON`的属性名必须用双引号扩起来，并且不能使用简写，如`{ x }`
* `JSON`的值只能是`strings`、`numbers`、`arrays`、`true`、`false`、`null`、或者另一个`JSON`对象
* 函数不能作为`JSON`的值
* `Date`对象会被parse为`string`
* `JSON.parse()`遇到计算属性（`getter`、`setter`）会报错

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

在一个对象上，定义一个新属性，或修改一个已存在的属性。不能重复调用，重复调用会报错。

Syntax

> `Object.defineProperty(obj, prop, descriptor)`

默认值：

* `configurable`: false
* `enumerable`: false
* `value`: undefined
* `writable`: false
* `get`: undefined
* `set`: undefined

当`descriptor`有`value`和`writable`，或者`get`和`set`之一时，被视为data descriptor。

相反的，当`descriptor`同时有`value`或`writable`，和`get`或`set`时，会报错。

```javascript
// 对
Object.defineProperty(obj, 'key', {
  value: 'static',
  writable: true
});

// 对
Object.defineProperty(obj, 'key', {
  get() { return 'static'; }
});

// 报错
Object.defineProperty(obj, 'key', {
  value: 'static',
  get() {return 'static'; }
})
```

[其他例子](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty/Additional_examples)。

### `Object.defineProperties()`

Syntax

> `Object.defineProperties(obj, props)`

polyfill

```javascript
function defineProperties(obj, properties) {
  function convertToDescriptor(desc) {
    function hasProperty(obj, prop) {
      return Object.prototype.hasOwnProperty.call(obj, prop);
    }

    function isCallable(v) {
      // NB: modify as necessary if other values than functions are callable
      return typeof v === 'function';
    }

    var d = {};

    if (hasProperty(desc, 'enumerable'))
      d.enumerable = !!desc.enumerable;
    if (hasProperty(desc, 'configurable'))
      d.configurable = !!desc.configurable;
    if (hasProperty(desc, 'value'))
      d.value = desc.value;
    if (hasProperty(desc, 'writable'))
      d.writable = !!desc.writable;
    if (hasProperty(desc, 'get')) {
      var g = desc.get;

      if (!isCallable(g) && typeof g !== 'undefined')
        throw new TypeError('bad get');
      d.get = g;
    }
    if (hasProperty(desc, 'set')) {
      var s = desc.set;

      if (!isCallable(s) && typeof s !== 'undefined')
        throw new TypeError('bad set');
      d.set = s;
    }

    if (('get' in d || 'set' in d) && ('value' in d || 'writable' in d)) 
      throw new TypeError('identity-confused descriptor');

    return d;
  }

  if (typeof obj !== 'object' || obj === null)
    throw new TypeError('bad obj');
  
  properties = Object(properties);

  var keys = Object.keys(properties);
  var descs = [];

  for (var i = 0; i < keys.length; i++)
    descs.push(keys[i], convertToDescriptor(properties[keys[i]]));
  
  for (var i = 0; i < descs.length; i++)
    Object.defineProperty(obj, descs[i][0], descs[i][1]);

  return obj;
}
```

### `Object.entries()`

返回给定对象的own enumerable string-keyed属性[key, value]对的数组。和`for...in`循环的顺序是一致的。

如果需要返回特定顺序，可以做个sort：

`Object.entries(obj).sort((a, b) => b[0].localCompare(a[0]));`

Syntax

> `Object.entries(obj)`

polyfill

```javascript
if (!Object.entries) {
  Object.entries = function (obj) {
    var ownProps = Object.keys(obj),
      i = ownProps.length,
      resArray = new Array(i); // prealocate the Array
    while(i--)
      resArray[i] = [ownProps[i], obj[ownProps[i]]];

    return resArray;
  }
}
```

### `Object.fromEntries()`

`Object.entries()`作用相反的方法。

### `Object.freeze()`

一个被冷冻的对象不能再修改。它的原型，即`__proto__`属性也不能被修改。

Syntax

> `Object.freeze(obj)`

冷冻有元素的[`ArrayBufferView`][]会报错。

```javascript
> Object.freeze(new Unit8Array(0)) // No elements
Unit8Array []

> Object.freeze(new Unit8Array(1)) // Has elements
TypeError: Cannot freeze array buffer views with elements

> Object.freeze(new DataView(new ArrayBuffer(32))) // No elements
DataView {}

> Object.freeze(new Float64Array(new ArrayBuffer(64), 63, 0)) // No elements
Float64Array []

> Object.freeze(new Float64Array(new ArrayBuffer(64), 32, 2)) // Has elements
TypeError: Cannot freeze array buffer views with elements
```

### `Object.seal()`

禁止添加新的属性，已有属性被设置为non-configurable。现有属性如果`writable`为`true`，则仍可修改。

Syntax

> `Object.seal(obj)`

### `Object.preventExtensions()`

不能再添加新的属性。

### `Object.isExtensible()`

### `Object.isFrozen()`

### `Object.isSealed()`

### `Object.getOwnPropertyDescriptor()`

返回own property的descriptor。

Syntax

> `Object.getOwnPropertyDescriptor(obj, prop)`

### `Object.getOwnPropertyDescriptors()`

应用：

#### 浅复制

```javascript
Object.create(
  Object.getPrototypeof(obj),
  Object.getOwnPropertyDescriptors(obj)
)
```

#### 创建子类

```javascript
function SuperClass() {}
SuperClass.prototype = {};

function SubClass() {}
SubClass.prototype = Object.create(
  SuperClass.prototype,
  {}
)
```

### `Object.getOwnPropertyNames()`

返回一个包含所有属性的数组（包括non-enumerable属性，不包括Symbol类型）.

### `Object.getOwnPropertySymbols()`

返回一个包含所有Symbol类型属性的数组。

### `Object.keys()`

返回一个包含所有own、enumerable、string属性的数组。

polyfill

```javascript
// From https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys
if (!Object.keys) {
  Object.keys = (function() {
    'use strict';
    var hasOwnProperty = Object.prototype.hasOwnProperty,
      hasDontEnumBug = !({ toString: null }).propertyIsEnumerable('toString'),
      dontEnums = [
        'toString',
        'toLocaleString',
        'valueOf',
        'hasOwnProperty',
        'isPrototypeOf',
        'propertyIsEnumerable',
        'constructor'
      ],
      dontEnumsLength = dontEnums.length;

    return function(obj) {
      if (typeof obj !== 'function' && (tyoeof obj !== 'object' || obj === null)) {
        throw new TypeError('Object.keys called on non-object');
      }

      var result = [], prop, i;

      for (prop in obj) {
        if (hasOwnProperty.call(obj, prop)) {
          result.push(prop);
        }
      }

      if (hasDontEnumBug) {
        for (i = 0; i < dontEnumsLength; i++) {
          if (hasOwnProperty.call(obj, dontEnums[i])) {
            result.push(dontEnums[i]);
          }
        }
      }

      return result;
    }
  }())
}
```

### `Object.values()`

返回一个包含own enumerable string属性的值的数组。

### `Object.is()`

比较两个值。

```javascript
Object.is('foo', 'foo'); // true
Object.is(window, window); // true

Object.is([], []); // false
Object.is(obj1, obj2); // false

Object.is(null, null); // true

// 特殊例子
Object.is(0, -0); // false
Object.is(-0, -0); // true
Object.is(NaN, 0/0); // true
```

polyfill

```javascript
if (!Object.is) {
  Object.is = function(x, y) {
    // SameValue algorithm
    if (x === y) { // Steps 1-5, 7-10
      // Steps 6.b-6.e: +0 != -0
      return x !== 0 || 1 / x === 1 / y;
    } else {
      // Step 6.a: NaN == NaN
      return x !== x && y !== y;
    }
  }
}
```

### `Object.getPrototypeOf()`

返回内部[[Prototype]]属性的值。

### `Object.setPrototypeOf()`

给一个对象设置原型（内部[[Prototype]]属性的值）。

> **注意：** 更改一个对象的[[Prototype]]是一个非常慢的操作，即使现代Javascript引擎做了优化。如果对性能有要求的话，避免修改[[Prototype]]，使用`Object.create()`方法创建一个新对象。

polyfill

```javascript
// Only works in Chrome and FireFox, does not work in IE:
Object.setPrototypeOf = Object.setPrototypeOf || function(obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

[`Function`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
[`Object`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
[`ArrayBufferView`]: https://developer.mozilla.org/en-US/docs/Web/API/ArrayBufferView
[`AsyncFunction`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction
[`GeneratorFunction`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/GeneratorFunction
[`ArrowFunctions`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
