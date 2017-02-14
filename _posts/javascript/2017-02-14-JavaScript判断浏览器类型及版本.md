---
layout: post
title:  "JavaScript判断浏览器类型及版本"
date:   2017-02-14
desc: "JavaScript判断浏览器类型及版本"
keywords: "mancoxu,javascript"
categories: [Javascript]
tags: [Javascript]
icon: icon-html
---

高效简洁 判断浏览器类型及版本代码 ，亲测！

```javascript

    var Sys = {};
    var ua = navigator.userAgent.toLowerCase();
    var s;

    (s = ua.match(/rv:([\d.]+)\) like gecko/)) ? Sys.ie = s[1] :
    (s = ua.match(/msie ([\d.]+)/)) ? Sys.ie = s[1] :
    (s = ua.match(/firefox\/([\d.]+)/)) ? Sys.firefox = s[1] :
    (s = ua.match(/chrome\/([\d.]+)/)) ? Sys.chrome = s[1] :
    (s = ua.match(/opera.([\d.]+)/)) ? Sys.opera = s[1] :
    (s = ua.match(/version\/([\d.]+).*safari/)) ? Sys.safari = s[1] : 0;
    
    if (Sys.ie) console.log('IE: ', Sys.ie);
    if (Sys.firefox) console.log('Firefox: ', Sys.firefox);
    if (Sys.chrome) console.log('Chrome: ', Sys.chrome);
    if (Sys.opera) console.log('Opera: ', Sys.opera);
    if (Sys.safari) console.log('Safari: ', Sys.safari);

```

[原文链接: http://keenwon.com/851.html](http://keenwon.com/851.html)