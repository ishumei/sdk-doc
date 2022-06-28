## 验证码web-sdk接入手册

### 浏览器兼容性说明

Chrome、Firefox、Safari、Opera、IE9(包含IE9)+、主流手机浏览器、iOS 及 Android 上的内嵌 Webview。

### 嵌入SDK的详细步骤

**1. 引入初始化js** 

引入最新版js入口文件，版本是：1.0.4

```js
    // 连接北京机房配置项，无特殊海外业务客户均选用此机房的js
    <script src="https://castatic.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>

    // 连接新加坡机房特殊配置项，仅供验证码数据上报新加坡机房客户使用
    // <script src="https://castatic-xjp.fengkongcloud.com/pr/v1.0.4/smcp.min.js"></script>

    // 连接美国机房特殊配置项，仅供验证码数据上报美国机房客户使用
    // <script src="https://castatic-fjny.fengkongcloud.com/pr/v1.0.4/smcp.min.js"></script>
```

**2. 初始化方法**
```js
    initSMCaptcha({
        organization: 'xxxx', // 数美后台可以查看公司标识

        // 连接新加坡机房特殊配置项，仅供验证码数据上报新加坡机房客户使用
        // domains: ["captcha-xjp.fengkongcloud.com"],

        // 连接美国机房特殊配置项，仅供验证码数据上报美国机房客户使用
        // domains: ["captcha-fjny.fengkongcloud.com"],
        product: 'popup', // 展现形式，具体选项见下表
        mode: 'slide', // 验证码模式，具体选项见下表
        appendTo: 'captchaId', // 验证码dom元素的id，
    }, function (instance) {
        // 验证码实例，可以调用实例上的方法
    });
```

初始化参数如下表所示：

| **字段** | **参数类型**  | **是否必填** | **默认值** | **字段说明** |
| -- | -- | -- | -- | -- |
| organization | string | 是 | 无 | 数美分配的公司标识，数美后台可以查看看 |
| appId | string | 否 | default | 应用标识，区分不同应用，数美后台可以管理 |
| channel | string | 否 | default | 推广渠道，可自定义 |
| product | string | 否 | embed（嵌入式）| 展现形式，支持：embed（嵌入式）、float（浮动式）、popup（弹出层） |
| mode | string  | 否 | slide | 模式：支持slide（滑动验证码）、select（文字点选验证码）、 icon_select（图标点选验证码）、  seq_select（成语语序验证码）、spatial_select（空间逻辑） |
| appendTo  | string  | 是 | 无  | 验证码dom元素的id。embed（嵌入式）、float（浮动式）必须配置，popup（弹出层）不需要 |
| lang | string | 否 | zh-cn（简体中文） | 模式：支持zh-cn（简体中文）、en（英文）、ph（菲律宾语）、ina（印尼语）、tha（泰语）、vn（越南语）、mys（马来语）、jp（日语）及kr(韩语) |
| useBrowserLang | boolean | 否 | false | 是否高优使用浏览器设置的语言作为验证码的语言 |
| customData | object | 否 | {} | 自定义数据 |
| tipsMessage.sliderPlaceholder | string | 否 | 向右滑动滑块填充拼图 | 自定义滑块默认文案,仅滑块验证支持自定义设置 |
| disabled  | boolean | 否 | false | 初始状态是否禁用验证码 |
| https | boolean | 否 | true| 加载资源是否是https |
| width | number或string | 否 | 100% | 验证码宽度，单位支持数字、%、px，建议最小宽度为300px，宽高比建议为2:1 |
| debug | boolean | 否 | false | 开发过程中可以开启，出现异常会打印log，方便开发；上线后，务必关闭 |
| maskBindClose | boolean | 是 | true | popup模式有效，是否可以通过点击遮罩层来关闭弹框 |
| onError | function | 否 | 无 | 监听配置获取接口异常，参数： errType, errMsg |
| onInit| function| 否 | 无 | 开始加载验证码的回调，其中有个参数是captchaUuid |
| style | object | 否 | {} | 支持自定义样式及风格设置，详细配置请见3.12 |
| captchaUuid | string | 否 | 32位随机字符串 | 支持业务侧传入一个32位随机字符串，与业务方自身埋点数据配合，便于后续定位问题或进行数据统计 |


