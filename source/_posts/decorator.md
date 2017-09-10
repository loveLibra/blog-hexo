title: decorator
date: 2017-09-05 22:59:08
tags: ES6
---

**Decorator**修饰器 

类和属性的decorator目前在[Stage2](https://github.com/tc39/proposal-decorators)，babel转义需要在`.babelrc`中加入Plugin支持此功能:
```shell
npm install babel-plugin-transform-decorators-legacy --save-dev
```
```json
{
  "plugins": ["transform-decorators-legacy"]
}
```

## 类的修饰器
先来个修饰器babel编译前后的对比，看看decorator到底是个啥
```javascript
// souce
@test
class A {}

function test(target) {
  target.isTestable = true;
}
```
```javascript
// compiled
var _class;

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var A = test(_class = function A() {
  _classCallCheck(this, A);
}) || _class;

function test(target) {
  target.isTestable = true;
}
```
说白了，类的修饰器就是个函数，类会被扔到decorator中进行2次处理

## 方法修饰器
不仅可以对类定义修饰器，也可以对类的属性或者方法定义
```javascript
class A {
    @readonly name() {
        return 'hello world';
    }
}

/*
 * @param target 目标属性／方法
 * @param name 属性名
 * @param descriptor 属性描述对象
 */
function readonly(target, name, descriptor) {
    descriptor.writable = false;

    return descriptor;
}
```

## 函数修饰器
函数因hoist的因素，无法实现decorator的功能，可以使用高阶函数替代
```javascript
const decorator = (target) => {
    return function() {
        // do something else
        target.apply(this, arguments);
    }
}
```