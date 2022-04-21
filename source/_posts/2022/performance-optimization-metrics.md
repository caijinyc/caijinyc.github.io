---
title: Web 性能指标定义和获取方法
subtitle: "Web performance optimization metrics and how to get them"
date: 2022-04-20
tags:
- 性能优化
---

如果要做前端页面的性能优化，那么肯定需要一些衡量指标来衡量优化收益。目前业界常用的衡量指标有：FP、FCP、LCP、TTFB 等等，本文简单解释一下这些指标以及如何获取方法。

<!--more-->

> 4.21 未完待续...由于本人性能优化实践还在进行中，后续会持续补充内容 & 个人理解。

>  本文仅仅是我个人的学习和整理，大多数内容来自于 `https://web.dev`，有部分总结如有错误请大家及时指出；
>  本文不会提及如何对各指标进行优化，优化实践方案后续会单独更一篇文章；

## First Paint (FP) 首次绘制

### 指标定义

FP 指页面导航与浏览器将该网页的**第一个像素渲染到屏幕上所用的中间时**，渲染是任何与输入网页导航前的屏幕上的内容不同的内容。

在性能统计指标中，从用户开始访问 Web 页面的时间点到 FP 的时间点这段时间可以被视为**白屏时间**，也就是说在用户访问 Web 网页的过程中，FP 时间点之前，用户看到的都是没有任何内容的白色屏幕，用户在这个阶段感知不到任何有效的工作在进行。

如果我们的白屏时间过长，那么用户的流失率就会升高。

我们可以通过 Performance 面板查看到页面的 FP：

<img src="/asset/img/2022/performance-optimizing-metrics/1.png" width="400">

### 获取方法

在 MDN 上可以查到 [PerformancePaintTiming API](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformancePaintTiming) 用来获取性能数据，我们可以通过 `performance.getEntriesByType('paint')` 的方法来获取到应用的 FP 值。

![](/asset/img/2022/performance-optimizing-metrics/2.png)

> 注意：这个指标是有兼容性问题的，如果用户的游览器不支持此 API，则在埋点上报处给这部分的用户做好标识即可。

## First Contentful Paint (FCP) 首次内容绘制

### 指标定义

首次内容绘制 (FCP) 指标测量页面从开始加载到页面内容的任何部分在屏幕上完成渲染的时间。对于该指标，["内容" 指的是文本、图像（包括背景图像）、svg 元素或非白色的 canvas 元素](https://web.dev/lcp/#yuan)。 相比 FP，我认为 FCP 对网站的重要性更为重要，因为 FP 时用户还是无法看到有用的信息，只有完成 FCP 时，用户才能看到网页上的部分信息。

![](/asset/img/2022/performance-optimizing-metrics/3.png)

[W3C 团队指出](https://web.dev/fcp/#fcp-2)：**为了提供良好的用户体验，网站应该努力将首次内容绘制控制在 1.8 秒或以内。**如果网站的 FCP > 3 秒，那么体验就非常糟糕了。

<img src="/asset/img/2022/performance-optimizing-metrics/img.png"  width="500">

### 获取方法

和 FP 相同，直接通过 [PerformancePaintTiming API](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformancePaintTiming) 就能获取到 FCP 的值。

![](/asset/img/2022/performance-optimizing-metrics/img_1.png)


## Largest Contentful Paint (LCP) 最大内容绘制

### 指标定义

LCP 是测量感知加载速度的一个以用户为中心的重要指标，因为该项指标会在页面的主要内容基本加载完成时，在页面加载时间轴中标记出相应的点，迅捷的 LCP 有助于让用户确信页面是有效的。

最大内容绘制 (LCP) 指标会根据页面首次开始加载的时间点来报告可视区域内可见的最大图像或文本块完成渲染的相对时间。

![](/asset/img/2022/performance-optimizing-metrics/img_2.png)

如果你对细节感兴趣，想知道 W3C 团队是如何定义 LCP 的，可以查看这里：[何时报告最大内容绘制？](https://web.dev/lcp/#yuan-2)

W3C 团队同样也给出了推荐的 LCP 时间：

<img src="/asset/img/2022/performance-optimizing-metrics/img_3.png" width="500">

### 获取方法

可以直接参考这个：[getLCP](https://github.com/GoogleChrome/web-vitals/blob/main/src/getLCP.ts)，我这就不贴代码了。

> 注意：浏览器需要支持 [Largest Contentful Paint API](https://developer.mozilla.org/en-US/docs/Web/API/Largest_Contentful_Paint_API) 和 PerformanceObserver API。


