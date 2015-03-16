title: Way To Pjax
date: 2015-03-16 13:11:37
tags:
---
Pjax = **P**ushState + A**jax**
Pjax使用在链接跳转页面的情形下，可以将跳转页面转化为页面内异步更新内容，并且同时浏览器URL、Title以及浏览器的回退等功能都能无异于页面跳转。效果可以参考Github上的文件/目录跳转加载，Github就是采用Pjax的方式去实现的。

Pjax的好处：
* 用户体验提升
无刷新页面极大的提升了用户的体验
* 减少资源加载
只更新页面中需要替换的那部分资源，layout框架部分不用重新加载
* 浏览器支持处理
在不支持PushState或者Ajax的浏览器中，Pjax的功能不被支持，但是不影响正常使用，Pjax会使用原始的页面跳转来处理这种情况

Pjax的实现原理：
pajx通过ajax从服务器端抓取html并把html填充到需要变更内容的容器中，然后通过PushState更新页面URL。(easy to understand,uh？But you are not the first one to eat crab ~~)

以上属于思想层面的东西，因为jquery.pjax的存在，下面将针对jquery.pjax做出使用说明。

--------

[Jquery.pajax](https://github.com/defunkt/jquery-pjax)

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
`$(document).pjax(selector, container, options)`
selector和container必须指定，但是方式可多选，可以直接通过参数列表传递也可以通过options对象传入。
* $.pjax.click
pajx的底层函数，使用方法：

        if ($.support.pjax) {
            $(document).on('click', '#switch', function(e) {
                $.pjax.click(e, {
                    container: $('#container')
                });
            });
        }
可以同样实现上述例子的效果。
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
