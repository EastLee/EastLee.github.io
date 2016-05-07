---

title: 移动端之webp图片
date: 2016-04-27 21:38:17
categories: 移动端性能优化
tags: 性能优化

---

## [webp是什么？](http://baike.baidu.com/link?url=tL9ztrug4h__GQfPznbD0O4-6iDZ5eLEsXdHZ-3crUE0GTcwmwXPtZi4Yvu4P4H2iGq4ru9XNBPG21zYm9FHqa)
WebP格式，谷歌（google）开发的一种旨在加快图片加载速度的图片格式。图片压缩体积大约只有JPEG的2/3，并能节省大量的服务器带宽资源和数据空间。Facebook Ebay等知名网站已经开始测试并使用WebP格式。但WebP是一种有损压缩。相较编码JPEG文件，编码同样质量的WebP文件需要占用更多的计算资源。

## 目前使用情况
国外的网站：Youtube、Facebook.etc
国内的网站：腾讯、淘宝、美团等
 <!--more-->
## webp兼容性
### 支持
chrome opera
4s uc
5 uc
5s uc
6 uc
6s uc
华为荣耀6 qq内置浏览器 微信 uc 自带浏览器
魅族 微信 uc 自带浏览器
三星（Galay S4） uc 自带浏览器 QQ浏览器
红米1s（andr4.3） uc QQ浏览器-v6.4.1 自带浏览器

### 不支持
58app-v6.5.7
4s safari
5 safari
5s safari
6 safari
6s safari
5s qq内置浏览器
6 qq内置浏览器
6s qq内置浏览器
QQ浏览器-v6.0
QQ浏览器-v6.4
5微信
5s微信
6微信
6s微信
5s 高速浏览器
## 如何转换
[智图](http://zhitu.isux.us/)
[iSparta](http://isparta.github.io/)
[libwebp](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html)
## 如何使用
### js检测，准备两套图片路径

```js    
    var kTestImages = {
        lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
        lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
        alpha: "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
        animation: "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA"
    };
    var img = new Image();
    img.onload = function () {
        var result = (img.width > 0) && (img.height > 0);
        callback(feature, result);
    };
    img.onerror = function () {
        callback(feature, false);
    };
    img.src = "data:image/webp;base64," + kTestImages[feature];
    }
```

### 使用webp.js
插件将会捕捉页面中使用WebP格式的img元素，并用Flash进行替换。图像的解码及显示都在Flash中完成，因此目前版本对CSS设置的背景图片无效。
当然，作为JPEG格式的替换，只有对较大的图像使用才有意义，否则过多的解码将消耗大量的资源。
3、用html5中提供的picture元素选择图片格式（浏览器支持的情况不好）

### server-response（Accept和varry）
#### Accept
iphone4s
uc无Accept请求头
Safari Accept:\*/\*
微信 Accept:\*/\*
QQ浏览器-v6.4 Accept:\*/\*
小米4
uc：Accept:image/webp
自带浏览器：Accept:image/webp
chrome：Accept:image/webp
QQ浏览器：Accept:image/webp
并不是所有的请求头都包含images/webp 目前只有opera有
#### varry
             ```
         (client)  >  Accept: image/jpeg, image/png, image/mif
         (server) >  Content-Type: image/mif
                  >  Vary: Accept
                  >  (object)

               ```
## 优势
WebP is a new image format that provides lossless and lossy compression for images on the web. WebP lossless images are 26% smaller in size compared to PNGs. WebP lossy images are 25-34% smaller in size compared to JPEG images at equivalent SSIM index. WebP supports lossless transparency (also known as alpha channel) with just 22% additional bytes. Transparency is also supported with lossy compression and typically provides 3x smaller file sizes compared to PNG when lossy compression is acceptable for the red/green/blue color channels.


## 参考文献：
[WebP 探寻之路](http://isux.tencent.com/introduction-of-webp.html)
[A new image format for the Web](https://developers.google.com/speed/webp/)
[Deploying New Image Formats on the Web](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)
[How To Reduce Image Size With WebP Automagically](How To Reduce Image Size With WebP Automagically)
[让你的页面支持WebP图像](http://www.etherdream.com/WebP/)

