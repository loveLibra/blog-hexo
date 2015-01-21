title: 在服务端和客户端使用Handlebars
date: 2015-01-21 09:11:27
tags:
---

英文原文：<http://tilomitra.com/handlebars-on-the-server-and-client/#comment-13139>

昨天在Node上尝试使用Handlebars，之前没搞清服务端Handlebars和客户端Handlebars的区别,导致在使用Handlebars的模板时很茫然，昨天终于google到来真爱，一下就解决了问题，下面阐述下我遇到的问题后就正式进入大神的文章，带着问题看理解更深。

问题：

    <script id="login-signup-tpl" type="text/x-handlebars-template">
        ...
        {{#if isLogin}}
            登录
        {{^}}
            注册
        {{/if}}
        ...
    </script>

目的就是想根据传入isLogin的值去渲染不同的值,但是在

    var tpl = Handlerbar.compile($('#login-signup-tpl').html());
    $('body').append(tpl({isLogin: true}));
后却在页面中得不到isLogin的值,也就是说会始终渲染后者。这也就是问题的所在，没有区分Handlebars在客户端和服务端使用的区别，下面带着问题看文章吧...


这篇文章讲的主要是关于如果在客户端和服务端使用Handlebars，以及在使用时如何避免一些陷阱。

##服务端使用Handlebar

NodeJS中我选择Handlebars作为我的视图引擎。我喜欢这种看上去只是多了一些{{}}和helper的HTML，而不是像EJS和Jade那样跟HTML有很大区别的视图引擎。如果你之前还没有使用过Handlebars，下面列举了一个简单的模板代码：

    <div class="entry">
        <h1>{{title}}</h1>
        <div class="body">
            {{body}}
        </div>
    </div>
在构建一个基于Express的Web应用的时候，我选择express-handlebars安装的组件去使用Handlebars作为视图引擎。它的作者是我的好朋友和大学同学Eric,他时我见过的最好的工程师之一。我强烈推荐express-handlebars。你可以很方便的通过npm安装：
    npm install express3-handlebars

查看express-handlebars的基本使用章节可以找到一些如果使用Express安装它的信息。

如果你想在基于Express的应用中使用Handlebars，可以参考我的node-boilerplate。

##客户端使用Handlebars
如果你正在使用任何Javascript MV*框架（Backbone,EmberJs,YAF等），你也许需要在客户端使用一个模板库。Handlebar在所有这些框架中都能很好的用起来。在客户端使用Handlebars最酷的事情之一就是你可以在客户端和服务端共享模板。你可以通过把模板分片存储，然后在服务端以及在客户端通过`<script>`标签来分别使用。

##服务端使用Handlebars时的客户端模板设置
如果你在客户端和服务端使用Handlebars，你会遇到这样的问题，客户端的模板会被服务端的视图引擎解析。例如：

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

你在浏览器上查看页面时。`<script id="lightbox-template">`会被解释成完全无用的代码段：

    <script type="text/x-handlebars" id="lightbox-template">
        <img class="lightbox-image" src="">
        <div class="lightbox-meta">
            <a class="pure-button lightbox-link" href="">View on Flickr</a>
            <button class="pure-button lightbox-link lightbox-hide">Hide</button>
        </div>
    </script>

不过幸运的是，解决这个问题的方法很简单。只用加一个`\`在{{之前就可以了。`\`会在编译的时候才会被解析，这样的话你可以得到一个可用的客户端模板:

    <!-- Add a \ before the handlebars -->
    <script type="text/x-handlebars" id="lightbox-template">
        <img class="lightbox-image" src="\{{large}}">
        <div class="lightbox-meta">
            <a class="pure-button lightbox-link" href="\{{url}}">View on Flickr</a>
            <button class="pure-button lightbox-link lightbox-hide">Hide</button>
        </div>
    </script>
