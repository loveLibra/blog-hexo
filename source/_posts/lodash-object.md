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

## assignIn
```javascript
_.assignIn(object, [sources])
```
类似`assign`，但是也会将source中的继承熟悉合并过去，那就不要`hasOwnproperty`的判断咯？

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

## pickBy
```javascript
_.pickBy(object, [predicate=_.identity])
```
pick除满足predicate为truthy值的属性。predicate接受两个参数（value、key）
```javascript
const pickBy = (object, predicate) => {
    if (object == null) {
        return {};
    }

    if (typeof predicate !== 'function') {
        return object;
    }

    let result = {};
    for(let attr in object) {
        if (predicate(object[attr], attr)) {
            result[attr] = object[attr];
        }
    }

    return result;
}
```

## omit
```javascript
_.omit(object, [paths])
```
pick的逆操作，排除某些paths(字符串或数组)指定的属性(自身属性和继承属性)
```javascript
const omit = (object, paths) => {
    if (object == null) {
        return {};
    }

    if (paths === undefined) {
        return object;
    }

    if (!Array.isArray(paths)) {
        paths = [String(paths)];
    }

    let result = {};
    for(let attr in object) {
        if (paths.indexOf(attr) < 0) {
            result[attr] = object[attr];
        }
    }

    return result;
}
```

## omitBy
```javascript
_.omitBy(object, [predicate=_.identity])
```
排除**不**满足predicate的object的属性，predicate接受两个参数(value, key)
```javascript
const omitBy = (object, predicate) => {
    if (object == null || predicate === undefined) {
        return {};
    }

    if (typeof predicate !== 'function') {
        return object;
    }

    let result = {};
    for(let attr in object) {
        if (predicate(object[attr], attr)) {
            continue;
        }
        result[attr] = object[attr];
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

## get
```javascript
_.get(object, path, [defaultValue])
```
获取object指定path的值，如果设置了defaultValue，在value为undefined时返回defaultValue
```javascript
const get = (obejct, path, defaultValue) => {
    if (object == null || path === undefined) {
        return;
    }

    object = Object(object);

    if (Array.isArray(path)) {
        path = path.join('.');
    }

    path = String(path).replace(/\[([^\]]*)\]/g, '.$1').split('.'); // 处理[0]这种情况的路径访问

    let index = -1;
    let value;
    while (++index < path.length) {
        let cur = path[index];

        if (index === 0) {
            value = object[cur];
            continue;
        }

		value = value[cur];

        if (value === undefined) {
            break;
        }
    }

    if (value === undefined) {
        value = defaultValue;
    }

    return value;
}
```

## set
```javascript
_.set(object, path, value)
```
设置指定路径的值，若中间路径不存在就新建，对于索引(数字)的情况，新建数组，其他情况新建对象
```javascript
const set = (object, path, value) => {
    if (object == null || path === undefined) {
        return object;
    }

    object = Object(object);

    if (Array.isArray(path)) {
        path = path.join('.');
    }

    path = String(path).replace(/\[([^\]]*)\]/g, '.$1').split('.');

    let index = -1;
    let nest = object;

	let length = path.length;
    while (++index < length) {
        let cur = path[index];

        let val = nest[cur];

		if (index !== length - 1) {
			if (val === undefined) {
				val = /^(?:0|[1-9]\d*)$/.test(path[index + 1]) ? [] : {};
			}
		} else {
			val = value;
		}

		nest[cur] = val;

        nest = nest[cur];
    }

    return object;
}
```

## toPairs
```javascript
_.toPairs(object)
```
以对象自身可枚举的键和值组成的数组做为返回的数组的每项，如果是Map或者Set，直接返回entries
```javascript
const toPairs = object => {
    if (object == null) {
        return [];
    }

    const toString = Object.prototype.toString.call(object);

    if (toString === '[object Set]' || toString === '[object Map]') {
        return object.entries();
    }

    let res = [];
    Object.keys(object).forEach(key => {
        res.push([key, object[key]]);
    });

    return res;
}
```

## toPairsIn
```javascript
_.toPairsIn(object)
```
类似`toPairs`，可遍历出继承的枚举属性
```javascript
const toPairsIn = object => {
    if (object == null) {
        return [];
    }

    const toString = Object.prototype.toString.call(object);

    if (toString === '[object Set]' || toString === '[object Map]') {
        return object.entries();
    }

    let res = [];
    for (let key in object) {
        res.push([key, object[key]]);
    }

    return res;
}
```

## values
```javascript
_.values(object)
```
返回包含object自身可枚举属性值的数组
```javascript
const values = object => {
    if (object == null) {
        return [];
    }

    object = Object(object);

    let index = -1;
    let res = [];

    let keys = Object.keys(object);

    while (++index < keys.length) {
        res[index] = object[keys[index]];
    }

    return res;
}
```

## valuesIn
```javascript
_.valuesIn(object)
```
类似于`values`，可返回继承属性的值
```javascript
const valuesIn = object => {
    if (object == null) {
        return [];
    }

    object = Object(object);

    let index = -1;
    let res = [];

    for (let index in object) {
        res.push(object[index]);
    }

    return res;
}
```