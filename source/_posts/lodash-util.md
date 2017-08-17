title: Lodash源码祭「Util」
date: 2017-08-17 21:32:26
tags: [Lodash, String]
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

# Util

## attempt
```javascript
_.attempt(func, [args])
```
尝试执行函数，args为函数执行时的参数...会返回执行的结果或者捕获到Error对象
```javascript
const attempt = (func, ...args) => {
    try {
        return func(...args);
    } catch(e) {
        return e; // 源码这块，给了个判定是否是Error对象，如果不是转化为Error
    }
}
```