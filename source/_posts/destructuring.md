title: destructuring
date: 2016-09-25 11:48:01
tags: Es6
---

**destructuring**，析构，可以方便coder们从Object/Array中提取想要的数据，Es6大杀器...

按照阮老师的说法，析构其实就属于一种数据结构的模式匹配，匹配上的模式对应的值会从表达式右侧赋值给表达式左侧，比如：
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
我们就能直接获取**模式**name和age的值，赋值给**变量**name和age，需要注意的是模式和变量的概念，这是理解析构的基础，
`{mode: var}`，`：`左侧为模式，右侧为变量，上述例子是当模式和变量名相同时的缩写，实际上是：
```javascript
let {name: name, info: {
    age: age
}} = ...
```
这样理解起来就清楚了。析构可以从数组和对象中提取值，分别来说明

## Array destructuring

### basic variable assignment
```javascript
let [one, two, three] = [1, 2, 3];`
```
当然，变量声明和析构赋值可以分开进行：
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
另外，阮老师的书里面说到的一种情况可能平时不太会经常用到，但是还是需要注意下：在默认值是通过函数返回值赋值的情况下，
函数是惰性求职的，即在不使用默认值的情况下，函数是不会被执行的
```javascript
const fn = () => {
    console.log('Hey, I am here');
    return 1;
};

let [a = fn()] = [1];

// fn didn't invoke
```
当然默认值也可以引用变量(外部变量/析构变量)，当默认值引用析构变量时，需要注意在设置默认值时必须确保引用的析构变量已经
声明，否则将报错
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

### parsing an array returned from a function && ignore some variable
我们常见的场景，需要通过函数返回多个变量，通过析构的话就可以拿到返回的多个值，并且可以忽略某个或多个值。
```javascript
const fn = () => [1, 2, 3];

let [a, , b] = fn();

a; // 1
b; // 3
```

## Object destructuring