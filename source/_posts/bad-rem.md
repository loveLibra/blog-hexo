title: REM是个坑，精确控制不适合
date: 2015-06-18 14:56:05
tags: [CSS,Solution]
---
前两天需要做一个涂鸦的活动页面，来瞅瞅![年中大促](https://dn-xuqi.qbox.me/yoho-q1.png)
简要介绍下这个页面，就是一个ABC选择，选中某项后加红色区分然后右侧动画闪动。*[图片版权声明：版权归YOHO所有]*

拿到这个页面第一反应就是好*蛋啊，就是个图片页面了然后需要点击的地方就绝对定位一个div盖在上面。考虑是手机端的页面，那就可以使用REM神器了:
1. 加上脚本
    	(function (doc, win) {
            var docEl = doc.documentElement,
                resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
                recalc = function () {
                    var clientWidth = docEl.clientWidth;
                    if (!clientWidth) {
                        return;
                    }
                    docEl.style.fontSize = 20 * (clientWidth / 320) + 'px';
                };
    
            if (!doc.addEventListener) {
                return;
            }
            win.addEventListener(resizeEvt, recalc, false);
            doc.addEventListener('DOMContentLoaded', recalc, false);
        })(document, window);
2. 使用方法
css中原来用px的换成rem就可以了,40px = 1rem
3. 脚本根据屏幕尺寸会在html元素上生成个font-size的样式，然后页面渲染时都会根据这个值和rem的数值等比例生成对应屏幕的尺寸或者位置。

大体功能都能满足，但是在进行选项图标200ms每帧切换的时候却出现了问题，会抖动，那找原因就是通过雪碧图切换background-position位置不准咯，而且在各个手机上不同的选项各有抖动与不抖动的情况。

就是使用rem的原因，不同的屏幕生成的根font-size不一样然后计算后就会出现定位不准的问题了，最后还是换成了直接切换background-image了，雪碧图用不了。（ps:如果用rem可以实现请告诉我...）


另外这个页面还有另外一个情况，看到会有选中项的颜色区分。最开始做的时候是把整个图片除掉选项右边的小图标(图标会有动画实现因此时单独的)作为底图然后选中后就在对应项上面加一个有颜色的div。事实肯定不如所意了，文字和ABC会被盖住....最后的解决办法是：灰色的底图单独是一层作为最底层，然后页面图片去底图在最上面，选中时的颜色div放在两层中间就可以了...但是这样对上层图切的要求就高了，要不然你会发现边边角角全是白色的锯齿。