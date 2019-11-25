---
title: IOS中new Date()的问题
date: 2019-11-25 11:19:37
tags:
categories:
---

IOS中new Date()的问题

<!-- more -->

```javascript

new Date('2019-08-07') // 正常
new Date('2019-08-07 11:26:30') // Invalid Date
new Date('2019-08-07T11:26:30') // 返回值快8小时
new Date('2019/08/07') // 正常
new Date('2019/08/07 11:26:30') // 正常
new Date('2019-8-7') // Invalid Date
new Date('2019-8-7 11:26:30') // Invalid Date
new Date('2019/8/7') // 正常
new Date('2019/8/7 11:26:30') // 正常
```
