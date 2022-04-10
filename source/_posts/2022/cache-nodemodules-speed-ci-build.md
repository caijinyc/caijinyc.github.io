---
title: 依赖缓存加速 CI 构建
subtitle: "Cache node_modules speed CI build"
date: 2022-04-11
tags:
- 构建
---

> 通过手动缓存 node_modules 目录来加速 CI 构建，真滴快（非 yarn offline 或 pnpm）！

<!--more-->


当我们使用 CI 构建前端项目时，对于一些 NPM 依赖项非常多的项目，依赖安装时间就会特别长。如果我们使用了 monorepo 的模式，整个团队的项目都使用同一个仓库，那么依赖数量就会指数级增长。这个时候进行依赖安装时，通常会需要 10+ 分钟以上的时间，极端情况下可能会达到 30+ 分钟。



这么长的编译时间，则会带来非常多的问题，例如：

1. 想部署一下看看效果？先等编译吧您；
2. 测试让你修 bug，你说改好了，但是要等几十分钟编译；
3. 需求要上线了，但是离封板时间剩下几十分钟，结果代码合并到 master 之后，光编译就花了几十分钟，还上锤子线；
4. ...

## 使用 yarn 缓存

我们可以使用 yarn / npm 的离线模式来安装依赖，这样就可以减少网络请求。但是使用这个缓存是没法减少依赖分析的时间的，所以使用缓存后的安装时间还是比较长。

## 使用 pnpm

pnpm 在机制上就比 yarn 和 npm 快很多，可以看一下相关测试：[https://pnpm.io/benchmarks](https://pnpm.io/benchmarks)

<img src="/asset/img/2022/cache-nodemodules-speed-ci-build/time.png" alt="time">

虽然使用 pnpm 能减少构建时间，但是依赖项太多的时候，分析依赖还是需要不少时间，剩下的分析依赖的时间能不能也节省掉呢？答案是可以的。

## 手动缓存 node_modules

有没有一种方案可以实现 10s 内依赖安装完成？答案是有的，且我们在业务上已经稳定运行两年多了，之前一直想写文章记录一下，一直拖到现在。

### 解决方案

在相同条件时，直接使用之前安装的 node_modules，不需要再运行 yarn install，减少依赖安装和索引需要的时间。

### 实现原理

1. 在安装依赖时，先将所有的 package.json 文件进行 hash，获得唯一标识；
2. 在 cache 中寻找是否有对应的 hash cache，如果有，那么就直接使用此 hash cache，将此 cache 软链到我们的项目中就可以完成依赖安装
3. 如果发现没有，那么就算首次安装依赖，这次安装是比较慢的；
4. 将安装完成后的依赖存储到 cache 中，当依赖无变更时，下次安装就可以直接使用 cache；

<img src="/asset/img/2022/cache-nodemodules-speed-ci-build/流程图.jpg" width="500px" alt="流程图">

给出的思路是最简单的版本，如果你的项目是 monorepo 则还需要考虑多个 package.json 和每个子项目中的 node_modules，如果项目严格使用 lock 文件的话，其实也可以使用 lock 文件来生成 cache hash 值。

当然这些都是细节上的事情，解决起来都很快，主要是这种缓存的想法。实际代码我就不贴了，因为真的不难。

## 参考

- 勇神的鬼点子