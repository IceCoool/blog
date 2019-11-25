---
title: wxs的使用
date: 2019-11-25 10:54:28
tags:
categories:
---

wxs的简单使用

<!-- more -->

在utils.wxs内注册并导出需要的函数
```javascript
function getStatus(status) {
  status = status.toString()
  var statusObj = {
    '3': '待接受',
    '4': '已拒绝',
    '5': '已接单',
    '6': '服务中',
    '7': '已完成'
  };
  var text = statusObj[status]
  return text
}

module.exports = {
  getStatus: getStatus
}
```

在需要的wxml文件内引入utils.wxs工具函数

```html
<wxs src="../wxs/utils.wxs" module="utils" />

<view>{{utils.getStatus(item.status)}}</view>
```