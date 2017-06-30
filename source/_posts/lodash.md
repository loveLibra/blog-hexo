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
        return array[length - 1];
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

实现sortIndex基础方法：
```javascript
const baseSortedIndex = (array, value, iteratee) => {
    let length = array ? array.length : 0;

    let low = 0;
    let high = length;

    let iteValue = iteratee ? iteratee(value) : value;

    if (typeof iteValue === undefined) {
        return high;
    }

    // 二分查找
    // 即找第一个大于待插入元素值的元素的位置
    while (low < high) {
        let middle = (low + high) >>> 1;

        let midValue = iteratee ? iteratee(array[middle]) : array[middle];

        if (midValue < iteValue) {
            low = middle + 1;
        } else {
            high = middle;
        }
    }

    return high;
};

const sortedIndex = (array, value) => baseSortedIndex(array, value);
```
源码中，利用位运算求中间值，可关注: `const mid = (low + high) >>> 1`

## sortedIndexBy
```javascript
_.sortedIndexBy(array, value, [iteratee=_.identity])
```
iteratee转化后进行元素比较，iteratee接受一个参数
```javascript
const sortedIndexBy = (array, value, iteratee) => baseSortedIndex(array, value, iteratee);
```

## sortedIndexOf
```javascript
_.sortedIndexOf(array, value)
```
使用`二分法`binary search查找一个value在已排序数组中的索引
```javascript
const sortedIndexOf = (array, value) => {
    let length = array ? array.length : 0;

    if (length) {
        // 调用baseSortedIndex找到value插入的位置，再判断插入位置的值是否与value相等
        // 若不等，则无对应值存在于数组中，返回-1
        // 另外，需要满足条件，得到的index需要小于length，排出插入到最后的情况
        let index = baseSortedIndex(array, value);

        if (index < length && array[index] === value) {
            return index;
        }
    }

    return -1;
}
```

## sortedLastIndex、 sortedLastIndexBy、sortedLastIndexOf
```javascript
_.sortedLastIndex(array, value)
_.sortedLastIndexBy(array, value, [iteratee=_.identity])
_.sortedLastIndexOf(array, value)
```
对于之前的三个sortedIndex的方法，还有三个对应的逆序，在原有的baseSortedIndex的基础上，将二分查找的条件修改下即可：
```javascript

    //[ < ] --->  [<=]这样在遇到第一个相等的元素后会继续向右查找直到找到最后一个
    if (midValue <= iteValue) {
        low = middle + 1;
    } else {
        high = middle;
    }
}
```
在此要求下对baseSortedIndex做个修正并提取baseSortedIndexBy方法，控制一下参数:
```javascript
const baseSortedIndex = (array, value, highest) => {
    return baseSortedIndexBy(array, value, val => val, highest);
};

const baseSortedIndexBy = (array, value, iteratee, highest) => {
    let length = array ? array.length : 0;

    let low = 0;
    let high = length;

    if (!iteratee) {
        iteratee = val => val;
    }

    let iteValue = iteratee(value);

    if (typeof iteValue === undefined) {
        return high;
    }

    // 二分查找
    // 即找第一个大于待插入元素值的元素的位置
    while (low < high) {
        let middle = (low + high) >>> 1;

        let midValue = iteratee(array[middle]);

        if (highest ? (midValue <= iteValue) : (midValue < iteValue)) {
            low = middle + 1;
        } else {
            high = middle;
        }
    }

    return high;
};

// 实现...
const sortedLastIndex = (array, value) => baseSortedIndex(array, value, true);
const sortedLastIndexBy = (array, value, ite) => baseSortedIndexBy(array, value, ite, true);
// sortedLastIndexOf类似
```

