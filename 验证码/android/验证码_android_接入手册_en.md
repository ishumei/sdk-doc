#  android Verification Code SDK Integration Steps

## Project Configuration

Android Verification Code SDK is compatible with a minimum of Android API Level 15.

### Import jar Package

Copy `captcha*.jar` to the libs directory of the project Module.

### Add Permission Dependencies

```groovy
dependencies {
  implementation fileTree(include: ['captcha*.jar'], dir: 'libs')
}
```

### Add Permissions

Add the following permissions in `AndroidManifest.xml`
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Add SDK proguard rules to the proguard-rules.pro file as follows

```
-keep class com.ishumei.** {*;}
```

## Integration

### Parameter Settings

   ```java
   SmCaptchaWebView.SmOption option = new SmCaptchaWebView.SmOption();
   // (Required) Replace with your own organization
   option.setOrganization("xxx");
   // (Required) Keep consistent with the backend management page, you can directly use default
   option.setAppId("default");
   // (Optional) Select verification mode, supports the following modes
   //      1. SmCaptchaWebView.MODE_SLIDE          Slide (default)
   //      2. SmCaptchaWebView.MODE_SELECT         Text selection
   //      3. SmCaptchaWebView.MODE_ICON_SELECT    Icon selection
   //      4. SmCaptchaWebView.MODE_SEQ_SELECT     Sequence selection
   //      5. SmCaptchaWebView.MODE_SPATIAL_SELECT Spatial logic reasoning
   option.setMode(SmCaptchaWebView.MODE_SLIDE);
   // (Optional) Channel identifier
   option.setChannel("CHANNEL");
   // (Optional) Custom prompt message, only supported in slide mode
   option.setTipMessage("YOUR_TIP_MESSAGE");
   // (Optional) captchaUuid, used for business statistics, if not set, the SDK will generate a captchaUuid internally
   // This captchaUuid is consistent with the captchaUuid in the callback
   option.setCaptchaUuid(UUID.randomUUID().toString());
   
   // (Optional) Style configuration
   Map<String, Object> extOption = new HashMap<>();
   // (Optional) Verification code English theme, default is Chinese, other languages refer to: https://help.ishumei.com/docs/tw/captcha/web/developDoc for the explanation of the lang value of the startup parameter
   extOption.put("lang", "en");
   // (Optional) Custom style, only supported in slide mode (MODE_SLIDE)
   extOption.put("style", 
                 new JSONObject(
                   "{\n" +
                   "  \"customFont\": { \n" +
                   "    \"name\": \"Walsheim\",\n" + // Font name, no special restrictions
                   // This ttf file must be configured by the customer, must be an absolute address, a URL starting with https or http, and must support cross-domain (set CORS)
                   "    \"url\": \"http://castatic.fengkongcloud.com/pr/v1.0" +
                   ".4/assets/GT-Walsheim-Pro-Bold.ttf\"\n" +
                   "  },\n" +
                   "  \"fontFamily\": \"Arial\",\n" + // Only one of customFont and fontFamily can exist, otherwise it will override the custom font
                   "  \"fontWeight\": 600,\n" + // Font weight
                   // This field is mainly used for the end, used to display the header when the product is embed
                   // In slide mode (MODE_SLIDE), when the custom style withTitle is true, the content aspect ratio is 6:5, other styles are 3:2
                   "  \"withTitle\": " + true + ",\n" +
                   "  \"headerTitle\": \"Vertification1\",\n" + // Custom header title, this attribute affects the page aspect ratio
                   "  \"hideRefreshOnImage\": true,\n" + // Whether to hide the refresh button on the image, the default is to have it
                   "  \"slideBar\": {\n" + // Custom style of the progress bar
                   "    \"color\": \"#141C30\",\n" + // Default font color on the progress bar
                   "    \"successColor\": \"#FFF\",\n" + // Font color when successful
                   "    \"showTipWhenMove\": true,\n" + // When dragging, the text behind still shows
                   "    \"process\": {\n" +
                   "      \"border\": \"none\",\n" + // Border of the progress bar, currently used to clear the border
                   "      \"background\": \"#F3F8CE\",\n" + // Default background color of the progress bar
                   "      \"successBackground\": \"#25BC73\",\n" + // Background color when successful
                   "      \"failBackground\": \"#F5E0E2\"\n" + // Background color when failed
                   "    },\n" +
                   "    \"button\": {\n" +
                   "      \"boxShadow\": \"none\",\n" + // Button shadow setting, generally used to remove the default shadow
                   "      \"color\": \"#333\",\n" + // Default icon color in the button
                   "      \"background\": \"#F6FF7E\",\n" + // Default button background color
                   "      \"successBackground\": \"#25BC73\",\n" + // Background color when successful
                   "      \"failBackground\": \"#ED816E\"\n" + // Background color when failed
                   "    }\n" +
                   "  }\n" +
                   "}"
                 ));
   option.setExtOption(extOption);
   
   // Overseas integration
   // (Optional) Special configuration items for connecting to the Singapore data center, only for customers reporting verification code data to the Singapore data center
   // option.setCdnHost("castatic-xjp.fengkongcloud.com");
   // option.setHost("captcha-xjp.fengkongcloud.com");
   // (Optional) Special configuration items for connecting to the US data center, only for customers reporting verification code data to the US data center
   // option.setCdnHost("castatic-fjny.fengkongcloud.com");
   // option.setHost("captcha-fjny.fengkongcloud.com");
   
   // Private integration
   // Set the verification code URL
   // option.setCaptchaHtml(captchaHtmlUrl);
   // Set the verification code API interface domain name: format host[:port]
   // option.setHost(host);
   ```

