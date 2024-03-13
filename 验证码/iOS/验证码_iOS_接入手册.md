# iOS 验证码SDK接入步骤

## 工程配置

iOS 验证码 SDK 最低兼容 iOS 9.0

### 导入.xcframework

在工程下导入 SmCaptcha.xcframework，具体方式如下：

首先将 SmCaptcha.xcframework 移动到项目中，接着在 Xcode 的 TARGETS -> General -> Framework,Libraries, and Enbeded Content 中，点击 + 号 -> Add Other -> Add Files，选择 SmCaptcha.xcframework，并将 Embed 方式设置为 "Embed & Sign"。

## 接入

### 导入头文件

在代码中使用 `#import ` 导入头文件：

```objective-c
#import <SmCaptcha/SmCaptchaWebView.h>
```

### 参数设置

```objective-c
// 启动验证码WebView
SmCaptchaOption *caOption = [[SmCaptchaOption alloc] init];
[caOption setOrganization: @”xxxxx”];   // 必填，组织标识
[caOption setAppId:@"default"];         // 必填，应用标识

// 选填，支持滑动式（默认） 和点选式
// SM_MODE_SLIDE  滑动点选
// SM_MODE_SELECT  文字点选
// SM_MODE_ICON_SELECT  图标点选
// SM_MODE_SEQ_SELECT  语序点选
// SM_MODE_SPATIAL_SELECT  空间逻辑推理
[caOption setMode: SM_MODE_SLIDE];    

[caOption setExtOption:@{
  //（选填项）验证码英文主题，默认中文，其他语言参考：https://help.ishumei.com/docs/tw/captcha/web/developDoc 启动参数 lang 值解释
  @"lang" : @"en",
  // (选填项) 自定义样式，仅 滑动式（MODE_SLIDE）模式 支持自定义样式
  @"style": @{
    @"customFont": @{ // 自定义字体
      @"name": @"Walsheim", // 字体名，没有特别的限制
      // 这个ttf文件必须由客户配置，必须是绝对地址，以https或是http开头的url，且必须支持跨域(设置CORS)。
      @"url": @"http://castatic.fengkongcloud.com/pr/v1.0.4/assets/GT-Walsheim-Pro-Bold.ttf",
    },
    // 滑动式（MODE_SLIDE）模式下，自定义样式 withTitle 为 true 时，内容宽高比为 6:5，其它样式 3:2
    @"withTitle":@(YES),
    @"fontFamily": @"Arial", // 与customFont只能存在一种，不然会覆盖自定义字体
    @"fontWeight": @(600), // 字体粗度
    @"headerTitle": @"Vertification1", // 顶部自定义标题
    @"hideRefreshOnImage": @(YES), // 是否隐藏图片上的刷新按钮，默认是有的
    @"slideBar": @{ // 进度条自定义样式
      @"color": @"#141C30", // 进度条上默认字体颜色
      @"successColor": @"#FFF", // 成功时字体颜色
      @"showTipWhenMove":@(YES), // 当拖动的时候后面文案依旧显示
      @"process": @{
        @"border": @"none", // 进度条边框，目前只是用来设置清除边框
        @"background": @"#F3F8CE", // 默认进度条背景色
        @"successBackground": @"#25BC73", // 成功时背景色
        @"failBackground": @"#F5E0E2", // 失败时背景色
      },
      @"button": @{
        @"boxShadow": @"none", // 按钮阴影设置，一般用来去除默认阴影的作用
        @"color": @"#333", // 默认按钮中的icon颜色
        @"background": @"#F6FF7E", // 默认按钮背景色
        @"successBackground": @"#25BC73", // 成功时按钮背景色
        @"failBackground": @"#ED816E", // 失败时按钮背景色
      }
    },
  },
}];

[caOption setTipMessage:@"xxxxx"];      // 选填，自定义提示文字，仅滑动式支持
[caOption setChannel:@"xxxxx"];         // 选填，渠道标识

// 连接新加坡机房特殊配置项，仅供验证码数据上报新加坡机房客户使用 
// [caOption setCdnHost:@"castatic-xjp.fengkongcloud.com"];
// [caOption setHost:@"captcha-xjp.fengkongcloud.com"];

// 连接美国机房特殊配置项，仅供验证码数据上报美国机房客户使用
// [caOption setCdnHost:@"castatic-fjny.fengkongcloud.com"];
// [caOption setHost:@"captcha-fjny.fengkongcloud.com"];

// 私有化
// 设置验证码 url
// [caOption setCaptchaHtml:@"YOUR_CAPTCHA_URL"];
// 设置验证码 api 接口域名：格式 host[:port]
// [caOption setHost:@"YOUR_HOST"];
```

