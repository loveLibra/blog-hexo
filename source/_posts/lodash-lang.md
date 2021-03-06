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

## isUndefined
```javascript
_.isUndefined(value)
```
检查是否为undefined
```javascript
const isUndefined = value => value === undefined;
```

## toArray
```javascript
_.toArray(value)
```
转化value为数组
```javascript
const toArray = value => {
    if (!value) {
        return [];
    }

    if (isArrayLike(value)) {
        if (typeof value === 'string') {
            return [].slice.call(value)
        } else {
            // arguments
            // DOM Nodes
            let {length} = value;

            let index = -1;

            let res = [];

            while (index++ < length) {
                res[index] = value[index];
            }

            return res;
        }
    }

    // map/set
    let tag = Object.prototype.toString.call(value)
    if (tag === '[object Set]' || tag === '[object Map]') {
        return Array.from(value);
    }

    // NOTICE:还有迭代器对象
    if (Symbol && Symbol.iterator && value[Symbol.iterator]) {
        let res = [];
        for (let val of value) {
            res.push(val);
        }

        return res;
    }

    // 其他
    return Object.values(value);  // 兼容性，懒得遍历了
}
```
源码对于这种`_.toArray({ 'a': 1, 'b': 2, length:2 });`情况的ArrayLike返回的是[undefined, undefined]...有点不合理了，但是似乎没办法区分这种这种带length的普通对象啊...

## toLength
```javascript
_.toLength(value)
```
转化为一个length的合法值：整数字且0 <= value <= Number.MAX_SAFE_INTEGER
```javascript
const toLength = value => {
    if (!value) {
        return 0;
    }

    // 化整的神奇方法
    let reminder = value % 1;

    value = reminder ? value - reminder : value;

    if (value < 0) {
        return 0;
    } else if (value > Number.MAX_SAFE_INTEGER) {
        return Number.MAX_SAFE_INTEGER;
    }

	return value;
}
```

## toString
```javascript
_.toString(value)
```
转化为字符串，对于undefined和null返回空字符串，-0会保留`-`
```javascript
const INFINITY = 1 / 0;

const toString = value => {
    if (value == undefined) {
        return '';
    }

    if (typeof value === 'string') {
        return value;
    }

    // 对于数组，递归，并且会flattenDeep
    if (Array.isArray(value)) {
        return `${value.map(val => toString(val))}`;
    }

    // 因为symbol类型的变量不可隐式转化为其他数据类型，symbol类型也要单独考虑（只可显示转化为字符串或symbol）
    if (typeof value === 'symbol') {
        if (Symbol && Symbol.prototype.toString) {
            return Symbol.prototype.toString.call(value);
        } else {
            return '';
        }
    }

    //  其他类型强转
    let res = `${value}`;

    // 此处有-0的判断技巧
    if (res === '0' && 1 / value === -INFINITY) {
        return '-0';
    } else {
        return res;
    }
}
```

## toNumber
```javascript
_.toNumber(value)
```
转化为数字
```javascript
const toNumber = value => {
    if (typeof value === 'number') {
        return value;
    }

    // symbol can not covert to number
    if (Object.prototype.toString.call(value) === '[object Symbol]') {
        return NaN;
    }

    // valueof for object
    if (isObject(value)) {
        let val = typeof value.valueOf === 'function' ? value.valueOf() : value;

        value = isObject(val) ? `${val}` : val; // what for?
    }

    // 字符串
    if (typeof value === 'string') {
        value = value.trim();

        // 对于2进制(0b)、8进制(0o)、16进制(0x)的转化
        // if (/^0b[01]+$/i.test(value) ||
        //    /^0o[0-7]+$/i.test(value) ||
        //    /^0x[0-9a-f]+$/i.test(value)) {
        //        return +value;
        // }

        // 源码对于这个正则的判断是为了啥?
        // /^[-+]0x[0-9a-f]+$/i，只用判断16进制么，2进制和8进制带符号的同样也转不过来的啊
        return +value;
    }

    return +value; // 其他情况
}
```

## toFinite
```javascript
_.toFinite(value)
```
转化为有限数字
```javascript
const INFINITY = 1 / 0;
const MAX_INTEGER = 1.7976931348623157e+308;

const toFinite = value => {
    if (!value) {
        return 0;
    }

    value = toNumber(value);

    if (value === INFINITY || value === -INFINITY) {
        let sign = value < 0 ? -1 : 1;

        return sign * MAX_INTEGER;
    }

    return value === value ? value : 0; // NaN
}
```

## toInteger
```javascript
_.toInteger(value)
```
转化为整数
```javascript
const toInteger = value => {
    value = toFinite(value);

    let reminder = value % 1;


    return value === value ? value - reminder : 0;
}
```

## toSafeInteger
```javascript
_.toSafeInteger(value)
```
转化为安全整数
```javascript
const toSafeInteger = value => {
    value = toInteger(value);

    if (value < Number.MIN_SAFE_INTEGER) {
        return Number.MIN_SAFE_INTEGER;
    } else if (value > Number.MAX_SAFE_INTEGER) {
        return Number.MAX_SAFE_INTEGER;
    }

    return value;
}
```

## toPlainObject
```javascript
_.toPlainObject(value)
```
转化为plain object，就是for...in遍历后取到的所有键值对（包括原型链上的属性值）
```javascript
const toPlainObject = value => {
    // convert to object
    value = Object(value);

    let res = {};

    for (let key in value) {
        res[key] = value[key];
    }

    return res;
}
```