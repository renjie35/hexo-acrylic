---
title: 'Grid布局 | 解决Flex布局难点？'
date: '2021-07-07'
categories: 'CSS'
keywords: 'layout-grid-flex'
description: '学习如何使用Grid解决Flex难完成的布局。'
cover: '/images/css.webp'
tags: ['css']
---


## 子元素宽度确定，表格形式排列，父宽度可变

```css
.items {
	display: grid;
	grid-template-columns: repeat(auto-fill, 200px);

}

.item {
	margin: 0 auto;
	width: 100%;
	height: 100px;
	background: red;
	border: 1px solid;
}
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
	<div class="item"></div>
	<div class="item"></div>
	<div class="item"></div>
</div>
```

## 每行n个，子元素自动计算宽度，间距可调

```css
.items {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 8px;
}

.item {
	margin: 0 auto;
	width: 100%;
	height: 100px;
	background: red;
	border: 1px solid;
}
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
	<div class="item"></div>
	<div class="item"></div>
	<div class="item"></div>
</div>
```

## 特殊万能公式

```css
.layout {
  display: grid;
  grid:
    "span12 span12 span12 span12 span12 span12 span12 span12 span12 span12 span12 span12" 1fr
    "span6 span6 span6 span6 span6 span6 . . . . . ." 1fr
    ". . . . . span3 span3 span3 . . . ." 1fr
    ". . . . span1 . . . . . . ." 1fr
    / 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  gap: 8px;
}

.span12 { grid-area: span12; }
.span6 { grid-area: span6; }
.span3 { grid-area: span3; }
.span1 { grid-area: span1; }
```

```html
<section class="layout">
  <div class="span12">1</div>
  <div class="span6">2</div>
  <div class="span3">3</div>
  <div class="span1">4</div>
</section>
```