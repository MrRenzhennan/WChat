# WChat
微信小程序开发总结

## 在微信小程序中使用animate.css 动画库
[微信小程序动画有自己的方法](https://developers.weixin.qq.com/miniprogram/dev/api/wx.createAnimation.html)  
[animate.css官网](https://daneden.github.io/animate.css/)  
由于小程序对代码大小限制比较大，所以删除了animate.css中 所有@-webkit-部分css，减少了一半体积  
再把文件后缀名改为wxss，以后出来的百度小程序、支付宝小程序、今日头条小程序估计修改对应的后缀名就可以直接使用了  
[点击下载](http://nodejs999.com/animate.wxss)
