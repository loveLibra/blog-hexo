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

## bindAll
```javascript
_.bindAll(object, methodNames)
```
将object中的methodNames的执行上下文绑定在object上。methodNames可为字符串和字符串的数组。执行的返回结果为object
```javascript
const bindAll = (object, methods) => {
    if (typeof methods === 'string') {
        methods = [methods];
    }

    if (!Array.isArray(methods)) {
        return object;
    }

    methods.forEach(method => {
        let fn = obejct[method].bind(object);

        object[method] = fn;
    });

    return object;
}
```

## flow
```javascript
_.flow([funcs])
```
组合函数的执行，前一个函数的返回值作为后一个函数的参数，返回值为组合的新函数，函数的列表的的执行上下文全部指向新生成的函数
```javascript
const flow = (funcs) => {
    const length = funcs ? funcs.length : 0;

    // 检查调用栈上是否全是函数
    let index = length
    while (index--) {
        if (typeof funcs[index] !== 'function') {
            throw new TypeError('Expected a function...');
        }
    }

    return function(...args) {
        let index = 0;

        // 如果没有funcs，则返回参数列表的第一个参数
        if (length === 0) {
            return args[0];
        }

        let res = funcs[index].apply(this, args);

        while (++index < length) {
            res = funcs[index].call(this, res);
        }

        return res;
    }

    return fn;
}
```

## identity
```javascript
_.identity(value)
```
返回第一个参数是什么鬼...
```javascript
const identity = (...value) => {
    return value[0];
}
```

## noop
```javascript
_.noop()
```
返回undefined的函数
```javascript
const noop = _ => {};
```

## nthArg
```javascript
_.nthArg([n=0])
```
生成一个返回第n个参数的函数...若n为负数，则从参数列表尾部开始算
```javascript
const nthArg = (n = 0) => {
    return (...args) => {
        let {length} = args;

        if (!length) {
            return;
        }

        return n < 0 ? args[length + n] : args[n];
    }
}
```

## over
```javascript
_.over([iteratees=[_.identity]])
```
生成一个函数，对函数的arguments，调用各iteratees，返回执行的结果
```javascript
const over = iteratees => {
    if (!Array.isArray(iteratees)) {
        iteratees = [iteratees];
    }

    return (...args) => {
        let res = [];

        iteratees.forEach((iteratee, index) => {
            res[index] = typeof iteratee === 'function' ? iteratee.apply(null, args) : undefined;
        });

        return res;
    }
}
```

## property
```javascript
_.property(path)
```
生成一个返回object指定path的属性的值的函数...就是把object的get包装了下
```javascript
const property = path => {
    return object => {
        return get(object, path);
    };
}
```

## range
```javascript
_.range([start=0], end, [step=1])
```
返回一个步长为step，从start到end(不包括end)区间内数字的数组;
```javascript
const range = (...args) => {
    let start;
    let end;
    let step;

    switch (args.length) {
        case 1:
            end = args[0];

            start = 0;

            step = end < start ? -1 : 1;
            break;
        case 2:
            [start, end] = args;
            step = 1;
            break;
        case 3:
            [start, end, step] = args;
            break;
    }

    if (start === end) {
        return [];
    }

    if (step === 0) {
        return new Array(end - start).fill(start);
    }

    let fill = start;
    let res = [];

	while (step < 0 ? fill > end : fill < end) {
        res.push(fill);

        fill += step;
    };

    return res;
}
```