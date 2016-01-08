title: "respondjs-proxy"
date: 2016-01-08 10:47:43
tags: [Solution, IE8]
---

**respond.js**用来解决IE9以下不支持媒体查询的问题。

在静态资源(CSS)和网站在同一域名的情况下使用起来很简单：
```html
    <link rel="stylesheet" href="path/to/cssfile.css">
    <script src="path/to/respond.js"></script>
```
引入HTML并置于css文件引用位置之后即可。  
<!-- more -->
然而大部分情况下并非如此，静态资源都会放在cdn上进行访问加速，这样就产生了域名不一致而导致respondjs在跨域的情况下失效的问题。Github上作者对该[问题](https://github.com/scottjehl/Respond#cdnx-domain-setup)的产生给出了阐述，感兴趣的可以去看下。[代码](https://github.com/scottjehl/Respond/tree/master/cross-domain)中也给出了解决示例。此处给出解决方法：
```html
<link href="path/to/respond-proxy.html" id="respond-proxy" rel="respond-proxy" />
<script src="path/to/respond.proxy.js"></script>
<link href="path/to/respond.proxy.gif" id="respond-redirect" rel="respond-redirect" />
```
再加载上述三行代码到HTML中即可。需要注意的是`respond-proxy.html`需要与静态资源放在同一域名下。

Enjoy it~