## sortedUniq
```javascript
_.sortedUniq(array)
```
对已排序数组去重，返回新数组
```javascript
const sortedUniq = array => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    let index = -1;
    let result = [];
    let seen;

    while (++index < length) {
        let value = array[index];

        // 相邻元素值不相等的
        // 注意：!index判断seen有没有被初始化，不可用!seen，因为seen可能是falsely值
        if (!index || !eq(value, seen)) {

            seen = value;
            result.push(seen);
        }
    }

    return result;
}

/**
 * value与other是否相等，NaN !== NaN
 */
const eq = (value, other) => {
    return value === other || (value !== value && other !== other);
}
```

## sortedUniqBy
```javascript
_.sortedUniqBy(array, [iteratee])
```
功能类似，接受迭代器参数，将值转化后做比较，综合`sortedUniq`提取baseSortedUniq方法...
```javascript
const baseSortedUniq = (array, iteratee) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    if (!iteratee) {
        iteratee = x => x;
    }

    let index = -1;
    let result = [];
    let seen;

    while (++index < length) {
        let value = array[index];

        let computed = iteratee(value);

        // 相邻元素值不相等的
        // 注意：!index判断seen有没有被初始化，不可用!seen，因为seen可能是falsely值
        if (!index || !eq(computed, seen)) {

            seen = computed;
            result.push(value);
        }
    }

    return result;
}

const sortedUniqBy = (array, iteratee) => {
    return baseSortedUniq(array, iteratee);
}
```

## tail
```javascript
_.tail(array)
```
获取除第一个元素的数组切片
```javascript
const tail = array => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    return array.slice(1);
}
```
看了下源码的实现，直接用slice有点小low啊，别人都用数组的解构啦！
```javascript
const [head, ...res] = array; // 666
return res;
```

## take
```javascript
_.take(array, [n=1])
```
获取从数组开始的n个元素的切片
```javascript
const take = (array, n = 1) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    n = n > 0 ? n : 0;

    return array.slice(0, n);
}
```

## takeRight
```javascript
_.takeRight(array, [n=1])
```
获取数组结尾的n个元素的切片
```javascript
const takeRight = (array, n = 1) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    if (n > length) {
        n = length;
    }

    return array.slice(length - n);
}
```

## takeWhile
```javascript
_.takeWhile(array, [predicate=_.identity])
```
从左侧开始切片数组，直到第一个predicate判断返回为false的值位置
```javascript
const takeWhile = (array, predicate) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    if (!predicate) {
        return array.slice(0);
    }

    let index = -1;
    while (++index < length) {
        if (!predicate(array[index])) {
            break;
        }
    }

    return array.slice(0, index);
}
```

## takeRightWhile
```javascript
_.takeRightWhile(array, [predicate=_.identity])
```
类似takeWhile，takeRightWhile从数组右侧开始寻找，综合takeWhile，来一个`baseTakeWhile`
```javascript
const baseTakeWhile = (array, predicate, fromRight) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    if (!predicate) {
        return array.slice(0);
    }

    let index = fromRight ? length : -1;

    while (fromRight ? index-- : ++index < length) {
        if (!predicate(array[index])) {
            break;
        }
    }

    return fromRight ? array.slice(index + 1) : array.slice(0, index);
}
```
## union
```javascript
_.union([arrays])
```
数组元素组合，参数接受n个数组
```javascript
const union = (...arrays) => {
    let length = arrays ? arrays.length : 0;

    if (!length) {
        return [];
    }

    let result = [];
    let index = -1;
    while (++index < length) {
        result.splice(result.length, 0, ...arrays[index]);
    }

    return Array.from(new Set(result));
}
```

