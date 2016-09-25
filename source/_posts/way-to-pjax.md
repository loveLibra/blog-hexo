title: Way To Pjax
date: 2015-03-16 13:11:37
tags: HandBook
---
### Pjax = **P**ushState + A**jax**

Pjax使用在链接跳转页面的情形下，可以将跳转页面转化为页面内异步更新内容，并且同时浏览器URL、Title以及浏览器的回退等功能都能无异于页面跳转。效果可以参考Github上的文件/目录跳转加载，Github就是采用Pjax的方式去实现的。

Pjax的好处：
* 用户体验提升
无刷新页面极大的提升了用户的体验
* 减少资源加载
只更新页面中需要替换的那部分资源，layout框架部分不用重新加载
* 浏览器支持处理
在不支持PushState或者Ajax的浏览器中，Pjax的功能不被支持，但是不影响正常使用，Pjax会使用原始的页面跳转来处理这种情况
<!-- more -->
Pjax的实现原理：
pajx阻止链接的点击默认事件，转为通过ajax从服务器端抓取html并把html填充到需要变更内容的容器中，然后通过PushState更新页面URL。(easy to understand,uh？But we are not the first one to eat crab ~~)

以上属于思想层面的东西，因为jquery.pjax的存在，下面将针对jquery.pjax做出使用说明。

--------

[Jquery.pjax](https://github.com/defunkt/jquery-pjax)

Start By An Example:

    //page1.html
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf8">
            <title>Pjax Example</title>
        </head>
        <body>
            <div id="container">
                Nothing
                <a id="switch-to-two" href="page2.html">Go to page2</a>
            </div>
            <div>Other parts</div>
            <script src="bower_components/jquery/dist/jquery.min.js"></script>
            <script src="bower_components/jquery-pjax/jquery.pjax.js"></script>
            <script>
                $(document).pjax('#switch-to-two', '#container');
            </script>
        </body>
    </html>


    //page2.html
    <p>I am page2</p>
    <a id="switch-to-one" href="page1.html">Go to pag1</a>

Tip：使用前bower安装一下jquery和jquery.pjax或者cdn引入下。动手运行感受下吧~

Usage:
* $.fn.pjax
初始化Pjax，可通过下述方法实现链接的pjax页面加载：


        $(document).pjax(selector, [container], options)
selector可以指定为a[data-pjax]，然后html中a标签中指定data-pjax为container即可省略列表中container参数。

还有另外一种方法初始化，但是不推荐：

    $('a[data-pjax]').pjax();

因为源码中初始化的过程是这样的：

    function fnPjax(selector, container, options) {
        var context = this
        return this.on('click.pjax', selector, function(event) {
            var opts = $.extend({}, optionsFor(container, options))
    
            if (!opts.container)
                opts.container = $(this).attr('data-pjax') || context
            handleClick(event, opts) 
        })
    }

在上下文环境（此处为document）中为selector绑定click.pjax事件并执行回调，回调中会首先拼接opts即将container和options两个参数合并成一个对象（`{container: '#container', otherOpt: ...}`）。
如果参数列表中没有传入container则会去读上下文环境中的data-pjax属性（上述不推荐的例子就是属于这种情况）或者直接将上下文环境当成container。由代码可以看到，如果采用不推荐的那种写法，基本功能虽然可以实现，但是此时如果我在列表中传入了一个options对象，其实这个时候options是无效的（会被当成selector），或者传入一个字符串选择器（本意是container），但是会被绑定一个原来链接元素才会有的事件。上述所有情况都会造成不预期的错误，虽然仍然以跳转页面进行处理，但这不是我们想要的结果。所以，尽量避免...（我看到好多stackoverflow上的都是这么写的，难道是我技术拙劣得不能理解了...如果你很知道的话一定要不吝指教啊）

options：

| Key            | Default   | Description                           |
| -------------- |:---------:| -------------------------------------:|
| timeout        | 650       | *timeout*毫秒后的强制刷新             |
| push           | true      | 使用pushState添加一个浏览器记录       |
| replace        | false     | url替换而不是添加(类比replaceState)   |
| maxCacheLength | 20        | 缓存以前container内容最大数目         |
| version        |           | string/function，当前pjax版本         |
| scrollTo       | 0         | 页面“刷新”后定位的scrollTop           |
| type           | 'GET'     |                                       |
| dataType       | 'html'    |                                       |
| container      |           |                                       |
| url            | link.href | string/function，URL for ajax request |
| target         | link      | 事件relatedTarget属性的值             |
| fragment       |           | 指定返回数据中某一段代码进行填充      |

另外，pjax允许你通过`$.pjax.defaults.xx`去修改默认值。
Ps: relatedTarget是当前事件涉及到的相关联的其他元素。比如mouseover会涉及到多个元素，从元素1离开然后进入元素2，此时relatedTarget就是元素1...你明白了吗？反正我明白了...涉及到这个属性的事件比较少，列如下：mouseenter，mouseout，mouseover，mouseleave，focus，blur。
* $.pjax.click
pajx的底层函数，功能是手动去把链接click事件转移到pjax实现，可以增加一些使用者对事件句柄event的控制。

        if ($.support.pjax) {//使用前应该判断浏览器是否支持pjax
            $(document).on('click', '#switch', function(event) {
                $.pjax.click(event, {
                    container: $('#container')
                });
            });
        }
通过上述可以同样实现上述例子的功能

* $.pjax.submit
用于实现通过Pjax提交表单数据

        $(document).on('submit', '#form', function(e) {
            $.pjax.submit(e, '#container');
        });

* $.pjax.reload
使用当前URL通过pjax机制向服务端请求数据并替换#container内的内容

        $.pjax.reload('#container', options);

* $.pjax
手动触发pjax，用于非click事件触发的一个请求

        $.pjax({
            url: url,
            container: '#container'
        });

* pjax event
pjax提供了很多事件来供我们实现Pjax过程中需要处理的细节功能，比如，上述例子中，在pjax请求完成后我打印一个'Complete'，就可以这样实现：

        $(document).on('pjax:complete', function() {
            console.log('Compelete');
        });

另外，pjax:send和pjax:complete是一对基友，可以帮我们实现类似于loading图标正确的显示和隐藏的功能，send时显示，complete时隐藏，语法同上，不赘述。

events list：
Ps：pjax:click和pjax:clicked是由链接元素触发的，其他事件由容器触发。
* `pjax:click`
* `pjax:beforeSend`
* `pjax:start`
* `pjax:send`
* `pjax:clicked`
* `pjax:beforeReplace`
* `pjax:success`
* `pjax:timeout`
* `pjax:error`
* `pjax:complete`
* `pjax:end`

下列事件会在浏览器后退/前进按钮时触发
* `pjax:popstate`
* `pjax:start` -- 内容替换前
* `pjax:beforeReplace` -- 从缓存中读取HTML替换之前
* `pjax:end` -- 内容替换后

---------
另外，如果你使用spm，jquery.pjax也被简单了封装了一下放在了上面，你可以通过`spm install jquery-pjax --save`获取。