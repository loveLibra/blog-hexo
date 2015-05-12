title: Handlebars使用指南
date: 2015-04-30 17:49:41
tags:
---

{% raw %}
[Handlebars](http://handlebarsjs.com/)为Mustache的超集，即：在完全兼容Mustache语法的基础上，提供了很多语法糖，不要太甜...

## Begin Now
### 写在前面
这里我们只介绍一些Handlebars中相对于Mustache超集的部分，若不了解Mustache，请[移步](http://i0011.com/2015/04/30/Mustache%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/)

### Expressions
表达式就是{{}}里面包的东西，Handlebars中语法的最小单位。{{xx}}表示在**当前上下文环境**中去寻找xx属性，并用xx属性的值去替换之。Handlebars支持`.`分隔的属性{{xx.yy}}，但在有的情况下`.`不能解决我们的问题，比如:

    data = {
        articles: [
            {
                '#comments': {
                    author: 'dabai',
                    date: '2015.4.30'
                }
            },
            {
                '#comments': [
                    {
                        author: 'xuqi',
                        date: '2015.4.30'
                    },
                    {
                        author: 'xuqi-two',
                        date: '2015.4.30'
                    },
                    {
                        author: 'xuqi-three',
                        date: '2015.4.30'
                    },
                    ...
                ],
                ...
            }
        ]
    }


需要渲染articles数组第2项的'#comments'的数据，我们用`.`怎么分隔？`articles.2.#comments`?...呵呵哒...跟js一样嘛，我们可以用`[]`:

    {{# each articles.[10].[#comment]}}
        <h1>{{author}}{{date}}</h1>
    {{/ each}}

上述这种情况在需要取数据某一元素或者渲染的属性名不是合理的Handlebars标识名时适用。

### Helper
Handlebars中可以让我们体验飞一般感觉的重量级语法糖，精华所在，可以帮助我们现在前段时间因为使用简单Mustache语法而引入的很多不必要的数据嵌套的问题，我们可以用`if`了，可以用`each`了，可以自定义了Helper了~欢呼雀跃吧....好吧，冷静，回归正题。

先从一个例子引入，然后逐步强化，里面有干货：
Level1：

    //tpl
    {{link 'http://www.baidu.com' '百度'}}
    
    //helper
    Handlebars.registerHelper('link', function(url, name) {
         return '<a href="' + url + '">' + name + '</a>';
    }); 

Level2：

    //tpl
    {{link site}}
     
    //helper
    Handlebars.registerHelper('link', function(site) {
        return '<a href="' + site.url + '">' + site.name + '</a>'
    });
     
    //data
    {
        site: {
            url: 'http://www.baidu.com',
            name: '百度'
        }
    };
     
Level3： 

    //tpl
    {{link '百度' url='http://www.baidu.com' class='site'}}
    
    //helper
    Handlebars.registerHelper('link', function(name, options) {
        var attrs = [],
            prop;
        for (prop in options.hash) {
            attrs.push(prop + '="' + options.hash[prop] + '"');
        }
        
        return '<a ' + attrs.join(' ') + '>' + name + '</a>'
    });
    
上述三个等级分别演示了Helper中参数传递的方法的功能递进， 下面介绍两个Handlebars的API为Helper打辅助：`Handlebars.SafeString()` 和 `Handlebars.escapeExpression()`。 

Helper返回的HTML字符串会被默认转义， 显示在界面上的只是文本节点而不是元素节点。所以， 对于这种自定义的Helper， 因为我们能确保返回的字符串是安全的， 因此我们需要告诉编译器这是一个安全的字符串， 放心当成元素节点插进去吧， 使用的方法就是SafeString()； 
另外， 因为Helper函数中有输入参数而我们不能确定这些数据是否含有XSS脚本， 如此那我们便手动将你转化成安全的字符串吧， 如此escapeExpression()的用法也自然就知晓了， 现在用这两个方法完善下上述Level3的Helper的例子:

    Handlebars.registerHelper('link', function(name, options) {
        var attrs = [],
            prop;
        for (prop in options.hash) {
            attrs.push(
                Handlebars.escapeExpression(prop) +'="' +
                Handlebars.escapeExpression(options.hash[prop]) + '"'
            );
        }
        
        return new Handlebars.SafeString('<a ' + attrs.join(' ') + '>' +
            Handlebars.escapeExpression(name) + '</a>');
    });


Level4：BLOCK-Heplers

    {{#bold}}
        My name is {{name}}
    {{/bold}}
    
    Handlebars.registerHelper('bold', function(options) {
        return new Handlebars.SafeString('<b>' + options.fn(this) + '</b>');
    });
    
    {name: 'xuqi'}

可以看到跟上述三个步骤中的Tpl不一样，这次好像不是简单的代码替换了，因为Helper变成了一个区块，区块中还有内容， 那该怎么办？注意本例中我需要一个文字加粗的功能，在返回字符串中我加了`<b>`标签实现加粗，中间那个options.fn(this)是干啥的？

前面讲了`<b>`实现了加粗，那加粗总该有个对象吧，也就是`My name is {{name}}`； 
那我也总不能直接把这个模板字符串给加粗输出吧，我得编译然后根据上下文环境的属性值name去生成My name is xuqi。所以呢，option.fn的功能就自然出来了，就是在this上下文中执行编译Block中的模板并填充数据得到HTML片段。
而this就是Block Helper执行的当前上下文环境。 

另外，可以通过this取得当前上下文的属性值手动进行某些操作，比如可以通过this.name获取name的属性值为xuqi。

#### Ps: 一些有用的Build-In Helper
* {{#each arg}}
循环内可以通过{{@index}}获取当前的循环的索引， 通过{{@../index}}可以获得父级遍历的索引；  
{{@key}}可以获取当前遍历的数组或者对象的键值，数组中等同于{{@index}}；  
{{@first}}，{{@last}}表示遍历中的第一个和最后一个的标识，返回true或false；  
each可选择性插入{{else}}， 在遍历的list为**空**(遍历对象为非空数组或者对象时不为空值)时执行。

* {{#if condition}} or {{else}} or {{else if conditionElse}}

* {{#unless condition}} inverse of the `if` helper

* {{with ctx}}
与Js的with一样，改变当前context，可以避免含有深层次嵌套属性时重复书写父级的名字

        //tpl
        <h1>{{title}}</h1>
        {{#with story}}
            <div>{{intro}}</div>
            <div>{{body}}</div>
        {{else}}
            {{!-- 可选择性插入else，仅当story为false值时渲染 --}}
            <div>I am Empty</div>
        {{/with}}
    
        //data
        {
            title: 'Hello world',
            story: {
                intro: 'Before the jump',
                body: 'After the jump'
            }
        }
* {{lookup context key}} 获取动态参数进行渲染，相当于获取context.key的值，看例子会容易理解点：

        //tpl
        {{#each names}}
            {{.}} : {{lookup ../foo @key}}
        {{/each}}
    
        //data
        {
            names: {
                a: 'Name1',
                b: 'Name2',
                c:'Name3'
            },
            foo: {
                a: 'Foo1',
                b: 'Foo2',
                c: 'Foo3'
            }
        }
    
        //output
        Name1 ：Foo1
        Name2 ：Foo2
        Name3 ：Foo3
        
* {{log info}} 打印调试

### Partials
Handlebars.registerPartial('parName', 'parContent'); 注册一个Partials；

需要调用Partials时 {{> parName}}即可。

调用Partials时也可以传入参数：{{> parName *parContext*}}指定Partials的执行上下文；{{> parName *attr=val*}}

### Path
{{xx}}中的xx可以不只是我们熟悉的一个属性的名字，或是xx.yy这样的嵌套路径。也能支持`../`这样的父级上下文环境和`./`这样的当前上下文环境的语法啦，来个例子：
    
    //tpl
    {{# nest}}
        {{../hello}}
    {{/ nest}}

    //data
    {
        nest: [
            ...
        ],
        hello: 'I am Hello'
    }

./是用来解决Helper名字和属性名重名的冲突的情况，例如./xx表示的是属性名而不是Helper名。另外this/name和this.name也实现相同的功能。

当然啦， 如果嵌套层级很深， 而我想访问到根作用域的时候是不是要写成`../../..`这样呢？ 是不是sa！ 那该咋办？ Handlebars提供了一个@root变量轻轻松松访问到根作用域：

    {{#each sub}}
        {{@root.someAttr}}
    {{/each sub}}
    
### Comments
{{!-- --}}和{{! }}不会将注释内容输出到HTML中，如果想输出注释到HTML中可以使用HTML的注释语法`<!-- -->`。另外，如果注释中有`}}`，则必须使用{{!-- --}}。

## 后语
上面讲的只是一些基本用法，冰山一角，官网上面还有很多文章中没有提及但是也许会给你的不一样的快感的功能。Helper的实现方式需要着重理解，几个Build-In Helper在官网上都提供了Helper实现方式，明白思想->模仿实现->定义自己的Helper实现特定的功能，go ~
{% endraw %}