title: Lodash源码祭「Lang」
date: 2017-07-12 18:58:41
tags: [Lodash, Lang]
---
> 巧妙的函数实现吸引着你想去看看他的实现方法，里面会有更多奇思妙想让你欣喜若狂...

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
<!-- more -->
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

## isArrayBuffer
```javascript
_.isArrayBuffer(value)
```
判断是否为ArrayBuffer。同样，通过`toString`也能得到ArrayBuffer的tag为`[object ArrayBuffer]`

## isArrayLike
```javascript
_.isArrayLike(value)
```
首先，明确ArrayLike的特征为：非函数的value有length属性，length为整数且0 <= value.length <= Number.MAX_SAFE_INTEGER
```javascript
const isLength = val => {

    // 数字 && 非负 && 除「-0」的非负「整数」 && 小于最大正整数
    return typeof val === 'number' && val > -1 && val % 1 === 0 && val <= Number.MAX_SAFE_INTEGER;
}

const isArrayLike = value => {
    return value !== null && typeof value !== 'function' && isLength(value.length);
}
```

## isArrayLikeObject
```javascript
_.isArrayLikeObject(value)
```
在`isArrayLike`的基础上加了个条件：判断value得是Object

## isBoolean
```javascript
_.isBoolean(value)
```
可以检验值是否为布尔型变量，可以是原始值类型（true / false）也可以是Boolean的实例对象
```javascript
const isBoolean = value => {
    return value === true || value === false ||
        (value !== null && Object.prototype.toString.call(value) === '[object Boolean]')
}
```

## isNumber
```javascript
_.isNumber(value);
```
判断参数是否为数字，同样对于原始类型和Number的实例都适用
```javascript
const isNumber = value => {
    return typeof value === 'number' ||
        Object.prototype.toString.call(value) === '[object Number]';
}
```

## isNaN
```javascript
_.isNaN(value)
```
原生`isNaN`方法会对非数字都返回true，而不光是NaN，lodash仅对NaN本身以及`new Number(NaN)`的情况返回true
```javascript
const isNaN = value => {

    // 首先判断是否为数字
    // 然后区分是否跟本身相等

    return isNumber(value) && value != +value; // 非强等
}
```

## isNil
```javascript
_.isNil(value)
```
判断参数是否为undefined或者null
```javascript
const isNil = value => {
    return value === undefined || value === null; // value == null
}
```

## isNull
```javascript
_.isNull(value)
```
判断参数是否为null
```javascript
const isNull = value => {
    return value === null;
}
```

## isObject
```javascript
_.isObject(value)
```
是否为对象(包括数组、函数、对象、正则等等非原始数据类型...)
```javascript
const isObject = value => {
    var type = typeof value;

    // null && 对象或者函数
    return value != null && (type === 'object' || type === 'function');
}
```

## isObjectLike
```javascript
_.isObjectLike(value)
```
检查value是否为Object-Like（非null并满足typeof vlaue === 'object'）
```javascript
const isObjectLike = value => {
    return value !== null && typeof value === 'object';
}
```

## isPlainObject
```javascript
_.isPlainObject(value)
```
检查是否为plain object（object.constructor === Object || object.[[prototype]] === null）
```javascript
const isPlainObject = value => {
    // 首先满足是个对象并且排除掉一些内置的对象,比如Date这些
    if (!isObjectLike(value) || Object.prototype.toString.call(value) !== '[object Object]') {
        return false;
    }

    // 先看value.[[prototype]]是否为null，[[prototype]]是不可见的属性，通过Object.getPrototype可以获取
    let proto = Object.getPrototypeOf(value);

    if (proto === null) {
        return true;
    }
    // 承上，这边需要说一下，默认创建的总是有[[prototype]]指向Object.prototype的，但是为啥会有null的情况呢，因为有Object.create(null)啊！指定[[prototype]]是null，也就变成了食物链顶端的男人...需要考虑这种情况

    // 接下来判断对象是不是通过Object构造函数构造出来的
    // contructor是可以篡改的...
    let constructor = Object.prototype.hasOwnProperty.call(proto, 'constructor') && proto.constructor;

    if (typeof constructor === 'function' &&
        // instanceof 检查是是否在左参数是否在右参数的原型链中
        // 除constructor是Object外，其他构造函数不满足此条件
        // 对于Object，Object.__proto__ === Function.prototype，Function.prototype.__proto__ === Object.prototype
        constructor instanceof constructor &&
        // 检查contructor字符串是否是Object的字符串
        Function.prototype.toString.call(constructor) === Function.prototype.toString.call(Object)) {
            return true;
        }
    return false;
}
```

## isRegExp
```javascript
_.isRegExp(value)
```
检查是否为正则
```javascript
const isRegExp = value => {
    // 官网似乎还做了个node环境下直接用util.isRegExp去判断的，但是看Node API似乎已经被Deprecated啦
    return isObjectLike(value) && Object.prototype.toString.call(value) === '[object RegExp]';
}
```

## isSafeInteger
```javascript
_.isSafeInteger(value)
```
也就是 -(2^53 - 1) ~ （2^53 - 1）之间的整数。ES6直接提供了方法`Number.isSafeInteger`可以直接实现...当然，考虑IE的话还是需要Polyfill的
```javascript
const isInteger = value => {
    return typeof value === 'number' &&
        isFinite(value) &&
        Math.floor(value) === value;
}

const isSafeInteger = value => {
    return isInteger(value) && Math.abs(value) <= Number.MAX_SAFE_INTEGER;
}
```

## isSet / isWeekSet / isMap / isWeekMap
Set和Map都可以通过`Object.prototype.toString.call(value)`的值去判断，分别为'[object Set]'、'[object WeekSet]'、'[object Map]'、'[object WeekMap]'

## isString
```javascript
_.isString(value)
```
检查是否为字符串
```javascript
const isString = value => typeof value === 'string' ||
    Obejct.prototype.toString.call(value) === '[object String]';
```

## isSymbol
```javascript
_.isSymbol(value)
```
检查是否为symbol值
```javascript
const isSymbol = value => typeof value === 'symbol' ||
    Obejct.prototype.toString.call(value) === '[object Symbol]';
```

## isTypedArray
```javascript
_.isTypedArray(value)
```
检查是否为typed array。对应的不同类型的数组，有不同的toString展示，通过正则规则统一匹配

```javascript
const reTypedTag = /^\[object (?:Float(?:32|64)|(?:Int|Uint)(?:8|16|32)|Uint8Clamped)\]$/;

const isTypedArray = value => typeof value === 'object' && reTypedTag.test(value);
```

link to description of [typed array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)...就是什么是Unit8ClampedArray好诡异的数据结构，限制数组元素值在0-255