## unionBy
```javascript
_.unionBy([arrays], [iteratee=_.identity])
```
接受迭代参数，迭代后相同的项入坑...这样就坑爹啦，上面那个简易的办法用不上啦
```javascript
const unionBy = (...arrays) => {
    let length = arrays ? arrays.length : 0;

    if (!length) {
        return [];
    }

    // let iteratee = last(arrays);
    let iteratee = arrays[length - 1];

    length--;

    let result = [];
    let seen = [];
    let index = -1;

    while (++index < length) {
        let array = arrays[index];

        let computed = array.map(iteratee);

        let inner = -1;
        let innerLen = array.length;

        while (++inner < innerLen) {
            if (seen.indexOf(computed[inner]) === -1) {
                seen.push(computed[inner]);

                result.push(array[inner]);
            }
        }
    }

    return result;

}
```
上述方法时间复杂度O(n<sup>2</sup>)，应该可以简化到O(n)
```javascript
// 先参考union的方法将所有元素合并到数组中，然后再针对iteratee做去重
const unionBy = (...arrays) => {
    // 参数拆解同上
    let length = arrays ? arrays.length : 0;

    if (!length) {
        return [];
    }

    let iteratee = arrays[length - 1];

    length--;


	let index = -1;
    let inter = [];
     while (++index < length) {
        inter.splice(inter.length, 0, ...arrays[index]);
    }

    // 去重
    let result = [];
    let seen = [];

    index = -1;
    length = inter.length;

    while (++index < length) {
        let val = inter[index];
        let computed = iteratee(val);

        if (seen.indexOf(computed) > -1) {
            continue;
        }

        seen.push(computed);
        result.push(val);
    }

    return result;
}
```

## unionWith
```javascript
_.unionWith([arrays], [comparator])
```
老三样，可指定元素间比较方式，只是O(n<sup>3</sup>)的复杂度啊...comparator也需要遍历比较


## uniq
```javascript
_.uniq(array)
```
所以这是跟`union`是有什么区别呢？一个参数是数组，一个是参数别表，每个参数项是数组？
包括`uniqBy`和`uniqWith`都是同理的实现

## zip
```javascript
_.zip([arrays])
```
接受参数为n个数组的参数列表，将参数列表中的数组按照index进行分组，返回分组的新数组
```javascript
const zip = (...arrays) => {
    let length = arrays.length ? arrays.length : 0;

    if (!length) {
        return [];
    }

    let max = 0;
    arrays.forEach(array => {
        max = Math.max(max, array.length);
    });

    let index = -1;
    let result = new Array(max);
    while (++index < max) {

        let sub = [];
        let inner = -1;
        while (++inner < length) {
            sub[inner] = arrays[inner][index];
        }

        result[index] = sub;
    }

    return result;
}
```

## unzip
```javascript
_.unzip(array)
```
接收参数跟zip不同，其实功能与`zip`一样，把对应index的元素组合即可，这个操作可以理解为双向操作， 因此通过`...`将列表转化为数组后，执行过程就一致了，我们将上述zip的执行移植过来：
```javascript
const unzip = array => {
    let length = array.length ? array.length : 0;

    if (!length) {
        return [];
    }

    let max = 0;
    array.forEach(array => {
        max = Math.max(max, array.length);
    });

    let index = -1;
    let result = new Array(max);
    while (++index < max) {

        let sub = [];
        let inner = -1;
        while (++inner < length) {
            sub[inner] = array[inner][index];
        }

        result[index] = sub;
    }

    return result;
}
```
这样zip操作就可以直接调用unzip
```javascript
const zip = (...arrays) => {
    return unzip(arrays);
}
```

## unzipWith
```javascript
_.unzipWith(array, [iteratee=_.identity])
```
对位的元素使用iteratee进行结合后吐出来，可以先试用unzip转化，然后对没个数组元素进行组合， `iteratee`为接收参数列表的函数
```javascript
const unzipWith = (array, iteratee) => {
    let length = array.length ? array.length : 0;

    if (!length) {
        return [];
    }

    let result = unzip(array);

    return result.map(res => {

        // 数组参数转参数列表
        return iteratee ? iteratee.apply(null, res) : res
    });
}
```
zip然后unzipWith能实现某种神奇的转化？yes...比如:
```javascript
// iteratee
const add = (...arr) => arr.reduce((acc, val) => acc + val, 0);

unzipWith(zip([1, 2], [10, 20], [100, 200]), add)
```
可实现原数组转化为每项的元素和的数组

