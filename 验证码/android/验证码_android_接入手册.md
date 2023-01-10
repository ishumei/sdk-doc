#  android 验证码SDK接入步骤

## 工程配置

Android 验证码 SDK 最低兼容 Android API Level 15。

### 导入jar 包

将 `captcha*.jar` 拷贝到项目Module 的 libs 目录下。

### 添加权限依赖

```groovy
dependencies {
  implementation fileTree(include: ['captcha*.jar'], dir: 'libs')
}
```

### 添加权限

在 `AndroidManifest.xml` 中添加如下权限
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

向 proguard-rules.pro 文件中添加 SDK 防混淆规则，如下

```
-keep class com.ishumei.** {*;}
```

## 接入

### 参数设置

   ```java
   SmCaptchaWebView.SmOption option = new SmCaptchaWebView.SmOption();
   //（必填项）替换成自己的 organization
   option.setOrganization("xxx");
   //（必填项）与后台管理页面保持一致，可直接使用 default
   option.setAppId("default");
   //（选填项）选择验证模式，支持以下模式
   //      1. SmCaptchaWebView.MODE_SLIDE          滑动式（默认）
   //      2. SmCaptchaWebView.MODE_SELECT         文字点选式
   //      3. SmCaptchaWebView.MODE_ICON_SELECT    图标点选式
   //      4. SmCaptchaWebView.MODE_SEQ_SELECT     语序点选
   //      5. SmCaptchaWebView.MODE_SPATIAL_SELECT 空间逻辑推理
   option.setMode(SmCaptchaWebView.MODE_SLIDE);
   //（选填项）渠道标识
   option.setChannel("CHANNEL");
   //（选填项）自定义提示文字，仅滑动式支持
   option.setTipMessage("YOUR_TIP_MESSAGE");
   //（选填项）captchaUuid，用于业务统计，如果不设置此值，SDK 内部会生成一个 captchaUuid
   // 此 captchaUuid 与回调中 captchaUuid 一致
   option.setCaptchaUuid(UUID.randomUUID().toString());
   
   //（选填项）样式配置
   Map<String, Object> extOption = new HashMap<>();
   //（选填项）验证码英文主题，默认中文，其他语言参考：https://help.ishumei.com/docs/tw/captcha/web/developDoc 初始化参数 lang 值解释
   extOption.put("lang", "en");
   // (选填项) 自定义样式，仅 滑动式（MODE_SLIDE）模式 支持自定义样式
   extOption.put("style", 
                 new JSONObject(
                   "{\n" +
                   "  \"customFont\": { \n" +
                   "    \"name\": \"Walsheim\",\n" + // 字体名，没有特别的限制
                   // 这个ttf文件必须由客户配置，必须是绝对地址，以https或是http开头的url，且必须支持跨域(设置CORS)
                   "    \"url\": \"http://castatic.fengkongcloud.com/pr/v1.0" +
                   ".4/assets/GT-Walsheim-Pro-Bold.ttf\"\n" +
                   "  },\n" +
                   "  \"fontFamily\": \"Arial\",\n" + // 与customFont只能存在一种，不然会覆盖自定义字体
                   "  \"fontWeight\": 600,\n" + // 字体粗度
                   // 这个字段主要给端上使用，用来在product是embed的时候展示头部的
                   // 滑动式（MODE_SLIDE）模式下，自定义样式 withTitle 为 true 时，内容宽高比为 6:5，其它样式 3:2
                   "  \"withTitle\": " + true + ",\n" +
                   "  \"headerTitle\": \"Vertification1\",\n" + // 顶部自定义标题，此属性影响页面宽高比
                   "  \"hideRefreshOnImage\": true,\n" + // 是否隐藏图片上的刷新按钮，默认是有的
                   "  \"slideBar\": {\n" + // 进度条自定义样式
                   "    \"color\": \"#141C30\",\n" + // 进度条上默认字体颜色
                   "    \"successColor\": \"#FFF\",\n" + // 成功时字体颜色
                   "    \"showTipWhenMove\": true,\n" + // 当拖动的时候后面文案依旧显示
                   "    \"process\": {\n" +
                   "      \"border\": \"none\",\n" + // 进度条边框，目前只是用来设置清除边框
                   "      \"background\": \"#F3F8CE\",\n" + // 默认进度条背景色
                   "      \"successBackground\": \"#25BC73\",\n" + // 成功时背景色
                   "      \"failBackground\": \"#F5E0E2\"\n" + // 失败时背景色
                   "    },\n" +
                   "    \"button\": {\n" +
                   "      \"boxShadow\": \"none\",\n" + // 按钮阴影设置，一般用来去除默认阴影的作用
                   "      \"color\": \"#333\",\n" + // 默认按钮中的icon颜色
                   "      \"background\": \"#F6FF7E\",\n" + // 默认按钮背景色
                   "      \"successBackground\": \"#25BC73\",\n" + // 成功时按钮背景色
                   "      \"failBackground\": \"#ED816E\"\n" + // 失败时按钮背景色
                   "    }\n" +
                   "  }\n" +
                   "}"
                 ));
   option.setExtOption(extOption);
   
   // 海外接入
   //（选填项）连接新加坡机房特殊配置项，仅供验证码数据上报新加坡机房客户使用
   // option.setCdnHost("castatic-xjp.fengkongcloud.com");
   // option.setHost("captcha-xjp.fengkongcloud.com");
   //（选填项）连接美国机房特殊配置项，仅供验证码数据上报美国机房客户使用
   // option.setCdnHost("castatic-fjny.fengkongcloud.com");
   // option.setHost("captcha-fjny.fengkongcloud.com");
   
   // 私有化接入
   // 设置验证码 url
   // option.setCaptchaHtml(captchaHtmlUrl);
   // 设置验证码 api 接口域名：格式 host[:port]
   // option.setHost(host);
   ```

