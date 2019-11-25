---
title: 获取wifi
date: 2019-05-22 15:14:54
tags:
categories:
---

测试手机的网络类型

<!-- more -->

<script>
            getNetworkType()

            function getNetworkType() {
                var ua = navigator.userAgent;
                // alert(navigator.userAgent)
                var connection = navigator.connection||navigator.mozConnection||navigator.webkitConnection||{tyep:'unknown'};
                alert(connection.bandwidth)
                var networkStr = ua.match(/NetType\/\w+/) ? ua.match(/NetType\/\w+/)[0] : 'NetType/other';
                networkStr = networkStr.toLowerCase().replace('nettype/', '');
                var networkType;
                switch(networkStr) {
                    case 'wifi':
                        networkType = 'wifi';
                        break;
                    case '4g':
                        networkType = '4g';
                        break;
                    case '3g':
                        networkType = '3g';
                        break;
                    case '3gnet':
                        networkType = '3g';
                        break;
                    case '2g':
                        networkType = '2g';
                        break;
                    default:
                        networkType = 'other';
                }
                // alert(networkStr)
            }
</script>