---
title: viewport
categories: HTML
tags: html
---

## viewport的概念

### layout viewport
通俗的讲，移动设备上的viewport就是设备的屏幕上能用来显示我们的网页的那一块区域，在具体一点，就是浏览器上(也可能是一个app中的webview)用来显示网页的那部分区域，但viewport又不局限于浏览器可视区域的大小，它可能比浏览器的可视区域要大，也可能比浏览器的可视区域要小。在默认情况下，一般来讲，移动设备上的viewport都是要大于浏览器可视区域的，这是因为考虑到移动设备的分辨率相对于桌面电脑来说都比较小，所以为了能在移动设备上正常显示那些传统的为桌面浏览器设计的网站，移动设备上的浏览器都会把自己默认的viewport设为980px或1024px（也可能是其它值，这个是由设备自己决定的），但带来的后果就是浏览器会出现横向滚动条，因为浏览器可视区域的宽度是比这个默认的viewport的宽度要小的。下图是一些设备上浏览器的默认viewport的宽度。{% asset_img icon1.png %}这个layout viewport的宽度可以通过 document.documentElement.clientWidth 来获取。
 <!--more-->
### visual viewport
它是在不同的缩放（initial-scale）情况下浏览器可视区域的大小（需要多少css像素填充），visual viewport的宽度可以通过window.innerWidth 来获取，但在Android 2, Oprea mini 和 UC 8中无法正确获取。

