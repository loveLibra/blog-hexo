title: Way To Pjax
date: 2015-03-16 13:11:37
tags:
---
###Pjax = **P**ushState + A**jax**

Pjax使用在链接跳转页面的情形下，可以将跳转页面转化为页面内异步更新内容，并且同时浏览器URL、Title以及浏览器的回退等功能都能无异于页面跳转。效果可以参考Github上的文件/目录跳转加载，Github就是采用Pjax的方式去实现的。

Pjax的好处：
* 用户体验提升
无刷新页面极大的提升了用户的体验
* 减少资源加载
只更新页面中需要替换的那部分资源，layout框架部分不用重新加载
* 浏览器支持处理
在不支持PushState或者Ajax的浏览器中，Pjax的功能不被支持，但是不影响正常使用，Pjax会使用原始的页面跳转来处理这种情况

Pjax的实现原理：
pajx通过ajax从服务器端抓取html并把html填充到需要变更内容的容器中，然后通过PushState更新页面URL。(easy to understand,uh？But we are not the first one to eat crab ~~)

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
初始化Pjax，可通过下述三种方案实现链接的pjax页面加载：
**方法一：**

        $(document).pjax(selector, container, options)
参数列表指定链接的选择器selector、HTML容器container以及其他一些选项以对象方式传入。
**方法二：**

        $(document).pjax('a[data-pjax]', options)
此方法可以指定所有包含data-pjax属性的链接元素使用pjax进行页面加载。在链接元素中我们指定data-pjax的属性值为container的选择器可以省略container参数；若不指定则按照方法一方式传入或者options中增加container属性对即可。
**方法三：**

        $(document).pjax({
            target: 'a',
            container: '#container',
            ...
        });
所有参数通过一个对象传入，包括target/selector和container。

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
| target         | link      |                                       |
| fragment       |           | (还没有理解，后面补充)                |

另外，pjax允许你通过`$.pjax.defaults.xx`去修改默认值。

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

另外，pjax:send和pjax:complete是一对基友可以帮我们实现正在加载的图标的显示和隐藏。send时显示，complete时隐藏，语法同上，不赘述。

events list：
1.pjax链接时事件

| event              | args                         | notes                   |
| ------------------ |:----------------------------:|:-----------------------:|
| pjax:click         | options                      | 链接激活后触发          |
| pjax:beforeSend    | xhr,options                  | 此时可以设置XHR头部信息 |
| pjax:start         | xhr,options                  |                         |
| pjax:end           | xhr,options                  |                         |
| pjax:clicked       | options                      | 链接点击后pjax开始触发  |
| pjax:beforeReplace | contents,options             | 替换HRML之前触发        |
| pjax:success       | data,status,xhr,options      | HTML替换之后触发        |
| pjax:timeout       | xhr,options                  | 超时timeout时触发       |
| pjax:error         | xhr,textStatus,error,options | ajax出错时触发          |
| pjax:complete      | xhr,textStatus,options       | ajax完成后触发          |
| pjax:end           | xhr, options                 |                         |

2.浏览器前进后退按钮时事件
pjax:popstate，pjax:start，pjax:beforeReplace，pjax:end