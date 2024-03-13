
# 数美号码认证 iOS SDK 接入文档

## 一、接入说明

开发环境：
* 最低部署版本：iOS 9.0及以上。
* Xcode开发工具：12.0及以上。

集成方式：

* 将`SmAuth.xcframework`拖拽到Xcode工程内（需要勾选Copy items if needed选项）。
* 点击项目target -> General -> Frameworks,Libraries,and Embedded Content -> SmAuth.xcframework，在Embed选项下，选择`Embed & Sign`。

-------

**一键登录使用步骤：**

* 导入头文件
```oc
#import <SmAuth/SmAuth.h>
```

* 启动SDK
```oc
// AppId
[[SmAuthHelper shareInstance] registerWithAppId:@"xxxxx"];
// 开启日志
[[SmAuthHelper shareInstance] consolePrintLoggerEnable:YES];
// 设置超时
[[SmAuthHelper shareInstance] setTimeoutInterval:6000];
```

* 预加载授权页（取号）
```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

* 加载授权页
```oc
- (void)loadLoginAuthorizePageWithViewConfig:(UAuthViewConfig *)viewConfig completion:(void (^)(NSError *error, NSString *token))completion;
```

* 关闭授权页
```oc
- (void)unloadLoginAuthorizePageAnimated:(BOOL)animated completion:(void (^_Nullable)(void))completion;
```

-------

**本机号码验证使用步骤：**

* 预加载授权页（取号）
```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

* 手机号码检测（获取验证Token）
```oc
- (void)verifyPhoneNumberCompletion:(void(^)(NSError * _Nullable, NSString * _Nullable))completion;
```

## 二、SDK接口说明

#### 2.1 启动SDK

**接口描述** 

>  启动SDK。

**接口示例**

```oc
[[SmAuthHelper shareInstance] registerWithAppId:@"xxxxx"];
```

**参数说明**

> appId: 数美号码认证提供的 ID。

-------

#### 2.2 预加载授权页（取号）

**接口描述** 

>  预加载授权页，使用前应先判断运营商信息、网络信息等。

**接口示例**

```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

**回调说明**

> 错误码:`200027`-->蜂窝网络未开启；`-10003`-->蜂窝网络权限未开启；其它错误码参照运营商错误码列表。

-------

#### 2.3 加载授权页

**接口描述** 

>  拉取授权页。

**接口示例**

```oc
- (void)loadLoginAuthorizePageWithViewConfig:(UAuthViewConfig *)viewConfig completion:(void (^)(NSError *error, NSString *token))completion;
```

**参数说明**

> viewConfig: 授权页配置model。

**回调说明**

> 错误码: 参照运营商错误码列表。
> token: 取号token，使用该token去开发者应用后台换取完整手机号码。

-------

#### 2.4 关闭授权页

**接口描述** 

>  关闭授权页。

**接口示例**

```oc
- (void)unloadLoginAuthorizePageAnimated:(BOOL)animated completion:(void (^_Nullable)(void))completion;
```

**参数说明**

> animated: 是否开启动画。

-------

#### 2.5 本机号码校验Token获取

**接口描述** 

>  本机号码校验Token获取。

**接口示例**

```oc
- (void)verifyPhoneNumberCompletion:(void(^)(NSError * _Nullable, NSString * _Nullable))completion;
```

**回调说明**

> 错误码: 参照运营商错误码列表。
> token: 本机号码验证token，使用该token去开发者应用后台验证是否是本机手机号码。

-------

#### 2.6 日志打印

**接口描述** 

>  日志打印。

**接口示例**

```oc
- (void)consolePrintLoggerEnable:(BOOL)enable;
```

**参数说明**

> enable: 是否开启控制台日志打印。

-------

#### 2.7 超时设置

**接口描述** 

>  超时设置。

**接口示例**

```oc
- (void)setTimeoutInterval:(NSTimeInterval)interval;
```

**参数说明**

> interval: 单位毫秒，建议设置为8000毫秒。

-------

#### 2.8 运营商类型

**接口描述** 

>  运营商类型。

**接口示例**

```oc
- (SmCarrierType)carrierType;
```

**参数说明**

> SmCarrierType: 

`SmCarrierTypeUnknown`-->未知；

`SmCarrierTypeMobile`-->中国移动; 

`SmCarrierTypeUnicome`-->中国联通;

`SmCarrierTypeTelecom`-->中国电信。

-------

#### 2.9 连接网络类型

**接口描述** 

>  当前连接网络类型。

**接口示例**

```oc
- (SmNetworkType)networkType;
```

**参数说明**

> SmNetworkType:

`SmNetworkTypeUnknown`-->未知

`SmNetworkTypeNotReachable`-->无网络

`SmNetworkTypeCellular`-->数据流量

`SmNetworkTypeWifi`-->WiFi

`SmNetworkTypeCellularAndWifi`-->数据流量+WiFi

-------

#### 2.10 SDK版本

**接口描述** 

>  当前SDK版本。

**接口示例**

```oc
- (NSString *)getSDKVersion;
```

## 三、其他注意事项

其它接口的使用方法，请参考Demo中的示例。        

## 四、运营商SDK错误码
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

| **状态码**                  | **状态码描述**                       |
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