## without
```javacript
_.without(array, [values])
```
生成一个数组中排出指定元素的新数组，与`pull`功能类似，pull会改变源数组
```javascript
const without = (array, ...values) => {
    let length = array ? array.length : 0;

    if (!length) {
        return [];
    }

    let result = [];
    let index = -1;
    while (++index < length) {
        if (values.indexOf(array[index]) > -1) {
            continue;
        }

        result.push(array[index]);
    }

    return result;
}
```

## xor
```javascript
_.xor([arrays])
```
亦或运算
```javascript
const xor = ...arrays => {
    let length = arrays.length;

    // 如果参数只传入一个数组，则对该数组元素去重后返回
    if (length < 2) {
        return length ? uniq(arrays[0]) : [];
    }

    let index = -1;
    let result = [];

    // TODO
}
```

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
// length合法值
const isLength = val => {

    // 数字 && 非负 && 非负0 && 小于最大正整数
    return typeof val === 'number' && val > -1 && val % 1 === 0 && val <= 9007199254740991;
}

const isArrayLike = arr => {
    return arr !== null && typeof arr !== 'function' && isLength(arr.length);
}

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
# Date

## now
```javascript
_.now()
```
获取当前时间的毫秒时间戳
```javascript
const now () => {
    return new Date().getTime();
}
```

# Number

## random
```javascript
_.random([lower=0], [upper=1], [floating])
```
返回lower-upper之间任意的随机数，若lower为传则从0开始，floating=true则会返回浮点数，另外，如果lower或者upper为浮点数，也会返回一个浮点数
```javascript
const random = (lower, upper, floating) => {
    // 接收参数情况较多，需进行参数校准

    // 参数不满
    if (typeof floating === 'undefined') {

        // 2个参数，第2个参数为浮点数，则第一个参数为upper
        if (typeof upper === 'boolean') {
            floating = upper;

            upper = lower;

            lower = undefined;
        } else if (typeof lower === 'boolean') {
            // 1个参数

            floating = lower;
            lower = undefined;
        }
    }

    if (typeof lower === 'undefined') {
        lower = 0;
    }

    if (typeof upper === 'undefined') {
        upper = 1;
    }

    // lower > upper的情况则交换
    if (lower > upper) {
        [lower, upper] = [upper, lower];
    }

    // 通过number % 1判断是否为浮点数
    if (floating || lower % 1 || upper % 1) {
        // 范围浮点数的方法有点诡异啊...
        let rand = Math.random(); // 0 - 1
        let len = `${rand}`.length;

        return Math.min(lower + (rand * (upper - lower + parseFloat(`1e-${len}`))), upper); // ...
    }

    // 整数
    return lower + Math.floor((Math.random() * (upper - lower + 1)));
}
```

## inRange
```javascript
_.inRange(number, [start=0], end)
```
判断一个数字是否在start和end(不包含end)之间，start为可选参数，默认为0，如果start大于end
```javascript
const inRange = (number, start, end) => {
    if (typeof end === 'undefined') {

        if (typeof start === 'undefined') {
            return false; // 无range
        }

        end = start;
        start = 0;
    }

    if (end > start) {
        return number >= start && number < end;
    } else {
        return number > end && number <= start;
    }
}
```

## clamp
```javascript
_.clamp(number, [lower], upper)
```
名字好隐晦...保证返回值在lower和upper之间，小于lower则返回lower，大于upper则返回upper
```javascript
const clamp = (number, lower, upper) => {
    number = ~~number;
    lower = ~~lower;
    upper = ~~upper;

    // 判断NaN
    lower = lower === lower ? lower : 0;
    upper = upper === upper ? upper : 0;

    if (number === number) {
        // compare to upper
        number = number <= upper ? number : upper;

        // compare to lower
        number = number >= lower ? number : lower;
    }

    return number;
}
```

