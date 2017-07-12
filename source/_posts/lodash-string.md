title: lodash-string
date: 2017-07-12 19:00:27
tags: [Lodash, String]
---
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

