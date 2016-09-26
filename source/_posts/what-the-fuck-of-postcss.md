title: "What the f**k of Postcss"
date: 2016-01-28 13:37:31
tags: ['Postcss', 'HandBook']
---
> PostCSS is a tool for transforming styles with JS plugins. These plugins can lint your CSS, support variables and mixins, transpile future CSS syntax, inline images, and more.

对于Postcss，这样介绍来开启话题最合适不过。大部分人对Postcss的认知还是CSS pre-preprocessor，因此经常会有将Postcss和Sass/Less/Stylus进行对比的情况。那Postcss到底是什么？

Postcss只是一个工具，获取CSS内容并将其转化成JS插件可以处理的数据，如此基于Postcss的JS插件便可基于Postcss提供的数据进行对应的处理。目前Postcss Plugin的列表数目已经200+，提供诸于：auto-prefix、next-css、linter、short-name等功能。另外，由于Postcss的职能是类似于工具箱，用户可以选择自己的工具放进去，所以你可以根据自身的开发喜好和需求去开发自己的“工具”扔到工具箱里去，心有多大，世界就有多大，勇敢的少年啊，快去创造奇迹~
<!-- more -->

在认清Postcss的真实面目后，搬出下面这段摘抄的话，会显得逼格很高：
> * It’s not a pre-processor, though it can optionally behave like one.
* It’s not a post-processor, though it can optionally behave like one.
* It’s not about “future syntax”, though it can facilitate support for future syntax
* It’s not a clean up / optimization tool, though it can provide such functionality.
* It’s not any one thing; it’s a means to potentially unlimited functionality configured as you choose.

也就是，它看起来什么都是，然而本质上它什么都不是，但是它什么都能干。万剑归宗，万法归一，嗯，就是这种意思了...

口说无凭，祭出插件杀器：
* [autoprefixer](https://github.com/postcss/autoprefixer) 为指定的browsers生成兼容性CSS，你只需按照W3C的标准去书写你的样式就好了，接下的事autoprefixer帮你摆平
* [precss](https://github.com/jonathantneal/precss) 如果你需要像Sass提供的:variables、mixins、conditionals等功能，precss是不二之选
* [postcss-import](https://github.com/postcss/postcss-import) 使用@import，并且可以获取第三方的样式(比如bower或者npm)
* [cssnano](https://github.com/ben-eb/cssnano) css**优化**(清除注释和尾分号、合并规则，字体权重优化等等等...)，压缩...强大
* [postcss-assets](https://github.com/assetsjs/postcss-assets) img/font加载路径解决方案，另外还提供获取图片尺寸以及将图片转化为base64写入css的功能
* [postcss-sprites](https://github.com/2createStudio/postcss-sprites) image sprite
* ...颜色处理，可读性处理，简化输入，优化，打包，[自行探索>>>](http://postcss.parts/)

怎么去用？Gulp！你应该享受现代工具~
```javascript
var gulp = require('gulp'),
    postcss = require('gulp-postcss'),
    sourcemap = require('gulp-sourcemaps');
gulp.task('css', function() {
    gulp.src('css/**/*.css')
        .pipe(sourcemap.init())
        .pipe(postcss([
            require('autoprefixer')({
                browsers: ['not ie < 8']
            }),
            require('precss'),
            require('postcss-assets')({
                loadPaths: ['font/', 'img/'],
                relativeTo: 'css/post/'
            }),
            require('postcss-sprites').default({
                basePath: './img', //img base path
                stylesheetPath: './css', //path of css generated
                spritePath: './img', //path of sprites generated
                spritesmith: {
                    padding: 2,
                },
                groupBy: function(file) {
                    var group = file.url.split('/')[1]; // 根据目录分组，防止合并后的图片太大
                    return group ? Promise.resolve(group) : Promise.reject();
                }
            })
        ]))
        .pipe(sourcemap.write('.'))
        .pipe(gulp.dest('dist/'))
});
```

### 题外话
由于正在使用Compass，不自主会把这两个工具进行比较：
Compass是Sass的扩展，提供了很多牛X的Minix和功能(比如：无可比拟的image-sprite)。用过Compass的可能会有疑问了，你上面讲的在Compass都能做啊，而且我还不用去找很多插件来实现我的功能，去网站翻使用指南和它的API就可以了，不要太强大...然而，出问题的就是他的强，大：
1. Compass需要依赖Ruby，这一点就足够判死刑
2. Compass提供了很多@mixin，其中也包括类似于autoprefix的功能，但是，前提是：我得知道我要写的属性需要进行prefix处理，所以对大部分人来说，这个功能属于鸡肋功能。对于一些辅助的功能实现的mixin，我们可能会想到去官网搜索，然而，这个比prefix的处理更让人头疼，在一个全站颜色统一，甚至对搜索结果都不做突出的网站上找到想要的API是有多难，搜索结果下你会发现不知所云的搜索结果
3. 我们并不需要Compass提供的那么多内置mixin
4. 对compass认知程度有限，上述叙述若有偏颇，勿怪，个人观点...当然，不可否认还是有很多出色的功能的~

对比Compass，我更喜欢Postcss带来的体验，把一切交给npm，无需其他乱七八糟的环境的配置，通过一些基本的插件，可以实现从Compass到Postcss的无缝切换，而且Postcss的开放性必然会给以后带来诸多Compass所不会有的好处。插件多样性可以帮你处理各种常见问题：嫌每天重复输入相同的width,height...你可以用`postcss-short`或者`postcss-size`；嫌`#3f3f3f`这种输入浪费时间，`postcss-color-short`帮你解决后顾之忧，你只用输入`#3f`；想像写Es6那样享受超前的快感，`cssnext`带你装逼；想摆脱老是写水平居中或者垂直居中的困扰，`postcss-center`助你一臂之力；clearfix的功能在你只用写`clear:fix`的情况下，也有插件`postcss-clearfix`帮你铺平道路；甚至连css的语法检查，也有插件给你两肋插刀...总之大到模块化，小到画三角，应有尽有，各取所需就好...

灵活性+开放性+易用性+扩展性....足够了，写Postcss，挺好~
