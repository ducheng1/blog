---
title: 给可任意选择一级的el-cascader赋值时子节点radio不默认选中的解决方式
published: 2023-02-06
description: 给可任意选择一级的el-cascader赋值时子节点radio不默认选中的解决方式
image: ../../../../assets/images/covers/pexels-photo-32347564.jpeg
tags: [ "Vue2", "ElementUI" ]
category: 前端
draft: false
lang: zh-CN
---

# 问题描述

问题呈现：
![](https://img.dcwedu.top/i/2024/04/01/660a6237e6bdc.png)

解决后：
![](https://img.dcwedu.top/i/2024/04/01/660a623aa8c4c.png)

# 解决方案

```javascript
// 解决el-cascader子节点不默认选中问题
this.$nextTick(() => {
  // 获取所有选中的节点
  let checkedLeaves = document.getElementsByClassName("in-checked-path");
  // 获取子节点
  let checkedLeaf = checkedLeaves[checkedLeaves.length - 1];
  // console.log('leaf', checkedLeaf)
  // 获取子节点中的radio元素
  let radio = checkedLeaf.childNodes[0].childNodes[0];
  radio.classList.add("is-checked");
});
```

# 实现原理

`.in-checked-path`在 elementui 中表示该节点路径被选中（即图中样式不同部分），需要给 radio 加上`.is-checked`使其变成被选中的节点
> 注：将该代码放在 mounted 生命周期中，由于此时元素不一定渲染完全，若有接口请求等异步操作，需要用`async/await`关键字，以此确保能获取到元素。
