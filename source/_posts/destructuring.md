title: destructuring
date: 2016-09-25 11:48:01
tags: ES6
---

**destructuring**，解构，可以方便coder们从Object/Array中提取想要的数据，Es6大杀器...

按照阮老师的说法，解构其实就属于一种数据结构的模式匹配，匹配上的模式对应的值会从表达式右侧赋值给表达式左侧，比如：
```javascript
let {name, info: {age}} = {
    name: 'xuqi',
    info: {
        age: 27
    }
};

name; // 'xuqi'
age; // 27
```
我们就能直接获取**模式**name和age的值，赋值给**变量**name和age，需要注意的是模式和变量的概念，这是理解解构的基础，
`{mode: var}`，`：`左侧为模式，右侧为变量，上述例子是当模式和变量名相同时的缩写，实际上是：
```javascript
let {name: name, info: {
    age: age
}} = ...
```
这样理解起来就清楚了。解构可以从数组和对象中提取值，分别来说明...
<!-- more -->
## Array destructuring

### basic variable assignment
```javascript
let [one, two, three] = [1, 2, 3];
```
当然，变量声明和解构赋值可以分开进行：
```javascript
let one, two, three;
[one, two, three] = [1, 2, 3];
```

### default values
```javascript
let [a = 1, b = 2] = [3];

a; // 3
b; // 2
```
默认值生效的条件是右侧对应位置的值**严格**等于`undefined`，下述情况是不会发生默认值赋值的：
```javascript
let [a = 1] = [null];

a; // null
```
另外，阮老师的书里面说到的一种情况可能平时不太会经常用到，但是还是需要注意下：在默认值是通过函数返回值赋值的情况下，函数是惰性求职的，即在不使用默认值的情况下，函数是不会被执行的
```javascript
const fn = () => {
    console.log('Hey, I am here');
    return 1;
};

let [a = fn()] = [1];

// fn didn't invoke
```
当然默认值也可以引用变量(外部变量/解构变量)，当默认值引用解构变量时，需要注意在设置默认值时必须确保引用的解构变量已经声明，否则将报错
```
let a = 1;
let [x = a, y = x] = [];

x; // 1
y; // 1
```

### swapping variable
轻松实现变量交换
```javascript
let a = 1;
let b = 2;

[a, b] = [b, a];
```

### parsing an array returned from a function & ignore some variable
我们常见的场景，需要通过函数返回多个变量，通过解构的话就可以拿到返回的多个值，并且可以忽略某个或多个值。
```javascript
const fn = () => [1, 2, 3];

let [a, , b] = fn();

a; // 1
b; // 3
```

## Object destructuring
同数组，对象也可以按照结构进行解构赋值，文章开头已经举了一个basic assignment的例子，这里就继续说明其他的使用场景和注意事项。
### assignment without declaration
```javascript
let a, b;
({a, b} = {a: 1, b: 2});
```
需要注意的是，当解构赋值跟声明分开进行时，解构表达式外的`()`是必须得，否则解析左侧是会当成一个块级表达式，从而导致因=号的出现而报错。

### assignment to new variable name
对象的解构可以指定赋值的变量名：
```javascript
let {name: theName, info: {age: theAge}} = {
    name: 'xuqi',
    info: {
        age: 27
    }
};

theName; // 'xuqi'
theAge; // 27

```
之前已经介绍过模式和变量名的形式，这里就很好理解了，由于指定了变量名，name和age只是模式

### default values
跟数组一样，对象解构也可以设置默认值
```javascript
let {a = 1, b:c = 2} = {a: 2};

a; // 2
c; // 2
```
对象解构的默认值有一种比较重要的应用场景：函数的默认参数
```javascript
const fn = ({a = 1, b = 2} = {}) => {
    return [a, b];
};

fn();
```

这篇流水账的起因是因为TS2.0发布了然后去看了下文档，瞬间被加了type的解构表达式搞晕了，然后就想着写个文章来捋一捋，毕竟看别人文章跟自己写出来还是有点差别的，想看详细的还是看文档或者阮老师的书比较好，想要了解下或者重新梳理思路，这篇文章适合你

参考资料：  
[阮一峰《ECMAScript6入门》变量的解构赋值](http://es6.ruanyifeng.com/#docs/destructuring)  
[MDN Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
