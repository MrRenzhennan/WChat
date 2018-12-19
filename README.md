# WChat
微信小程序开发总结  
- [官方文档](https://developers.weixin.qq.com/miniprogram/dev/)

## 在微信小程序中使用animate.css 动画库
[微信小程序动画有自己的方法](https://developers.weixin.qq.com/miniprogram/dev/api/wx.createAnimation.html)  
[animate.css官网](https://daneden.github.io/animate.css/)  
由于小程序对代码大小限制比较大，所以删除了animate.css中 所有@-webkit-部分css，减少了一半体积  
再把文件后缀名改为wxss，以后出来的百度小程序、支付宝小程序、今日头条小程序估计修改对应的后缀名就可以直接使用了  
[点击下载](http://nodejs999.com/animate.wxss)  
放到小程序代码中，然后@import到app.wxss文件中。   
我项目是把animate.wxss文件放在utils文件夹下。  
所以在app.wxss中加入 @import './utils/animate.wxss'; 即可


## 小程序颜色问题
在写tabBar的时候配置了字体颜色，代码如下  
![..](https://img-blog.csdn.net/20170705145035143?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenp3d2pqZGox/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
结果安卓上正常显示,苹果手机却无法显示tabBar上的文字，仔细看了下官网，  
需要16进制颜色值  
改成这样就好了。  
![..](https://img-blog.csdn.net/20170705145142733?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenp3d2pqZGox/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  
然后又发现页面设置的背景颜色，ios正常，安卓手机又无法显示了。。。

```css
background: #666;
```

经过多次测试，得到的结果是，这个16进制颜色值不能简写，6位要写全。

```css
background: #666666; 这样就正常了。
```
只能说明小程序真任性！！

## 安卓手机wx.hideLoading()无效

首先在onLoad()中
```js
wx.showLoading({
  title: “数据加载中”,
  mask: true
});
```
异步获取数据后  

`wx.hideLoading();`  

测试结果  
在微信开发者工具和iOS上都能正常隐藏loading框，安卓手机上却无法隐藏。  
**解决办法**  
wx.hideLoading()方法外层加个setTimeout居然就解决了。  
```js
setTimeout(() => {
   wx.hideLoading();
}, 100);
```
