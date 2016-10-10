title: console what u need
date: 2015-03-06 14:13:14
tags: Js Basic
---

console能用来干嘛？console.log啊，用来debug我们的程序，浏览器或者terminal就能看到输出了。没错，引用[ZHANGXIANGLIANG同学挂在sf上面文章](https://segmentfault.com/a/1190000006721606?utm_source=weekly&utm_medium=email&utm_campaign=email_weekly)里面的话，这是凡人视角，还有不为人知的上帝视角~

**建议**先控制台打印下`console`本身，先感受下能有多少方法给我们用

* console.log
console.log输出信息到控制台，意思很简单，但格式化输出的功能可能会用的不多，这边提供一个列表供参考:
<table><tr><th>Format Specifier</th><th>Description</th></tr><tr><td>%s</td><td>Formats the value as a string.</td></tr><tr><td>%d or %i</td><td>Formats the value as an integer.</td></tr><tr><td>%f</td><td>Formats the value as a floating point value.</td></tr><tr><td>%o</td><td>Formats the value as an expandable DOM element</td></tr><tr><td>%O</td><td>Formats the value as an expandable JavaScript object.</td></tr><tr><td>%c</td><td>Formats the output string according to CSS styles you provide.</td></tr></table>
<!-- more -->
另外，console还提供了另外3种显示信息的方法：console.info、console.warn、console.error，分别用来显示一般信息、警告提示、错误提示，而console.debug只是console.log的一个alias

* console.clear
清空控制台

* console.group
分组显示信息:
```javascript
console.group('first group');
console.log('info1');
console.log('info2');
console.groupEnd();
```
group打印的信息是展开的，如果想默认折叠，可以用`console.groupCollapsed`  

另，分组是可以嵌套的哦，groupEnd就近匹配，与group[Collapsed]成对出现

* console.table
用表格显示需要打印的**对象数组**的信息，清晰明了

* console.dir
显示一个对象的所有属性和方法

* console.dirxml
显示网页某个节点的html/xml代码

* console.assert
断言判定

* console.trace
追踪函数的调用轨迹

* console.time / console.timeEnd
显示代码运行时间
```javascript
console.time('timer');
...
console.timeEnd('timer');
```
注意到console还有一对方法timeline / timelineEnd，用一下，发现提示`deprecated`，哦，跟time/timeEnd的用途一样的，快被废了，忽视~

* console.profile
性能分析，同chrome F12中的Profiler功能，chrome下console的分析的结果会直接显示在chrome的`Profiles`中
```javascript
console.profile('profile1');
...
console.profileEnd();
```
* console.count
统计执行次数，可以不用单独申明一个变量来帮我们做数字统计

* console.memory
这是console的一个**属性**，打印JS运行行堆栈的内存情况...这个有点蒙B，MDN文档好像都没提啊...

* console.timeStamp
在浏览器的timeline工具中加入一个marker(在timeline上会显示一个黄点)，这样的话可以将代码与timeline中的节点关联起来，分析起来更清晰一点

参考资料：  
[阮一峰 - Firebug控制台详解](http://www.ruanyifeng.com/blog/2011/03/firebug_console_tutorial.html)  
[sf - 你所不知道的Console](https://segmentfault.com/a/1190000006721606?utm_source=weekly&utm_medium=email&utm_campaign=email_weekly)  
[MDN - console](https://developer.mozilla.org/en-US/docs/Web/API/Console)