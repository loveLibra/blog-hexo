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