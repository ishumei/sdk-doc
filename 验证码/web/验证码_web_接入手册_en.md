## Verification Code Web SDK Integration Manual

### Browser Compatibility Description

Chrome, Firefox, Safari, Opera, IE9 (including IE9)+, mainstream mobile browsers, embedded Webview on iOS and Android.

### Detailed Steps to Embed SDK

**1. Import js**

Import the latest js entry file, version: 1.0.4

```js
    // Configuration for connecting to the Beijing data center, used by default for customers without special overseas business
    <script src="https://castatic.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>

    // Special configuration for connecting to the Singapore data center, only for customers reporting verification code data to the Singapore data center
    // <script src="https://castatic-xjp.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>

    // Special configuration for connecting to the US data center, only for customers reporting verification code data to the US data center
    // <script src="https://castatic-fjny.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>
```

**2. Initialization Method**
```js
    initSMCaptcha({
        organization: 'xxxx', // Company identifier viewable in the backend of Shumei

        // Special configuration for connecting to the Singapore data center, only for customers reporting verification code data to the Singapore data center
        // domains: ["captcha-xjp.fengkongcloud.cn"],

        // Special configuration for connecting to the US data center, only for customers reporting verification code data to the US data center
        // domains: ["captcha-fjny.fengkongcloud.cn"],
        product: 'popup', // Display form, specific options are shown in the table below
        mode: 'slide', // Verification code mode, specific options are shown in the table below
        appendTo: 'captchaId', // ID of the verification code DOM element
    }, function (instance) {
        // Verification code instance, methods on the instance can be called
    });
```

The startup parameters are shown in the table below:

| **Field** | **Parameter Type**  | **Required** | **Default Value** | **Field Description** |
| -- | -- | -- | -- | -- |
| organization | string | Yes | None | Company identifier assigned by Shumei, viewable in the backend of Shumei |
| appId | string | No | default | Application identifier, used to distinguish different applications, manageable in the backend of Shumei |
| channel | string | No | default | Promotion channel, customizable |
| product | string | No | embed (embedded) | Display form, supports: embed (embedded), float (floating), popup (popup) |
| mode | string  | No | slide | Mode: supports slide (sliding verification code), auto_slide (no image sliding verification code), select (text selection verification code), icon_select (icon selection verification code), seq_select (idiom sequence verification code), spatial_select (spatial logic) |
| appendTo  | string  | Yes | None  | ID of the verification code DOM element. Must be configured for embed (embedded) and float (floating), not needed for popup (popup) |
| lang | string | No | zh-cn (Simplified Chinese) | Mode: supports zh-cn (Simplified Chinese), en (English), ph (Filipino), ina (Indonesian), tha (Thai), vn (Vietnamese), mys (Malay), jp (Japanese), kr (Korean), Spanish (es), Bengali (bn), Portuguese (pt), German (de), French (fr), Hindi (hi), Italian (it), Urdu (ur), Russian (ru), Swedish (sv), Turkish (tr), Traditional Chinese (zh-tw), Arabic (ar). Note: In idiom sequence and text selection modes, the verification code image content defaults to Chinese and does not change with the language. |
| useBrowserLang | boolean | No | false | Whether to prioritize using the browser's language setting as the verification code language |
| customData | object | No | {} | Custom data |
| tipsMessage.sliderPlaceholder | string | No | Slide the slider to fill the puzzle | Custom default text for the slider, only supported in slider verification |
| disabled  | boolean | No | false | Whether the initial state disables the verification code |
| https | boolean | No | true| Whether to load resources via https |
| width | number or string | No | 100% | Verification code width, supports numbers, %, px. The recommended minimum width is 300px, and the aspect ratio is recommended to be 2:1 |
| debug | boolean | No | false | Can be enabled during development to print logs for debugging; must be turned off after going live |
| maskBindClose | boolean | Yes | true | Effective in popup mode, whether the popup can be closed by clicking the mask layer |
| onError | function | No | None | Listener for configuration fetch interface exceptions, parameters: errType, errMsg |
| onInit| function| No | None | Callback when the verification code starts loading, with a parameter captchaUuid |
| style | object | No | {} | Supports custom styles and settings, detailed configuration in 3.12 |
| captchaUuid | string | No | 32-character random string | Supports passing a 32-character random string from the business side, used in conjunction with the business side's own buried point data, facilitating subsequent problem location or data statistics |

**3. Instance Methods**

3.1 appendTo, where to load the verification code on the page, effective for product as embed and float, recommended to be directly written in the configuration's appendTo
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // Verification code instance, methods on the instance can be called
    });