### 初始化

```java
SmCaptchaWebView mCaptchaView = new SmCaptchaWebView(context);
int code = mCaptchaView.initWithOption(option, mSimpleResultListener);
if (SmCaptchaWebView.SMCAPTCHA_SUCCESS != code) {
  // 初始化错误，参考错误码含义进行修改
} else {
  // 将 mCaptchaView 添加到指定容器中
  // 验证码宽高比为 3:2
  // 滑动式（MODE_SLIDE）模式下，自定义样式 withTitle 为 true 时，内容宽高比为 6:5
}
```

### 回调方法

```java
SimpleResultListener mSimpleResultListener = new SimpleResultListener() {

  @Override
  public void onInitWithContent(JSONObject content) {
    super.onInitWithContent(content);
    // web 参数初始化完成时，回调此方法
    // 通过 content.optString("captchaUuid") 获取 uuid，用于业务统计
  }

  @Override
  public void onReadyWithContent(JSONObject content) {
    super.onReadyWithContent(content);
    // 验证码图片加载成功时回调此方法
    // 通过 content.optString("captchaUuid") 获取 uuid，用于业务统计
  }

  @Override
  public void onSuccessWithContent(JSONObject content) {
    super.onSuccessWithContent(content);
    // 验证结束时回调此方法
    // content.optString("captchaUuid") 获取 uuid，用于业务统计
    // content.optBoolean("pass") 获取验证结果：true 为验证通过，false 为未通过
    // content.optString("rid") 获取 rid
  }

  @Override
  public void onErrorWithContent(JSONObject content) {
    super.onErrorWithContent(content);
    // 加载或验证过程中出现错误时，回调此方法，code 码见下文
    // content.optString("captchaUuid") 获取 uuid，用于业务统计
    // content.optInt("code") 获取错误码，code 码含义见下文
  }

  @Override
  public void onCloseWithContent(JSONObject content) {
    super.onCloseWithContent(content);
    // 带关闭按钮样式的验证码点击关闭按钮时回调此方法
    // content.optString("captchaUuid") 获取 uuid，用于业务统计
  }

  // 以下方法用于版本升级过渡，后续会被移除

  @Override
  public void onReady() {
    // 验证码图片加载成功时回调此方法
    // 此方法后续会移除，新接入客户请使用 onReadyWithContent 方法
  }

  @Override
  public void onSuccess(CharSequence rid, boolean pass) {
    // 验证结束时回调此方法
    // 验证未通过 pass 为 false；验证通过 pass 为 true
    // 此方法后续会移除，新接入客户请使用 onSuccessWithContent 方法
  }

  @Override
  public void onError(int code) {
    // 加载或验证过程中出现错误时，回调此方法，code 码含义见下文
    // 此方法后续会移除，新接入客户请使用 onErrorWithContent 方法
  }

  @Override
  public void onClose() {
    // 带关闭按钮样式的验证码点击关闭按钮时回调此方法
    // 此方法后续会移除，新接入客户请使用 onCloseWithContent 方法
  }
}
```

### 其它方法

```java
// 使滑动验证码不可滑动
mCaptchaView.disableCaptcha();
// 使滑动验证码可滑动
mCaptchaView.enableCaptcha();
// 销毁 CaptachView
mCaptchaView.destroy();
```

## 错误码

| 返回码 | 备注                       |
| ------ | -------------------------- |
| 0      | 成功                       |
| 1001   | 传入参数为空               |
| 1002   | 未传入 organization        |
| 1003   | 未传入 appId               |
| 1004   | 未设置回调                 |
| 1005   | 网络超时或者主页面加载失败 |
| 1006   | 返回结果出错               |
| 2003   | 传入参数错误               |
| 2005   | 网络错误                   |
