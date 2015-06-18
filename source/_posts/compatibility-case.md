title: 兼容性解决方案
date: 2015-04-13 15:40:22
tags: [IE8,兼容性,Solution]
---
 
开发中得到的一些IE/FF/Safari下的兼容性处理方案，请叫我兼容性处理大师，谢谢~

1 图片链接边框
       

    <a href="signup.html">
        <img class="banner" src="src/banner.jpg">
    </a>
常用的图片链接，在IE8下，图片会被加上蓝色边框，什么鬼...这当然不是我们想要的结果了，加个

    img {
        border: none;
    }

干掉IE8给加的边框。All is well~
<!-- more -->
2 未设置背景的div真的只是四条边线
什么意思呢？就是IE8如果你一个普通的div，设置width,height后，恩，你想这个div应该有一个cursor:pointer，然后再绑定个click事件什么的....然后你会发现？wtf！鼠标移上去完全没作用啊！点击也完全没作用啊！诶，不对，似乎鼠标在经过边框的时候有反应...好吧，你已经发现真理了。来，解决这个问题，填充一下就可以了嘛：


    div {
        height: 100px;
        width: 100px;
        cursor: pointer;
        background-color: #fff;
        opacity: 0;
        filter: Alpha(opacity=0);
    }
Perfect!

3 设置高度的select中的文字没有垂直居中
那你就加line-height去控制啊，look down:


    select {
        height: 36px;
        line-height: 18px;
        padding: 9px 0;
    }

4 placeholderIE10以下处理方案
简单的东西当然可以自己写了， 但是网上有那么多， 没必要啦...最基本的实现方式都是给输入框一个值然后在keyup的时候去清除， 但是这种情况， 因为是直接把内容填进去的， 对于有验证的输入框来说就有点痛苦了而且密码框要么就是placeholder=“密码”会被转成两个`..`， 要么就是输入密码直接给显示出来， 太恶心太麻烦。
我还是比较喜欢[这个插件](https://github.com/amerikan/placeholder-polyfill)的做法， 在input的外面包裹一个wrapper， 然后加一个input同级的label来显示placeholder， placeholder其实就是label， 然后控制label的显示与隐藏就可以了。但是label的样式可能需要手动去改一下， 要么直接在插件里面扩展， 要么传入参数扩展。 

另外， 当然也不是一帆风顺了， 当遇到`自动填入`（浏览器记住密码） 的时候就会发现IE10往下是全部不会让"placeholder"消失的， 估计写插件的人也没考虑这个， 于是乎就给他补了一个chang的事件监听....啊， IE10好了， IE9也好了....擦， IE8什么鬼， 不会触发change....苦苦寻找， 好吧有`propertychange
` 可以用也一样...IE真坑爹

5 Mac safari输入框placeholder文字底部被截取
开始的输入框一直是:

    height: 43px;
    line-height: 43px;
    
诶， 测试没有mac下了一个windows的safari， 怎么输入框placeholder全坏的啊...拿掉line-height好了...于是不用line-height， 用
    
    height: 18px;
    padding: 12px 0;
    
这样的替代方案...其他的好了， 到mac上的safari就有文字截取了， 最后查了一下原因还是因为没有line-height的原因， 那就加上吧， 然后来个兼容方案:

    height: 43px;
    font-size: 16px;
    line-height: 1.2;
    line-height: 43px\0/; /* for IE 8 */

6 windows safari的password输入框似乎不支持黑体这样的字体
对于设置了黑体的password-input， 你是看不到密码的黑点的， 输入也没有反应（其实已经有了） ， 坑爹货。 

Windows safari已经停止更新了， 不要再用这个来折磨前端了...

7 IE8是不支持rgba的颜色的
因此只能用相近色去替代了， 或者直接不要用rgba了， 毛用， 就是来找麻烦的...这样写可以兼容:

    color: #9a9a9a;
    color: rgba(0,0,0,.5);
上面是替换色, IE8下会执行, 其他浏览器使用下面的颜色, 按顺序写就行了。 

End with: 没什么高深的东西，慢慢积累就会变成财富。