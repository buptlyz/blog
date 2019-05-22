---
title: array-prototype-reduce
date: 2019-05-18 11:30:16
tags:
- JavaScript
- Array
- reduce
---

`reduce`方法对数组中的每个元素执行一个由我们定义的`reducer`函数（升序执行），将其结果汇总为单个返回值。

`reducer`函数接收4个参数：

1. Accumulator（acc）（累计器）
2. Current Value（cur）（当前值）
3. Current Index（idx）（当前索引）
4. Source Array（src）（源数组）

polyfill

```javascript
Array.prototype.reduce = function(callback) {
  var arr = this, len = arr.length;
  var idx = 0, value;
  if (arguments.length >= 2) {
    value = arguments[1];
  } else {
    while(idx < len && !(idx in arr)) {
      idx++;
    }
    if (idx >= len) {
      throw new TypeError('empty array');
    }
    value = arr[idx++];
  }
  while(idx < len) {
    if (idx in arr) {
      value = callback(value, arr[idx], idx, arr);
    }
    idx++;
  }
  return value;
};
```

### 示例

#### 按顺序执行Promise

```javascript
/**
 * Runs Promise from array of functions that can return promises in chained manner
 *
 * @param {array} arr - promise arr
 * @return {Object} promise object
 */
function runPromiseInSequence(arr, input) {
  return arr.reduce(
    (promiseChain, currentFn) => promiseChain.then(currentFn),
    Promise.resolve(input)
  );
}

runPromiseInSequence([getUser, getAuth, getData], {});
```

#### 功能型函数管道

```javascript
var double = x => x*2
var triple = x => x*3
var quadruple = x => x*4

var pipe = (...fns) => input => fns.reduce(
  (acc, fn) => fn(acc),
  input
);

var multiply6 = pipe(double, triple);
var multiply9 = pipe(triple, triple);
var multiply16 = pipe(quadruple, quadruple);
var multiply24 = pipe(quadruple, multiply6);
```
