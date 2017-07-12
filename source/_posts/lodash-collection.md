title: Lodash源码祭(Collection)
date: 2017-07-12 19:05:36
tags: [Lodash, Collection]
---
# Collection

## countBy
```javascript
_.countBy(collection, [iteratee=_.identity])
```
collection中统计各项的数目，也可指定iteratee进行迭代处理
```javascript
const countBy = (collection, iteratee) => {
    let result = {};

    collection.forEach(key => {

		let ite = iteratee ? iteratee(key) : key;

        if (typeof result[ite] === 'undefined') {
            result[ite] = 1;
        } else {
            result[ite] += 1;
        }
    });

    return result;
}
```
reduce?

## forEach
```javascript
_.forEach(collection, [iteratee=_.identity])
```
遍历元素，iteratee接受三个参数`value`,'key','collection'。但是对于对象的each，官方更推荐使用`forIn`或者`forOwn`
```javascript
const forEach = (collection, iteratee) => {
    let isArr = Array.isArray(collection);

    if (isArr) {
        collection.forEach(iteratee);
    } else {
        // 对象情况，用forin遍历，顺序不可控
        Object.keys(collection).forEach(key => {
            iteratee(collection[key], key, collection);
        });
    }

    return collection;
}
```
有一点忽略了，某一次iteratee执行`return false`是可以终结整个的循环的，原生的就没有此功能了，因此在上面的基础上要略做改造
```javascript
const forEach = (collection, iteratee) => {
    let isArr = Array.isArray(collection);

    if (isArr) {
        let length = collection.length;
        let index = -1;

        while (++index < length) {
            let res = iteratee(collection[index], index, collection);

            if (res === false) {
                break;
            }
        }
    } else {
        // 对象情况，用forin遍历，顺序不可控
        let keys = Object.keys(collection);
        let length = keys.length;
        let index = -1;

        while (++index < length) {
            let key = keys[index];
            let res = iteratee(collection[key], key, collection);

            if (res === false) {
                break;
            }
        }
    }

    return collection;
}
```

## forEachRight
```javascript
_.forEachRight(collection, [iteratee=_.identity])
```
与forEach类似，只是从尾向头迭代，不赘述

## every
```javascript
_.every(collection, [predicate=_.identity])
```
若collection所有项通过predicate check则返回true，否则返回false。需要注意的是，对空的collection返回值为true，约定
```javascript
const every = (collection, predicate) => {
    let isArr = Array.isArray(collection);

    let keys = !isArr && Object.keys(collection);
    let length = isArr ? collection.length : keys.length;

    // 空collection返回true
    if (length === 0) {
        return true;
    }

    let index = -1;
    while (++index < length) {
        let key;
        if (isArr) {
            key = index;
        } else {
            key = keys[index];
        }

        if (!predicate(collection[key], key, collection)) {
            return false;
        }
    }

    return true;
}
```

## filter
```javascript
_.filter(collection, [predicate=_.identity])
```
满足preficate为truthy的元素的数组
```javascript
const filter = (collection, predicate) => {
    let isArr = Array.isArray(collection);

    if (isArr) {
        return collection.filter(predicate);
    } else {
        let result = [];

        let index = -1;
        let keys = Object.keys(collection);

        let length = keys.length;

        while (++index < length) {
            let key = keys[index];

            if (predicate(collection[key], key, collection)) {
				result.push({
					[key]: collection[key]
				});
            }
        }

        return result;
    }
}
```

## reject
```javascript
_.reject(collection, [predicate=_.identity])
```
`filter`的反向操作，过滤掉predicate为truthy值的想，返回剩余项的数组集合
```javascript
const filter = (collection, predicate) => {
    let result = [];

    forEach(collection, (item, index, collection) => {
        if (predicate(item)) {
            return;
        }


		result.push(Array.isArray(collection) ? item : {
			[index]: item
		});
    });

    return result;
}
```

## find
```javascript
_.find(collection, [predicate=_.identity], [fromIndex=0])
```
从指定的fromIndex开始查找满足predicate的项并返回，若无则返回undefined
```javascript
const find = (collection, predicate, fromIndex) => {
    let isArr = Array.isArray(collection);

    let keys = Object.keys(collection); // turn array key to string
    let length = keys.length;
    let index = fromIndex ? fromIndex - 1 : -1;

    while (++index < length) {
        let key = keys[index];

        if (isArr) {
            key = key >>> 0; // turn key to number
        }

        if (predicate(collection[key], key, collection)) {

           return isArr ? collection[key] : {
                [key]: collection[key]
            };
        }
    }

    return;
}
```

## findLast
```javascript
_.findLast(collection, [predicate=_.identity], [fromIndex=collection.length-1])
```
与find类似，只是从右往左查找
```javascript
const findLast = (collection, predicate, fromIndex) => {
    let isArr = Array.isArray(collection);

    let keys = Object.keys(collection); // turn array key to string
    let length = keys.length;
    let index;

    // 下标为范围
    if (fromIndex !== undefined) {
        index = fromIndex < 0 ? Math.max(fromIndex + length, 0) : Math.min(length - 1, fromIndex);
	} else {
		index = length - 1;
	}

    index += 1;
    while (--index > 0) {
        let key = keys[index];

        if (isArr) {
            key = key >>> 0; // turn key to number
        }

        if (predicate(collection[key], key, collection)) {

           return isArr ? collection[key] : {
                [key]: collection[key]
            };
        }
    }

    return;
}
```

