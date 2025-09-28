---
title: 解决gin-swagger找不到doc.json问题
published: 2023-02-27
image: https://images.unsplash.com/photo-1749371930388-50c782b0acea?ixid=M3wzNTk3NzR8MHwxfHRvcGljfHxibzhqUUtUYUUwWXx8fHx8Mnx8MTc0OTYzMTAzNHw&ixlib=rb-4.1.0
tags: [ "Golang", "Gin" ]
category: 后端
---

使用 gin-swagger 时出现找不到 doc.json 的问题，只需在 main.go 中使用`docs.SwaggerInfo.ReadDoc()`即可。

官方注释：`ReadDoc`会把`SwaggerTemplate`转换成为 swagger 文档，即把 docs 文件夹中的内容转换成 doc.json。
> 注意：引入的docs应为工程内 swag-cmd 生成的文件夹
