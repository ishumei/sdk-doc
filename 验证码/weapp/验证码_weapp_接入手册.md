## 验证码weapp-sdk接入手册

### 支持的小程序

支持微信小程序、qq小程序、头条小程序、支付宝小程序，同时支持Taro框架集成各版本集成验证码组件。本文档介绍微信插件集成方式，其他小程序接入方式请与数美技术支持团队联系。

### 1. 在如下页面申请小程序插件

https://mp.weixin.qq.com/wxopen/plugindevdoc?appid=wxfb52904a0e24dc20&token=&lang=zh_CN

申请完成后请通知数美，待数美审批。审批完成后，按照如下流程接入SDK。

### 2. 初始化小程序验证码
```js
// app.json
{
    "pages": [],
    "plugins": {
        "smCaptcha": {
            "version": "1.3.4",
            "provider": "wxfb52904a0e24dc20"
        }
    }
}

// pages/index/index.json 这里指的是需要用验证码组件的页面配置文件
{
    "usingComponents": {
        "sm-captcha": "plugin://smCaptcha/captcha"
    }
}

// pages/index/index.js
const plugin = requirePlugin("smCaptcha");

Page({
    data: {
        options: { // 具体请查看config参数配置
            organization: 'xxx', // 公司标识必填
            tipsMessage: { // 非必填
                slidePlaceholder: '自定义滑块提示文案'
            },
            product: 'popup',
            mode: 'slide',
        }
    },

    onCaptchaReady() { // 验证码资源加载就绪时触发
        console.log('验证码准备就绪');
    },

    onSuccess(e) { // 验证成功时触发
        console.log('我是验证成功的回调', e.detail);
    },

    onFailure(e) { // 验证失败时触发
        console.log('我是验证失败的回调', e.detail);
    },

    onError(e) { // 监听错误事件
        console.log('出错啦', e.detail);
        // 如果 e.detal.isValid === false 表示当基础库版本低于2.1.0，不兼容
    },

    onClose() { // 当引用方式是popup时，有效， 监听验证码关闭
        console.log('监听验证码关闭');
    },

    resetCaptcha() { // 某些场景需要初始化验证码
        plugin.reset();
    },

    disableCaptcha() { // 某些场景需要主动禁用验证码使用
        plugin.disableCaptcha();
    },

    enableCaptcha() { // 某些场景需要主动启用验证码使用
        plugin.enableCaptcha()
    },

    showCaptcha() { // 当引用方式是popup时，有效，调用验证码弹窗
        plugin.verify();
    },

    getResult() { //主动获取验证结果，如：验证码正确可以走后续逻辑，验证失败做提示
        const result = plugin.getResult();
        console.log('主动获取验证结果', result);
    },
});


// pages/index/index.wxml 页面代码
<view class="btn-box">
    <!--showCaptcha方法 在product为 'popup' 有效-->
    <button wx:if="{{options.product==='popup'}}" bindtap="showCaptcha">显示验证码</button>
    <button id="add" bindtap="resetCaptcha">刷新</button>
    <button bindtap="enableCaptcha">启用</button>
    <button bindtap="disableCaptcha">禁用</button>
    <button bindtap="getResult">获取验证结果</button>
</view>

<sm-captcha
    options="{{options}}"
    bindcaptchaReady="onCaptchaReady"
    bindsuccess="onSuccess"
    bindfailure="onFailure"
    binderror="onError"
    bindclose="onClose"
>
</sm-captcha>
```
 

参数配置如下所示：

| **字段** | **参数类型** | **是否必填** | **默认值** | **备注** |
| -- | -- | -- | -- | -- |
| organization | string  | 是  | 无 | 数美分配的公司标识，数美后台可以查看 |
| appId | string  | 否  | default| 应用标识，区分不同应用，数美后台可以管理  |
| channel | string  | 否  | default | 推广渠道，可自定义 |
| product | string  | 否  | embed（嵌入式）| 展现形式，支持：embed（嵌入式）、popup（弹出层） |
| mode | string  | 否  | slide  | 模式：支持slide（滑动验证码） 、select（文字点选验证码）、 icon_select(图标点选验证码)、 seq_select(成语语序验证码)、spatial_select(空间逻辑) |
| lang | string  | 否  | zh-cn  | 模式：支持zh-cn en，分别为中文 英文 |
| customData | object  | 否  | {} | 自定义数据 |
| tipsMessage.sliderPlaceholder | string  | 否  | 向右滑动滑块填充拼图 | 自定义滑块样式，仅滑块验证支持自定义设置 |
| disabled | boolean | 否  | false | 初始状态是否禁用 |
| width | string  | 否  | 100% | 验证码宽度，单位支持rpx、%、px。验证码最小宽度200px，最大宽度600px |
