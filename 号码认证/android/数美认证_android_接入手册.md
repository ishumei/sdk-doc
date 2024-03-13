# smauth-demo-android

smauth 手机号一键登录 SDK Demo for Android

## SDK 位置

`Demo/app/libs/smauth*.aar`

## 工程配置

### 导入 SDK 及依赖包

``` java
implementation 'com.google.code.gson:gson:2.8.8'
implementation fileTree(dir: "libs", include: "smauth*.aar")
```

### 配置 AndroidManifest

```xml
<application>
    <activity
            android:name="com.cmic.gen.sdk.view.GenLoginAuthActivity"
            android:configChanges="orientation|keyboardHidden|screenSize"
            android:launchMode="singleTop"
            android:screenOrientation="behind" />
</application>
```



## 启动SDK

``` java
SmAuthHelper.create(context).register(applicationId, listener);
```



## API

### SmAuthHelper

#### 获取SDK version
``` java
public static String sdkVersion()
```

#### 创建SmAuthHelper实例
``` java
public static SmAuthHelper create(Context context)
```

* parameter:
    * context: Android 上下文
* return:
    * SmAuthHelper实例


#### 获取SmAuthHelper实例
``` java
public static SmAuthHelper getInstance()
```

* parameter:
    * -
* return:
    * SmAuthHelper实例

#### 销毁SmAuthHelper实例
``` java
public static void destroy()
```

* parameter:
    * -
* return:
    * -

#### 注册服务
``` java
public void register(SmAuthRegisterListener listener)
```
* parameter:
    * listener: 注册callback
* return:
    * -

``` java
public void register(String applicationId, SmAuthRegisterListener listener)
```
* parameter:
    * applicationId: 控制台上登记APP信息后分配的applicationId
    * listener: 注册callback
* return:
    * -

#### 预取号
``` java
public void preloadAuthorization(SmAuthPreloadListener listener, int requestCode)
```

* parameter:
    * listener: 预取号callback
    * requestCode: 请求码，callback中会返回该请求码
* return:
    * -

#### 自定义授权页UI配置
``` java
public void setAuthThemeConfigure(ThemeConfig theme)
```

* parameter:
    * theme: UI配置项
* return:
    * -

#### 取号授权（弹出授权页）
``` java
public void loginAuth(SmAuthTokenListener listener, int requestCode)
```

* parameter:
    * listener: 取号授权callback
    * requestCode: 请求码，callback中会返回该请求码
* return:
    * -

#### 退出授权页
``` java
public void quitLoginAuth()
```

* parameter:
    * -
* return:
    * -

#### 手机号校验授权
``` java
public void verifyMobile(SmAuthVerifyMobileListener listener, int requestCode)
```

* parameter:
    * listener: 手机号校验授权callback
    * requestCode: 请求码，callback中会返回该请求码
* return:
    * -

#### 获取当前网络状态及运营商信息
``` java
public NetworkInfo getNetworkInfo()
```

* parameter:
    * -
* return:
    * 当前网络状态及运营商信息

#### 设置服务接口超时时间
``` java
public void setTimeout(long overtime)
```

* parameter:
    * overtime: 超时时间（毫秒）
* return:
    * -

