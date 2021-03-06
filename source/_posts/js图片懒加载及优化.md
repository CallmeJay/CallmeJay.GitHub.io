---
title: js图片懒加载及优化
date: 2017-06-18 22:20:23
categories: JavaScript
tags: image-lazyload
---

## 为啥要用图片懒加载

对页面加载速度影响最大的就是图片，一张普通的图片可以达到几 M 的大小，而代码也许就只有几十 KB。当页面图片很多时，页面的加载速度缓慢，几 S 钟内页面没有加载完成，也许会失去很多的用户。
所以，对于图片过多的页面，为了加速页面加载速度，所以很多时候我们需要将页面内未出现在可视区域内的图片先不做加载， 等到滚动到可视区域后再去加载。这样子对于页面加载性能上会有很大的提升，也提高了用户体验。

<!--more-->

## 原理

将页面中的 img 标签 src 指向一张小图片或者 src 为空，然后定义 data-src（这个属性可以自定义命名，我才用 data-src）属性指向真实的图片。src 指向一张默认的图片，否则当 src 为空时也会向服务器发送一次请求（指向默认的一张图那就只需请求一次）。可以指向 loading 的地址。

当载入页面时，先把可视区域内的 img 标签的 data-src 属性值负给 src，然后监听滚动事件，把用户即将看到的图片加载。这样便实现了懒加载。

