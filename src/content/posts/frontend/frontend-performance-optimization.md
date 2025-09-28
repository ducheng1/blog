---
title: 总结一些前端性能优化的方式
published: 2025-06-11
image: https://images.unsplash.com/photo-1748426156241-6e5239022c03?ixid=M3wzNTk3NzR8MHwxfHRvcGljfHxibzhqUUtUYUUwWXx8fHx8Mnx8MTc0OTYzMTAzNHw&ixlib=rb-4.1.0
tags: [ "性能优化" ]
category: 前端
---

# 网络层优化

## 减少http请求

- 打包时进行代码拆分
- 关键的js、css资源内联到html

## 压缩资源

- 启用服务器gzip/brotli压缩
- 图片静态资源体积压缩（使用webp等）
- 代码压缩（使用vite、terser等进行代码压缩）

## 资源缓存

- 浏览器缓存（Cache-Control）
- PWA离线缓存

## CDN

- 静态资源（js、css、图片等）迁移到CDN

# 渲染层优化

## 资源加载优化

- js异步async/延迟defer加载，防止阻塞渲染

## 减少重排（Reflow）和重绘（Repaint）

- 减少DOM操作
- 使用transform、opacity等css动画或requestAnimationFrame跳过重排

## 使用虚拟列表

## 图片懒加载

- 部分组件库提供lazyload属性
- 原生img标签的lazyload属性
- 使用IntersectionObserver（交叉视图观察者）监听图片，确保元素进入视图后再加载

# js执行效率优化

## 防抖、节流

## 使用WebWorker创建多线程任务

## 避免内存泄露

- 使用WeakMap
- 及时移除eventListener
- 及时清除setInterval、setTimeout等定时器
- 释放闭包内存（如闭包内部引用的外部变量未被释放）
- 滥用全局变量，长期占用内存

# 打包优化

## 使用TreeShaking移除未使用的代码（依赖ESM）

## 按需加载第三方库

## 代码分割

# 其他方式

## 使用服务端渲染（SSR）

## 使用WASM加速复杂运算
