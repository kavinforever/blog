---
title: 精通CSS高级web解决方案读书笔记一
date: 2017-02-01 12:10:40
comments: true
reward: true
tags:
    - css
    - 读书笔记
---
读书笔记

# 第一章 基础知识

## 一、css的出现
表现（css）与内容（html）的分离

不要使用表现性元素来命名，应该根据“它们是什么”来为元素命名，而不是根据“它们的外观如何”来命名。
坏的命名red，好的命名error


## 二、使用id还是类
类应用在概念上相似的元素，这些元素可以出现在同一个页面的多个位置
ID应该应用于不同的唯一的元素

只有在绝对确定这个元素只会出现一次的情况下才使用ID。如果以后可能需要相似的元素，就使用类。

不要试图在每一个东西上都添加类！！（**==多类症==**）
<!-- more -->
## 三、DOCTYPE
dtd(文档类型定义)是一组机器可读的规则，用这些规则来检查页面的有效性并采取相应的措施。
DOCTYPE声明是指html文档开头处的一行两行代码，描述使用哪个dtd，但是html5就不需要特定dtd文件的url
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="lib/d3.v3.js"></script>
    <link rel="stylesheet" href="css/jquery.jqtimeline.css">
</head>
<body>
</body>
</html>
```

# 第二章 为样式找到应用目标--各种选择器
## 一、常用选择器
1. 类型选择器 div p h1
2. 后代选择器：由两个选择器之间的空格表示 .news p
3. ID选择器 #container
4. 类选择器 .news
分析元素的差异性，如果元素的唯一差异只是在页面上的位置，那么不要给这些元素指定不同的类，而是将一个类应用于它们的祖先，然后后代选择器定位它们。
5. 伪类 :hover
6. 通用选择器 *
7. 高级选择器
- 子选择器
    后代选择器选择一个元素的所有的后代，而子选择器只选择元素的直接后代，即子元素。
    #nav>li
- 相邻同胞选择器
    顶级标题后面一段粗体标识
    ```css
    h2+p{
        font-weight:bold;
    }
    ```
- 属性选择器
    对具有title属性的元素应用特殊的样式
    ![image](http://imglf0.nosdn.127.net/img/dk1xY2tOVlo0b2I4dnpnN2dZVG5kSUJiSGFqeVUxUDU5RVFybjVKZmhINi8zMUZSdEEwSHBRPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0&type=jpg)

## 二、层叠重要度和特殊性
重要度：
- 标有！important的用户样式
- 标有！important的开发者样式
- 开发者样式
- 用户样式
- 浏览器、用户代理应用的样式

特殊性：
- 行内样式style             1000
- ID选择器                  100
- 类、伪类、属性选择器      10
- 类型选择器                1


## 三、继承
不要把继承和层叠混为一谈！
继承的定义是应用样式的元素的后代会继承样式的某些属性，例如颜色和字号。
应用在元素上的任务样式会覆盖继承而来的样式。