# Math

## mean
```javascript
_.mean(array)
```
求数字平均数
```javascript
const mean = array => {
    let sum = array.reduce((total, i) => {
        return total + i;
    }, 0);
    let {length} = array;

    if (!length) {
        return NaN;
    }

    return sum / length;
}
```

## add
```javascript
_.add(augend, addend)
```
不仅可以数字相加，也可以实现字符串的相加，并且参数为空时返回默认值0
```javascript
const add = (augend, addend) => {
    if (typeof augend === 'undefined' && typeof addend === 'undefined') {
        return 0; // default val
    }

    if (typeof addend === 'undefined') {
        addend = typeof augend === 'string' ? '' : 0;
    }

    return augend + addend;
}
```

## divide
```javascript
_.divide(dividend, divisor)
```
两数字相除，默认值为1
```javascript
const divide = (dividend, divisor) => {
    if (typeof dividend === 'undefined' && typeof divisor === 'undefined') {
        return 1; // default val
    }

    if (typeof divisor === 'undefined') {
        divisor = 1;
    }

    return dividend / divisor;
}
```

## ceil
```javascript
_.ceil(number, [precision=0])
```
数字向上取整，并可指定精度
```javascript
const ceil = (number, precision = 0) => {
    // 这一波precision操作666啊，保持关注
    if (precision) {

        // 排除科学计数法的数字影响？
        let pair = `${number}e`.split('e');

        let val = Math.ceil(`${pair[0]}e${~~pair[1] + precision}`); // 可以接受字符串参数...

        pair = `${val}e`.split('e');

        return +`${pair[0]}e${~~pair[1] - precision}`
    }
    return Math.ceil(number);
}
```
`Math.ceil(4.231, -2) === 100`哈哈

## floor
```javascript
_.floor(number, [precision=0])
```
同理ceil，方法换为`Math.floor`;还有`round`实现四舍五入的同理

## max
```javascript
_.max(array)
```
接受数字数组，求max... 还有另外一个扩展的方法`_.maxBy(array, [iteratee=_.identity])`可以一起实现
```javascript
const baseMax = (array, iteratee) => {
    let index = 0;
    let length = array ? array.length : 0;

    if (!iteratee) {
        iteratee = val => val;
    }

    let max = array[0];
    let computedMax = iteratee(max);

    while (++index < length) {
        let cur = array[index];

        let computed = iteratee(cur);

        // NOTE：symbol不可隐式转化为数字参与比较
        if (typeof computed !== 'symbol' && computed > computedMax) {
            max = cur;
            computedMax = computed;
        }
    }

    return max;
}
```
另外，同理的还有`min`和`minBy`，不赘述

## mean
```javascript
_.mean(array)
```
求数组的平均值... 同样也有扩展方法`_.meanBy(array, [iteratee=_.identity])`，iteratee接受一个参数value
```javascript
const baseMean = (array, iteratee) => {
    let {length} = array;

    if (!length) {
        return NaN;
    }

    if (!iteratee) {
        iteratee = val => val;
    }

    // 因为iteratee的因素，还不能直接用reduce来求sum值
    let index = -1;
    let sum = 0;
    while(++index < length) {
        let cur = iteratee(array[index]);
        sum += (cur === undefined ? 0 : cur);
    }

    return sum / length;
}
```

## multiply
```javacript
_.multiply(multiplier, multiplicand)
```
两变量相乘，参数为空时默认值为1
```javascript
const multiply = (multiplier, multiplicand) => {
    if (multiplier === undefined && multiplicand === undefined) {
        return 1;
    }

    if (multiplicand === undefined) {
        return multiplier;
    }

    return multiplier * multiplicand;
}
```

## subtract
```javascript
_.subtract(minuend, subtrahend)
```
减法，有默认值为0...类似乘法不赘述