> 注：图片要指定宽高。
> 关于窗口各种宽度，可以看下面两篇文章：
> [scrollWidth,clientWidth,offsetWidth 的区别](http://www.cnblogs.com/kongxianghai/p/4192032.html)
> [JS 中关于 clientWidth offsetWidth scrollWidth 等的含义](http://www.cnblogs.com/fullhouse/archive/2012/01/16/2324131.html)

## 图片懒加载的实现代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
    <style>
      img {
        display: block;
        margin-bottom: 50px;
        width: 400px;
        height: 400px;
      }
    </style>
  </head>
  <body>
    <img
      src=""
      data-src="http://pic.58pic.com/58pic/17/18/97/01U58PIC4Xr_1024.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://cover.read.duokan.com/mfsv2/download/fdsc3/p01N203pHTU7/Wr5314kcLAtVCi.jpg!t"
      alt=""
    />
    <img
      src=""
      data-src="http://77fkxu.com1.z0.glb.clouddn.com/20160308/1457402219_73571.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://pic1.cxtuku.com/00/16/18/b3809a2ba0f3.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://img.bitscn.com/upimg/allimg/c150708/14363B06253120-6060O.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://cover.read.duokan.com/mfsv2/download/fdsc3/p015trgKM7vw/H0iyDPPneOVrA4.jpg!t"
      alt=""
    />
    <img
      src=""
      data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://imgsrc.baidu.com/baike/pic/item/2f9cbdcc5e0bcf5c00e9283b.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg"
      alt=""
    />
    <script>
      (function() {
        let num = document.getElementsByTagName("img").length;
        let img = document.getElementsByTagName("img");
        let n = 0; //存储图片加载到的位置，避免每次都从第一张图片开始遍历
        lazyload(); //页面载入完毕加载可是区域内的图片
        window.onscroll = lazyload;
        function lazyload() {
          //监听页面滚动事件
          let seeHeight = document.documentElement.clientHeight; //可见区域高度
          let scrollTop =
            document.documentElement.scrollTop || document.body.scrollTop; //滚动条距离顶部高度
          for (let i = n; i < num; i++) {
            // 图片未出现时距离顶部的距离大于滚动条距顶部的距离+可视区的高度
            if (img[i].offsetTop < seeHeight + scrollTop) {
              if (img[i].getAttribute("src") == "") {
                img[i].src = img[i].getAttribute("data-src");
              }
              n = i + 1;
            }
          }
        }
      })();
    </script>
  </body>
</html>
```

## 使用节流函数进行优化

如果直接将函数绑定在 scroll 事件上，当页面滚动时，函数会被高频触发，这非常影响浏览器的性能。

同时还有以下场景往往由于事件频繁被触发，因而频繁执行 DOM 操作、资源加载等重行为，导致 UI 停顿甚至浏览器崩溃。
1.window 对象的 resize、scroll 事件 2.拖拽时的 mousemove 事件 3.射击游戏中的 mousedown、keydown 事件 4.文字输入、自动完成的 keyup 事件
**解决这个问题的方法有去抖动和节流的方法**

- 去抖动原理： 当调用动作 n 毫秒后，才会执行该动作，若在这 n 毫秒内又调用此动作则将重新计算执行时间。
  > 不足:当我一直滚动鼠标的时候，lazyload 函数就会不断被延迟，这样只有停下来的时候才会执行，那么再有些需要及时显示的情况下，就显得不那么友好了
- 节流原理：预设一个执行周期，如果这个周期结束了都还没触发函数，那就会执行一次函数；如果这个周期还没结束就触发了函数，那定时器将重置，开始新周期。
  > 达到了想要的效果，既没有频繁的执行也没有延迟执行
  > <br/>

---

## 运用节流函数的图片懒加载代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
    <style>
      img {
        display: block;
        margin-bottom: 50px;
        width: 400px;
        height: 400px;
      }
    </style>
  </head>
  <body>
    <img
      src=""
      data-src="http://pic.58pic.com/58pic/17/18/97/01U58PIC4Xr_1024.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://cover.read.duokan.com/mfsv2/download/fdsc3/p01N203pHTU7/Wr5314kcLAtVCi.jpg!t"
      alt=""
    />
    <img
      src=""
      data-src="http://77fkxu.com1.z0.glb.clouddn.com/20160308/1457402219_73571.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://pic1.cxtuku.com/00/16/18/b3809a2ba0f3.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://img.bitscn.com/upimg/allimg/c150708/14363B06253120-6060O.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://cover.read.duokan.com/mfsv2/download/fdsc3/p015trgKM7vw/H0iyDPPneOVrA4.jpg!t"
      alt=""
    />
    <img
      src=""
      data-src="http://ww1.sinaimg.cn/large/006y8mN6gw1fa7kaed2hpj30sg0l9q54.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://imgsrc.baidu.com/baike/pic/item/2f9cbdcc5e0bcf5c00e9283b.jpg"
      alt=""
    />
    <img
      src=""
      data-src="http://ww4.sinaimg.cn/large/006y8mN6gw1fa5obmqrmvj305k05k3yh.jpg"
      alt=""
    />
    <script>
      (function() {
        let num = document.getElementsByTagName("img").length;
        let img = document.getElementsByTagName("img");
        let n = 0; //存储图片加载到的位置，避免每次都从第一张图片开始遍历
        lazyload(); //页面载入完毕加载可是区域内的图片
        function lazyload() {
          //监听页面滚动事件
          let seeHeight = document.documentElement.clientHeight; //可见区域高度
          let scrollTop =
            document.documentElement.scrollTop || document.body.scrollTop; //滚动条距离顶部高度
          for (let i = n; i < num; i++) {
            // 图片未出现时距离顶部的距离大于滚动条距顶部的距离+可视区的高度
            if (img[i].offsetTop < seeHeight + scrollTop) {
              if (img[i].getAttribute("src") == "") {
                img[i].src = img[i].getAttribute("data-src");
              }
              n = i + 1;
            }
          }
        }
        // 采用了节流函数
        function throttle(fun, delay, time) {
          let timeout,
            startTime = new Date();
          return function() {
            let context = this,
              args = arguments,
              curTime = new Date();
            clearTimeout(timeout);
            // 如果达到了规定的触发时间间隔，触发 handler
            if (curTime - startTime >= time) {
              fun.apply(context, args);
              startTime = curTime;
              // 没达到触发间隔，重新设定定时器
            } else {
              timeout = setTimeout(fun, delay);
            }
          };
        }
        window.addEventListener("scroll", throttle(lazyload, 500, 1000));
      })();
    </script>
  </body>
</html>
```
