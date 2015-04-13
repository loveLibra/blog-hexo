title: 一些IE8下的兼容性解决方案
date: 2015-04-13 15:40:22
tags: ie8 兼容性
---

纪念一下在YOHO的第一个上线的WEB(页面)[]，虽然很简单，但是也是一个新的开始~ 
开发中得到的一些IE8下的兼容性处理方案，请叫我兼容性处理大师，谢谢~

1 图片链接边框
       

    <a href="signup.html">
        <img class="banner" src="src/banner.jpg">
    </a>
常用的图片链接，在IE8下，图片会被加上蓝色边框，什么鬼...这当然不是我们想要的结果了，加个

    img {
        border: none;
    }

干掉IE8给加的边框。All is well~

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


没什么高深的东西，慢慢积累就会变成财富