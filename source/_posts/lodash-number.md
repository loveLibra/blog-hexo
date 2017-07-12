title: Lodash源码祭(Number)
date: 2017-07-12 19:02:12
tags: [Lodash, Number]
---
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