### 自定义授权页配置项 (ThemeConfig.Builder)
``` java
// 设置状态栏颜色
public Builder setStatusBar(int statusBarColor, boolean isLightColor)
// 设置自定义ContentView
public Builder setAuthContentView(View contentView)
// 设置手机号码字体大小
public Builder setNumberSize(int numberSize, boolean isBold)
// 设置手机号码字体颜色
public Builder setNumberColor(int numberColor)
// 设置手机号码的横向位置偏移
public Builder setNumberOffsetX(int numberOffsetX)
// 设置手机号码的纵向位置相对于顶部偏移
public Builder setNumFieldOffsetY(int numFieldOffsetY)
// 设置手机号码的纵向位置相对于底部偏移
public Builder setNumFieldOffsetY_B(int numFieldOffsetY_B)
// 设置授权按钮文本
public Builder setLogBtnText(String logBtnText)
// 设置授权按钮文本以及字体信息
public Builder setLogBtnText(String logBtnText, int logBtnTextColor, int logBtnTextSize, boolean isBold)
// 设置授权按钮字体颜色
public Builder setLogBtnTextColor(int logBtnTextColor)
// 设置授权按钮背景资源名称（btn.png就填btn）
public Builder setLogBtnImgPath(String logBtnBackgroundPath)
// 设置授权按钮尺寸
public Builder setLogBtn(int width, int height)
// 设置授权按钮横向margin
public Builder setLogBtnMargin(int marginLeft, int marginRight)
// 设置授权按钮的纵向位置相对于顶部偏移
public Builder setLogBtnOffsetY(int logBtnOffsetY)
// 设置授权按钮的纵向位置相对于底部偏移
public Builder setLogBtnOffsetY_B(int logBtnOffsetY_B)
// 设置协议勾选框未选中时的提示文本
public Builder setCheckTipText(String checkTipText)
// 设置协议勾选框选中状态图片资源名称
public Builder setCheckedImgPath(String checkedImgPath)
// 设置协议勾选框未选中状态图片资源名称
public Builder setUncheckedImgPath(String uncheckedImgPath)
// 设置协议勾选框详情
public Builder setCheckBoxImgPath(String checkedImgPath, String uncheckedImgPath, int width, int height)
// 设置协议默认勾选状态
public Builder setPrivacyState(boolean privacyState)
// 设置协议字体颜色
public Builder setClauseColor(int clauseBaseColor, int clauseColor)
// 设置协议横向margin
public Builder setPrivacyMargin(int privacyMarginLeft, int privacyMarginRight)
// 设置协议纵向相对于顶部的偏移
public Builder setPrivacyOffsetY(int privacyOffsetY)
// 设置协议纵向相对于底部的偏移
public Builder setPrivacyOffsetY_B(int privacyOffsetY_B)
// 设置协议名称是否显示书名号
public Builder setPrivacyBookSymbol(boolean haveBookSymbol)
// 设置复选框相对右侧协议文案居上或者居中，默认居上。0-居上，1-居中
public Builder setCheckBoxLocation(int checkBoxLocation)
// 设置协议政策勾选框的勾选状态切换回调
public Builder setPrivacyCheckedChangeListener(SmAuthPrivacyCheckedChangeListener listener)
// 设置授权页的进场动画
public Builder setAuthPageActIn(String authPageActIn, String activityOut)
// 设置授权页的退场动画
public Builder setAuthPageActOut(String activityIn, String authPageActOut)
// 设置授权页窗口宽高比例
public Builder setAuthPageWindowMode(int windowWidth, int windowHeight)
// 设置授权页窗口X轴Y轴偏移
public Builder setAuthPageWindowOffset(int windowX, int windowY)
// 设置授权页是否居于底部，0=居中；1=底部，设置为1Y轴的偏移失效
public Builder setWindowBottom(int windowBottom)
// 设置授权页弹窗主题，也可在Manifest设置
public Builder setThemeId(int themeId)
// 0.中文简体1.中文繁体2.英文
public Builder setAppLanguageType(int appLanguageType)
/** 开启安卓底部导航栏自适应，开启后，导航栏唤起时，授权页面元素也会相对变化；
  * 不开启自适应，自定义内容可以铺满全屏，设置状态栏透明后，可以达到沉浸式显示效果。
  * 0-开启自适应，1-关闭自适应，默认开启。
**/
public Builder setFitsSystemWindows(boolean isFitsSystemWindows)
```

### SmAuthRegisterListener

``` java
// 注册完成
void onRegisted(boolean isSuccess, Exception exception);
```

### SmAuthPreloadListener

``` java
// 预取号完成
void onPreloaded(int requestCode, PreloadResultBean token);
// 预取号失败
void onPreloadFailed(int requestCode, Exception exception);
```

### SmAuthTokenListener

``` java
// 获取授权Token完成
void onGetToken(int requestCode, TokenBean token);
// 获取授权Token失败
void onGetTokenFailed(int requestCode, Exception exception);
```

