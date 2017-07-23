title: Lodash源码祭「Object」
date: 2017-07-20 22:33:33
tags: [Lodash, String]
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

# Object

## forIn
```javascript
_.forIn(object, [iteratee=_.identity])
```
遍历所有可枚举的`字符串`(非symbol)键和对应的值，iteratee接受三个参数：value、key、object...跟数组的forEach一样，`return false`会退出遍历
```javascript
const forIn = (object, iteratee) => {
    if (object == null) {
        return object;
    }

    object = Object(object);

    //let attrs = Object.keys(object); // 获取所有可枚举属性
    let attrs = [];

    // 通过原生for in获取所有可枚举的属性
	for (let attr in object) {
		attrs.push(attr);
	}
    // 或者通过如下方式获取，但是要保存object副本
    //while (object) {
    //    let attr = Object.keys(object);

    //    attrs = [...attrs, ...attr];

    //    object = Object.getPrototypeOf(object);
    //}

    let index = -1;

    while (++index < attrs.length) {
        let attr = attrs[index];

        if (iteratee(object[attr], attr, object) === false) {
            break;
        }
    }

    return object;
}
```

## forOwn
```javascript
_.forOwn(object, [iteratee=_.identity])
```
类似于`forIn`，但是只遍历自己的可枚举属性，直接用`Object.keys`就可以啦
```javascript
const forOwn = (object, iteratee) => {
    if (object == null) {
        return object;
    }

    object = Object(object);

    let attrs = Object.keys(object);

    let index = -1;

    while (++index < attrs.length) {
        let attr = attrs[index];

        if (iteratee(object[attr], attr, object) === false) {
            break;
        }
    }

    return object;
}
```

## keys
```javascript
_.keys(object)
```
返回可枚举的键名。非对象变量会强制转化为对象
```javascript
const keys = object => {
    if (object == null) {
        return [];
    }

    object = Object(object);

    return Object.keys(obejct);
}
```