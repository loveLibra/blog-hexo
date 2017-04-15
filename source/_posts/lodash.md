title: lodash源码祭
date: 2017-04-03 22:08:37
tags: Lodash
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

# Array

## chunk
```
_.chunk(array, [size=1])
```
猜想下实现方法，功能就是将数组拆分成`arrar.length / size`个数组，每个数组`size`个元素，剩余的元素作为最后一个分组，数组操作中`slice`不改变原数组并实现数组切分:
```javascript
const chunk = (array, size = 1) => {
    let count = Math.ceil(array.length / size);

    let _chunk = new Array(count);

    let index = -1;
    let start = 0;

    while (++index < count) {
        _chunk[index] = array.slice(start, start += size);
    }

    return _chunk;
}
```
<!-- more -->
验证下，功能OK，再来看下lodash的实现方法:
* 首先，**对参数进行验证**
```javascript
size = Math.max(size, 0)
const length = array == null ? 0 : array.length
if (!length || size < 1) {
    return []
}
```
确保size非负以及length为合法值...

虽然处理了数组，但是需不需要考虑ArrayLike的Object伪装Array的情况
```javascript
_.chunk({a:1, b:2, length:2}, 2)
```
将得到一个包含两个undefined的数组的数组，是否加上数组判断是更nice呢:
```javascript
if (!length || !(array instanceof Array) || size < 1) {
    return [];
}
```

