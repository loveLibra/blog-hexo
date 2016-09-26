title: "React server side render"
date: 2016-01-24 20:20:17
tags: ['React', 'Ssr', 'Koa', 'Browserify', 'Postcss', 'Es6']
---
如何使用React开发高效可<del>装B</del>用的WEB应用:) ... 融合React以及React-ssr、koa、Es6、Postcss，绽放吧，骚年...

[DEMO传送门>>>>](https://github.com/loveLibra/React-ssr-demo)

1. JSX支持Runtime
`require('node-jsx').install({harmony: true});`。添加harmony选项可在JSX中使用部分ES6的特性，但是未完全支持.

2. browserify打包客户端脚本
```javascript
browserify('./js/index.js')
    .transform(babelify, {
        presets: ['es2015', 'react']
    })
    .bundle()
    .pipe(fs.createWriteStream('dist/bundle.js'));
```
3. React服务端渲染
将一个纯View用在此处其实有点别扭，但是为了SEO以及一些页面渲染性能的考虑，需要服务端渲染。React-SSR就是通过`ReactDOMServer.renderToString`得到HTML渲染到页面中，然后在客户端重新初始化组件，React并不会蠢的去重新渲染页面，所以此处可以忽略前端再次初始化带来的性能和使用体验的问题...前端初始化组件也是需要带数据的，所以在后端渲染组件时，我们将数据也带带页面中，然后再前端初始化后再删除数据DOM即可.
另外，在渲染字符串到页面中时，`views/partials/content.html`中放置渲染好的HTML的标签:
```html
<!-- 不能有换行，否则客户端渲染时因为识别空白节点的问题在前端初始化失败。-->
<div id="yoho-container">{{{content}}}</div>
```
4. Koa
koa + yield真的是太明智了，让代码不再一层层的去嵌套回调，棒！路由，接口取数据，数据格式化，页面渲染能对程序员展现以同步的形式去实现，跳出恶魔金字塔吧，Koa带你飞~
5. Postcss
Postcss相对于Compass的优势，在于其灵活性和轻便性，需要什么功能，扩展postcss插件就行了，autoprefixer，sprite-image应有尽有，Get what You Need...

后记：前端现在愈发膨胀和夸张，无数的轮子和框架来方便你的开发，然而得有度，别让自己的程序失去控制...前端现在就像一匹飞奔的马，跑的很快，但是跑到哪你却不知道，你唯一能做的就是练好骑马的技术，别从马上摔下来。夯实基础，成就你的精湛马术吧

精彩的参考资料:
[Rendering React Components on the Server](http://www.crmarsh.com/react-ssr/)