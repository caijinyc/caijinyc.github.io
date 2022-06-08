---
title: 复杂表单场景解决方案”原子化表单” Part 1 - 痛点
subtitle: Complex Form Scenario Solution "Atomic Form"
date: 2022-06-9
tags:
- Form
---

> 糊了两年多表单，顺便写点总结，先简单聊聊常见的表单库在复杂表单场景（例如发布商品）中的痛点。

<!--more-->

## 痛点

作为前端开发者，相信大家或多或少都使用过组件库自带的表单框架，例如 [Ant Design](https://ant-design.antgroup.com/components/form-cn/)
提供的 [rc-filed-form](https://github.com/react-component/field-form)（我们在使用 antd 时使用的其实是封装后的 Form，但底层是 rc-field-form）。

Antd Form 在开发一般的表单需求时，开发体验其实还不错。例如这种：

![Untitled](/asset/img/2022/complex-form-scenario-solution-atomic-form/part-1/img_1.png)

但是遇到复杂的表单场景时，例如这种：

![Untitled](/asset/img/2022/complex-form-scenario-solution-atomic-form/part-1/img_2.png)

这张图是电商场景下发布商品巨型表单页面中的的一小部分（国内的电商页面是这样的，表单又多又杂，可以理解…），如图所示，我们可以看到这个表单非常复杂，首先它的数据结构很复杂，还有各种深层嵌套、表单联动、动态表单（商品规格）、自增对象（SKU）等等。

如果这个时候再继续使用 Antd Form 开发的话，开发体验就比较差了，其次是维护成本也会很高。

如果你有开发过后台复杂表单项目且用的是 Antd Form 的话，相信你应该能理解笔者讲述的痛点。

对于这种复杂表单，想降低复杂度的话，思路就是进行拆分。只要能够将表单拆解的足够细，那么复杂度就可以直线降低。

## 尝试拆分

拆表单？听起来这个思路不错，那我们尝试用 Antd Form 拆分一下试试。

![Untitled](/asset/img/2022/complex-form-scenario-solution-atomic-form/part-1/img_3.png)

如图所示，来把这个表格中的价格输入框拆出来，移除无关业务逻辑，大概长这样：

```tsx
const FieldFormDemo = ({skuId}: { skuId: string }) => {
  const value = form.getFieldValue(["skuInfoObject", skuId, "price"]);
  // ... more action
  return (
    <Field name={ ["skuInfo0bject", skuld, "price"] }>
      <Input/>
    </Field>
  )
}
```

看起来好像没什么问题？不对不对，怎么感觉越看味道越不对呢。

不是说拆分之后，复杂度就可以直接降低么？为什么我只是修改个价格的 Input 里面的 form 会需要关心 `skuInfo, skuId` 这些外层的表单？如果没有表单联动的话，我不应该只关心 `price` 吗。

但是好像不太行，如果全局使用一个 Form 实例的话，那么内部的 Form 对于表单的处理，都需要依赖 `["skuInfoObject", skuId, "price"]` 这个 Field 数据。

那再优化下代码：

```tsx
const FieldFormDemo = (fieldArray: string[]) => {
  const value = form.getFieldValue(fieldArray);
  // ... more action
  return (
    <Field name={ fieldArray }>
      <Input/>
    </Field>
  )
}

<FieldFormDemo fieldArray={ ["skuInfoObject", skuId, "price"] }/>
```

好像行了，但感觉又没那么优雅，每个表单的拆分难道都要这么写吗？

**其实最大的问题，就是看起来是做了拆分，但是拆分后的表单其实和外面还是藕断丝连，非常依赖传入的 FieldArray，也就是没有完全做到解耦。**

拆表单说起来简单，但是除了 Field 的问题，表单的拆分需要考虑的问题非常多，例如整个发布商品**页面的全量表单校验、拆分的子表单校验、表单之间的联动、数据的监听**等等，都是我们需要考虑的问题。

这些问题在我们做表单拆分时，如果使用 Antd Form，那么每个问题都需要去做复杂的处理。

## 原子化表单

那么是不是可以有一个表单解决方案，可以帮我们解决这种复杂场景的问题？其实就是标题提到的 “原子化表单”。

“原子化表单”指的是天然支持原子化的表单拆解。

我们可以把包含数百个表单的巨型表单页面拆分成一个个独立的原子表单，他们内部可以**单独进行校验、订阅表单数据、更新等等，表单实例可以做的事情，它都可以做**。同时这些原子化的表单还可以自由组合成一个大型表单。

## 总结

本文简述了一般的表单库在面对巨型表单场景的无力以及由此引出的原子化表单解决方案。但因为篇幅的关系，本文暂时不对原子化表单进行深入讲解，后面会补充。
