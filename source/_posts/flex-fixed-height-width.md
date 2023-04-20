---
title: 'Flex垂直主轴布局，固定高度 | 如何实现自适应宽度？'
date: '2021-06-10'
categories: '技术分享'
keywords: 'flex-fixed-height-width'
description: '学习如何使用Flex实现固定高度的垂直主轴布局，同时使宽度自适应。'
cover: '/images/css.webp'
tags: ['css']
---

由于直接使用`flex-direction: column;`会导致高度坍塌

所以这边使用`writing-mode: vertical-lr;`来实现垂直布局, 由于`writing-mode`的垂直布局会导致子元素内容改变行进方向，所以这边使用`writing-mode: horizontal-tb;`来实现将子元素摆正

以下是 demo：

```css
<style>
  .items {
    display: inline-flex;
    writing-mode: vertical-lr;
    flex-wrap: wrap;
    align-content: flex-start;
    height: 350px;
    background: blue;
  }
  .item {
    writing-mode: horizontal-tb;
    width: 150px;
    height: 100px;
    background: red;
    margin: 2px;
  }
</style>
```
```html
<div class="items">
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
  <div class="item"></div>
</div>
```
