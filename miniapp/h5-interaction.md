# 小程序与H5页面交互

## 前言

&emsp;&emsp;目前小程序页面回退H5是无法带入数据的，例如用户A在H5上选择拍照，跳转到小程序页面拍照压缩并将照片地址带回H5展示出来。<br>

&emsp;&emsp;小程序与H5之间的交互目前只有官方给出的postMessage接口，并且只能在满足特定时机下触发message事件才能拿到数据。<br>

&emsp;&emsp;为满足日益繁杂的产品需求，提高迭代速度，又不能放弃小程序的优势功能，因此需要打通小程序与H5的数据交换。

## 实现方案
利用H5的hash特性，使H5知道有数据变化，并且页面不会刷新，如下图所示流程：

<img src="https://file.40017.cn/trainwechat/lct/h5lct.png" width="70%">

### 关于window.history.go(-1)
hash变更会导致页面历史栈长度+1，执行window.history.go(-1)保证和之前的url一致，并且history.go(-1)只后退不刷新，而history.back()后退+刷新

### 踩坑：
* webview的src是固定的，不会随着H5地址的变化而改变，因此H5跳转小程序需要将当前页面的url传递过去，设置H5的数据时回传过去，然后在onShow中重新组合url
* 改变的hash值必须要和之前的不同，确保webview的src更新，触发H5的hashchange

## 使用示例

### 小程序跳转H5 

```js
wx.navigateTo({
    url: 'webview路径' + `?src=${encodeURIComponent(h5地址)}`
})
```

### H5跳转小程序 

```js
window.wx.miniProgram.navigateTo({
    //backUrl参数跳转的小程序页面会记住，以供后续回传使用
    url: '小程序页面地址...' + `&backUrl=${encodeURIComponent(H5当前页面的地址location.href)}`
})
```

## 小程序代码示例

```js
Page({
    data: {
        src: '',
        canHtml: wx.canIUse && wx.canIUse('web-view')
    },
    onLoad(options) {
        let src = decodeURIComponent(options.src || '').replace(/(^\s*)|(\s*$)/g, '');
        //记录h5初始的URL
        this.baseUrl = src;
        this.setData({
            src: src || ''
        })
    },
    /**
    * 设置H5的数据
    * @param {Object} res - { data: 'xxx', backUrl: 'xxx' }
    */
    setH5Data(res) {
        //这里处理上面提到踩坑的第二点，v: Date.now()时间戳确保拼接webview的src与上一次的不同，触发H5的hashchange
        this.miniappData = encodeURIComponent(JSON.stringify({data: res.data, v: Date.now()}));
        this.backUrl = res.backUrl ? decodeURIComponent(res.backUrl) : this.baseUrl;
    },
    onShow() {
        //h5站使用的是vue路由hash模式，这里直接拼接数据
        if (this.miniappData) {
            let src = this.backUrl + (this.backUrl.indexOf('?') > -1 ? '&' : '?') + `_miniappData=${this.miniappData}`;
            this.miniappData = null;
            this.setData({
                src
            })
        }
    }
})
```

## 小程序传值给H5

#### 注意：小程序返回H5先调用setH5Data方法传递数据再返回，参数格式 { data: 'xxx', backUrl: 'xxx' }

```js
goH5Page(data) {
    let pages = getCurrentPages();
    let pre = pages[pages.length - 2];
    pre.setH5Data && pre.setH5Data({
        data: data,
        backUrl: this.backUrl
    });
    wx.navigateBack();
}
```

## 监听示例：vue路由hash模式

```js
watch: {
    "$route.query._miniappData": {
        handler(val) {
            //vue有子路由，this.routePath记录了当前页的地址，如果当前是子路由，则只触发子路由的监听
            if (val && this.$route.path == this.routePath) {
                window.history.go(-1); //这个是核心
                let res = JSON.parse(decodeURIComponent(val)).data || "";
                xxx 业务逻辑
            }
        },
        deep: true
    }
}
```

#### Tips
* 上述示例是基础实现，有其他业务需求可扩展
* 该功能已上线，体验很棒

![小程序与H5页面交互](https://file.40017.cn/train4in1/img/ezgif-5-50060ffa55ec.gif)