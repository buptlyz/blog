---
title: 函数式编程第二章——高阶JavaScript
date: 2019-05-17 23:40:14
tags:
- 函数式编程
- functional programming
---

## 2.1 为什么要使用JavaScript

## 2.2 函数式与面向对象的程序设计

面向对象的应用程序大多是命令式的，因此在很大程度上依赖于使用基于对象的封装来保护其自身和继承的可变状态的完整性，再通过实例方法来暴露或修改这些状态。

其结果是，对象的数据与其具体的行为以一种内聚的包裹的形式紧耦合在一起。

函数式编程不需要对调用者隐藏数据，通常使用一些更小且非常简单的数据类型。由于一切都是不可变的，对象都是可以直接拿来使用的，而且是通过定义在对象作用域外的函数来实现的。

换句话说，数据与行为是松耦合的。

```javascript
class Person {
  constructor(firstname, lastname, ssn) {
    this._firstname = firstname;
    this._lastname = lastname;
    this._ssn = ssn;
    this._address = null;
    this._birthYear = null;
  }

  get ssn() {
    return this._ssn;
  }

  get firstname() {
    return this._firstname;
  }

  get lastname() {
    return this._lastname;
  }

  get address() {
    return this._address;
  }

  get birthYear() {
    return this._birthYear;
  }

  set birthYear(year) {
    this._birthYear = year;
  }

  set address(address) {
    this._address = address;
  }

  toString() {
    return `Person(${this._firstname}, ${this._lastname})`;
  }

  peopleInSameCountry(friends) {
    var result = [];
    for (let friend of friends) {
      if (this.address.country === friend.address.country) {
        result.push(friend);
      }
    }
    return result;
  }
}

class Student extends Person {
  constructor(firstname, lastname, ssn, school) {
    super(firstname, lastname, ssn);
    this._school = school;
  }

  get school() {
    return this._school;
  }

  studentInSameCountryAndSchool(friends) {
    var closeFriends = super.peopleInSameCountry(friends);
    var result = [];
    for (let friend of closeFriends) {
      if (friend._school === this._school) {
        result.push(friend);
      }
    }
    return result;
  }
}
```

函数式

```javascript
function selector(country, school) {
  return function(student) {
    return student.address.country() === country &&
      student.school() === school;
  }
}
function findStudentsBy(friends, selector) {
  return friends.filter(selector);
}
findStudentsBy([...], selector('China', 'BUPT'));
```