## sum
```javascript
_.sum(array)
```
求数组的“加法”运算，也包括字符串，引用上面`mean`求值的sum算式即可。另外，也有`sumBy`方法，接收2个参数，另外一个参数`iteratee`
```javascript
const baseSum = (array, iteratee) => {
    let {length} = array;

    if (!length) {
        return 0;
    }

    if (!iteratee) {
        iteratee = val => val;
    }

    let index = -1;
    let sum = 0;
    while(++index < length) {
        let cur = iteratee(array[index]);

        if (cur === undefined) {
            continue;
        }

        sum += cur;
    }

    return sum;
}
```

# String

## loweFrirst
```javascript
_.lowerFirst([string=''])
```
小写字符串的首字母
```javascript
const lowerFirst = (string = '') => {
    if (string !== null) {
        string = typeof string === 'string' ? string : string.toString();

        let head = string.match(/^[A-Z]/);

        head = head ? head[0] : head;

        if (head) {
            return string.replace(/^([A-Z])(.*)/, `${head.toLowerCase()}$2`)
        } else {
            return string;
        }
    }

    return '';
}
```

同理，首字母大写是同样的实现，不赘述

## toLower / toUpper
```javascript
// 小写
_.toLower([string=''])
// 大写
_.toUpper([string=''])
```
将字符串中的字母转换为小写或者大写，直接调用标准方法即可
```javascript
const toLower = (string = '') => {
    if (string !== null) {
        string = typeof string === 'string' ? string : string.toString();

        return string.toLowerCase();
    }

    return '';
}
```

## capitalize
```javascript
_.capitalize([string=''])
```
首字母大写，剩余的小写。先全部转化为小写，再首字母大写就可以了
```javascript
const capitalize = (string = '') => {
    return upperFirst(toLower(string));
}
```

## endsWith
```javascript
_.endsWith([string=''], [target], [position=string.length])
```
判断字符串到position位置是否是以target结尾的
```javascript
const endsWith = (string, target, position) => {
    if (string === undefined || target === undefined) {
        return false;
    }

    position = position === undefined ?  string.length : +position;

    if (position < 0 || position !== position) {
        return false;
    }

    // 正则
    let sliceStr = string.slice(0, position);

	let regx = new RegExp(`${target}$`);

    return regx.test(sliceStr);

    // 或者也可以通过字符串截取
    // let end = position;
    // position -= target.length;

    // return position >= 0 && string.slice(position, end) == target; // 未强等，'123'找数字3也是可以的
}
```

## pad
```javascript
_.pad([string=''], [length=0], [chars=' '])
```
用chars补全字符串到指定length
```javascript
const repeat = (str, times) => {
    let res = str;

    while (--times) {
        res += str;
    }

    return res;
}

const pad = (string, length, chars) => {
    if (string === undefined) {
        return '';
    }

    length = +length;

    let {length: len} = string;

    if (length <= len) {
        return string;
    }

    let offset = length - len;

    let left = Math.floor(offset / 2);
    let right = offset - left;

    chars = chars === undefined ? ' ' : chars.toString();

    let charsLen = chars.length;

    return repeat(chars, Math.ceil(left / charsLen)).slice(0, left) +
    string +
    repeat(chars, Math.ceil(right / charsLen)).slice(0, right)
}
```

## padEnd / padStart
```javascript
_.padEnd([string=''], [length=0], [chars=' '])
//
_.padStart([string=''], [length=0], [chars=' '])
```
类似于pad，向字符串尾部或者头部补全指定字符串

## repeat
```javascript
_.repeat([string=''], [n=1])
```
在实现`pad`功能的时候，我们有一个重复字符串的函数
```javascript
const repeat = (str, times) => {
    let res = str;

    while (--times) {
        res += str;
    }

    return res;
}
```
So easy？no，至少不是那么完美。比如把'a'重复4次，上面做法需要执行4次字符串合并，那常规做法，我可以把第一次合并的结果再做一次合并就可以了。优化的算法：
```javascript
const repeat = (str, times) => {
    let result = '';

    if (!str || times < 1) {
        return result;
    }

    do {
        if (times % 2) {
            result += str;
        }

        times = Math.floor(times / 2);

        if (times) {
            str += str;
        }
    } while (times)

    return result;
}
```

