---
title: 关于Date相关方法的兼容性问题
date: 2019-02-12 14:50:49
tags:
categories:
---

工作中经常使用到Date的相关方法，也趟过莫名其妙的坑，记录一下印象深刻的事情。
<!-- more -->
> 其中一个是不同浏览器对于`toLocaleDateString`和`toLocaleTimeString`的解析,通常会出现以下几种

* "2019/2/12" "14:48:25"
* "2019-2-12" "下午2:44:12"
* "2019年2月12" 
因此在格式化时间的时候，需要写好不同的正则表达式来处理。

比较恶心的是关于IE浏览器的处理，附上对于IE浏览器的判断
```javascript
function isIE() {
    if (!!window.ActiveXObject || "ActiveXObject" in window){
        return true;
    }else{
        return false;
    }
}
```
> 另一个是IE浏览器cooike设置的问题

之前接到过一个需求，在不需要登录的情况下实现点赞功能，一天只能点一次，相关代码如下
```javascript
function seeCookie() {
        var flag = $.cookie('flag')
        if (flag) {
            alert('已存在flag')
            return
        }
        alert('下面代码执行')
        // 是否为IE浏览器 
        function isIE() {
            if (!!window.ActiveXObject || "ActiveXObject" in window) {
                return true;
            } else {
                return false;
            }
        }

        var curDate = new Date();
        //当前时间戳
        var curTamp = curDate.getTime();
        //当日凌晨的时间戳,减去一毫秒是为了防止后续得到的时间不会达到00:00:00的状态
        var timeString = curDate.toLocaleDateString();
        isIE() ? timeString = timeString.replace('年', '/').replace('月', '/').replace('日', '') : '';
        var curWeeHours = new Date(timeString).getTime() - 1;
        //当日已经过去的时间（毫秒）
        var passedTamp = curTamp - curWeeHours;
        //当日剩余时间
        var leftTamp = 24 * 60 * 60 * 1000 - passedTamp;
        var leftTime = new Date();
        leftTime.setTime(leftTamp + curTamp);
        $.cookie("flag", "1", {
            path: '/',
            expires: leftTime
        });
    }
    seeCookie()
```
可以把这段代码放到页面中尝试一下，会发现其他浏览器没有问题，但是IE会出现当浏览器关闭后重新打开页面，总是会alert`console.log('下面代码执行')`，这说明没有获取到cookie，cookie的过期时间没起作用。那IE浏览器点赞功能就出现问题了。没办法，找原因吧。打开控制台一步步打点，终于发现了问题。但是表示没看懂。。。纳尼？看图
![](/images/IEcookie.png)
竟然是这里出现了问题，不应该啊。
精简代码，并打印
```javascript
    function seeCookie() {
        function isIE() {
            if (!!window.ActiveXObject || "ActiveXObject" in window) {
                return true;
            } else {
                return false;
            }
        }
        var curDate = new Date();
        var timeString = curDate.toLocaleDateString();
        isIE() ? timeString = timeString.replace('年', '/').replace('月', '/').replace('日', '') : '';
        console.log(typeof timeString)
        console.log(timeString)
        console.log(new Date(timeString))
        console.log(new Date('2019/2/12'))
    }
    seeCookie()
```
![](/images/IEcookie2.png)
还是这种结果。。
为什么会是这样，直接写日期没有问题，通过`toLocaleDateString`获取传值就不行，一样的字符串，表示一脸懵**
难道`toLocaleDateString`获取到的并不是一个单纯的字符串？再改改代码确保是字符串无疑
```javascript
    function seeCookie() {
        function isIE() {
            if (!!window.ActiveXObject || "ActiveXObject" in window) {
                return true;
            } else {
                return false;
            }
        }
        var curDate = new Date();
        var timeString = curDate.toLocaleDateString();
        isIE() ? timeString = timeString.replace('年', '/').replace('月', '/').replace('日', '') : '';
        var arr = timeString.split('/');
        timeString = arr[0] + '/' + arr[1] + '/' + arr[2]
        var timeString = year + '/' + month + '/' + day
        console.log(timeString)
        console.log(new Date(timeString))
        console.log(new Date('2019/2/12'))
    }
    seeCookie()
```
结果同上，哇！！恶心

最后在改吧
```javascript
    function seeCookie() {
        function isIE() {
            if (!!window.ActiveXObject || "ActiveXObject" in window) {
                return true;
            } else {
                return false;
            }
        }
        var curDate = new Date();
        var year = curDate.getFullYear(),
            month = curDate.getMonth() + 1,
            day = curDate.getDate()
        var timeString = year + '/' + month + '/' + day
        console.log(typeof timeString)
        console.log(timeString)
        console.log(new Date(timeString))
        console.log(new Date('2019/2/12'))
    }
    seeCookie()
```
结果如下
![](/images/IEcookie3.png)
莫名其妙的好了！！！
后来才发现，`toLocaleDateString`这个方法IE10及以下的版本是没有问题的，但是IE11和IE Edge使用则会出现上述问题，嗯。。为什么越高的版本会出现不兼容低版本的情况，不明白。。。



