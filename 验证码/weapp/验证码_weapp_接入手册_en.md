## Verification Code weapp-sdk Integration Manual

### Supported Mini Programs

Supports WeChat Mini Program, QQ Mini Program, Toutiao Mini Program, Alipay Mini Program, and also supports Taro framework integration for various versions of the verification code component. This document introduces the WeChat plugin integration method. For other mini program integration methods, please contact the Shumei technical support team.

### 1. Apply for Mini Program Plugin on the following page

https://mp.weixin.qq.com/wxopen/plugindevdoc?appid=wxfb52904a0e24dc20&token=&lang=zh_CN

After the application is completed, please notify Shumei and wait for Shumei's approval. After the approval is completed, follow the process below to integrate the SDK.

### 2. Start Mini Program Verification Code
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

// pages/index/index.json This refers to the page configuration file that needs to use the verification code component
{
    "usingComponents": {
        "sm-captcha": "plugin://smCaptcha/captcha"
    }
}

// pages/index/index.js
const plugin = requirePlugin("smCaptcha");

Page({
    data: {
        options: { // For specific configuration, please refer to the config parameter configuration
            organization: 'xxx', // Company identifier, required
            tipsMessage: { // Optional
                slidePlaceholder: 'Custom slide prompt text'
            },
            product: 'popup',
            mode: 'slide',
        }
    },

    onCaptchaReady() { // Triggered when the verification code resource is ready
        console.log('Verification code is ready');
    },

    onSuccess(e) { // Triggered when verification is successful
        console.log('Verification successful callback', e.detail);
    },

    onFailure(e) { // Triggered when verification fails
        console.log('Verification failed callback', e.detail);
    },

    onError(e) { // Listen for error events
        console.log('An error occurred', e.detail);
        // If e.detail.isValid === false, it means that the basic library version is lower than 2.1.0 and is not compatible
    },

    onClose() { // Effective when the reference method is popup, listen for verification code closure
        console.log('Listen for verification code closure');
    },

    resetCaptcha() { // Some scenarios require starting the verification code
        plugin.reset();
    },

    disableCaptcha() { // Some scenarios require actively disabling the use of the verification code
        plugin.disableCaptcha();
    },

    enableCaptcha() { // Some scenarios require actively enabling the use of the verification code
        plugin.enableCaptcha()
    },

    showCaptcha() { // Effective when the reference method is popup, call the verification code popup
        plugin.verify();
    },

    getResult() { // Actively obtain the verification result, such as: the verification code is correct and can proceed with subsequent logic, and prompt if the verification fails
        const result = plugin.getResult();
        console.log('Actively obtain verification result', result);
    },
});


// pages/index/index.wxml Page code
<view class="btn-box">
    <!--showCaptcha method is effective when product is 'popup'-->
    <button wx:if="{{options.product==='popup'}}" bindtap="showCaptcha">Show Verification Code</button>
    <button id="add" bindtap="resetCaptcha">Refresh</button>
    <button bindtap="enableCaptcha">Enable</button>
    <button bindtap="disableCaptcha">Disable</button>
    <button bindtap="getResult">Get Verification Result</button>
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

The parameter configuration is as follows:

| **Field** | **Parameter Type** | **Required** | **Default Value** | **Remarks** |
| -- | -- | -- | -- | -- |
| organization | string  | Yes  | None | Company identifier assigned by Shumei, can be viewed in the Shumei backend |
| appId | string  | No  | default| Application identifier, distinguishes different applications, can be managed in the Shumei backend  |
| channel | string  | No  | default | Promotion channel, customizable |
| product | string  | No  | embed (embedded) | Display form, supports: embed (embedded), popup (popup layer) |
| mode | string  | No  | slide  | Mode: supports slide (slide verification code), select (text selection verification code), icon_select (icon selection verification code), seq_select (idiom sequence verification code), spatial_select (spatial logic) |
| lang | string  | No  | zh-cn  | Mode: supports zh-cn en, respectively Chinese and English. Note: In idiom sequence and text selection modes, the verification code image content defaults to Chinese and does not change with the language. |
| customData | object  | No  | {} | Custom data |
| tipsMessage.slidePlaceholder | string  | No  | Slide the slider to fill the puzzle | Custom slider style, only slide verification supports custom settings |
| disabled | boolean | No  | false | Initial state whether to disable |
| width | string  | No  | 100% | Verification code width, units support rpx, %, px. The minimum width of the verification code is 200px, and the maximum width is 600px |