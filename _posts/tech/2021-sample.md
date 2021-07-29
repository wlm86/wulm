---
layout: post
title: systemtap
category: VIRTUAL
tags: VIRTUAL
description:  systemtap
---

### 1.**注意事项**

1. 代码注意事项 

   1) "#", // , /**/都可以

   2) 如果多个probe需要共用变量，需要通过global声明全局变量。

      例：global list[400] 声明了一个数组，可以存储400个元素


2. 目标代码内核变量 