### SmAuthVerifyMobileListener

``` java
// 校验完成
void onVerified(int requestCode, VerifyMobileBean result);
// 校验失败
void onVerifiedFailed(int requestCode, Exception exception);
```

## 运营商SDK错误码

> 在调用号码认证服务一键登录和本机号码校验功能的接口时，会返回运营商对应接口的错误码。常见接口错误码、错误描述，可参考下述

### 中国移动

| 状态码 | 状态码描述                                                                                                                                                                                                                                                                                                      |
| ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 103000 | 成功                                                                                                                                                                                                                                                                                                            |
| 102101 | 无网络                                                                                                                                                                                                                                                                                                          |
| 102102 | 网络异常(网络请求出错，一般出现在网络安全策略限制了不能使用http、设备开启了代理或连接的WiFi有网络链接安全策略限制的场景，建议结合SDK日志具体分析)                                                                                                                                                               |
| 102103 | 未开启数据网络                                                                                                                                                                                                                                                                                                  |
| 102121 | 用户取消登录                                                                                                                                                                                                                                                                                                    |
| 102203 | 输入参数错误                                                                                                                                                                                                                                                                                                    |
| 102223 | 数据解析异常                                                                                                                                                                                                                                                                                                    |
| 102507 | 登录超时（授权页点登录按钮时）                                                                                                                                                                                                                                                                                  |
| 102508 | 数据网络切换失败                                                                                                                                                                                                                                                                                                |
| 103101 | 请求签名错误(请求签名错误（若发生在客户端，可能是appkey传错，可检查是否跟appsecret弄混，或者有空格。若发生在服务端接口，需要检查验签方式是MD5还是RSA，如果是MD5，则排查signType字段，若为appsecret，需确认是否误用了appkey生签。如果是RSA，需要检查使用的私钥跟报备的公钥是否对应和报文拼接是否符合文档要求。） |
| 103102 | 包签名Bundle ID错误（报备的和实际使用的对不上）                                                                                                                                                                                                                                                                |
| 103103 | 用户不存在                                                                                                                                                                                                                                                                                                      |
| 103111 | 网关IP错误（检查是否开了vpn或者境外ip）                                                                                                                                                                                                                                                                         |
| 103119 | AppID不存在。（检查传的appid是否正确或是否有空格）                                                                                                                                                                                                                                                              |
| 103211 | 其他错误（常见于报文格式不对，先请检查是否符合这三个要求：a、json形式的报文交互必须是标准的json格式；b、发送时请设置content type为`application/json`；c、参数类型都是String                                                                                                                                       |
| 103412 | 无效的请求有加密方式错误、非JSON格式和空请求等                                                                                                                                                                                                                                                                  |
| 103414 | 参数校验异常                                                                                                                                                                                                                                                                                                    |
| 103511 | 服务器IP白名单校验失败                                                                                                                                                                                                                                                                                          |
| 103811 | Token为空                                                                                                                                                                                                                                                                                                       |
| 103902 | 短时间内重复登录，造成script失效                                                                                                                                                                                                                                                                                |
| 103911 | Token请求过于频繁，10分钟内获取Token且未使用的数量不超过30个                                                                                                                                                                                                                                                    |
| 104201 | Token重复校验失败、失效或不存在                                                                                                                                                                                                                                                                                 |
| 105002 | 移动取号失败（一般是物联网卡）                                                                                                                                                                                                                                                                                  |
| 105013 | 不支持联通取号                                                                                                                                                                                                                                                                                                  |
| 105018 | 使用了本机号码校验的Token获取号码，导致Token权限不足                                                                                                                                                                                                                                                            |
| 105019 | 应用未授权                                                                                                                                                                                                                                                                                                      |
| 105021 | 已达当天取号限额                                                                                                                                                                                                                                                                                                |
| 105302 | AppID不在白名单                                                                                                                                                                                                                                                                                                 |
| 105313 | 非法请求                                                                                                                                                                                                                                                                                                        |
| 200002 | 手机未安装sim卡                                                                                                                                                                                                                                                                                                 |
| 200005 | 用户未授权（READ_PHONE_STATE）                                                                                                                                                                                                                                                                                  |
| 200006 | 用户未授权（SEND_SMS）                                                                                                                                                                                                                                                                                          |
| 200007 | authType仅使用短信验证码认证                                                                                                                                                                                                                                                                                    |
| 200008 | 1. authType参数为空；2. authType参数不合法；                                                                                                                                                                                                                                                                    |
| 200009 | 应用合法性校验失败（包名包签名未填写正确）                                                                                                                                                                                                                                                                      |
| 200010 | 无法识别SIM卡或没有SIM卡（Android）                                                                                                                                                                                                                                                                             |
| 200012 | 取号失败，跳短信验证码登录                                                                                                                                                                                                                                                                                      |
| 200013 | 短信上行发送短信失败（短信上行）                                                                                                                                                                                                                                                                                |
| 200014 | 手机号码格式错误（短验）                                                                                                                                                                                                                                                                                        |
| 200015 | 短信验证码格式错误                                                                                                                                                                                                                                                                                              |
| 200016 | 更新KS失败                                                                                                                                                                                                                                                                                                      |
| 200017 | 非移动卡不支持短信上行                                                                                                                                                                                                                                                                                          |
| 200018 | 不支持网关登录                                                                                                                                                                                                                                                                                                  |
| 200019 | 不支持短信验证码登录                                                                                                                                                                                                                                                                                            |
| 200020 | 用户取消登录                                                                                                                                                                                                                                                                                                    |
| 200021 | 数据解析异常                                                                                                                                                                                                                                                                                                    |
| 200022 | 无网络                                                                                                                                                                                                                                                                                                          |
| 200023 | 请求超时                                                                                                                                                                                                                                                                                                        |
| 200024 | 数据网络切换失败                                                                                                                                                                                                                                                                                                |
| 200025 | 位置错误（一般是线程捕获异常、socket、系统未授权移动数据网络权限等，请提工单联系工程师）                                                                                                                                                                                                                        |
| 200026 | 输入参数错误。                                                                                                                                                                                                                                                                                                  |
| 200027 | 未开启移动数据网络或网络不稳定                                                                                                                                                                                                                                                                                  |
| 200028 | 网络请求出错                                                                                                                                                                                                                                                                                                    |
| 200029 | 请求出错,上次请求未完成                                                                                                                                                                                                                                                                                         |
| 200030 | 没有启动参数                                                                                                                                                                                                                                                                                                  |
| 200031 | 生成token失败                                                                                                                                                                                                                                                                                                   |
| 200032 | KS缓存不存在                                                                                                                                                                                                                                                                                                    |
| 200033 | 复用中间件获取Token失败                                                                                                                                                                                                                                                                                         |
| 200034 | 预取号token失效                                                                                                                                                                                                                                                                                                 |
| 200035 | 协商ks失败                                                                                                                                                                                                                                                                                                      |
| 200036 | 预取号失败                                                                                                                                                                                                                                                                                                      |
| 200037 | 获取不到openid                                                                                                                                                                                                                                                                                                  |
| 200038 | 异网取号网络请求失败                                                                                                                                                                                                                                                                                            |
| 200039 | 异网取号网关取号失败                                                                                                                                                                                                                                                                                            |
| 200040 | UI资源加载异常                                                                                                                                                                                                                                                                                                  |
| 200042 | 授权页弹出异常                                                                                                                                                                                                                                                                                                  |
| 200048 | 用户未安装SIM卡                                                                                                                                                                                                                                                                                                 |
| 200050 | EOF异常                                                                                                                                                                                                                                                                                                         |
| 200061 | 授权页面异常                                                                                                                                                                                                                                                                                                    |
| 200064 | 服务端返回数据异常                                                                                                                                                                                                                                                                                              |
| 200072 | CA根证书校验失败                                                                                                                                                                                                                                                                                                |
| 200082 | 服务器繁忙                                                                                                                                                                                                                                                                                                      |
| 200086 | ppLocation为空                                                                                                                                                                                                                                                                                                  |
| 200087 | 仅用于监听授权页成功拉起                                                                                                                                                                                                                                                                                        |
| 200096 | 当前网络不支持取号                                                                                                                                                                                                                                                                                              |

### 中国电信

| 状态码  | 状态码描述                                                         |
| ------- | ------------------------------------------------------------------ |
| 0       | 请求成功                                                           |
| -64     | Permission-denied（无权限访问）                                    |
| -65     | API-request-rates-Exceed-Limitations（调用接口超限）               |
| -10001  | 取号失败，mdn为空                                                  |
| -10002  | 参数错误                                                           |
| -10003  | 解密失败                                                           |
| -10004  | IP受限                                                             |
| -10005  | 异网取号回调参数异常                                               |
| -10006  | mdn取号失败，且属于电信网络                                        |
| -10007  | 重定向到异网取号                                                   |
| -10008  | 超过预设取号阈值                                                   |
| -10009  | 时间戳过期                                                         |
| -10013  | Perator_unsupported，提工单联系工程师                              |
| -20005  | Sign-invalid（签名错误）                                           |
| -20006  | 应用不存在                                                         |
| -20007  | 公钥数据不存在                                                     |
| -20100  | 内部解析错误                                                       |
| -20102  | 加密参数解析失败                                                   |
| -30001  | 时间戳非法                                                         |
| -30003  | topClass-invalid,topclass无效                                      |
| 51002   | 参数为空                                                           |
| 51114   | 无法获取手机号数据                                                 |
| -8001   | 网络异常，请求失败                                                 |
| -8002   | 请求参数错误                                                       |
| -8003   | 请求超时                                                           |
| -8004   | 移动数据网络未开启                                                 |
| -8010   | 无网络连接（网络错误）                                             |
| -720001 | Wi-Fi切换4G请求异常                                                |
| -720002 | Wi-Fi切换4G超时                                                    |
| 80000   | 请求超时                                                           |
| 80001   | 网络连接失败、网络链接已中断、Invalid argument、目前不允许数据连接 |
| 80002   | 响应码错误404                                                      |
| 80003   | 网络无连接                                                         |
| 80005   | socket超时异常                                                     |
| 80007   | IO异常                                                             |
| 80008   | No route to host                                                   |
| 80009   | Nodename nor servname provided, or not known                       |
| 80010   | Socket closed by remote peer                                       |
| 80800   | Wi-Fi切换超时                                                      |
| 80801   | Wi-Fi切换异常                                                      |
| -9999   | 网络故障(networkauth-fail)                                         |
| -720001 | 切换异常,切换流量卡时网络不稳定                                    |
| -720002 | 切换异常超时                                                       |

### 中国联通

| 状态码                      | 状态码描述                           |
| --------------------------- | ------------------------------------ |
| 0                           | 表示请求成功                         |
| -10008                      | JSON转换失败                         |
| 1，请求超时                 | 请求超时                             |
| 1，私网IP查找号码失败       | 私网IP查找号码失败                   |
| 1，私网IP校验错误           | 私网IP校验错误                       |
| 1，源IP鉴权失败             | 源IP鉴权失败                         |
| 1，获取鉴权信息失败         | 获取鉴权信息失败                     |
| 1，获得的手机授权码失败     | 一般是由于请求SDK超时导致的失败      |
| 1，网关取号失败             | 网关取号失败                         |
| 1，网络请求失败             | 网络请求失败                         |
| 1，验签失败                 | 签名校验失败                         |
| 1，传入code和AppID不匹配    | 传入code和AppID不匹配                |
| 1，似乎已断开与互联网的链接 | 网络不稳定导致连接断开               |
| 1，connect address error    | 连接地址错误，一般是由于超时导致失败 |
| 1，select socket error      | 选择socket错误                       |
| 1，handshake failed         | 握手失败                             |
| 1，decode ret_url fail      | URL解码失败                          |
| 1，connect error            | 连接错误                             |
| 2，请求超时                 | 接口请求耗时超过timeout设定的值      |