再看`import baseSlice from './.internal/baseSlice.js'`，lodash并未用Array.prototype.slice去做数组切割，而是自己写了一个，为啥？slice是基础特性啊，浏览器都支持的，咋不直接用的...实现上首先将`start`和`end`都转化为正值并做start <= end的验证，在算切割的数组length的时候，用了这样的语句，可以关注一下：
```javascript
length = start > end ? 0 : ((end - start) >>> 0)
```
`>>> 0`有什么用？Check [stackoverlfow](http://stackoverflow.com/questions/1822350/what-is-the-javascript-operator-and-how-do-you-use-it)...其实就是将值转化为32位无符号整数，即Array.length的合法值，参见[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/length)，非"数字"转化后值为0，比如:
```javascript
null >>> 0 // 0

'1' >>> 0 // 1 数字字符串将转化为对应的数字
```

## compact
```javascript
_.compact(array)
```
compact -> “压紧、简化”，这个就简单了，移除数组中所有的`falsey`值：
```javascript
const compact = array => {
    let _compact = [];
    if (array && array.length && array instanceof Array) {
        let index = -1;
        let length = array.length;

        while (index++ < length) {
            if (array[index]) {
                _compact.push(array[index]);
            }
        }
    }
    return _compact;
}
```
下意识的用`push`插入数组项，而源码惯用下标递增：`result[resIndex++] = value`。性能有影响？

## concat
```javascript
_.concat(array, [values])
```
实现的就是原生concat的功能，参数处理一下即可：
```javascript
const concat = (array, ...values) => {
    if (array && array instanceof Array) {
        return array.concat(...values);
    }
    return [];
}
```
Done! rest参数和数组析构教你做人...对比对比源码看看自己的实现漏了些啥...
* 基础方法实现es6提供的一些便利，包括arguments对象的参数拆分、数组flatten
* array参数可以接受非数组，若array非数组，[array]将作为基础数组参与运算
```javascript
return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));
```

关于**数组类型**的判断，我们上面用了`array instanceof Array`这种方式，lodash中`isArray`即`Array.isArray`，来看MDN关于isArray的Polyfill:
```javascript
if (!Array.isArray) {
    Array.isArray = function(arg) {
        return Object.prototype.toString.call(arg) === '[object Array]';
    };
}
```
如果toString能吐出`[object Array]`我们就认为他是一个数组，那`instanceof Array`和`isArray`有啥区别？
> When checking for Array instance, Array.isArray is preferred over instanceof because it works through iframes.
```javascript
let xArray = otherFrame.Array();

let arr = new xArray(1,2,3);

arr instanceof Array; // false

Array.isArray(arr); // true
```
显然，arr只是xArray(ohterFrame的Array)的instance而并非当前frame的Array的instance，而isArray不受此影响

## difference
```javascript
_.difference(array, [values])
```
生成一个从array中过滤掉values中所有元素的数组，即做个数组去重：
```javascript
const difference = (array, values) => {
    if (!values || values.length === 0) {
        return array;
    }

    let result = [];

    for (let i = 0; i < array.length; i++) {
        let the = array[i];

        if (values.indexOf(the) === -1) {
            result.push(the);
        }
    }

    return result;
};
```
源码如何实现，坑有点深啊...

## differenceBy
```javascript
_.differenceBy(array, [values], [iteratee=_.identity])
```
不同于difference的是，differenceBy接受第三个参数iteratee，会按照array中每项转化后的值进行去重，比如：
```javascript
_.differenceBy([2.1, 1.2], [2.3, 3.4], Math.floor); // [1.2]
```
另外，对于对象数组，iteratee可以为属性字符串，这其实是_.property的简写

不严谨实现方法：
```javascript
const differenceBy = (array, ...params) => {
    if (params.length === 1) {
        return difference(array, params);
    }

    let result = [];

    let ite = params.pop();

    let exclude = params.shift().map(i => ite(i));

    for (let i = 0; i < array.length; i++) {
        let computed = ite(array[i]);

        if (exclude.indexOf(computed)) {
            result.push(array[i]);
        }
    }

    return result;
}
```

还有一个difference方法...

## differenceWith
```javascript
_.differenceWith(array, [values], [comparator])
```
differenceWith通过指定comparator，改变默认比较方式去得到相应的去重数组

```javascript
const differenceWith = (array, ...params) => {
    if (params.length === 1) {
        return difference(array, params);
    }

    let result = [];

    let comparator = params.pop();

    let exclude = params.shift();

    let eLen = exclude.length;

    outer:
    for (let i = 0; i < array.length; i++) {
        let the = array[i];
        let index = 0;

        while (index++ < eLen) {
            if (comparator(the, exclude[index])) {
                continue outer;
            }
        }

        result.push(the);
    }

    return result;
}
```
上述三个difference方法在去重操作上统一，可以抽象出来个baseDifference基础方法，且上面我们只考虑了简单的值类型的比较，并未考虑引用类型和NaN这种特殊类型的情况
```javascript
/**
 * base difference
 * @param array [Array]
 * @param exclude [Array]
 * @param iteratee [function] 迭代器，对每个元素转化后进行比较
 * @param comparator [function] 比较器，比较方法
 * @param return [array]
 */
const isArray = Array.isArray;

const baseDifference = (array, exclude, iteratee, comparator) => {

    // 非数组或者数组为空
    if (!isArray(array) || !array.length) {
        return [];
    }

    // 非数组或为空直接返回源数组
    if (!isArray(exclude) || !exclude.length) {
        return array;
    }

    let result = [];
    let {length} = array;

    if (iteratee) {
        exclude = exclude.map(i => iteratee(i));
    }

    if (!comparator) {
        comparator = function(a, b) {
            if (a === b) {
                return true;
            }

            return false;
        }
    }

    let index = -1;

    outer:
    while (++index < length) {
        let val = array[index];

        let iteVal = iteratee ? iteratee(val) : val;

        if (iteVal === iteVal) {
            // 值类型

            let excludeLength = exclude.length;
            while (excludeLength--) {
                let excludei = exclude[excludeLength];

                if (comparator(iteVal, iteratee ? iteratee(excludei) : excludei)) {
                    continue outer;
                }
            }

            result.push(val);
        }
    }

    return result;
};
```

## drop
```javascript
_.drop(array, [n=1])
```
生成一个从array开头移除n个元素后剩余的元素的切片，相当于`array.slice(n)`，源数组不变
```javascript
const drop = (array, n = 1) => {
    if (!Array.isArray(array)) {
        return [];
    }

    return array.slice(n);
}
```
