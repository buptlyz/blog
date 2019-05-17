---
title: jest笔记
date: 2019-05-17 10:55:14
tags:
- jest
- 单元测试
---

### 支持的`Matchers`

* `toBe`&& `toEqual`
* 真值，结果同if(/\*statement\*/)
* 数值，`toBeGreaterThan`, `toBeGreaterThanOrEqual`, `toBeLessThan`, `toBeLessThanOrEqual`, `toBe`, `toEqual`
* String，`toMatch`
* 数组和Iterables，`toContain`
* 异常，`toThrow`

### 异步测试

* 回调，`done`
* `Promise`，`then`里直接`expect`，`catch`需要加一条语句`expect.assertions(1);`
* `.resolves`/`.rejects`
* `Async`/`Await`，类似于`Promise`

### `describe`

`jest`会先调用`describe`，因此一些有意义的逻辑应该放在`beforeEach`、`afterEach`里。
`jest`会按照`test`定义的顺序来执行`test`。

```javascript
describe('1', () => {
  console.log('1-describe');
  beforeAll(() => console.log('1-once-before'));
  afterAll(() => console.log('1-once-after'));
  beforeEach(() => console.log('1-each-before'));
  afterEach(() => console.log('1-each-after'));

  test('1', () => {
    console.log('1-test');
    expect(1).toBe(1);
  })

  describe('2', () => {
    console.log('2-describe');
    beforeAll(() => console.log('2-once-before'));
    afterAll(() => console.log('2-once-after'));
    beforeEach(() => console.log('2-each-before'));
    afterEach(() => console.log('2-each-after'));

    test('2', () => {
      console.log('2-test');
      expect(2).toBe(2);
    })
  });
})

//   1-describe
//   2-describe
//   1-once-before
//   1-each-before
//   1-test
//   1-each-after
//   2-once-before
//   1-each-before
//   2-each-before
//   2-test
//   2-each-after
//   1-each-after
//   2-once-after
//   1-once-after
```

### 调试技巧

当某个`test`出错时，可以使用`test.only`来检查是它本身的错误，还是与其他`test`有关。

如果`test.only`时通过，和其他`test`一起就不通过，猜测很可能与共享状态有关。可以在`beforeEach`里重置共享状态来验证。

```javascript
test.only(() => {
  expect(1).toBe(1);
})

test(() => {
  expect('aaa').toMatch(/a/);
})
```
