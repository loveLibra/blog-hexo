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