### 启动

```objective-c
// 构造数美验证码对象，验证码宽高比为 3:2，滑动式（MODE_SLIDE）模式下，自定义样式 withTitle 为 true 时，内容宽高比为 6:5
SmCaptchaWKWebView *_webview = [[SmCaptchaWKWebView alloc] init];
CGRect captchaRect = CGRectMake(0, 0, width, width * 2 / 3);
[_webview setFrame:captchaRect];

// 启动错误，参考错误码含义进行修改；self 实现 SmCaptchaProtocol 协议
NSInteger code = [_webview createWithOption: caOption delegate: self];
if (SmCaptchaSuccess != code) {
  NSLog(@"SmCaptchaWKWebView init failed %ld", code);
} else {
  [_captchaContainer addSubview:_webview];
}
```

### SmCaptchaProtocol 协议

```objective-c
- (void)onInitWithContent:(NSDictionary *)content
{
  // web 参数启动完成时，回调此方法
  // [content objectForKey:@"captchaUuid"] 获取uuid，用于业务统计
  NSLog(@"onInitWithContent : %@",content);
} 

- (void)onReadyWithContent:(NSDictionary *)content
{
  // 验证码图片加载成功时回调此方法
  // 通过content中的 [content objectForKey:@"captchaUuid"] 获取uuid，用于业务统计
  NSLog(@"onReadyWithContent : %@",content);
} 

- (void)onSuccessWithContent:(NSDictionary *)content
{
  // 验证结束时回调此方法
  // [content objectForKey:@"captchaUuid"] 获取uuid，用于业务统计
  // [content objectForKey:@"pass"]; 获取验证结果：YES 为验证通过，NO 为未通过
  // [content objectForKey:@"rid"]; 获取 rid
  NSLog(@"onSuccessWithContent : %@",content);
} 

- (void)onErrorWithContent:(NSDictionary *)content
{
  // 加载或验证过程中出现错误时，回调此方法，code 码见下文
  // [content objectForKey:@"captchaUuid"] 获取uuid，用于业务统计
  // [content objectForKey:@"code"] 获取错误码，code 码含义见下文
  NSLog(@"onErrorWithContent : %@",content);
} 

- (void)onCloseWithContent:(NSDictionary *)content
{
  // 带关闭按钮样式的验证码点击关闭按钮时回调此方法
  // [content objectForKey:@"captchaUuid"] 获取uuid，用于业务统计
  NSLog(@"onCloseWithContent : %@",content);
}

// 以下方法用于版本升级过渡，后续会被移除

// 验证码图片加载成功回调函数
- (void) onReady {
  NSLog(@"view onReady");
}

// 处理过程出现异常回调函数
- (void) onError:(NSInteger)code {
  NSLog(@"view onError:%ld", code);
}	

// 用户操作结束回调函数，操作验证未通过pass为false，操作验证通过pass为true。
- (void) onSuccess:(NSString *)rid pass:(BOOL)pass {
  NSLog(@"view onSuccess:%@", rid);
}

// 验证码图片关闭回调函数
- (void) onClose {
  NSLog(@"view onClose"); 
}
```

### 错误码

| 返回码 | 备注                |
| ------ | ------------------- |
| 0      | 成功                |
| 1001   | 传入参数为空        |
| 1002   | 未传入 organization |
| 1003   | 未传入 appId        |
| 1004   | 未设置回调          |
| 1005   | 网络问题            |
| 1006   | 返回结果出错        |
| 2003   | 传入参数参数错误    |