```

3.2 bindForm, insert the verification result into the FORM form on the page
```js
    initSMCaptcha({
      organization: 'xxxx', // Viewable in the backend of Shumei
      appendTo: captchaId,
    }, function (SMCaptcha) {
        // Bind the result to the form on the page, $form is a DOM object
        SMCaptcha.bindForm($form);
    });
```

3.3 getValidate/getResult, used in the success callback to get the result. If the verification code is correct, proceed with the subsequent business logic; if the verification fails, provide a prompt.
```js
     initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
     }, function(SMCaptcha){
        SMCaptcha.onSuccess(function (data) {
            // {rid: "2017080810405655b3377d25de478233", pass: false} var data1 = instance.getResult();
            var data2 = instance.getValidate();
        });
     });
```
3.4 onReady, callback after all verification code resources (js, css, and images) are loaded,
returns the type field to distinguish the three different cases of successful loading: init: after startup, refresh: after manually refreshing the image, afterFail: after verification failure
```js
    initSMCaptcha({ 
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;

        // After all resources are successfully loaded
        SMCaptcha.onReady(function (rootEl, params) {
            // params contains the type field, 'init'-after startup | 'refresh'-after manually refreshing the image | 'afterFail'-after verification failure
            console.log('onReady', params);
        });
    });
```
3.5 onSuccess, callback after front-end verification
```js
    initSMCaptcha({ 
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;
        // Callback for verification code verification status
        SMCaptcha.onSuccess(function (data) {
            // data format: {rid: '2017080810405655b3377d25de478233', pass: false}
            console.log(data);
            if (data.pass) {
                // After verification is passed
            }
            else {
                // After verification fails
            }
        });
    });
```
3.6 onError, callback for registration, verification, resource loading, and other exceptions
```js
    initSMCaptcha({ 
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;
        // Listen for verification code exceptions, recommended to listen during development to facilitate error discovery, and for the business side to log errors in production
        SMCaptcha.onError(function (errType, errMsg) {
            // errType format: 'SERVER_ERROR'
            // errMsg format: {code: 2002, message: 'Server exception: unauthorized operation (invalid organization)'}
            console.log('onError', errType, errMsg);
        });
    });
```
3.7 verify, effective when the reference method is popup, call the verification code popup
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // Call the popup verification code SMCaptcha.verify() at a specific time as needed
        // For example, call it after clicking a button to open the verification popup
    });
```
3.8 onClose, effective when the reference method is popup, callback for closing the verification code popup
```js
     initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // Listen for popup closure
        SMCaptcha.onClose(function () {
            console.log('Verification code popup closed');
        });
    });
```
3.9 disableCaptcha, some scenarios require actively disabling the use of the verification code
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.disableCaptcha();
    });
```
3.10 enableCaptcha, some scenarios require actively enabling the use of the verification code
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.enableCaptcha();
    });
```
3.11 reset, some scenarios require actively resetting the verification code
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.reset();
    });
```
3.12 Custom styles, note: currently only supports modes slide and auto_slide, see the comments below for details
```js
    initSMCaptcha({
        organization: 'xxxx', // Viewable in the backend of Shumei
        appendTo: captchaId,
        style: {
            // Custom font
            customFont: {
                name: 'Walsheim', // Font name
                // This font file must be maintained by the user, must be an absolute address, must support https, and must set CORS to support cross-domain
                url: 'https://castatic.fengkongcloud.cn/pr/v1.0.4/assets/GT-Walsheim-Pro-Bold.ttf', 
            },
            fontFamily: '', // Custom browser-supported font
            fontWeight: '', // Set custom font weight
            headerTitle: 'Vertification1', // Custom title at the top
            hideRefreshOnImage: true, // Whether to hide the refresh button on the image
            slideBar: {
                color: '#141C30', // Default font color on the progress bar
                background: '#F3F7FA', // Default background color of the progress bar
                successColor: '#FFF', // Font color when successful
                border: 'none', // Border of the drag progress bar, generally used to clear the border
                process: {
                    background: '#F3F8CE', // Default background color of the progress bar
                    successBackground: '#25BC73', // Background color when successful
                    failBackground: '#F5E0E2' // Background color when failed
                },
                button: {
                    boxShadow: 'none', // Used to remove the shadow of the button
                    color: '#333', // Default color in the button
                    successColor: '#FFF' // Font color when successful
                    failColor: '#FFF' // Font color when verification fails
                    background: '#F6FF7E', // Default button background color
                    successBackground: '#25BC73', // Button background color when successful
                    failBackground: '#ED816E' // Button background color when failed
                }
            }
        }
    }, function (instance) {
        // ...
    });
