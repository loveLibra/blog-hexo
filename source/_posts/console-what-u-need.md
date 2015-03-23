title: console what u need
date: 2015-03-06 14:13:14
tags:
---
Mark~，console是前端er最熟知的工具了，但是冰山一角的道理咱还是自知的，学习学习...此处列出简单的列表和功能说明，若有不理解之处，请移步[阮一峰博客](http://www.ruanyifeng.com/blog/2011/03/firebug_console_tutorial.html)查看详细信息。

* console.log
console.log输出信息到控制台，意思很简单，但格式化输出的功能可能会用的不多，这边提供一个列表供参考:
<table><tr><th>Format Specifier</th><th>Description</th></tr><tr><td>%s</td><td>Formats the value as a string.</td></tr><tr><td>%d or %i</td><td>Formats the value as an integer.</td></tr><tr><td>%f</td><td>Formats the value as a floating point value.</td></tr><tr><td>%o</td><td>Formats the value as an expandable DOM element</td></tr><tr><td>%O</td><td>Formats the value as an expandable JavaScript object.</td></tr><tr><td>%c</td><td>Formats the output string according to CSS styles you provide.</td></tr></table>

另外，console还提供了另外4种显示信息的方法：console.info、console.debug、console.warn、console.error，分别用来显示一般信息、除错信息、警告提示、错误提示

* console.group
分组显示信息,usage:

        console.group('first group');
        console.log('info1');
        console.log('info2');
        console.groupEnd();

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


        console.time('timer');
        ...
        console.timeEnd('timer');

* console.profile
性能分析，同chrome F12中的Profiler功能


        console.profile('profile1');
        ...
        console.profileEnd();