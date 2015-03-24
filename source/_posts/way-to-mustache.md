title: way-to-mustache
date: 2015-03-23 10:51:41
tags:
---

先拐远一点：mustache，读[ˈmʌsˌtæʃ]，而不是[ˈmʌstˌek]，胡子的意思，取这名字大概是将模板引擎中的`{`顺时针转个90度就像个人的胡子了，嗯，大概是这样...

再进入正题：mustache是一个轻逻辑的模板引擎（为啥叫“轻逻辑”呢，因为没有if else for等逻辑语句，取而代之的是只用标签实现 ）

mustache.js是mustache模板系统的js实现，或者叫解析器。下面介绍下使用方法，很简单的语法~

## quick example

    Mustache.render('<p>I am {{name}}, I am {{age}} years old</p>', {
        title: 'xuqi',
        age: function() {
            return 24;
        }
    });
上述例子输出一段HTML片段：`<p>I am xuqi, I am 24 years old</p>`。

首先上述例子给我们最直观的印象就是：templates + data => html-partials。使用render(tpls, dataObj)。使用dataObj去渲染tpls得到html片段。

## 模板
模板就是一段包含任意数目`{{keyname}}`的字符串~  
`{{keyname}}`中的包裹的keyname即为mustache模板的键名（稍微了解下就好）。下面介绍三种加载模板的方法
1 直接在js中声明模板字符串，如quick example所示
2 从HTML中读取模板

    <script id="ex-tpl" type="x-tmpl-mustache">
        My name is {{name}}, I am {{age}} years old
    </script>

    //example.js
    var tpl = $('#ex-tpl').html(),
        html;
    html = Mustache.render(tpl, data);
3 从.mst文件中读取/通过ajax异步渲染  

    function loadTpl() {
        $.get('tpl.mst', function(tpl) {
            var html = Mustache.render(tpl, data);
            //Operation of html
        });
    }

## 变量
最简单的标签就是一个变量`{{key}}`...使用当前上下文环境中的key值替换标签，如果key不存在，则不会被渲染。

`{{key}}`会默认转义一些HTML标记，如果不想使用转义， 可以使用`{{{}}}`或者`{{&}}`。

另外，JS中的`.`也可以在mustache中使用，比如：`{{person.name}}`

## 区块

    {{section}}
        {{name}}
        //other code
    {{/section}}

根据上下文环境中的section的值去渲染区块内的代码。如果section键名不存在或者存在但是值为'false'值（null，undefined，false，0，NaN，''，[]），区块内代码不会被渲染。

键名存在也为'true'值的情况下，区块内代码被渲染1+次。比如：

    //view
    {
        section: [
            {name: 'xuqi'},
            {name: 'xuqi2'}
        ]
    }

    //template
    {{section}}
        <b>{{name}}</b>
    {{/section}}

    //output
    <b>xuqi</b>
    <b>xuqi2</b>
我们把上面的view简化一下：

    {
        section: ['xuqi', 'xuqi2']
    }
对，就是把section的元素由对象简化为了字符串，那键名没了我怎么去渲染？`{{.}}`来帮你~

    {{section}}<b>{{.}}</b>{{/section}}
这样就可以得到同样的output啦。

再来看一种稍微复杂一点的情况：外国人都有firstname和lastname，如果按照上面的情况去实现就得`{{firstname}}.{{lastname}}`去手动拼接。所以现在要引入函数来帮我们实现拼接了，而不是把逻辑扔在template中去实现。

如果section中有个变量是一个函数，那么会遍历对应上下文中的各个项并执行，例如：
  
    //view
    {
        'section': [
            { 'firstName': 'qi', 'lastName': 'xu' },
            { 'firstName': 'qi2', 'lastName': 'xu' }
        ],
        'name': function () {
            return this.firstName + '.' + this.lastName;
        }
    }

    //template
    {{#section}}
        <b>{{name}}</b>
    {{/section}}

    //output
    <b>qi.xu</b>
    <b>qi2.xu</b>
## 取反区块
`{{^section}}`与区块的情况相反，section的值为'false'值时执行，否则不执行

## 区块函数
如果section键名为函数，即`{{#sectionFn}}{{/sectionFn}}`中sectionFn为函数时，是个奇葩，官方文档看不懂是什么意思，直接来看例子吧~~

    //view
    {
        name: 'xuqi',
        bold: function() {
            return function(text, render) {
                return '<b>' + render(text) + '</b>';
            }
        }
    }

    //tpl
    {{#bold}}Hi {{name}}{{/bold}}

    //output
    <b>Hi xuqi</b>

function里面的实现意思就是：render(text)替换`{{bold}}`块中的内容并将数据渲染进去，然后把`{{bold}}`替换成`<b>`标签就行了
## 注释
`{{!}}` 内容不显示

## partials
`{{> partial-name}}`引入某个小部件到模板中，比如

    base.mst
    {{#name}}
        {{> user}}
    {{/name}}

    user.mst
    <b>{{name}}</b>
mustache将user.mst的内容插入到base.mst中`{{> user}}`的位置。 

如果在Mustache.render中使用partials的话，将partials当做render的第三个参数即可，即：

    Mustache.render(tpl, data, {
        user: userTplText,
        ...
    });

## 预编译和缓存模板
默认情况下Mustache在第一次编译模板后会缓存起来，如果第二次的模板没有变化，就直接从缓存中取编译后的模板去渲染，加快了渲染速度。如果你需要提前预编译和缓存，可调用Mustache.parse(tpl);.....一会儿以后，你就可以享受预编译模板带来的渲染快感了