```
3.13) The startup parameters support passing the field captchaUuid
The startup parameters support passing the field captchaUuid (32-character random string), used to associate the buried point logs on the Shumei side with the buried point logs on the business side, which is useful for subsequent problem location or data alignment. See the example code below.
```js
initSMCaptcha({ 
    organization: 'xxxx', // Viewable in the backend of Shumei
    appendTo: captchaId,
    captchaUuid: '20220512203521NAdPbGRptrBEFd8cFd', // 32-character random string generated by the business side, if not passed, the verification code will also generate one internally
    onInit(captchaUuid) {
        // Callback when the verification code starts loading, the business side can get the captchaUuid of the current lifecycle of the verification code "at the first time"
        console.log('Received', captchaUuid);
    }
}, function (SMCaptcha) {
    // The SMCaptcha object has a captchaUuid attribute
    const captchaUuid = SMCaptcha.captchaUuid;

    // After all resources are successfully loaded
    SMCaptcha.onReady(function (dom, params) {
        // params has a type field, 'init'-after startup | 'refresh'-after manually refreshing the image | 'afterFail'-after verification failure
        console.log('onReady', params);

        // The business side can log by itself
        fetch('https://.../api/log', {
            method: 'post',
            body: JSON.stringify({
                action: 'smCaptchaReady',
                content: {
                    type: params.type,
                    captchaUuid: captchaUuid, // Send the uuid to the business side's buried point interface
                    userId: userId // Some other information that needs to be logged on the business side, for example
                }
            }),
            headers: {
                'Content-Type': 'application/json'
            }
        });
    });

    // After verification code resources fail to load
    SMCaptcha.onError(function (errType, errMsg) {
        // errType format: 'SERVER_ERROR'
        // errMsg format: {code: 2002, message: 'Server exception: unauthorized operation (invalid organization)'}
        console.log('onError', errType, errMsg);

        // The business side can log by itself
        fetch('https://.../api/log', {
            method: 'post',
            body: JSON.stringify({
                action: 'smCaptchaError',
                content: {
                    errType: errType,
                    errMsg: errMsg,
                    captchaUuid: captchaUuid,
                    userId: userId // Some other information that needs to be logged on the business side, for example
                }
            }),
            headers: {
                'Content-Type': 'application/json'
            }
        });
    });
});
```

**4. Exception Code Description**

| Exception Code | Remarks |
| ------ | ---------- |
| 2001   | Resource exception   |
| 2002   | Server exception |
| 2003   | Parameter exception   |
| 2004   | SDK startup exception |
| 2005   | Network timeout   |

**5. Complete Code Example**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Shumei Sliding Verification Code DEMO</title>
    <!-- The following is just the demo style, no need to import the following css for formal integration -->
    <link rel="stylesheet" href="https://castatic.fengkongcloud.cn/pr/v1.0.4/style/demo.css">
</head>
<body>
<form class="shumei_captcha_demo_form" id="shumei_captcha_form">
    <div class="shumei_form_item">
        <label>Username</label>
        <input type="text" name="account" placeholder="Please enter your account" />
    </div>

    <div class="shumei_form_item">
        <label>Password</label>
        <input type="text" name="password" placeholder="Please enter your password" />
    </div>
    <div class="shumei_form_item">
        <label style="vertical-align: top">Verification Code</label>
        <span id="shumei_form_captcha_wrapper">Sliding verification code loading...</span>
    </div>
    <div class="shumei_form_item">
        <label></label>
        <a class="shumei_login_btn" href="###">Login Now</a>
    </div>
</form>
</body>
<!-- For demo operation convenience, the verification code SDK does not depend on any third-party libraries -->
<script src="http://apps.bdimg.com/libs/jquery/1.9.0/jquery.js"></script>
<!-- Core entry js of the verification code, web verification code only needs to import the following one js, the current latest version is 1.0.4 -->
<script src="https://castatic.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>
<script>
    $(function () {
        var $body = $('body');
        var $form = $body.find('#shumei_captcha_form');
        var captchaId = 'shumei_form_captcha_wrapper';
        var $subBtn = $body.find('.shumei_login_btn');

        /**
         *      Shumei verification code callback
         *      @param SMCaptcha
         */
        function smCaptchaCallback(SMCaptcha) {
            // The uuid of the current lifecycle, it is recommended to add this field when logging on the business side, so that the problem can be located through the log query
            var captchaUuid = SMC