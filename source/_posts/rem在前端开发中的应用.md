---
title: rem在前端开发中的应用
date: 2016-08-01 14:39:38
comments: true
reward: true
tags:
    - rem
    - css
    - css3
---
	
前段时间在练习地铁H5切图的过程中，尝试使用rem来进行css单位的设置，感觉rem是最适合web app的单位，没有之一。

目录：

	1. CSS中的单位介绍
	2. rem在移动端开发中的使用
	3. 总结
	4. 参考文档


# 一、CSS中的单位介绍 #
## 1.1 像素的概念 ##
### 设备像素(device pixel): ###
设备像素是物理概念，指的是设备中使用的物理像素。
比如iPhone 5的分辨率640 x 1136px。

### 设备独立像素(device-independent pixels (dips)) ###
(也叫密度无关像素)，可以认为是计算机坐标系统中得一个点，这个点代表一个可以由程序使用并控制的虚拟像素(比如：CSS 像素,只是在android机中CSS像素就不叫”CSS 像素”了而是叫”设备独立像素”)，然后由相关系统转换为物理像素。

在PC端可以通过 screen.width/height 属性来获取设备独立像素值，在PC端这个值把它当成我们常说的屏幕分辨率（实际上它不是，但是由于在PC端设备像素和设备独立像素数值相等，才有这么一个不准确的说法）。

### CSS像素(css pixel): ###
CSS像素是Web编程的概念，指的是CSS样式代码中使用的逻辑像素。
在CSS规范中，长度单位可以分为两类，绝对(absolute)单位以及相对(relative)单位。px是一个相对单位，相对的是设备像素(device pixel)。

