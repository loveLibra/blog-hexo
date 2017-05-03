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

    n = n < 0 ? 0 : n;

    return array.slice(n);
}
```

同理，可以从右侧移除n个元素...

## dropRight
```javascript
_.dropRight(array, [n=1])
```

与drop类似，从右侧移除n个元素后返回剩余元素的切片即可

```javascript
const dropRight = (array, [n = 1]) => {
    if (!Array.isArray(array)) {
        return [];
    }

    n = n < 0 ? 0 : n;

    let {length} = array;

    return array.slice(0, length - n);
    //return array.reverse().slice(n).reverse();
}
```

在某些情况下，我们需要drop的可能不是从头或者从尾的N个元素这么简单，而是需要从满足某一条件的元素开始，此时就需要用到`dropWhile`和`dropRightWhile`

## dropWhile 和 dropRightWhile
```javascript
_.drop[Right]While(array, [predicate=_.identity])
```
接受参数`predicate`，接受三个参数`(value, key, array)`，从左/右找到满足条件的第一个元素后返回切片
```javascript
const baseDropWhile = (array, predicate, fromRight) => {
    let {length} = array;

    if (!length || !Array.isArray(array) ||
        !predicate || typeof predicate !== 'function') {
        return [];
    }

    let index = fromRight ? length : -1;

    // nice code
    while ((fromRight ? index-- : ++index < length) && predicate(array[index], index, array)) {}

    // 注意从右slice时index的值需要+1
    // slice(start, end)是不包括end元素的
    return fromRight ? array.slice(0, index + 1) : array.slice(index);
}
```

## fill
```javascript
_.fill(array, value, [start=0], [end=array.length])
```
将array从start到end位置填充value

```javascript
const fill = (array, value, start = 0, end = array.length >>> 0) => {

    let {length} = array;

    if (!length) {
        return [];
    }

    // 确保正确的start和end
    if (start < 0) {
        start = -start > length ? 0 : (length + start);
    }

    if (end < 0) {
        end += length;
    }

    end = start > end ? 0 : end;

    while (start < end) {
        array[start++] = value;
    }

    return array;
};
```

## findLastIndex
```javascript
_.findIndex(array, [predicate=_.identity], [fromIndex=0])
```
找到array中满足predicate返回为true值的条件的第一个元素的index，可指定查找的起始索引
```javascript
const findIndex = (array, predicate, fromIndex = 0) => {
    let {length} = array;

    if (!length) {
        return -1;
    }

    if (!predicate || typeof predicate !== 'function') {
        return -1;
    }

    let index = fromIndex >> 0;

    if (index < 0) {
        index += length;
    }

    if (index > length) {
        return -1;
    }

    while (index < length) {
        if (predicate(array[index])) {
            return index;
        }
        index++;
    }

    return -1;
}
```
同理，也可用findLastIndex从数组尾部找第一个满足条件的元素的index，不赘述

另外，lodash也提供了一些针对predicate的快捷方法，例如：predicate为字符串时，默认使用_.property(predicate)去做条件判断

## flatten
```javascript
_.flatten(array)
```
扁平array，一层
```javascript
const flatten = (array) => {
    let {length} = array;

    if (!length || !Array.isArray(array)) {
        return [];
    }

    let result = [];
    let index = -1;
    while (++index < length) {
        let item = array[index];

        if (Array.isArray(item)) {
           result.push(...item);
        } else {
            result.push(item);
        }
    }

    return result;
}
```

关于flatten，还有2个方法flattenDeep(array)和flattenDepth(array, [depth=1])，前者递归扁平所有嵌套数组，后者可指定扁平层级，可以实现baseFlatten基础功能
```javascript
const baseFlatten = (array, depth = 1, result = []) => {
    let {length} = array;

    if (!length) {
        return [];
    }

    let index = -1;
    while(++index < length) {
        let item = array[index];
        if (depth > 0) {
            if (Array.isArray(item)) {
                baseFlatten(item, depth - 1, result);
            } else {
                result.push(item);
            }
        }
    }

    return result;
}
```

flattenDepth即直接调用baseFlatten并传入array和depth参数；flattenDeep的depth为无穷大，lodash使用`const INFINITY = 1 / 0`重新算出来Infinity，怕覆盖么

## fromPairs
```javascript
_.fromPairs(pairs)
```
将[[key, value]...]转化为键值对对象{key: value...}
```javascript
const fromPairs = pairs => {
    let {length} = pairs;

    if (!length) {
        return;
    }

    let index = -1;

    let result = {};
    while (++index < length) {
        let item = pairs[index];

        let key = item[0];
        let val = item[1];

        result[key] = val;
    }

    return result;
}
```

## head
```javascript
_.head(array)
```
获取数组的第一个元素
```javascript
const head = array => {
    if (array && array.length) {
        return array[0];
    }
    return;
}
```

## indexOf
```javascript
_.indexOf(array, value, [fromIndex=0])
```
从array的指定位置fromIndex开始查找value值的索引，若无则返回-1
```javascript
const indexOf = (array, value, fromIndex = 0) => {
    let length = array ? array.length : 0;

    if (!length) {
        return -1;
    }

    // index转化为正整数
    let index = fromIndex >> 0;

    if (index < 0) {
        index = Math.max(index + length, 0);
    }

    // 需要考虑NaN的情况
    if (value === value) {
        while (index < length) {
            if (array[index] === value) {
                return index;
            }

            index++;
        }
    } else {
        while (index < length) {
            if (isNaN(array[index])) {
                return index;
            }

            index++;
        }
    }


    return -1;
}
```

## initial
```javascript
_.initial(array)
```
获取数组除最后一个元素外的切片
```javascript
const initial = array => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    return array.slice(0, -1);
}
```

## intersection
```javascript
_.intersection([arrays])
_.intersectionBy([arrays], [iteratee=_.identity])
_.intersectionWith([arrays], [comparator])
```
取数组的元素交集，并可以通过iteratee和comparator指定元素预处理或者比较方法来纠正计算结果，此处可以抽象下基础方法
```javascript
const include = (array, value, comparator) => {
    if (comparator) {
        let {length} = array;

        while (length--) {
            if (comparator(array[length], value)) {
                return true;
            }
        }
    } else {
        return array.indexOf(value) > -1
    }

    return false;
};