### Start

```java
SmCaptchaWebView mCaptchaView = new SmCaptchaWebView(context);
int code = mCaptchaView.initWithOption(option, mSimpleResultListener);
if (SmCaptchaWebView.SMCAPTCHA_SUCCESS != code) {
  // Startup error, refer to the error code meaning for modification
} else {
  // Add mCaptchaView to the specified container
  // The aspect ratio of the verification code is 3:2
  // In slide mode (MODE_SLIDE), when the custom style withTitle is true, the content aspect ratio is 6:5
}
```

### Callback Methods

```java
SimpleResultListener mSimpleResultListener = new SimpleResultListener() {

  @Override
  public void onInitWithContent(JSONObject content) {
    super.onInitWithContent(content);
    // This method is called when the web parameters are started
    // Get uuid for business statistics through content.optString("captchaUuid")
  }

  @Override
  public void onReadyWithContent(JSONObject content) {
    super.onReadyWithContent(content);
    // This method is called when the verification code image is successfully loaded
    // Get uuid for business statistics through content.optString("captchaUuid")
  }

  @Override
  public void onSuccessWithContent(JSONObject content) {
    super.onSuccessWithContent(content);
    // This method is called when the verification ends
    // Get uuid for business statistics through content.optString("captchaUuid")
    // Get verification result through content.optBoolean("pass"): true for verification passed, false for not passed
    // Get rid through content.optString("rid")
  }

  @Override
  public void onErrorWithContent(JSONObject content) {
    super.onErrorWithContent(content);
    // This method is called when an error occurs during loading or verification, see the error code below
    // Get uuid for business statistics through content.optString("captchaUuid")
    // Get error code through content.optInt("code"), see the error code meaning below
  }

  @Override
  public void onCloseWithContent(JSONObject content) {
    super.onCloseWithContent(content);
    // This method is called when the close button of the verification code with the close button style is clicked
    // Get uuid for business statistics through content.optString("captchaUuid")
  }

  // The following methods are for version upgrade transition and will be removed later

  @Override
  public void onReady() {
    // This method is called when the verification code image is successfully loaded
    // This method will be removed later, new customers please use the onReadyWithContent method
  }

  @Override
  public void onSuccess(CharSequence rid, boolean pass) {
    // This method is called when the verification ends
    // Verification not passed pass is false; verification passed pass is true
    // This method will be removed later, new customers please use the onSuccessWithContent method
  }

  @Override
  public void onError(int code) {
    // This method is called when an error occurs during loading or verification, see the error code meaning below
    // This method will be removed later, new customers please use the onErrorWithContent method
  }

  @Override
  public void onClose() {
    // This method is called when the close button of the verification code with the close button style is clicked
    // This method will be removed later, new customers please use the onCloseWithContent method
  }
}
```

### Other Methods

```java
// Make the slide verification code unmovable
mCaptchaView.disableCaptcha();
// Make the slide verification code movable
mCaptchaView.enableCaptcha();
// Destroy CaptachView
mCaptchaView.destroy();
```

## Error Codes

| Return Code | Remarks                       |
| ----------- | ----------------------------- |
| 0           | Success                       |
| 1001        | Input parameter is empty      |
| 1002        | Organization not provided     |
| 1003        | AppId not provided            |
| 1004        | Callback not set              |
| 1005        | Network timeout or main page load failed |
| 1006        | Error in return result        |
| 2003        | Input parameter error         |
| 2005        | Network error                 |