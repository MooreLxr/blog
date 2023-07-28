---
layout: post
title: 移动端适配方案
date: 2023-04-03 19:31:09
tags: 前端
---

# 移动端适配：

在做移动端开发时，为了使移动端的页面在不同的手机上以同样的大小来显示，我们可以将页面的宽度固定，然后获取设备的宽度，可以得到我们之前设定的宽度与设备宽度的比例，再使用HTML5新增的viewport来对页面进行缩放，并固定不允许用户再重新缩放。

在index.html增加该配置
```
<meta
  name="viewport"
  content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no,viewport-fit=cover"
>
```

## rem适配(淘宝和百度方案)
计算公式：
根字体 = （设备视口宽度 * 100） / 设计稿宽度
```
<script>
// 获取布局视口宽度
const dpWidth = document.documentElement.clientWidth
// 计算根字体大小
const rootFontSize = (dpWidth * 100) / 375
// 设置根字体大小
document.documentElement.style.fontSize = rootFontSize + 'px'
</script>
```
设置完根字体大小后，后续所有页面都以rem为单位，值为 设计稿的像素值/100，例如14px = 0.14rem，这样在不用设备上都是0.14rem,但不同设备的rem代表的像素值不同，从而实现适配

## vw适配
它的特点很明显，没有js代码，但是兼容性却不好
![App Screenshot](./mobile-adaption/caniuse.png)

vw:把布局视口分成100份，1vw = 1% 的布局视口 = 1% 的视觉视口
vh:就是1%的视口高度
用less计算
```
@basic: 375 / 100vw;
*{
  margin: 0;
  padding: 0;
}
.demo{
  width: (100/@basic);
  height: (100/@basic);
  background-color: black;
}
```