### ideal viewport
这个viewport可以理解为最适合当前机型的viewport。不同的手机的可能拥有不同的ideal viewport,一般iphone手机都是320px，但是安卓设备就比较复杂了，有320px的，有360px的，有384px的等等，关于不同的设备ideal viewport的宽度都为多少，可以到[http://viewportsizes.com](http://viewportsizes.com)去查看一下，里面收集了众多设备的理想宽度。

## 像素的理解

### 物理像素(physical pixel)
一个物理像素是显示器(手机屏幕)上最小的物理显示单元，在操作系统的调度下，每一个设备像素都有自己的颜色值和亮度值。

### 设备独立像素(density-independent pixel)
设备独立像素(也叫密度无关像素)，可以认为是计算机坐标系统中得一个点，这个点代表一个可以由程序使用的虚拟像素(比如: css像素)，然后由相关系统转换为物理像素。

### 设备像素比(device pixel ratio )
定义了物理像素和设备独立像素的对应关系，它的值可以按如下的公式的得到：

设备像素比 = 物理像素 / 设备独立像素 // 在某一方向上，x方向或者y方向

在js中，dpr = window.devicePixelRatio

在css中，可以通过-webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio和 -webkit-max-device-pixel-ratio进行媒体查询，对不同dpr的设备，做一些样式适配(这里只针对webkit内核的浏览器和webview)。

综合上面几个概念，一起举例说明下：

以iphone6为例：设备宽高为375×667，可以理解为设备独立像素(或css像素)。
dpr为2，根据上面的计算公式，其物理像素就应该×2，为750×1334。{% asset_img icon2.gif %}在css像素大小不变的情况下，普通屏幕下，1个css像素 对应 1个物理像素(1:1)，retina屏幕下，1个css像素对应 4个物理像素(1:4)。

## meta.viewport详解
### 几个概念
一般在做移动端页面时，会把<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">这行代码复制到页面中。该meta标签的作用是让当前viewport的宽度等于设备的宽度，同时不允许用户对其进行缩放。一般，width设置layout viewport 的宽度，为一个正整数，或字符串"width-device"；initial-scale 设置页面的初始缩放值，为一个数字，可以带小数；minimum-scale允许用户的最小缩放值，为一个数字，可以带小数；maximum-scale    允许用户的最大缩放值，为一个数字，可以带小数；user-scalable    是否允许用户进行缩放，值为"no"或"yes", no 代表不允许，yes代表允许。当然还可以设置其他的值，用逗号隔开就可以。
### width和initial-scale在移动端页面中的作用
（1）通过width=device-width，所有浏览器都能把当前的viewport宽度变成ideal viewport的宽度，但要注意的是，在iphone和ipad上，无论是竖屏还是横屏，宽度都是竖屏时ideal viewport的宽度。
（2）initial-scale的缩放是相对于ideal viewport来进行缩放的，当对ideal viewport进行100%的缩放，也就是缩放值为1的时候，得到的就是ideal viewport
（3）ideal viewport的值会取width和initial-scale两个中较大的那个值

### 关于initial-scale的缩放
initial-scale是对页面就行缩放（个人理解就是缩放单位css像素的大小），测试如下，在meta中去掉width这一项，visual viewport得出如下数据：{% asset_img icon3.png This is an example image %}从上图可以看出，在屏幕宽度不变的情况下，当initial-scale变小，预示着css像素变小，导致填充整个屏幕需要的css像素就会变多。因此，我们可以得出一个公式：visual viewport宽度 = ideal viewport宽度 / 当前缩放值

### 关于width
去掉initial-scale，测试width。当width=160时，{% asset_img icon4.png This is an example image %}这是因为有些元素把宽高都写死了，此时initial-scale=2，元素都变为原来的2倍，所以导致页面很挤。当width=1280时，{% asset_img icon5.png This is an example image %}这是因为有些元素把宽高都写死了，此时initial-scale=2，元素都变为原来的0.25倍，所以导致页面的元素很小。

### 结论
在iphone和ipad上，无论你给viewport设的宽的是多少，如果没有指定默认的缩放值，则iphone和ipad会自动计算这个缩放值，以达到当前页面不会出现横向滚动条(或者说viewport的宽度就是屏幕的宽度)的目的。安卓设备上的initial-scale默认值好像没有方法能够得到，或者就是干脆它就没有默认值，一定要你显示的写出来这个东西才会起作用！

## 利用meta.viewport对多屏适配布局

### retina下，border: 1px问题
经常border：1px在不同手机上的宽度不一样，如下：{% asset_img icon6.jpg This is an example image %}上图中，对于一条1px宽的直线，它们在屏幕上的物理尺寸(灰色区域)的确是相同的，不同的其实是屏幕上最小的物理显示单元，即物理像素，所以对于一条直线，iphone5它能显示的最小宽度其实是图中的红线圈出来的灰色区域，用css来表示，理论上说是0.5px。所以，设计师想要的retina下border:1px;，其实就是1物理像素宽，对于css而言，可以认为是border: 0.5px;，这是retina下(dpr=2)下能显示的最小单位。

### 多屏适配布局
移动端布局，为了适配各种大屏手机，目前最好用的方案莫过于使用相对单位rem。
rem = document.documentElement.clientWidth * dpr / 10 javascript方式，通过上面的公式，计算出基准值rem，然后写入样式。

以下是淘宝的代码：

```js
(function (doc, win) {
  var docEl = doc.documentElement,
    resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
    recalc = function () {
      var clientWidth = docEl.clientWidth;
      if (!clientWidth) return;
      docEl.style.fontSize = 20 * (clientWidth / 320) + 'px';
    };
  if (!doc.addEventListener) return;
  win.addEventListener(resizeEvt, recalc, false);
  doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);
```

可以看出，他们也仅仅按照手机屏幕的宽度在做适配，没有动用dpr和scale，个人认为这两个必须同时运用，页面才会在理想情况下展现！

一般方法大概如下：

```js       
        var dpr, rem, scale;
        var docEl = document.documentElement;
        var fontEl = document.createElement('style');
        var metaEl = document.querySelector('meta[name="viewport"]');
        dpr = win.devicePixelRatio || 1;
        scale = 1 / dpr;
        rem = docEl.clientWidth * dpr / 10;
        // 设置viewport，进行缩放，达到高清效果
        metaEl.setAttribute('content', 'width=' + dpr * docEl.clientWidth + ',initial-scale=' + scale + ',maximum-scale=' + scale + ', minimum-scale=' + scale + ',user-scalable=no');
        // 设置data-dpr属性，留作的css hack之用
        docEl.setAttribute('data-dpr', dpr);
        // 动态写入样式
        docEl.firstElementChild.appendChild(fontEl);
        fontEl.innerHTML = 'html{font-size:' + rem + 'px!important;}';
        // 给js调用的，某一dpr下rem和px之间的转换函数
        window.rem2px = function(v) {
            v = parseFloat(v);
            return v * rem;
        };
        window.px2rem: function(v) {
            v = parseFloat(v);
            return v / rem;
        };
        window.dpr = dpr;
        window.rem = rem;
```

以上设置是为了保证，1个css像素占据一个物理像素。如果有一个区块，在psd文件中量出：宽高750×300px的div，那么如何转换成rem单位呢？如果scale是1，对于iphone6来说就是取一半，如果scale是0.5，量出多少是多少，换句话说就是设计稿的宽度W=docEl.clientWidth * dpr，对于不符合的设计稿，计算公式如下：{% asset_img icon7.jpg This is an example image %}我们现在写页面参考的机型一般是iphone4以上，dpr一般都是2，所以按照以上写法其实只有viewport的宽度在起作用，但是如果遇到宽度和dpr都不一样的时候，两者的倍数都会起作用，如果遇到iphone3这样的，宽度不变而dpr变化的，它上面的元素就会比iphone4要小，遇到这样的机型，如果不改变scale，样式有可能出问题！