## escape
```javascript
_.escape([string=''])
```
转义特殊字符：`&`、`<`、`>`、`"`、`'`
```javascript
const escapeMap = {
    '&': '&amp',
    '<': '&lt',
    '>': '&gt',
    '"': '&quot',
    "'": '&#39'
};

const escape = (string = '') => {
    if (string && /[&<>"']/g.test(string)) {
        return string.replace(/[&<>"']/g, char => escapeMap[char]);
    }

    return string;
}
```

## escapeRegExp
```javascript
_.escapeRegExp([string=''])
```
转义字符串中的正则表达式字符`^`、 `$`、 `.`、 `\`、 `*`、 `+`、 `?`、 `(`、 `)`、 `[`、 `]`、 `{`、 `}`、 `|`
```javascript
const reg = /[\\^$.*+?()[\]{}|]/g; // 这里注意\]，防止闭合正则的前[

const escapeRegExp = (string = '') => {
    if (string && reg.test(string)) {
        return string.replace(reg, '\\$&'); // $& is matched substring
    }

    return string;
}
```
顺带小case，除`$&`，字符串替换还有多种模式，Turn to [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_string_as_a_parameter)

## parseInt
```javascript
_.parseInt(string, [radix=10])
```
字符串转化为数字
```javascript
const parseInt = (string, radix) => {
    if (radix === null) {
        radix = 0;
    } else {
        radix = +radix;
    }

    return nativeParseInt(string.replace(/^\s+/, ''), radix); //  调native的parseint实现，补：消除头尾空格
}
```
脑补radix值的以及默认参数转化方法，如果radix是`undefined`或者`0`，会自动根据字符串的前缀进行对应的转换数字操作：
* If the input string begins with "0x" or "0X", radix is 16 (hexadecimal) and the remainder of the string is parsed.
* If the input string begins with "0", radix is eight (octal) or 10 (decimal).  Exactly which radix is chosen is implementation-dependent.  ECMAScript 5 specifies that 10 (decimal) is used, but not all browsers support this yet.  For this reason always specify a radix when using parseInt.（对于`0`开头的字符串，浏览器解析有差异，所以建议parseInt指定radix参数）
* If the input string begins with any other value, the radix is 10 (decimal).

## replace
```javascript
_.replace([string=''], pattern, replacement)
```
字符串替换，pattern可以是正则表达式或者字符串
```javascript
const replace = (...args) => {

    if (args.length < 3) {
        return `${args[0]}`;
    } else {
        let [string, pattern, replacement] = args;

        return `${string}`.replace(pattern, replacement);
    }
}
```

## split
```javascript
_.split([string=''], separator, [limit])
```
字符串分割，也可以指定界限（也就是最终得到的数组的元素个数，原生也有这个参数...）；
另外，separator可以是字符串也可以是正则，原生也是支持的...当separator=undefined或者separator没有出现在string中时，返回原字符串；当separator=''时，返回字符串分割成单个字母的数组
```javascript
const MAX_ARRAY_LENGTH = 4294967295;

const split = (string, separator, limit) => {
    // 确保limit是一个正整数
    limit = limit === undefined ? MAX_ARRAY_LENGTH : limit >>> 0;

    if (limit === 0) {
        return [];
    }

    if (string === undefined) {
        return '';
    }

    return string.toString().split(separator, limit);
}
```
源码
```javascript
  if (string && (
        typeof separator == 'string' ||
        (separator != null && !isRegExp(separator))
      )) {
    if (!separator && hasUnicode(string)) {
      return castSlice(stringToArray(string), 0, limit)
    }
  }