比如iPhone 5使用的是Retina视网膜屏幕，使用2px x 2px的 device pixel 代表 1px x 1px 的 css pixel，所以设备像素数为640 x 1136px，而CSS逻辑像素数为320 x 568px。
![](http://i.imgur.com/7SWgfGs.png)
<!-- more -->
### 每英寸像素PPI&& 设备像素比DPI###
每英寸像素(pixel per inch,PPI):
表示每英寸所拥有的像素(pixel)数目，数值越高，代表显示屏能够以越高的密度显示图像。
要计算显示器的每英寸像素值，首先要确定**屏幕的尺寸**和**分辨率**。
![](http://i.imgur.com/cIk9QOM.png)

以尺寸为4.7英寸、分辨率为1334*750的iphone6为例
![](http://i.imgur.com/vaMQKyy.png)

设备像素比(device pixel ratio,DPI)：
window.devicePixelRatio是设备上物理像素和设备独立像素(device-independent pixels (dips))的比例。
	公式表示就是：window.devicePixelRatio = 物理像素 / dips
以上计算出ppi是为了得到密度分界，获得默认缩放比例，即**设备像素比**。
可以通过JavaScript 中的***window.devicePixelRatio***来获取设备中的像素比值。

ppi在120-160之间的手机被归为低密度手机，160-240被归为中密度，240-320被归为高密度，320以上被归为超高密度（Apple给了它一个高大上的名字——Retina）。

获得设备像素比后，便可得知设备像素与CSS像素之间的比例。当这个比率为1:1时，使用1个设备像素显示1个CSS像素。当这个比率为2:1时，使用4个设备像素显示1个CSS像素，当这个比率为3:1时，使用9（3*3）个设备像素显示1个CSS像素。

下面给出iphone各个信息的设备像素、css像素、设备像素比等参数
![](http://i.imgur.com/vbDgWdr.png)
具体可以参考[http://screensiz.es/phone](http://screensiz.es/phone "screensiz")，有各种设备的详细参数。

### 分辨率 ###
分辨率：泛指量测或显示系统对细节的分辨能力。以PC屏幕，手机屏幕为例，分辨率1920*1080 是指屏幕纵向能显示1920个像素，横向能显示1080个像素。这里的像素指的是设备像素。

描述分辨率的单位有：dpi（点每英寸）、lpi（线每英寸）和ppi（像素 每英寸）。从技术角度说，“像素”只存在于电脑、手机显示领域，而“点”只出现于打印或印刷领域。对于web开发者，我们需要清楚ppi（ pixel per inch）

## 1.2 px介绍 ##
px是国内目前使用最多的css单位。 
在chrome浏览器下默认加载的字体大小是16px
最小字体是12px，如果想改变chrome浏览器下的最小字体，可以使用

```css
font-size:12px;
transform: scale(0.5);
```
当用户和Ctrl滚页面的时候（ctrl+，ctrl-），你会发现页面结构产生了不可预知的错乱，因此有人倡导使用em替代px

## 1.3 em介绍 ##
em是指相对于父元素的大小.简单的讲px是绝对单位，1px就是1px，2px就是2px，以此类推，而em是相对单位，em相对的基准点就是浏览器的字体大小，浏览器默认字体大小是16px，也就是1em默认等于16px，如果你想给某个文字设定为14px，就这样写 font-size:0.875em.

你可以在根节点<html>上重定义基准字号 **html {font-size:62.5%}** ，此时页面基准字号就是 16px * 62.5% = 10px , 那么此时 1em = 10px，那么此时14px = 1.4em，15px=1.5em，依次类推。

em准确的说是相对于父节点的字号来计算的，在整个页面内1em并不是一个固定不变的值，1em不停的变换。所以这个地方比较蛋疼不建议采用。

## 1.4 rem的介绍 ##
rem（font size of the root element）是指相对于根元素的字体大小的单位。简单的说它就是一个相对单位。看到rem大家一定会想起em单位，em（font size of the element）是指相对于父元素的字体大小的单位。它们之间其实很相似，只不过一个计算的规则是依赖根元素一个是依赖父元素计算。

# 二、rem在移动端开发中的使用 #
## 2.1 什么情况下使用rem ##
对于只需要适配少部分手机设备，且分辨率对页面影响不大的，使用px即可

对于需要适配各种移动设备，使用rem，例如只需要适配iPhone和iPad等分辨率差别比较挺大的设备

在实际项目中，把与元素尺寸有关的css，如width,height,line-height,margin,padding等都以rem作为单位，这样页面在不同设备下就能保持一致的网页布局。

## 2.2 一种老式的自适应解决方案 ##
rem配置参考，适合视觉稿宽度为640px的：

```
<meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
```
```html
html{font-size:10px}/*默认配置*/
@media screen and (min-width:321px) and (max-width:375px){html{font-size:11px}}
@media screen and (min-width:376px) and (max-width:414px){html{font-size:12px}}
@media screen and (min-width:415px) and (max-width:639px){html{font-size:15px}}
@media screen and (min-width:640px) and (max-width:719px){html{font-size:20px}}
@media screen and (min-width:720px) and (max-width:749px){html{font-size:22.5px}}
@media screen and (min-width:750px) and (max-width:799px){html{font-size:23.5px}}
@media screen and (min-width:800px){html{font-size:25px}}
```
例如在我在笔记本屏幕下面
```
<!DOCTYPE html>
<html class="narrow-screen">
<head>
<meta charset="utf-8">
<meta content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no" name="viewport">
<meta content="yes" name="apple-mobile-web-app-capable">
<meta content="black" name="apple-mobile-web-app-status-bar-style">
<meta content="telephone=no" name="format-detection">
<meta content="email=no" name="format-detection">
<title>rem参考案例</title>
<style type="text/css">
*{padding: 0;margin: 0;}

html{font-size:10px}/*默认配置*/
@media screen and (min-width:321px) and (max-width:375px){html{font-size:11px}}
@media screen and (min-width:376px) and (max-width:414px){html{font-size:12px}}
@media screen and (min-width:415px) and (max-width:639px){html{font-size:15px}}
@media screen and (min-width:640px) and (max-width:719px){html{font-size:20px}}
@media screen and (min-width:720px) and (max-width:749px){html{font-size:22.5px}}
@media screen and (min-width:750px) and (max-width:799px){html{font-size:23.5px}}
@media screen and (min-width:800px){html{font-size:25px}}

/*
使用px前的代码
.btn{display:block;width: 250px;height:42px;line-height:42px;border-radius:5px;text-align:center;font-size:18px;background-color:#04BE02;color:#FFFFFF;margin: 50px;}
*/

.btn{display:block;width: 25rem;height:4.2rem;line-height:4.2rem;border-radius:0.5rem;text-align:center;font-size:1.8rem;background-color:#04BE02;color:#FFFFFF;margin: 5rem;}

</style>
</head>

<body>
<a href="javascript:;" class="btn">拖动缩放窗口看下我的变化</a>
</body>
</html>
```
a的长度为625px，设置的为25rem。换算规则就是先查询媒介，得到最小宽度大于800px，所以默认的html的字体大小是25px == 1rem。那么btn这个class的width属性为25*25 = 625px。

以上代码乍看没啥问题，响应式设计不就应该是这么干的吗？但是从工作量和复杂度方面来考虑，它有以下几个不足：
1. .item类在所有设备下的width都是3.4rem，但在不同分辨率下的实际像素是不一样的，所以在有些分辨率下，width的界面效果不一定合适，有可能太宽，有可能太窄，这时候就要对width进行调整，那么就需要针对.item写媒介查询的代码，为该分辨率重新设计一个rem值。然而，这里有7种媒介查询的情况，css又有很多跟尺寸相关的属性，哪个属性在哪个分辨率范围不合适都是不定的，最后会导致要写很多的媒介查询才能适配所有设备，而且在写的时候rem都得根据某个分辨率html的font-size去算，这个计算可不见得每次都那么容易，比如40px / 23.5px，这个rem值口算不出来吧！由此可见这其中的麻烦有多少。
2. 以上代码中给出的7个范围下的font-size不一定是合适的，这7个范围也不一定合适，实际有可能不需要这么多，所以找出这些个范围，以及每个范围最合适的font-size也很麻烦
3. 设计稿都是以分辨率来标明尺寸的，前端在根据设计稿里各个元素的像素尺寸转换为rem时，该以哪个font-size为准呢？这需要去写才能知道。



## 2.3 网易新闻的自适应实现 ##
先来看看网易在不同分辨率下，呈现的效果：
![](http://i.imgur.com/F7oglW1.png)
![](http://i.imgur.com/V65nW5F.png)

从上面几张图可以看出，随着分辨率的增大，页面的效果会发生明显变化，主要体现在各个元素的宽高与间距。能够达到这种效果的根本原因就是因为网易页面里除了font-size之外的其它css尺寸都使用了rem作为单位。

可是在上部分提到，使用rem布局结合在html上根据不同分辨率设置不同font-size有很多不好解决的麻烦，网易是如何解决的呢？最根本的原因在于，网易页面上html的font-size不是预先通过媒介查询在css里定义好的，而是通过js计算出来的，所以当分辨率发生变化时，html的font-size就会变，不过这得在你调整分辨率后，刷新页面才能看得到效果。你看代码就知道为啥font-size是直接写到html的style上面的了（js设置的原因）：
![](http://i.imgur.com/jaVRfJa.png)

它是根据什么计算的，这就跟设计稿有关了，拿网易来说，它的设计稿应该是基于iphone6，所以它的设计稿竖直放时的横向分辨率为750px，为了计算方便，取一个100px的font-size为参照，那么body元素的宽度就可以设置为width: 7.5rem，于是html的font-size=deviceWidth / 7.5。这个deviceWidth就是viewport设置中的那个deviceWidth。根据这个计算规则，可得出html的font-size大小如下：
```
deviceWidth = 320，font-size = 320 / 7.5 = 42.6667px
deviceWidth = 375，font-size = 375 / 7.5 = 50px
deviceWidth = 414，font-size = 414 / 7.5 = 55.2px
```
这个deviceWidth通过document.documentElement.clientWidth就能取到了，所以当页面的dom ready后，做的第一件事情就是：
```javascript
document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.5 + 'px';
```
这个7.5怎么来的，当然是根据设计稿的横向分辨率/100得来的。下面总结下网易的这种做法：
1. 先拿设计稿竖着的横向分辨率除以100得到body元素的宽度：
2. 布局时，设计图标注的尺寸除以100得到css中的尺寸，比如下图：
3. 播放器高度为210px，写样式的时候css应该这么写：height: 2.1rem。之所以取一个100作为参照，就是为了这里计算rem的方便！
4. 在dom ready以后，通过以下代码设置html的font-size:
5. font-size可能需要额外的媒介查询，并且font-size不能使用rem，如网易的设置：
```
@media screen and (max-width:321px){
    .m-navlist{font-size:15px}
}

@media screen and (min-width:321px) and (max-width:400px){
    .m-navlist{font-size:16px}
}

@media screen and (min-width:400px){
    .m-navlist{font-size:18px}
}
```
6. 视口要如下设置： 
```
<meta name="viewport" content="initial-scale=1,maximum-scale=1, minimum-scale=1">
```
7. 当deviceWidth大于设计稿的横向分辨率时，html的font-size始终等于横向分辨率/body元素宽：
```
var deviceWidth = document.documentElement.clientWidth;
if(deviceWidth > 750) deviceWidth = 750;
document.documentElement.style.fontSize = deviceWidth / 7.5 + 'px';
```
之所以这么干，是因为当deviceWidth大于750时，则物理分辨率大于1500（这就看设备的devicePixelRatio这个值了），应该去访问pc网站了。事实就是这样，你从手机访问网易，看到的是触屏版的页面，如果从pad访问，看到的就是电脑版的页面。如果你也想这么干，只要把总结中第三步的代码稍微改一下就行了：

# 三、总结 #
暂时还未总结完成，没有总结。

# 五、参考文档 #
1.http://www.zhihu.com/question/32011095
2.http://screensiz.es/phone
3.[http://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/](http://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/ "设备像素比devicePixelRatio简单介绍")
4.[http://www.cnblogs.com/PeunZhang/p/3407453.html](http://www.cnblogs.com/PeunZhang/p/3407453.html "移动web资源整理")
5.[http://www.codeceo.com/article/font-size-web-design.html](http://www.codeceo.com/article/font-size-web-design.html)