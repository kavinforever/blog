---
title: css实现三角形图形原理解析
date: 2016-10-14 05:06:19
comments: true
reward: true
tags:
    - css
---

在写select组件的时候需要用css实现向上和向下的箭头，这个地方网上有很多的实现，但是想要了解具体的实现原理，因此写了这篇博客。


# border属性的用法 #

- border 四条边框设置
- border-left 设置左边框，一般单独设置左边框样式使用
- border-right 设置右边框，一般单独设置右边框样式使用
- border-top 设置上边框，一般单独设置上边框样式使用
- border-bottom 设置下边框，一般单独设置下边框样式使用,有时可将下边框样式作为CSS下划线效果应用
<!-- more -->
# 实现 #
## 四个边框 ##

```html
  <div class='wrap'>我是一个居中的元素</div>
```

```css
.wrap{
  border-left:15px solid red;
  border-right:15px solid green;
  border-top:15px solid yellow;
  border-bottom:15px solid black;
  text-align:center;
  
}
```
实现效果如下：
![](file://)


```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <div class='wrap'>我是一个居中的元素</div>
  <br>
  <div class='wrap'></div>
  <div class="test-border-0"></div>
  <div class="test-border-1"></div>
  <br>
  <div class="test-border-2"></div>
  <br>
  <div class="test-border-3"></div>
  <br>
  <div class="test-border-4"></div>
  <br>
  <div class="test-border-5"></div>
  <br>
  <div class="test-border-6"></div>
  
</body>
</html>

```