**3. 实例方法**

3.1 appendTo，验证码加载到页面什么位置，product 为 embed 和 float 有效，建议直接写在配置里的 appendTo
```js
    initSMCaptcha({
        organization: 'xxxx', // 数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // 验证码实例，可以调用实例上的方法
    });
```

3.2 bindForm，将验证结果插入到页面 FORM 表单内
```js
    initSMCaptcha({
      organization: 'xxxx', //数美后台查看
      appendTo: captchaId,
    }, function (SMCaptcha) {
        // 将结果绑定到页面的 form 中, $form为dom对象
        SMCaptcha.bindForm($form);
    });
```

3.3 getValidate/getResult，在成功回调中使用，可以获得结果，验证码正确，可以走业务后续逻辑；验证失败做提示。
```js
     initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
     }, function(SMCaptcha){
        SMCaptcha.onSuccess(function (data) {
            // {rid: "2017080810405655b3377d25de478233", pass: false} var data1 = instance.getResult();
            var data2 = instance.getValidate();
        });
     });
```
3.4 onReady，验证码所有资源（js、css及图片）加载就绪后的回调，
返回type字段用以区分3种不同情况下的加载成功，init：初始化后 refresh：手动刷新图片后 afterFail：验证失败后
```js
    initSMCaptcha({ 
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;

        // 全部资源加载成功后
        SMCaptcha.onReady(function (rootEl, params) {
            // params中包含type字段，'init'-初始化后 | 'refresh'-手动刷新图片后 | 'afterFail'-验证失败后
            console.log('onReady', params);
        });
    });
```
3.5 onSuccess，前端验证后的回调
```js
    initSMCaptcha({ 
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;
        // 验证码校验情况回调
        SMCaptcha.onSuccess(function (data) {
            // data格式：{rid: '2017080810405655b3377d25de478233', pass: false}
            console.log(data);
            if (data.pass) {
                // 验证通过后
            }
            else {
                // 验证失败后
            }
        });
    });
```
3.6 onError，注册、校验、资源加载等异常回调
```js
    initSMCaptcha({ 
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId
    }, function (SMCaptcha) {
        const captchaUuid = SMCaptcha.captchaUuid;
        // 监听验证码异常情况，建议开发时监听便于发现错误，生产环境监听后业务端自行埋点记录错误情况
        SMCaptcha.onError(function (errType, errMsg) {
            // errType格式: 'SERVER_ERROR'
            // errMsg格式: {code: 2002, message: '服务器异常 : 无权限操作 (invalid organization)'}
            console.log('onError', errType, errMsg);
        });
    });
```
3.7 verify，当引用方式是popup时有效，调用验证码弹窗
```js
    initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // 根据自己的需要，特定的时机调用弹窗验证码SMCaptcha.verify()
        // 比如在点击某个按钮后调用，打开验证弹框
    });
```
3.8 onClose，当引用方式是popup时有效，关闭验证码弹窗回调
```js
     initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        // 监听弹窗关闭
        SMCaptcha.onClose(function () {
            console.log('关闭验证码弹框了');
        });
    });
```
3.9 disableCaptcha，某些场景需要主动禁用验证码使用
```js
    initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.disableCaptcha();
    });
```
3.10 enableCaptcha，某些场景需要主动启用验证码使用
```js
    initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.enableCaptcha();
    });
```
3.11 reset某些场景需要主动重置验证码使用
```js
    initSMCaptcha({
        organization: 'xxxx', // 数美后台查看
        appendTo: captchaId,
    }, function (SMCaptcha) {
        SMCaptcha.reset();
    });
```
3.12 自定义样式，注意：当时只支持mode是slide两种模式，具体详见下面注释说明
```js
    initSMCaptcha({
        organization: 'xxxx', //数美后台查看
        appendTo: captchaId,
        style: {
            // 自定义字体
            customFont: {
                name: 'Walsheim', // 字体名
                // 此字体文件须由使用方维护，必须是绝对地址，必须支持https，且必须设置CORS来支持跨域
                url: 'https://castatic.fengkongcloud.cn/pr/v1.0.4/assets/GT-Walsheim-Pro-Bold.ttf', 
            },
            fontFamily: '', // 自定义浏览器支持的字体
            fontWeight: '', // 设置自定义字体粗细
            headerTitle: 'Vertification1', // 顶部自定义标题
            hideRefreshOnImage: true, // 是否隐藏图片上的刷新按钮
            slideBar: {
                color: '#141C30', // 进度条上默认字体颜色
                background: '#F3F7FA', // 进度条默认背景色
                successColor: '#FFF', // 成功时字体颜色
                border: 'none', // 拖动进度条边框，一般用来清除边框
                process: {
                    background: '#F3F8CE', // 默认进度条背景色
                    successBackground: '#25BC73', // 成功时背景色
                    failBackground: '#F5E0E2' // 失败时背景色
                },
                button: {
                    boxShadow: 'none', // 用来去除按钮的阴影
                    color: '#333', // 按钮中的默认颜色
                    successColor: '#FFF' // 成功时字体颜色
                    failColor: '#FFF' // 验证失败时字体颜色
                    background: '#F6FF7E', // 默认按钮背景色
                    successBackground: '#25BC73', // 成功时按钮背景色
                    failBackground: '#ED816E' // 失败时按钮背景色
                }
            }
        }
    }, function (instance) {
        // ...
    });
```
3.13） 初始化参数中支持传入字段captchaUuid
初始化参数中支持传入字段captchaUuid（32位随机字符串），用来将数美侧的埋点日志与业务端的埋点日志关联上，后续无论是定位问题或是对齐埋点数据都是有用的。详见下述示例代码。
```js
initSMCaptcha({ 
    organization: 'xxxx', //数美后台查看
    appendTo: captchaId,
    captchaUuid: '20220512203521NAdPbGRptrBEFd8cFd', // 业务端生成的32位随机字符串，如果不传则验证码内部也会生成
    onInit(captchaUuid) {
        // 开始加载验证码的回调，业务端可以“第一时间”获取到验证码当前生命周期的captchaUuid，
        console.log('收到', captchaUuid);
    }
}, function (SMCaptcha) {
    // SMCaptcha对象上有captchaUuid属性
    const captchaUuid = SMCaptcha.captchaUuid;

    // 全部资源加载成功后
    SMCaptcha.onReady(function (dom, params) {
        // params中有type字段，'init'-初始化后 | 'refresh'-手动刷新图片后 | 'afterFail'-验证失败后
        console.log('onReady', params);

        // 业务端可以自行埋点
        fetch('https://.../api/log', {
            method: 'post',
            body: JSON.stringify({
                action: 'smCaptchaReady',
                content: {
                    type: params.type,
                    captchaUuid: captchaUuid, // 将uuid发到业务侧埋点接口中
                    userId: userId // 业务侧其他一些需要埋点的信息举例
                }
            }),
            headers: {
                'Content-Type': 'application/json'
            }
        });
    });

    // 验证码资源加载出错后
    SMCaptcha.onError(function (errType, errMsg) {
        // errType格式: 'SERVER_ERROR'
        // errMsg格式: {code: 2002, message: '服务器异常 : 无权限操作 (invalid organization)'}
        console.log('onError', errType, errMsg);

        // 业务端可以自行埋点
        fetch('https://.../api/log', {
            method: 'post',
            body: JSON.stringify({
                action: 'smCaptchaError',
                content: {
                    errType: errType,
                    errMsg: errMsg,
                    captchaUuid: captchaUuid,
                    userId: userId // 业务侧其他一些需要埋点的信息举例
                }
            }),
            headers: {
                'Content-Type': 'application/json'
            }
        });
    });
});
```

