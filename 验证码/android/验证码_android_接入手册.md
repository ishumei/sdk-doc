#  android 验证码SDK接入步骤

**Android 验证码 SDK目前支持兼容的最低android系统版本为 4.0.3**

1. **导入jar 包**

   将captcha.jar拷贝到项目Module的libs目录下。

2. **添加权限**

   在`AndroidManifest.xml`中添加如下权限：

   ```
   <uses-permission android:name="android.permission.INTERNET" />
   ```

3. **初始化数美SDK WebView控件类**

   ```java
    // 构造数美验证码WEBVIEW对象，目前宽高比为3:2
   SmCaptchaWebView mCaptchaView = new SmCaptchaWebView(this);
   LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(width, width / 3f * 2);
   mCaptchaView.setLayoutParams(layoutParams);
   
   // 定义结果回调方法
   SmCaptchaWebView.ResultListener listener = new SmCaptchaWebView.ResultListener() {
     // 验证码图片加载成功回调函数
     @Override
     public void onReady() {
       Log.i(TAG, "onReady");
     }
   
     // 处理过程出现异常回调函数
     @Override
     public void onError(int code) {
       Log.i(TAG, "code:" + code);
     }
   
     // 用户操作结束回调函数，操作验证未通过pass为false，操作验证通过pass为true。
     @Override
     public void onSuccess(CharSequence rid, boolean pass) {
       Log.d(TAG, "onSuccess rid:" + rid + " pass:" + pass);
     }
   };
   
   // 初始化验证码WebView
   SmCaptchaWebView.SmOption option = new SmCaptchaWebView.SmOption();
   option.setOrganization("xxxxx");     // 必填，组织标识
   option.setAppId("xxxx");                 // 必填，应用标识
   
   // 选填，支持滑动式（默认） 和点选式
   // MODE_SLIDE  滑动验证
   // MODE_SELECT  文字点选
   // MODE_ICON_SELECT  图标点选
   // MODE_SEQ_SELECT  语序点选
   // MODE_SPATIAL_SELECT  空间逻辑推理
   option.setMode(SmCaptchaWebView.MODE_SLIDE); 
   
   // 选填，支持自定义样式
   Map<String, Object> extOption = new HashMap<>();
   extOption.put("lang", "en"); // 验证码英文主题，默认中文
   
   // 自定义主题需要更换验证码主页
   option.setCaptchaHtml("https://castatic.fengkongcloud.com/pr/v1.0.4/index.html");
   extOption.put("style", new JSONObject("{\n" +
           "  \"customFont\": { \n" +
           "    \"name\": \"Walsheim\",\n" + // 字体名，没有特别的限制
           // 这个ttf文件必须由客户配置，必须是绝对地址，以https或是http开头的url，且必须支持跨域(设置CORS)
           "    \"url\": \"http://castatic-dev.fengkongcloud.com/pr/v1.0" +
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
           "}"));
   extOption.put("style", json);
   option.setExtOption(extOption);
   
   option.setTipMessage ("xxxx");       // 选填，自定义提示文字，仅滑动式支持
   option.setChannel("xxxx");           // 选填，APP的渠道标识
   option.setDeviceId("xxxxx");          // 选填，数美反欺诈SDK获取到的设备标识
   
   // 连接新加坡机房特殊配置项，仅供验证码数据上报新加坡机房客户使用 
   //option.setCdnHost("castatic-xjp.fengkongcloud.com");
   //option.setHost("captcha-xjp.fengkongcloud.com");
   
   // 连接美国机房特殊配置项，仅供验证码数据上报美国机房客户使用
   //option.setCdnHost("castatic-fjny.fengkongcloud.com");
   //option.setHost("captcha-fjny.fengkongcloud.com");
     
   // 私有化
   // 设置验证码 url
   // option.setCaptchaHtml("YOUR_CAPTCHA_URL");
   // 设置验证码 api 接口域名：格式 host[:port]
   // option.setHost("YOUR_HOST")
   
   int code = mCaptchaView.initWithOption(option, listener);
   if (SmCaptchaWebView.SMCAPTCHA_SUCCESS != code) {
     Log.e(TAG, "init failed:" + code);
   }
   
   ```

4. **将验证码WebView添加到需要展示的容器内**

5. **获取用户操作结果**

   用户操作结束后，会回调ResultListener中相应函数。如果用户操作验证结束，回调ResultListener中onSuccess函数，同时返回当前操作请求的唯一标识rid，以及操作是否通过。

6. **返回码说明**

   | 返回码 | 备注               |
   | ------ | ------------------ |
   | 0      | 成功               |
   | 1001   | 传入参数为空       |
   | 1002   | 未传入organization |
   | 1003   | 未传入appId        |
   | 1004   | 未设置回调         |
   | 1005   | 网络问题           |
   | 1006   | 返回结果出错       |
   | 2003   | 传入参数参数错误   |
