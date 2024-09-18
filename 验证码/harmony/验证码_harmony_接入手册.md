数美智能验证码 SDK（即 'smcaptcha'）compatibleSdkVersion 10

## 1 工程配置

本章节为 DevEco Studio 工程配置步骤

1. 将 `smcaptcha_x.x.x.har` 拷贝到 Module（例如 entry 模块） 的 libs 目录下，如果没有 libs 目录则新建 libs 文件夹，或者放到其它位置（确保后续依赖路径一致即可）

2. 在 entry 目录 oh-package.json5 文件中添加 har 引用

   ```json5
   {
     "license": "",
     "devDependencies": {},
     "author": "",
     "name": "entry",
     "description": "Please describe the basic information.",
     "main": "",
     "version": "1.0.0",
     "dependencies": {
       "@ohos/library": "file:./libs/smcaptcha_x.x.x.har" // 版本号需要与实际导入 smsdk 版本一直
     }
   }
   ```

3. 声明权限，在 module.json5 中添加如下权限

   ```xml
   "requestPermissions": [
       {
         // 联网权限，必选权限
         "name": "ohos.permission.INTERNET",
       }
     ]
   }
   ```

   权限作用

   | 权限             | 作用     |
   | ---------------- | -------- |
   | INTERNET（必选） | 网络通讯 |

## 2 接入

### 2.1 参数设置

```typescript
// （必填项） 验证码回调方法
this.captchaController.onInit = this.onInit;
this.captchaController.onReady = this.onReady;
this.captchaController.onSuccess = this.onSuccess;
this.captchaController.onError = this.onError;
this.captchaController.onClose = this.onClose;

// （必填项）todo: 替换成自己的 organization
this.captchaOption.organization = YOUR-ORGANIZATION
// （必填项）todo: 与后台管理页面保持一致，可直接使用 default
this.captchaOption.appId = 'default'
// （选填项）选择验证模式，支持以下模式
//      1. SmCaptchaOption.MODE_SLIDE          滑动式（默认）
//      2. SmCaptchaOption.MODE_SELECT         文字点选式
//      3. SmCaptchaOption.MODE_ICON_SELECT    图标点选式
//      4. SmCaptchaOption.MODE_SEQ_SELECT     语序点选
//      5. SmCaptchaOption.MODE_SPATIAL_SELECT 空间逻辑推理
this.captchaOption.mode = SmCaptchaOption.MODE_SLIDE
// （选填项）渠道标识
this.captchaOption.channel = 'YOUR-CHANNEL'
// （选填项）自定义提示文字，仅滑动式支持
this.captchaOption.tipMessage = '自定义提示文字'
// （选填项）captchaUuid，用于业务统计，如果不设置此值，SDK 内部会生成一个 captchaUuid 此 captchaUuid 与回调中 captchaUuid 一致
this.captchaOption.captchaUuid = util.generateRandomUUID();
// （选填项）验证码英文主题，默认简体中文
this.captchaOption.lang = `zh-cn` // 语言支持参考：https://help.ishumei.com/docs/tw/captcha/web/developDoc
// （选填项）设置 验证码 domain ip host
this.captchaOption.host = 'YOUR-HOST'
// （选填项）设置自定义字段，用于传输业务事件需要的自定义数据
this.captchaOption.extData = 'YOUR-EXTRA-DATA'
// （选填项) 自定义样式，仅 滑动式（MODE_SLIDE）模式
let style: SmWebStyle = new SmWebStyle()
style.fontFamily = "Arial" // 与customFont只能存在一种，不然会覆盖自定义字体
style.fontWeight = 600 // 字体粗度
style.withTitle = true // 自定义样式 withTitle 为 true 时，内容宽高比为 6:5，其它样式 3:2
style.headerTitle = "Vertification1" // 顶部自定义标题，此属性影响页面宽高比
style.hideRefreshOnImage = true // 是否隐藏图片上的刷新按钮，默认是有的
style.slideBar.color = "#141C30" // 进度条上默认字体颜色
style.slideBar.successColor = "#FFF" // 成功时字体颜色
style.slideBar.showTipWhenMove = true // 当拖动的时候后面文案依旧显示
style.slideBar.process.border = "none" // 进度条边框，目前只是用来设置清除边框
style.slideBar.process.background = "#F3F8CE" // 默认进度条背景色
style.slideBar.process.successBackground = "#25BC73" // 成功时背景色
style.slideBar.process.failBackground = "#F5E0E2" // 失败时背景色
style.slideBar.button.boxShadow = "none" // 按钮阴影设置，一般用来去除默认阴影的作用
style.slideBar.button.color = "#333" // 默认按钮中的icon颜色
style.slideBar.button.background = "#F6FF7E" // 默认按钮背景色
style.slideBar.button.successBackground = "#25BC73" // 成功时按钮背景色
style.slideBar.button.failBackground = "#ED816E" // 失败时按钮背景色
style.customFont.name = "Walsheim" // 字体名，没有特别的限制, 这个ttf文件必须由客户配置
style.customFont.url = "http:\/\/castatic.fengkongcloud.com\/pr\/v1.0.4\/assets\/GT-Walsheim-Pro-Bold.ttf" // 必须是绝对地址，以https或是http开头的url，且必须支持跨域(设置CORS)
this.captchaOption.style = style
```

### 2.2 初始化

```typescript
// 创建 UI 组件 SmCaptchaWeb
build() {
  Row() {
    ...
		SmCaptchaWeb({ controller: this.captchaController }).id('smcaptcha_web')
          .width('100%')
          .aspectRatio(1.5) // withTitle 宽高为 6:5；noTitle 宽高比为 3:2
          .alignRules({
            center: { anchor: '__container__', align: VerticalAlign.Center }
          })
    ...
}
    
// 加载验证码
captchaController = new SmCaptchaController()
this.captchaController.initOption(this.captchaOption)
```

### 2.3 回调方法

```typescript
onControllerAttached: () => void = () => {
  // 验证码内部 Web onControllerAttached 回调方法
}
onInit: (scc: SmCallbackContent) => void = (scc) => {
  // web 参数初始化完成时，回调此方法
  // 通过 scc?.captchaUuid 获取 uuid，用于业务统计
}
onReady: (scc: SmCallbackContent) => void = (scc) => {
  // 验证码图片加载成功时回调此方法
  // 通过 scc?.captchaUuid 获取 uuid，用于业务统计
}
onSuccess: (scc: SmCallbackContent) => void = (scc) => {
  // 验证结束时回调此方法
  // scc?.captchaUuid 获取 uuid，用于业务统计
  // scc?.pass 获取验证结果：true 为验证通过，false 为未通过
  // scc?.rid 获取 rid
}
onError: (scc: SmCallbackContent) => void = (scc) => {
  // 加载或验证过程中出现错误时，回调此方法，code 码见下文
  // scc?.captchaUuid 获取 uuid，用于业务统计
  // scc?.code 获取错误码，code 码含义见下文
  // scc?.message 错误信息
}
onClose: (scc: SmCallbackContent) => void = (scc) => {
  // 带关闭按钮样式的验证码点击关闭按钮时回调此方法
  // scc?.captchaUuid 获取 uuid，用于业务统计
}
```

## 错误码

| 返回码 | 备注                       |
| ------ | -------------------------- |
| 0      | 成功                       |
| 1001   | 传入参数为空               |
| 1002   | 未传入 organization        |
| 1005   | 网络超时或者主页面加载失败 |
| 1006   | 返回结果出错               |
| 2003   | 传入参数错误               |
| 2005   | 网络错误                   |