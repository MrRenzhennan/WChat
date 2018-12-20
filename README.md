# WChat
微信小程序开发总结  
- [官方文档](https://developers.weixin.qq.com/miniprogram/dev/)
- [在微信小程序中使用animatecss-动画库](#在微信小程序中使用animatecss-动画库)  
- [小程序颜色问题](#小程序颜色问题)  
- [安卓手机wx.hideLoading()无效](#安卓手机wxhideloading无效)
- [微信小程序使用Promise](#微信小程序使用Promise)
- [微信小程序-获取用户session_key,openid,unionid](#微信小程序-获取用户session_key-openid-unionid)
---
:smile:  :laughing:  :blush:  :relaxed:
***
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

## 微信小程序使用Promise
微信小程序使用Promise，其实只需要在API方法外层包一个promise就行了。

本文以微信登陆和获取用户信息接口为例。

封装代码：wechat.js  
```js
/**
 * Promise化小程序接口
 */
class Wechat {
  /**
   * 登陆
   * @return {Promise} 
   */
  static login() {
    return new Promise((resolve, reject) => wx.login({ success: resolve, fail: reject }));
  };
 
  /**
   * 获取用户信息
   * @return {Promise} 
   */
  static getUserInfo() {
    return new Promise((resolve, reject) => wx.getUserInfo({ success: resolve, fail: reject }));
  };
 
};
 
module.exports = Wechat;
```  
调用代码：  
```js
let wechat = require('./wechat.js');
    wechat.login()
      .then(d => {
        console.log("登陆", d);
        return wechat.getUserInfo();
      })
      .then(d => {
        console.log("获取用户信息", d);
      })
      .catch(e => {
        console.log(e);
      })
```  
如果需要传递参数，比如设置本地数据缓存接口：  
```js
  static setStorage(key, value) {
    return new Promise((resolve, reject) => wx.setStorage({ key: key, data: value, success: resolve, fail: reject }));
  };
```  

## 微信小程序-获取用户session_key,openid,unionid
1. 通过wx.login接口获取code既jscode，传递到后端  
2. 后端请求https://api.weixin.qq.com/sns/jscode2session?appid=APPID&secret=SECRET&js_code=JSCODE&grant_type=authorization_code地址，就能获取到openid和unionid。  

**小程序接口promise化和封装**  

**1、utils文件夹下创建wechat.js文件**  
```js
/**
 * Promise化小程序接口
 */
class Wechat {
  /**
   * 登陆
   * @return {Promise} 
   */
  static login() {
    return new Promise((resolve, reject) => wx.login({ success: resolve, fail: reject }));
  };
 
  /**
   * 获取用户信息
   * @return {Promise} 
   */
  static getUserInfo() {
    return new Promise((resolve, reject) => wx.getUserInfo({ success: resolve, fail: reject }));
  };
 
  /**
   * 发起网络请求
   * @param {string} url  
   * @param {object} params 
   * @return {Promise} 
   */
  static request(url, params, method = "GET", type = "json") {
    console.log("向后端传递的参数", params);
    return new Promise((resolve, reject) => {
      let opts = {
        url: url,
        data: Object.assign({}, params),
        method: method,
        header: { 'Content-Type': type },
        success: resolve,
        fail: reject
      }
      console.log("请求的URL", opts.url);
      wx.request(opts);
    });
  };
 
  /**
   * 获取微信数据,传递给后端
   */
  static getCryptoData() {
    let code = "";
    return this.login()
      .then(data => {
        code = data.code;
        console.log("login接口获取的code:", code);
        return this.getUserInfo();
      })
      .then(data => {
        console.log("getUserInfo接口", data);
        let obj = {
          js_code: code,
        };
        return Promise.resolve(obj);
      })
      .catch(e => {
        console.log(e);
        return Promise.reject(e);
      })
  };
 
  /**
   * 从后端获取openid
   * @param {object} params 
   */
  static getMyOpenid(params) {
    let url = 'https://xx.xxxxxx.cn/api/openid';
    return this.request(url, params, "POST", "application/x-www-form-urlencoded");
  };
}
module.exports = Wechat;
```  
**2、修改小程序的app.js文件**  
```js
let wechat = require('./utils/wechat.js');
App({
  onLaunch() {
    this.getUserInfo();
  },
  getUserInfo() {
    wechat.getCryptoData()
      .then(d => {
        return wechat.getMyOpenid(d);
      })
      .then(d => {
        console.log("从后端获取的openid", d.data);
      })
      .catch(e => {
        console.log(e);
      })
  }
})
```

**后端nodejs，是用的express命令行生成的项目框架，**

**1、创建common文件夹，创建utils文件，使用request模块请求接口，promise化request**  

```js
const request = require("request");
class Ut {
 
    /**
     * promise化request
     * @param {object} opts 
     * @return {Promise<[]>}
     */
    static promiseReq(opts = {}) {
	return new Promise((resolve, reject) => {
	    request(opts, (e, r, d) => {
		if (e) {
		    return reject(e);
		}
	        if (r.statusCode != 200) {
		    return reject(`back statusCode：${r.statusCode}`);
		}
		return resolve(d);
	    });
	})
    };
 
};
 
module.exports = Ut;
```

**2、新增路由，appId、secret在小程序的后台获取**  
```js
router.post("/openid", async (req, res) => {
  const Ut = require("../common/utils");
  try {
    console.log(req.body);
    let appId = "wx70xxxxxxbed01b";
    let secret = "5ec6exxxxxx49bf161a79dd4";
    let { js_code } = req.body;
    let opts = {
      url: `https://api.weixin.qq.com/sns/jscode2session?appid=${appId}&secret=${secret}&js_code=${js_code}&grant_type=authorization_code`
    }
    let r1 = await Ut.promiseReq(opts);
    r1 = JSON.parse(r1);
    console.log(r1);
    res.json(r1);
  }
  catch (e) {
    console.log(e);
    res.json('');
  }
})
```  
结果：  
![..](https://img-blog.csdn.net/20180223101525677)  
这个返回结果没有unionid，按照官方的说法，需要在[微信开放平台](https://open.weixin.qq.com/)绑定小程序；  
如果需要解密和数据校验，[请跳转这里](#微信小程序用户数据的签名校验和加解密-后端nodejs)。


## 微信小程序用户数据的签名校验和加解密 - 后端nodejs