**4. 异常码说明**

| 异常码 | 备注|
| ------ | ---------- |
| 2001   | 资源异常   |
| 2002   | 服务器异常 |
| 2003   | 参数异常   |
| 2004   | 初始化异常 |
| 2005   | 网络超时   |

**5. 完整代码示例**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>数美滑动验证码DEMO</title>
    <!-- 下面这个只是demo的样式，正式接入无需引入下面这个css -->
    <link rel="stylesheet" href="https://castatic.fengkongcloud.cn/pr/v1.0.4/style/demo.css">
</head>
<body>
<form class="shumei_captcha_demo_form" id="shumei_captcha_form">
    <div class="shumei_form_item">
        <label>用户名</label>
        <input type="text" name="account" placeholder="请输入您的账号" />
    </div>

    <div class="shumei_form_item">
        <label>密码</label>
        <input type="text" name="password" placeholder="请输入您的密码" />
    </div>
    <div class="shumei_form_item">
        <label style="vertical-align: top">验证码</label>
        <span id="shumei_form_captcha_wrapper">滑动验证码加载中...</span>
    </div>
    <div class="shumei_form_item">
        <label></label>
        <a class="shumei_login_btn" href="###">立即登录</a>
    </div>
</form>
</body>
<!-- demo 操作方便使用，验证码 SDK 不依赖任何第三方库 -->
<script src="http://apps.bdimg.com/libs/jquery/1.9.0/jquery.js"></script>
<!-- 验证码核心入口js，web验证码只需要引入下面一个js，当前最新版本是1.0.4 -->
<script src="https://castatic.fengkongcloud.cn/pr/v1.0.4/smcp.min.js"></script>
<script>
    $(function () {
        var $body = $('body');
        var $form = $body.find('#shumei_captcha_form');
        var captchaId = 'shumei_form_captcha_wrapper';
        var $subBtn = $body.find('.shumei_login_btn');

        /**
         *      数美验证码回调
         *      @param SMCaptcha
         */
        function smCaptchaCallback(SMCaptcha) {
            // 当前生命周期的uuid，业务端埋点的话建议加上这个字段，到时可以根据这个字段通过日志查询定位问题
            var captchaUuid = SMCaptcha.captchaUuid;
            
            // 将验证结果插入到页面form表单内
            SMCaptcha.bindForm($form[0]);

            // 全部资源加载成功后
            SMCaptcha.onReady(function (dom, params) {
                // params中有type字段，'init'-初始化后 | 'refresh'-手动刷新图片后 | 'afterFail'-验证失败后
                console.log('onReady', params);
            });

            // 验证码校验情况回调
            SMCaptcha.onSuccess(function (data) {
                // data格式：{rid: "2017080810405655b3377d25de478233", pass: false}
                console.log(data);
                if (data.pass) {
                    // 验证通过后
                }
                else {
                    // 验证失败后
                }
            });
            
            // 资源加载异常,建议开发时进行监听,便于查询错误
            SMCaptcha.onError(function (errType, errMsg) {
                // errType格式: 'SERVER_ERROR'
                // errMsg格式: {code: 2002, message: " 服 务 器 异 常 : 无 权 限 操 作 (invalid organization)"}
                console.log('onError', errType, errMsg);
            });

            SMCaptcha.onClose(function () {
                console.log('关闭弹框了');
            });

            // 监听按钮
            $subBtn.on('click', function (e) {
                e.preventDefault();
                var account = $body.find('[name=account]').val();
                var password = $body.find('[name=password]').val();
                var result = SMCaptcha.getResult();

                var rid = result.rid;
                var pass = result.pass;

                if (pass) {
                    console.log('账号: ' + account + '\n 密码: ' + password + '\n 验证码: ' + rid);
                }
                else {
                    console.log('请选择验证码');
                }
            });
        }

        // 初始化验证码
        initSMCaptcha({
            organization: 'xxx',
            appendTo: captchaId
        }, smCaptchaCallback);
    });
</script>
</html>

```
 

**6. web验证码更多Demo**

在线体验地址：https://castatic.fengkongcloud.cn/pr/v1.0.4/demo.html
