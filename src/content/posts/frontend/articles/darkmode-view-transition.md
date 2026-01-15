---
title: 深色模式切换动画实现
published: 2025-11-11
description: 使用View Transition API实现丝滑的深色模式切换动画
image: ../../../../assets/images/covers/photo-1762632173940-d08a7b9a19b3.avif
tags: [ "随笔" ]
category: 前端
draft: false
lang: zh-CN
---

实现效果如下：
![效果图](/static/images/darkmode-view-transition/shot.gif)

## 前置条件

- 浏览器支持View Transition API（目前仅支持Chrome 116+）
- 用户未开启减少动画偏好

``` typescript
function supportTransitions() {
  return (
    'startViewTransition' in document &&
    window.matchMedia('(prefers-reduced-motion: no-preference)').matches
  )
}
```

## 动画实现

说明：这里动画从鼠标点击位置开始，也可以按需调整。

``` typescript
async function toggleTransition({ clientX: x, clientY: y }: MouseEvent) {
  // 判断是否支持动画
  if (!supportTransitions()) {
    themeStore.toggleColorScheme()
    return
  }
  // 计算圆的路径
  const clipPath = [
    `circle(0px at ${x}px ${y}px)`,
    `circle(${Math.hypot(
      Math.max(x, innerWidth - x),
      Math.max(y, innerHeight - y),
    )}px at ${x}px ${y}px)`,
  ]
  // View Transition回调
  await document.startViewTransition(async () => {
    themeStore.toggleColorScheme()
    await nextTick()
  }).ready
  // 动画实现
  document.documentElement.animate(
    { clipPath: themeStore.darkMode ? clipPath.reverse() : clipPath },
    {
      duration: 300,
      easing: 'cubic-bezier(.4, 0, .2, 1)',
      // 这行配置说明在动画结束之后保留元素，防止视图闪烁
      // https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect/KeyframeEffect#fill
      fill: 'both',
      pseudoElement: `::view-transition-${themeStore.darkMode ? 'old' : 'new'}(root)`,
    },
  )
}
```
添加如下样式，保证动画切换时新元素覆盖在旧元素之上
``` css
::view-transition-old(root),
::view-transition-new(root) {
  animation: none;
  mix-blend-mode: normal;
}

::view-transition-old(root),
.dark::view-transition-new(root) {
  z-index: 9998;
}

::view-transition-new(root),
.dark::view-transition-old(root) {
  z-index: 9999;
}
```
