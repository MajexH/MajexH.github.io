---
title: 签到二维码生成
category: code
tags:
  - js
  - qrcode
toc: true
date: 2019-05-17 19:06:13
thumbnail:
---

<p>做这个东西的灵感来自于[lufer blog]('http://coder.lufer.cc/2018/06/11/%E7%8C%AB%E9%80%94%E6%A0%A1%E5%9B%AD%E7%AD%BE%E5%88%B0%E7%9A%84%E4%BA%8C%E7%BB%B4%E7%A0%81%E4%BC%AA%E9%80%A0/')以及<strong>室友</strong>的微信小程序拆包帮助</p>

## 做这个事儿的原因

嘛，最近老师准备派出去出差去学习 2 周，但是人出去了，课还是要上的，尤其是`杜博士`的课又在最近开了。杜博士还是有点怕的，而好巧不巧杜博士又喜欢用微信小程序的猫途校园来进行二维码签到，所以才有了做这个东西的动力和原因。

<!-- more -->

## 猫途校园签到二维码生成

去年已经有学长大致做过了这个二维码的伪造签到，粗略的看了一下，大概对面只是在二维码里面`明文`放了一个`课程ID&学生ID&&当前时间`的三元组，通过在一两秒的时间内不断变换当前时间来变化二维码，达到必须在现场才能签到的目的。可是咱另外给你生成一个二维码不就行了嘛，23333。

### 问题

这周，专门截图了一个签到二维码来扫码试试，扫码结果如下。

```
x%97%C5%94joj%97%93%92%98ggf%60%60f%9C%C8%C6%C7%94a%95%8B%5B%98%C7%C6%9Bke%98%98chni%95%92%60foo%99%9Anm%99%89Wfjloqljnnikp%5D%u5F46%uC46F%uC8C6
```

嗯？！！这和说好的有点不太一样呀，不是说好的明文存储嘛。在看到%后面跟着数字的时候，果断猜想是使用了 js 的`encodeURI()`来转换了一下无法在浏览器路径栏里面识别的字符，然后果然···猜错了。

### 小程序反编译

（这时候又要膜一波室友了）在尝试了常见了编码和加密方式都没办法拿到加密信息后，在室友的帮助下吧微信小程序反编译拿到了源代码。(微信小程序会在本地缓存一个`.wxapkg`的文件，该文件可以反编译出源代码，[反编译工具](https://github.com/qwerty472123/wxappUnpacker))（以后就算他的加解密方法变了，仍然可以通过这种方法重新拿到他的加解密方法）。总值最后拿到的加解密方法如下。

```javascript
// 加密
function encrypt(str) {
  for (
    var t = String.fromCharCode(str.charCodeAt(0) + str.length), n = 1;
    n < str.length;
    n++
  ) {
    t += String.fromCharCode(str.charCodeAt(n) + str.charCodeAt(n - 1));
  }
  return escape(t);
}

// 解密
function decode(str) {
  str = unescape(str);
  for (
    var t = String.fromCharCode(str.charCodeAt(0) - str.length), n = 1;
    n < str.length;
    n++
  )
    t += String.fromCharCode(str.charCodeAt(n) - t.charCodeAt(n - 1));
  return t;
}
```

总体来说就是通过 js 来产生了 unicode 码的偏移，在加密时，除了第一位偏移了字符串长度的位数，其他的均是偏移了后一个位置的位数。唯一不懂的是`escape()`和`unescape()`方法，一查，这两方法都`已废弃`···，与之对应的是` encodeURI()`和`decodeURI()`。于是将之前扫码拿到的字符串丢进去解码，可以得到

{% asset_img info.png test %}

现在变成了一个四元组(&分隔)，第三个是标准的 unix 时间戳，因此我们只需要拿到四元组的其他三个信息，然后自己生成一个时间戳，就可以生成跟他完全一样的二维码了。最后它每 3 秒钟生成一次二维码，不超过 5s 的时间差就接受该二维码。

{% blockquote %}
自己写了个生成的页面，需要`上传一个自己的签到截图`，就会每 1s 中在页面中生成一个二维码和分享链接，把`分享链接`发送给在教室的大佬们帮忙签到即可。实例如下[majexh.xyz/qrcode](https://www.majexh.xyz/qrcode)
{% endblockquote %}