## groupBy
```javascript
_.groupBy(collection, [iteratee=_.identity])
```
根据指定iteratee分组，iteratee迭代每个元素后的值为键，对应的值为之前的元素对应的值所组成的数组，这个借助之前的`forEach`可以很轻松的实现，但这个对单层数组还可以一用，对多层数组或者对象完全鸡肋啊...
```javascript
const groupBy = (collection, iteratee) => {
    let result = {};

    forEach(collection, function(val, key, collection) {
        let tag = iteratee(val, key, collection);

        tag = tag.toString();

        if (!result[tag]) {
            result[tag] = [];
        }

        result[tag].push(val);
    });

    return result;
}
```

## includes
```javascript
_.includes(collection, value, [fromIndex=0])
```
检查collection中从指定的fromIndex开始是否包含value值，此处collection可以为数组、对象和**字符串**，字符串按照`substring`进行查找
```javascript
const includes = (collection, value, fromIndex = 0) => {
    let length;

    let isStr = typeof collection === 'string';

    if (isStr) {
        length = collection.length;
    } else {
        length = Object.keys(collection).length;
    }

    if (fromIndex < 0) {
        fromIndex += length;
    }

    fromIndex = (fromIndex >= length - 1) ? (length - 1) : fromIndex;

    if (isStr) {
        return collection.substring(fromIndex).indexOf(value) > -1;
    } else {
        let ite = -1;
        let _include = false;
        forEach(collection, function(val) {
            ite++;

            if (ite < fromIndex) {
                return;
            } else {
                if (val === value) {
                    _include = true;
                    return false; // take it over
                }
            }
        });

        return _include;
    }
}
```

## keyBy
```javascript
_.keyBy(collection, [iteratee=_.identity])
```
生成键值对，键名由iteratee迭代每个元素后生成，值为collection对应项，注意iteratee只接受一个参数value，并且对于迭代后重复的键名会覆盖，同样使用`forEach`可以简单的完成这个功能
```javascript
const keyBy = (collection, iteratee) => {
	let result = {};
	forEach(collection, (val, key, collection) => {
		result[iteratee(val)] = val;
	});

	return result;
}
```

## invokeMap
```javascript
_.invokeMap(collection, path, [args])
```
path可接收`Array`、`String`和`Function`...接收数组参数是什么套路，没理解？先以函数为例吧，函数的this在迭代中绑定到collection中的每个元素。最后，还有一个可选参数args，可作为参数传给path
```javascript
const invokeMap = (collection, path, ...args) => {
    let result = [];

    forEach(collection, val => {
        result.push(path.apply(val, args));
    });

    return result;
}
```

## orderBy
```javascript
_.orderBy(collection, [iteratees=[_.identity]], [orders])
```
TODO留坑

## sample
```javascript
_.sample(collection)
```
任意返回collection中的一个抽样值
```javascript
const sample = collection => {
    let keys = Object.keys(collection);

    let {length} = keys;

    let rand = keys[Math.floor((Math.random() * length))]; // 0 ~ length-1

    return collection[Array.isArray(collection) ? ~~rand : rand];
}
```

## sampleSize
```javascript
_.sampleSize(collection, [n=1])
```
获取n个collection中的随机元素，扩展下`sample`方法，然后返回的是个数组
```javascript
const sampleSize = (collection, n = 1) => {
    let result = [];
    let keys = Object.keys(collection)

    let {length} = keys;

    if (n > length) {
        n = length;
    }

    if (n < 1) {
        n = 1;
    }

    // 生成n个不同的随机数
    let i = 0;
    let chosed = [];
    while (i < n) {
        let rand = keys[Math.floor((Math.random() * length))]; // 随机数

        // 保证键值不同
        if (chosed.indexOf(rand) > -1) {
            continue;
        }

		i++;

        chosed.push(rand);

        result.push(collection[Array.isArray(collection) ? ~~rand : rand])
    }

    return result;
}
```

## map
```javascript
_.map(collection, [iteratee=_.identity])
```
返回映射后的数组
```javascript
const map = (collection, iteratee) => {
	let result = [];

	forEach(collection, (val, key, collection) => {
		result.push(iteratee(val, key, collection));
	});
	
	return result;
}
```

## size
```javascript
_.size(collection)
```
可求数组、对象和字符串的尺寸，包括Map和Set
```javascript
const size = collection => {
    if (typeof collection === 'string') {
        return collection.length;
    }
    
    let tag = collection.toString();
    if (tag === '[object Set]' || tag === '[object Map]'){
        return collection.size;
    }

    return Object.keys(collection).length; // 数组或者对象
}
```
源码一直将包含`length`的非函数变量作为arrayLike，包括数组、字符串和含length属性的类数组对象，可以直接访问到length属性作为size返回。需要修正一下，要不然对于`{a: 1, length: 1}`这样的类数组会出现问题
```javascript
const size = collection => {
    if (isArrayLike(collection)) {
        return collection.length;
    }

    let tag = collection.toString();

    if (tag === '[object Set]' || tag === '[object Map]'){
        return collection.size;
    }

    return Object.keys(collection).length;
}
```

## some
```javascript
_.some(collection, [predicate=_.identity])
```
collection中任一元素predicate返回truthy值，循环**终止**并返回true，否则返回false
```javascript
const some = (collection, predicate) => {
    let _some = false;

    forEach(collection, (val, key, collection) => {
        if (predicate(val, key, collection)) {
            _some = true;
            return false;
        }
    });

    return _some;
}
```