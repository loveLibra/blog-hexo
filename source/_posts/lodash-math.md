title: lodash源码祭(Math)
date: 2017-07-12 19:02:21
tags: [Lodash, Math]
---
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

