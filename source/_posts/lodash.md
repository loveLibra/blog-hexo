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
下意识的用数组`push`，源码惯用下标递增：`result[resIndex++] = value`。性能有影响？