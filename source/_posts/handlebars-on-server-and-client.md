title: 在服务端和客户端使用Handlebars
date: 2015-01-21 09:11:27
tags: [Plugin,Solution]
---

英文原文：<http://tilomitra.com/handlebars-on-the-server-and-client/#comment-13139>

昨天在Node上尝试使用Handlebars，之前没搞清服务端Handlebars和客户端Handlebars的区别,导致在使用Handlebars的模板时很茫然，昨天终于在google找到真爱，解决了问题，下面阐述下我遇到的问题后就正式进入大神的文章，带着问题看可以理解更深。

问题：
```html
    <script id="login-signup-tpl" type="text/x-handlebars-template">
        ...
        {{#if isLogin}}
            登录
        {{^}}
            注册
        {{/if}}
        ...
    </script>
    <script>
        var tpl = Handlebars.compile($('#login-signup-tpl').html());
        $('body').append(tpl({isLogin: true}));
    </script>
```
功能就是使用Handlebars编译模板并传入json然后得到对应的HTML片段。不过试一下就知道，这样时得不到你想要的东西的。下面带着问题看文章吧...

<!-- more -->

# 在服务端和客户端使用Handlebars

这篇文章讲的主要是关于如果在客户端和服务端使用Handlebars，以及如何避免一些使用中的陷阱。

## 服务端使用Handlebars

NodeJS中我选择[Handlebars](http://handlebarsjs.com/)作为我的视图引擎。我喜欢这种看上去只是多了一些花括号和helper的HTML，而不是像EJS和Jade那样跟HTML有很大区别的视图引擎。如果你之前还没有使用过Handlebars，下面的代码段就是一段简单的模板示例：
```html
    <div class="entry">
        <h1>{{title}}</h1>
        <div class="body">
            {{body}}
        </div>
    </div>
```
在构建一个基于Express的Web应用的时候，我选择[express-handlebars](https://github.com/ericf/express-handlebars)去建立管道来使用Handlebars作为视图引擎。它的作者是我的好朋友和大学同学Eric，他是我见过的最好的工程师之一。我强烈推荐express-handlebars。你可以很方便的通过npm安装：

    npm install express3-handlebars
查看express-handlebars的[基本使用](https://github.com/ericf/express-handlebars#basic-usage)可以找到一些如何使用Express使用它的信息。

如果你想在基于Express的应用中使用Handlebars，可以参考我的[node-boilerplate](https://github.com/tilomitra/node-boilerplate/)。

## 客户端使用Handlebars
如果你正在使用任何Javascript MV*框架（BackboneJs，EmberJs，YAF等），你也许需要在客户端使用一个模板库。Handlebars在所有这些框架中都能很好的被用起来。在客户端使用Handlebars最酷的事情之一就是你可以在客户端和服务端之间共享模板。你可以通过把模板存储在partials，然后在服务端以及在客户端通过`script`标签来分别使用。

## 服务端使用Handlebars时的客户端模板设置
如果你在客户端和服务端使用Handlebars，你会遇到这样的[问题](http://stackoverflow.com/questions/10037936/node-js-with-handlebars-js-on-server-and-client)，客户端的模板会被服务端的视图引擎解析。例如：
```html
    <h1>My Page Title</h1>
    <!-- This template should be transformed into HTML by the server -->
    <div id="photoList" class="pure-g">
        {{#each photos}}
            <div class="photo pure-u-1-12" data-photo-id="{{id}}">
                <img class="pure-u-1" src="{{src}}">
            </div>
        {{/each}}
    </div>
    <!-- This template should not be touched. It's for the client -->
    <script type="text/x-handlebars" id="lightbox-template">
        <img class="lightbox-image" src="{{large}}">
        <div class="lightbox-meta">
            <a class="pure-button lightbox-link" href="{{url}}">View on Flickr</a>
            <button class="pure-button lightbox-link lightbox-hide">Hide</button>
        </div>
    </script>
```
在浏览器上查看页面时,handlebars模板脚本会被解释成完全无用的代码段：
```html
    <script type="text/x-handlebars" id="lightbox-template">
        <img class="lightbox-image" src="">
        <div class="lightbox-meta">
            <a class="pure-button lightbox-link" href="">View on Flickr</a>
            <button class="pure-button lightbox-link lightbox-hide">Hide</button>
        </div>
    </script>
```
不过幸运的是，解决这个问题的方法很简单。只用在Hbs表达式之前加一个反斜杠就可以了。反斜杠会在编译的时候才会被解析，这样的话你可以得到一个可用的客户端模板:
```html
    <!-- Add a \ before the handlebars -->
    <script type="text/x-handlebars" id="lightbox-template">
        <img class="lightbox-image" src="\{{large}}">
        <div class="lightbox-meta">
            <a class="pure-button lightbox-link" href="\{{url}}">View on Flickr</a>
            <button class="pure-button lightbox-link lightbox-hide">Hide</button>
        </div>
    </script>
```
最后，回到文章开始的问题上。客户端使用Handlebars不能正确解析的原因是因为在服务器端渲染页面的时候，Handlebars模板已经被编译了，加上反斜杠后就可以阻止服务端的编译。