const baseIntersection = (arrays, iteratee, comparator) => {
    let {length} = arrays;

    if (length < 2) {
        return [];
    }

    let first = arrays.shift();

    let result = [];

    let index = -1;

    while (++index < first.length) {
        let item = first[index];

        let computed = iteratee ? iteratee(item) : item;

        let i = -1;

        let has = 0;

        while (++i < length - 1) {
            let inner = arrays[i];

            iteratee && (inner = inner.map(iteratee));

            if (include(inner, computed, comparator)) {
                has++;
            } else {
                continue;
            }
        }

        if (has === length - 1) {
            result.push(item);
        }
    }

    return result;
}
```

## join
```javascript
_.join(array, [separator=','])
```
array join
```javascript
const join = (array, separator = ',') => {
    let length = array && array.length;

    length = length || 0;

    if (length) {
        return array.join(separator);
    }
    return '';
}
```

## last
```javascript
_.last(array)
```
获取数组的最后一个元素
```javascript
const last = array => {
    let length = array && array.length;

    length = length || 0;

    if (length) {
        return array.slice(-1);
    }

    return;
}
```

## lastIndexOf
```javascript
_.lastIndexOf(array, value, [fromIndex=array.length-1])
```
获取最后一个匹配的元素的index，可指定查找位置，默认length-1，即从后往前查找
```javascript
const lastIndexOf = (array, value, fromIndex) => {
    let length = array && array.length;

    if (!length) {
        return -1;
    }

    fromIndex = fromIndex || length - 1;

    let index = indexOf(array.reverse(), value, (length - 1 - fromIndex)); // 调indexOf方法查找反转的数组

    return index === -1 ? -1 : ((length - 1) - index);
}
```

## nth
```javascript
_.nth(array, [n=0])
```
获取数组的第n个元素，n可为负数
```javascript
const nth = (array, n = 0) => {
    let length = array && array.length;

    if (!length) {
        return;
    }

    // 转化or验证?

    // 数字
    // n < maxinter
    // /^(?:0|[1-9]\d*)$/ 验证无符号整数
    // n > -1 && n < length
    // n % 1 === 0
    n = n >> 0;

    if (n < 0) {
        n = n + length;
    }

    if (n > length - 1 || n < 0) {
        return;
    }

    return array[n];
}
```

## pullAll
```javascript
_.pullAll(array, values)
```
移除array中所有values包含的值，会改变目标数组，接受参数values为数组
```javascript
const pullAll = (array, values) => {
    let {length} = array;

    if (!length) {
        return [];
    }

    values.forEach(value => {
        while (true) {

            let index = array.indexOf(value);

            if (index === -1) {
                break;
            } else {
                array.splice(index, 1);
            }
        }
    });

    return array;
}
```

## pull
```javascript
_.pull(array, [values])
```
类似pullAll，只是接受的移除的参数为参数列表

```javascript
const pull = (array, ...values) => {
    return pullAll(array, values);
}
```

## pullAllBy
```javascript
_.pullAllBy(array, values, [iteratee=_.identity])
```
使用iteratee迭代每个元素进行后进行比较排除
```javascript
const pullAllBy = (array, values, iteratee) => {
    if (iteratee) {

        values.forEach(value => {
            while (true) {

                let index = array.map(iteratee).indexOf(iteratee(value));

                if (index === -1) {
                    break;
                } else {
                    array.splice(index, 1);
                }
            }
        });
	} else {
		return pullAll(array, values)
	}

	return array;
}
```

## pullAllWith
```javascript
_.pullAllWith(array, values, [comparator])
```
按指定比较规则进行排除
```javascript
const pullAllWith = (array, values, comparator) => {
    if (comparator) {

        values.forEach(value => {

            let index = -1;
			let {length} = array;
			let removes = [];
            while (++index < length) {
                if (comparator(value, array[index])) {
					removes.push(index);
                }
            }

			// removes为升序index的序列
			let offset = 0;
			removes.forEach(i => {
				array.splice(i - offset++, 1);
			})

        });
    } else {
        return pullAll(array, values);
    }

    return array;
}
```

## pullAt
```javascript
_.pullAt(array, [indexes])
```
移除指定位置的元素，并返回移除元素的数组
```javascript
const pullAt = (array, indexes = []) => {
    let {length} = array;

    if (!array) {
        return [];
    }

    let offset = 0; // index偏移修正    

    let result = [];

    // 从小到大排序，方便进行偏移修正
    indexes.sort().forEach(i => {
        if (!(/^\d+$/).test(i + '')) {
            return;
        }

        if (i < 0) {
            i += length;
        }

        if (i >= length) {
            return;
        }

        result.push(...array.splice(i - offset++, 1));
    });

    return result;
}
```

## remove
```javascript
_.remove(array, [predicate=_.identity])
```
移除array中满足predicate为truthy的元素，predicate接收三个参数(value, index, array)；remove返回删除的元素，并会改变array
```javascript
const remove = (array, predicate) => {
    let length = array ? array.length : 0;

    let result = [];

    if (!length || !predicate) {
        return result;
    }


    let indexes = [];
    result = array.filter((item, index) => {
        if (predicate(item, index, array)) {
            indexes.push(index);

            return true;
        }

        return false;
    });

    pullAt(array, indexes);

    return result;
}
```

## reverse
```javascript
const reverse = array => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    return array.reverse();
}
```

## slice
```javascript
_.slice(array, [start=0], [end=array.length])
```
slice的功能我们在实现chunk的时候就已经完整实现过了，在原生slice方法的基础上，做一些参数的限定和转化
```javascript
const slice = (array, start, end) => {
    let length = array ? array.length : 0;

    if (!lenght) {
        return [];
    }

    start = start ? start : 0;
    end = end ? end : length;

    if (start < 0) {
        start = -start > length ? 0 : (start + length);
    }

    if (end < 0) {
        end += length;
    }

    end = end > length ? length : end;

    if (start > end) {
        return [];
    }

    return array.slice(start, end);
}
```

## sortedIndex
```javascript
_.sortedIndex(array, value)
```
在已排序的array数组中，插入value后维持顺序，确定并返回最小的插入位置索引
```javascript
// 二分查找
const sortedIndex = (array, value) => {
    let length = array ? array.length : 0;

    let low = 0;
    let high = length;

    if (typeof value === undefined) {
        return high;
    }

    // 即找第一个大于待插入元素值的元素的位置
    while (low < high) {
        let middle = (low + high) >>> 1;

        if (array[middle] <= value) {
            low = middle + 1;
        } else {
            high = middle;
        }
    }

    return high;
}
```
源码中，利用位运算求中间值，可关注: `const mid = (low + high) >>> 1`