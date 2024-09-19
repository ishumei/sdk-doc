Shumei Intelligent Captcha SDK (i.e., 'smcaptcha') compatibleSdkVersion 10

## 1 Project Configuration

This chapter provides the DevEco Studio project configuration steps

1. Copy `smcaptcha_x.x.x.har` to the libs directory of the Module (e.g., entry module). If there is no libs directory, create a new libs folder, or place it in another location (just ensure the subsequent dependency path is consistent).

2. Add the har reference in the oh-package.json5 file in the entry directory

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
       "@ohos/library": "file:./libs/smcaptcha_x.x.x.har" // The version number needs to be consistent with the actual imported smsdk version
     }
   }
   ```

3. Declare permissions, add the following permissions in module.json5

   ```xml
   "requestPermissions": [
       {
         // Internet permission, mandatory
         "name": "ohos.permission.INTERNET",
       }
     ]
   ```

   Permission purpose

   | Permission        | Purpose       |
   | ----------------- | ------------- |
   | INTERNET (mandatory) | Network communication |

## 2 Integration

### 2.1 Parameter Settings

```typescript
// (Mandatory) Captcha callback methods
this.captchaController.onInit = this.onInit;
this.captchaController.onReady = this.onReady;
this.captchaController.onSuccess = this.onSuccess;
this.captchaController.onError = this.onError;
this.captchaController.onClose = this.onClose;

// (Mandatory) todo: Replace with your own organization
this.captchaOption.organization = YOUR-ORGANIZATION
// (Mandatory) todo: Keep consistent with the backend management page, you can directly use default
this.captchaOption.appId = 'default'
// (Optional) Choose verification mode, supports the following modes
//      1. SmCaptchaOption.MODE_SLIDE          Slide (default)
//      2. SmCaptchaOption.MODE_SELECT         Text selection
//      3. SmCaptchaOption.MODE_ICON_SELECT    Icon selection
//      4. SmCaptchaOption.MODE_SEQ_SELECT     Sequence selection
//      5. SmCaptchaOption.MODE_SPATIAL_SELECT Spatial logical reasoning
this.captchaOption.mode = SmCaptchaOption.MODE_SLIDE
// (Optional) Channel identifier
this.captchaOption.channel = 'YOUR-CHANNEL'
// (Optional) Custom prompt text, only supported in slide mode
this.captchaOption.tipMessage = 'Custom prompt text'
// (Optional) captchaUuid, used for business statistics. If this value is not set, the SDK will generate a captchaUuid internally. This captchaUuid is consistent with the captchaUuid in the callback.
this.captchaOption.captchaUuid = util.generateRandomUUID();
// (Optional) Captcha language theme, default is Simplified Chinese
this.captchaOption.lang = `zh-cn` // Language support reference: https://help.ishumei.com/docs/tw/captcha/web/developDoc
// (Optional) Set captcha domain IP host
this.captchaOption.host = 'YOUR-HOST'
// (Optional) Set custom fields, used to transmit custom data required for business events
this.captchaOption.extData = 'YOUR-EXTRA-DATA'
// (Optional) Custom styles, only for slide mode (MODE_SLIDE)
let style: SmWebStyle = new SmWebStyle()
style.fontFamily = "Arial" // Only one of customFont or fontFamily can exist, otherwise it will override the custom font
style.fontWeight = 600 // Font weight
style.withTitle = true // When withTitle is true, the content aspect ratio is 6:5, other styles are 3:2
style.headerTitle = "Vertification1" // Custom title at the top, this property affects the page aspect ratio
style.hideRefreshOnImage = true // Whether to hide the refresh button on the image, the default is to have it
style.slideBar.color = "#141C30" // Default font color on the progress bar
style.slideBar.successColor = "#FFF" // Font color when successful
style.slideBar.showTipWhenMove = true // When dragging, the text behind still shows
style.slideBar.process.border = "none" // Progress bar border, currently used to clear the border
style.slideBar.process.background = "#F3F8CE" // Default progress bar background color
style.slideBar.process.successBackground = "#25BC73" // Background color when successful
style.slideBar.process.failBackground = "#F5E0E2" // Background color when failed
style.slideBar.button.boxShadow = "none" // Button shadow setting, generally used to remove the default shadow
style.slideBar.button.color = "#333" // Default icon color in the button
style.slideBar.button.background = "#F6FF7E" // Default button background color
style.slideBar.button.successBackground = "#25BC73" // Button background color when successful
style.slideBar.button.failBackground = "#ED816E" // Button background color when failed
style.customFont.name = "Walsheim" // Font name, no special restrictions, this ttf file must be configured by the customer
style.customFont.url = "http:\/\/castatic.fengkongcloud.com\/pr\/v1.0.4\/assets\/GT-Walsheim-Pro-Bold.ttf" // Must be an absolute address, starting with https or http, and must support cross-origin (set CORS)
this.captchaOption.style = style
```

### 2.2 Initialization

```typescript
// Create UI component SmCaptchaWeb
build() {
  Row() {
    ...
		SmCaptchaWeb({ controller: this.captchaController }).id('smcaptcha_web')
          .width('100%')
          .aspectRatio(1.5) // withTitle aspect ratio is 6:5; noTitle aspect ratio is 3:2
          .alignRules({
            center: { anchor: '__container__', align: VerticalAlign.Center }
          })
    ...
}
    
// Load captcha
captchaController = new SmCaptchaController()
this.captchaController.initOption(this.captchaOption)
```

### 2.3 Callback Methods

```typescript
onInit: (scc: SmCallbackContent) => void = (scc) => {
	// This method is called when the web parameters are initialized
  // Get uuid through scc?.captchaUuid, used for business statistics
}
onReady: (scc: SmCallbackContent) => void = (scc) => {
  // This method is called when the captcha image is successfully loaded
  // Get uuid through scc?.captchaUuid, used for business statistics
}
onSuccess: (scc: SmCallbackContent) => void = (scc) => {
  // This method is called when the verification ends
  // Get uuid through scc?.captchaUuid, used for business statistics
  // Get verification result through scc?.pass: true for verification passed, false for not passed
  // Get rid through scc?.rid
}
onError: (scc: SmCallbackContent) => void = (scc) => {
	// This method is called when an error occurs during loading or verification, see the code below
  // Get uuid through scc?.captchaUuid, used for business statistics
  // Get error code through scc?.code, see the code meaning below
  // Error message through scc?.message
}
onClose: (scc: SmCallbackContent) => void = (scc) => {
	// This method is called when the close button is clicked on the captcha with the close button style
  // Get uuid through scc?.captchaUuid, used for business statistics
}
```

## Error Codes

| Return Code | Remarks                    |
| ----------- | -------------------------- |
| 0           | Success                    |
| 1001        | Input parameter is empty   |
| 1002        | Organization not provided  |
| 1005        | Network timeout or main page load failed |
| 1006        | Error in returned result   |
| 2003        | Input parameter error      |
| 2005        | Network error              |