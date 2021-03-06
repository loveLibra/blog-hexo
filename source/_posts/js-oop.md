title: "js-oop"
date: 2016-01-11 15:50:22
tags: ["Js Basic", "OOP", "Es6"]
---
Javascript可能是一门让人感觉“不适”的语言，实现`类`、`继承`等OOP概念竟然要通过`function`和`prototype`，Jser们可能理解原型机制就能伤半管子HP了。然而，这种情况随着ES6的到来变得好了起来，ES6提供了`class`、`extends`等语法糖可以帮你摆脱让人苦闷的原型（当然，如果你有高追求，你得理解），虽然JS的OOP的实现原理还是通过原型的，但是至少对开发者的友好性提升了很多，让JS更有OOP的味道也更接近于其他OOP的语言。现在我们也能很简单的来声明一个类：
```javascript
class People {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    say() {
        alert(`Hello, I am ${this.name}`);
    }
}
```
<!-- more -->
这要是放ES5时代呢？
```javascript
function People(name, age) {
    this.name = name;
    this.age = age;
}

People.prototype.say = function() {
    alert('Hello, I am ' + this.name);
}
```
两者new对象的实现方式一致：
```javascript
    var p = new People('xuqi', 26);
    p.say();
```
对比两者的实现，可以大致了解ES6提供的类的实现细节是怎样的。

接下来，我们来构建一个Woman类，继承自People类，并实现`buy`方法。先看ES5的两种实现方式（PS：可结合扩展，比如添加自己的factory方法等）:
```javascript
//方法一：Prototype
function Woman() {}

Woman.prototype = new People();

Woman.prototype.buy = function() {
    alert('双11，买买买');
};

Woman.prototype.constructor = Woman; //修正constructor
```
```javascript
//方法二：call/apply
function Woman(name, age) {
    People.call(this, name, age);
}

Woman.prototype.buy = function() {
    alert('双11，买买买');
};
```
方法一通过原型机制实现继承，设置子类的prototype为父类的一个实例化对象即可；方法二通过call，在子类的上下文环境中执行父类的构造函数即可获得父类的属性和方法。两者的方法扩展都需通过prototype去实现。

但是上述两种方法在例子中都有问题：通过原型的实现在绑定子类prototype为父类的实例化对象时，必须初始化父类，要不然父类的参数(name, age)就不能使用了；通过call/apply方法实现的子类的实例化对象无法使用父类绑定在prototype上的方法(say)。因此我们需要结合两者：
```javascript
function Woman(name, age) {
    People.call(this, name, age);
}

Woman.prototype = new People();

Woman.prototype.constructor = Woman;

Woman.prototype.buy = function() {
    alert('双11，买买买');
};
```

再来看看ES6的继承是怎样实现的：
```javascript
class Woman extends People {
    constructor(name, age) {
        super(name, age);
    }

    buy() {
        alert('双11，买买买');
    }
}
```
ES6的代码简洁明了，通过`extends`可以简单的实现类的继承。需要注意的是，子类的构造函数中需先调用`super()`从而生成一个父类的实例化对象即`this`，然后再对this进行扩展，因此在super调用前使用this会报错；在ES5中实现对象则相反，先生成子类实例化对象this，然后向this中扩展父类的方法和属性.

### 扩展：
1. ES5的继承实现中为什么需要`Woman.prototype.constructor = Woman;`？
对象都有`constructor`属性，为一个函数，标识构造出该对象的构造函数，对象默认的constructor为`function Object(){...}`；  
因此该问题中，若不重新制定constructor到Woman(){...}，通过Woman实例化出来的对象的constructor = Woman.prototype.constructor = (new People()).constructor = People(){...}。这显然不是我们想看到的。
2. ES5中父类和子类的关系到底是怎样串联起来的？
构造两条**继承链**：
（1）实例继承
（2）构造函数继承/静态属性方法继承
实现模式：
```javascript
//实例继承
Object.setPrototypeOf(Sub.prototype, Parent.prototype);
//等同于
Sub.prototype.__proto__ = Parent.prototype;

//构造函数继承
Object.setPrototypeOf(Sub, Parent);
//等同于
Sub.__proto__ = Parent;
```
3. 在ES5的例子中，我们通过`Woman.prototype = new People();`实现了实例继承，推导式：Woman.prototype.\__proto\__  =  (new People()).\__proto\__  =  People.prototype；根据2，我们可以完善上述ES5的继承的实现--补充构造函数的继承，添加如下代码：
```javascript
Object.setPrototypeOf ?
    Object.setPrototypeOf(Woman, People) :
    Woman.__proto__ = People;
```
4. 完整的继承实现后，我们可以得到如下等式：
```javascript
var sub = new Sub();

//true
sub.__proto__ === Sub.prototype;
sub.constructor === Sub;

sub instanceof Sub;
sub instanceof Parent;
Sub.prototype instanceof Parent;

Parent.prototype.constructor === Parent;

Object.getPrototypeOf(Sub) === Parent;
```

附：
![原型原理图](https://dn-xuqi.qbox.me/proto.jpg)