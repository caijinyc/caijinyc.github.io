---
title: 字体加载的几种方案
subtitle: "Several methods for loading fonts"
date: 2022-06-8
tags:
- Font
---

> 众所周知，中文字体文件都是比较大的，且还有不同的字重，如果需要加载所有字重的字体文件，那体积就更加夸张，本文简述字体加载的几种方案，以及如何选择合适的方案。

<!--more-->

#### 碎碎念

这篇文章真的拖跟的有点久，其实一个多月前就想写了，但是自己拖延症的毛病实在是 ...（已经不想吐槽了，我真的好懒），好在最后还是写完了。

我知道类似的这种文章网上一搜一大把，但我还是倾向于把自己学习到的内容通过文章的方式记录下来，虽然自己好奇的东西有很多，但是如果只是简单了解的话，往往又觉得不够透彻。只有把自己的学到的记录下来，才能够帮助自己更深入的理解。

> 好奇 -> 了解 -> 学习 -> 记录 -> 分享-> 成长

## 正文

我们可以看到，一个包含所有字重的思源黑体文件大小有 30MB，这个体积是非常夸张的。

<img src="/asset/img/2022/several-methods-for-loading-fonts/img.png" width="300px" alt="img">

如果我们需要对所有字体文件做加载，那么加载时间就会过长，很影响首屏时间。所以我们可以根据业务场景来使用不同的字体加载方案。

## 全量加载字体
虽然全量加载字体的操作对首屏的性能影响比较大，但是部分场景还是很常用的，例如图文编辑器(稿定设计、Canva)。所以这部分也简单记录一下。

如果是首屏的字体加载，直接写在 CSS 里面就可以了，例如这样：
``` css
      @font-face {
        font-family: FONT_FAMILY_NAME;
        font-display: swap;
        src: url("字体 CDN 地址");
      }
      
      body {
        font-family: FONT_FAMILY_NAME;
      }
      
```
如果非首屏，且是通过用户交互触发的字体加载，例如：

<img src="/asset/img/2022/several-methods-for-loading-fonts/img_1.png" width="500px" alt="img_1">

这种场景则可以使用 [Font Face Observer](https://github.com/bramstein/fontfaceobserver) 来帮助我们监听字体加载的状态态，让我们可以方便地获取字体加载成功、失败等状态，来做其他处理。

``` TS
      var font = new FontFaceObserver('My Family', {
        weight: 400
      });
      
      font.load().then(function () {
        console.log('Font is available');
      }, function () {
        console.log('Font is not available');
      });
```

## 部分加载字体

全量加载的体验不是很好，所以我们需要提供一个方案来加载部分字体，下面列举部分加载的几种方法。

### 使用图片

最简单最暴力的方案就是直接把设计师给的图片丢到页面上去，又快又方便。当然缺点也很明显：

1. 不能再对文案进行修改
2. 不能对文本设置样式
3. 如果使用的是矢量图就会导致图片放大后失真(例如 1x 图在高分屏上就模糊了)，当然你可以使用 svg 解决这个问题，如果用 figma 出图的话可以直接选择导出 svg 图片

如果这的时间紧迫，用这个方案也不是不可以，例如要做个简单的官网项目，时间又非常紧迫，直接在 Figma / Sketch 上导出 svg 图片就可以开心的糊页面了，又快又简单。

### 字体裁切
使用 [Fontmin](http://ecomfe.github.io/fontmin/#usage) 对使用到的文案进行选取，这样打包出来的字体文件就只会包含你选中的文案。

当你的文案相对固定时，就可以使用这个方案。例如之前我做的需求，设计只想用某个字体中的数字，这时我就可以直接把字体中的数字裁切出来，只需要加载几 kb 的字体。

<img src="/asset/img/2022/several-methods-for-loading-fonts/img_2.png" width="600px">

<img src="/asset/img/2022/several-methods-for-loading-fonts/img_3.png" width="400px">


### 动态加载字体 ⭐️

终极方案，按需加载字体，根据页面中的使用到的字体动态的加载字体文件。优点是**不再限制固定文案，纵使页面的文案不断变更，也不影响字体展示**。

<img src="/asset/img/2022/several-methods-for-loading-fonts/font-load_1654579848120_0.gif" width="400px" >

我们可以使用 [Google Fonts](https://fonts.google.com/noto/specimen/Noto+Serif+SC?subset=chinese-simplified) 提供的字体动态加载。 

<img src="/asset/img/2022/several-methods-for-loading-fonts/img_5.png" width="400px" >

如图所示，当我们需要使用动态加载服务时，直接复制示例代码到项目中就可以了，真的非常简单。效果如下：

<img src="/asset/img/2022/several-methods-for-loading-fonts/google-fonts.gif" width="700px" >

简单是简单，但是因为墙的问题，所以 Google Fonts 提供的字体在国内是无法使用的 😅。

这时我们可以也可以尝试自己来部署这套加载服务，先看看动态加载的原理吧。

#### 原理简述

分析一下动态原理，涉及的主要知识点就是 [unicode-range](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/unicode-range)，它的用途可以看一下这篇文章 [CSS unicode-range特定字符使用font-face自定义字体](https://www.zhangxinxu.com/wordpress/2016/11/css-unicode-range-character-font-face/)。

简单来说 `unicode-range` 设置了 `@font-face` 定义的字体中要使用的特定字符范围，如果页面在此范围内未使用任何字符，则不会下载字体；如果使用至少一种，则将下载整个字体。

让我们打开 Google Fonts 提供的 [字体使用地址](https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@100&display=swap) 看看里面都是什么：

<img src="/asset/img/2022/several-methods-for-loading-fonts/img_6.png" alt="img">

里面定义了无数的 `unicode-range`，相信你已经能够看出来了：**Google Fonts 将一个字体拆分成了上百个字体文件**（截图只展示了部分，其实还有很多，点开链接就知道了）。

当我们页面动态输入字体时，`unicode-range` 会判断字体是否命中，当命中切片字体范围时，就会加载这部分字体文件。

**那么怎么把一整个大的字体文件拆成若干个小的字体文件呢？？**

我们可以通过 [fonteditor-core](https://github.com/kekee000/fonteditor-core) 这个库来处理字体文件，将其根据 `unicode-range` 切片，具体使用方案请参考文档，感兴趣的朋友可以深入研究。

当然，如果你需要自己部署字体动态加载的服务，还有很多因素需要考虑，例如缓存、切片力度等等等。但是因为我自己也没搞过，只是去了解和分析动态加载的原理，所以我就不在这瞎吹了，有需要实现的朋友可以自己去研究 🤣。

如果我有时间实现一个的话，我会再写文章详细讲一讲的~

## 总结

本文简述了字体加载的几种方案以及部分原理，笔者也在持续学习中，如有错误，欢迎指正。

## 参考资料

[CSS unicode-range特定字符使用font-face自定义字体](https://www.zhangxinxu.com/wordpress/2016/11/css-unicode-range-character-font-face/)
