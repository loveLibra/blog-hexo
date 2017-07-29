title: Lodash源码祭「Object」
date: 2017-07-20 22:33:33
tags: [Lodash, String]
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

# Object

## assign
```javascript
_.assign(object, [sources])
```
将source(可多个，参数列表)中可枚举的字符串属性赋到object，合并顺序是从左到右合并到object，同名会覆盖，方法返回object
```javascript
const assign = (object, ...sources) => {
    if (object == null) {
        object = {};
    }

    if (sources && sources.length > 0) {
        sources.forEach(source => {
            for (let key in source) {
                if (source.hasOwnProperty(key)) {
                    object[key] = source[key];
                }
            }
        });
    }
    // Or: 直接Object.assign，行为一致

    return object;
}
```

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

## keysIn
```javascript
_.keysIn(object)
```
类似`keys`，返回原型链上的所有属性
```javascript
const keysIn = object => {
    if (object == null) {
        return [];
    }

    object = Object(object);

    let attrs = [];

    for (let attr in object) {
		attrs.push(attr);
	}

    return attrs;
}
```

## pick
```javascript
_.pick(object, [paths])
```
pick出obejct中对应path（字符串或数组）的键值对，返回pick到的新对象
```javascript
const pick = (object, paths) => {
    if (object == null || paths === undefined) {
        return {};
    }

    if (!Array.isArray(paths)) {
        paths = [String(paths)];
    }

    let result = {};
    for (let i = 0; i < paths.length; i++) {
        let key = paths[i];

        if (object[key] === undefined) {
            continue;
        }

		result[key] = object[key];
    }
	
	return result;
}
```

## has
```javascript
_.has(object, path)
```
object**自身**是否包含path所提供的属性。path可以是字符串(单个属性或者以`.`分割的路径)，也可以是数组
```javascript
const has = (object, path) => {
    if (object == null || !path) {
        return false;
    }

    path = Array.isArray(path) ? path : path.split('.');

    let index = -1;
    while (++index < path.length) {
        let attr = path[index];

        if (object.hasOwnProperty(attr)) {
            object = object[attr];
        } else {
            return false;
        }
    }

    return true;
}
```

## hasIn
```javascript
_.hasIn(object, path)
```
跟`has`类似，但是可以查找原型链上的属性，可以直接用`in`操作符
```javascript
const hasIn = (object, path) => {
    if (object == null || !path) {
        return false;
    }

    path = Array.isArray(path) ? path : path.split('.');

    let index = -1;
    while (++index < path.length) {
        let attr = path[index];

        if (attr in object) {
            object = object[attr];
        } else {
            return false;
        }
    }

    return true;
}
```

## invert
```javascript
_.invert(object)
```
反转对象的键值对，反转后重复的键的值会被覆盖
```javascript
const invert = object => {
    if (object == null) {
        return {};
    }

    object = Object(object);

    let res = {};

    for (let key in object) {

        if (object.hasOwnProperty(key)) {
            res[object[key]] = key;
        }
    }

    return res;
}
```

## invertBy
```javascript
_.invertBy(object, [iteratee=_.identity])
```
与`invert`类似，参数可有iteratee函数去迭代每个值...另外，对于重复键的该方法不会覆盖，键转化成值后是一个数组
```javascript
const invertBy = (object, iteratee) => {
    if (!iteratee) {
        iteratee = value => value;
    }

    if (object == null) {
        return {};
    }

    object = Object(object);

    let res = {};

    for (let key in object) {
        if (object.hasOwnProperty(key)) {

            let _key = iteratee(object[key]);

            if (res[_key]) {
                res[_key].push(key);
            } else {
                res[_key] = [key];
            }
        }
    }

    return res;
}
```

## mapKeys
```javascript
_.mapKeys(object, [iteratee=_.identity])
```
保留对象的值，通过iteratee重新映射键，iteratee接受3个参数（value, key, object），返回map后的新对象
PS: 对于官网`_.mapKeys({a: 1, b:2})`输出{1:1, 2:2}持保留意见啊，这个方法直接用Object.keys去遍历也太随便了吧，怎么滴也应该原样返回啊，这块就按照自己的想法去实现啊
```javascript
const mapKeys = (object, iteratee) => {
    if (object == null) {
        return {};
    }

    object = Object(object);

    if (!iteratee || typeof iteratee !== 'function') {
        iteratee = (value, key) => key;
    }

    let res = {};
    for (let key in object) {
        if (object.hasOwnProperty(key)) {
            res[iteratee(object[key], key, object)] = object[key];
        }
    }

    return res;
}
```
如果值是个引用（ 比如：{a: {b: 'hello world'}} ），怎么解决map后的对象和源对象的值是指向同一内存的事实啊...

## mapValues
```javascript
_.mapValues(object, [iteratee=_.identity])
```
同`mapKeys`，会映射值
```javascript
const mapValues = (object, iteratee) => {
    if (object == null) {
        return {};
    }

    object = Object(object);

    if (!iteratee || typeof iteratee !== 'function') {
        iteratee = (value, key) => value;
    }

    let res = {};
    for (let key in object) {
        if (object.hasOwnProperty(key)) {
            res[key] = iteratee(object[key], key, object);
        }
    }

    return res;
}
```