```
这是什么样的一个套路，没看明白...

## trim
```javascript
_.trim([string=''], [chars=whitespace])
```
移除字符串开头和结尾的空格或者指定的字符
```javascript
const trim = (string, chars) => {
    if (string === undefined) {
        return '';
    }

    if (typeof string !== 'string') {
        string = string.toString();
    }

    // 默认替换空格
    if (chars === undefined) {
        return string.trim();
    }

    let {length} = string;
    let start = 0;

    while (start < length) {
        if (chars.indexOf(string[start]) === -1) {
            break;
        }

        start++;
    }

    let end = length - 1;

    while (end > 0) {
        if (chars.indexOf(string[end]) === -1) {
            break;
        }

        end--;
    }

    return string.slice(start, end + 1);
}
```

然后trimStart和trimEnd同理，`String`也有`trimLeft`和`trimRight`方法

## words
```javascript
_.words([string=''], [pattern])
```
字符串分解为单词的数组，而且可以指定匹配的单词的模式
```javascript
const words = (string, pattern) => {
    if (!string) {
        return [];
    }

    if (pattern === undefined) {
        pattern = /\w+/g;
    }

    return string.match(pattern) || []; // match也是可以接受字符串pattern的...
}
```
这边对于pattern=undefined的情况其实考虑的很简单了，源码还考虑了`ascii`和`unicode`的情况分别处理...这个跑`中文 你好`的时候就挂了... `word.js`有点弱鸡啊，急需靠拢源码

## camelCase
```javascript
_.camelCase([string=''])
```
字符串驼峰，这之类的所有的字符串格式化方法都基于之前的单词匹配，匹配到单词后拼接起来
```javascript
const camelCase = (string = '') => {
    return toLower(string).match(/\w+/g).reduce((res, world, index) => {
        return res + (index ? upperFirst(world) : world);
    }, ''); // reduce函数的初始值可选，不给时默认为数组第一个元素，但当数组是空的时候会报错，此处有风险，应当赋默认值
}
```

## kebabCase
```javascript
_.kebabCase([string=''])
```
转化为中划线分割单词的字符串
```javascript
const kebabCase = (string = '') => {
    return toLower(string).match(/\w+/g).join('-');
}
```

包括`lowerCase`小写空格分割、`snakeCase`以下划线分割，也是同理

## template
```javascript
//TODO
```

# Lang

## castArray
```javascript
_.castArray(value)
```
如果value不是一个数组，就给他包成一个数组
```javascript
const castArray = (...arg) => {
    // 没有参数的情况，返回空数组，需要特殊处理，跟参数为undefined不同
    if (arg.length === 0) {
        return [];
    }

    let value = arg[0];

    if (Array.isArray(value)) {
        return value;
    }

    return [value];
}
```

## eq
```javascript
_.eq(value, other)
```
value和other是否「强相等」，但这里NaN和NaN相等
```javascript
const eq = (value, other) => value === other || (value !== value && other !== other);
```

## gt
```javascript
_.gt(value, other)
```
value > other则返回true，否则返回false；需要注意的时，当两个参数都是字符串时，进行字符串比较，其余的都转化为数字进行比较
```javascript
const gt = (value, other) => {
    if (!(typeof value === 'string' && typeof other === 'string')) {
        value = +value;
        other = +other;
    }

    return value > other;
}
```
`lt`、`lte`和`gte`同理...

## isArguments
```javascript
_.isArguments(value)
```
检查传入值是否是arguments对象
```javascript
const isArguments = value => {
    return typeof value === 'object' && value !== null && value.toString() === '[object Arguments]';
}
```

## isArray
```javascript
_.isArray(value)
```
判断是否为数组
```javascript
const isArray = value => {
    const _isArray = Array.isArray;

    if (!_isArray) {
        _isArray = function(arg) {
            return Object.prototype.toString.call(arg) === '[object Array]';
        };
    }

    return _isArray(value);
}
```