---
title: 解决gin-swagger找不到doc.json问题
published: 2023-02-27
description: gin-swagger找不到doc.json解决方案
image: https://cdn.dcwedu.top/common/2026/06/24/1782290637-c6e25b098b7da9e73b736aa3076241e4.jpg
tags: [ "Golang", "Gin" ]
category: 后端
draft: false
lang: zh-CN
---

使用 gin-swagger 时出现找不到 doc.json 的问题，只需在 main.go 中使用`docs.SwaggerInfo.ReadDoc()`即可。

官方注释：`ReadDoc`会把`SwaggerTemplate`转换成为 swagger 文档，即把 docs 文件夹中的内容转换成 doc.json。
> 注意：引入的docs应为工程内 swag-cmd 生成的文件夹
