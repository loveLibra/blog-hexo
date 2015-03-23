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

首先上述例子给我们最直观的印象就是：templates + data => html-partials。使用`render(tpls, dataObj)`。使用dataObj去渲染tpls得到html片段。

## 模板
模板就是一段包含任意数目`{{}}`的字符串~  
{{keyname}}中的包裹的keyname即为mustache模板的键名（稍微了解下就好）。下面介绍三种加载模板的方法
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

{{}}会默认转义一些HTML标记，如果不想使用转义， 可以使用{{{unescaped-key}}}或者{{&unescaped-key}}。

另外，JS中的`.`也可以在mustache中使用，比如：{{person.name}}

## 区块

    {{section}}
        {{name}}
        //other code
    {{/section}}
根据上下文环境中的section的值去渲染区块内的代码。如果section键名不存在或者存在但是值为'false'值（null，undefined，false，0，NaN，''，[]），区块内代码不会被渲染。

键名存在也为'true'值的情况下，区块内代码被渲染1+次。
