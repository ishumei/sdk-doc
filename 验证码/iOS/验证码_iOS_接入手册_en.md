# iOS Verification Code SDK Integration Steps

## Project Configuration

iOS Verification Code SDK is compatible with iOS 9.0 and above.

### Import .xcframework

Import SmCaptcha.xcframework into the project as follows:

First, move SmCaptcha.xcframework into the project. Then, in Xcode, go to TARGETS -> General -> Framework, Libraries, and Embedded Content, click the + button -> Add Other -> Add Files, select SmCaptcha.xcframework, and set the Embed option to "Embed & Sign".

## Integration

### Import Header File

In the code, use `#import` to import the header file:

```objective-c
#import <SmCaptcha/SmCaptchaWebView.h>
```

### Parameter Settings

```objective-c
// Launch the verification code WebView
SmCaptchaOption *caOption = [[SmCaptchaOption alloc] init];
[caOption setOrganization: @”xxxxx”];   // Required, organization identifier
[caOption setAppId:@"default"];         // Required, application identifier

// Optional, supports sliding (default) and click-to-select modes
// SM_MODE_SLIDE  Sliding selection
// SM_MODE_SELECT  Text selection
// SM_MODE_ICON_SELECT  Icon selection
// SM_MODE_SEQ_SELECT  Sequence selection
// SM_MODE_SPATIAL_SELECT  Spatial logic reasoning
[caOption setMode: SM_MODE_SLIDE];    

[caOption setExtOption:@{
  // (Optional) Verification code English theme, default is Chinese, other languages refer to: https://help.ishumei.com/docs/tw/captcha/web/developDoc for the explanation of the lang value in the startup parameters
  @"lang" : @"en",
  // (Optional) Custom style, only supported in sliding (MODE_SLIDE) mode
  @"style": @{
    @"customFont": @{ // Custom font
      @"name": @"Walsheim", // Font name, no special restrictions
      // This ttf file must be configured by the customer, must be an absolute address, starting with https or http, and must support cross-domain (set CORS).
      @"url": @"http://castatic.fengkongcloud.com/pr/v1.0.4/assets/GT-Walsheim-Pro-Bold.ttf",
    },
    // In sliding (MODE_SLIDE) mode, when withTitle is true, the content aspect ratio is 6:5, other styles are 3:2
    @"withTitle":@(YES),
    @"fontFamily": @"Arial", // Only one of customFont or fontFamily can exist, otherwise it will override the custom font
    @"fontWeight": @(600), // Font weight
    @"headerTitle": @"Vertification1", // Custom header title
    @"hideRefreshOnImage": @(YES), // Whether to hide the refresh button on the image, default is visible
    @"slideBar": @{ // Custom style for the progress bar
      @"color": @"#141C30", // Default font color on the progress bar
      @"successColor": @"#FFF", // Font color when successful
      @"showTipWhenMove":@(YES), // Whether to show the text when dragging
      @"process": @{
        @"border": @"none", // Border of the progress bar, currently used to clear the border
        @"background": @"#F3F8CE", // Default background color of the progress bar
        @"successBackground": @"#25BC73", // Background color when successful
        @"failBackground": @"#F5E0E2", // Background color when failed
      },
      @"button": @{
        @"boxShadow": @"none", // Button shadow setting, generally used to remove the default shadow
        @"color": @"#333", // Default icon color in the button
        @"background": @"#F6FF7E", // Default button background color
        @"successBackground": @"#25BC73", // Button background color when successful
        @"failBackground": @"#ED816E", // Button background color when failed
      }
    },
  },
}];

[caOption setTipMessage:@"xxxxx"];      // Optional, custom prompt text, only supported in sliding mode
[caOption setChannel:@"xxxxx"];         // Optional, channel identifier

// Special configuration items for connecting to the Singapore data center, only for customers reporting verification code data to the Singapore data center
// [caOption setCdnHost:@"castatic-xjp.fengkongcloud.com"];
// [caOption setHost:@"captcha-xjp.fengkongcloud.com"];

// Special configuration items for connecting to the US data center, only for customers reporting verification code data to the US data center
// [caOption setCdnHost:@"castatic-fjny.fengkongcloud.com"];
// [caOption setHost:@"captcha-fjny.fengkongcloud.com"];

// Privatization
// Set the verification code URL
// [caOption setCaptchaHtml:@"YOUR_CAPTCHA_URL"];
// Set the verification code API interface domain name: format host[:port]
// [caOption setHost:@"YOUR_HOST"];
```

### Launch

```objective-c
// Construct the SmCaptcha verification code object, the aspect ratio of the verification code is 3:2, in sliding (MODE_SLIDE) mode, when custom style withTitle is true, the content aspect ratio is 6:5
SmCaptchaWKWebView *_webview = [[SmCaptchaWKWebView alloc] init];
CGRect captchaRect = CGRectMake(0, 0, width, width * 2 / 3);
[_webview setFrame:captchaRect];

// Launch error, refer to the error code meaning for modification; self implements the SmCaptchaProtocol protocol
NSInteger code = [_webview createWithOption: caOption delegate: self];
if (SmCaptchaSuccess != code) {
  NSLog(@"SmCaptchaWKWebView init failed %ld", code);
} else {
  [_captchaContainer addSubview:_webview];
}
```

### SmCaptchaProtocol Protocol

```objective-c
- (void)onInitWithContent:(NSDictionary *)content
{
  // This method is called when the web parameters are initialized
  // [content objectForKey:@"captchaUuid"] to get the uuid for business statistics
  NSLog(@"onInitWithContent : %@",content);
} 

- (void)onReadyWithContent:(NSDictionary *)content
{
  // This method is called when the verification code image is successfully loaded
  // Get the uuid for business statistics through [content objectForKey:@"captchaUuid"]
  NSLog(@"onReadyWithContent : %@",content);
} 

- (void)onSuccessWithContent:(NSDictionary *)content
{
  // This method is called when the verification is completed
  // [content objectForKey:@"captchaUuid"] to get the uuid for business statistics
  // [content objectForKey:@"pass"]; to get the verification result: YES for pass, NO for fail
  // [content objectForKey:@"rid"]; to get the rid
  NSLog(@"onSuccessWithContent : %@",content);
} 

- (void)onErrorWithContent:(NSDictionary *)content
{
  // This method is called when an error occurs during loading or verification, see the error code below
  // [content objectForKey:@"captchaUuid"] to get the uuid for business statistics
  // [content objectForKey:@"code"] to get the error code, see the error code meaning below
  NSLog(@"onErrorWithContent : %@",content);
} 

- (void)onCloseWithContent:(NSDictionary *)content
{
  // This method is called when the close button is clicked on the verification code with a close button style
  // [content objectForKey:@"captchaUuid"] to get the uuid for business statistics
  NSLog(@"onCloseWithContent : %@",content);
}

// The following methods are for version upgrade transition and will be removed later

// Callback function when the verification code image is successfully loaded
- (void) onReady {
  NSLog(@"view onReady");
}

// Callback function when an exception occurs during processing
- (void) onError:(NSInteger)code {
  NSLog(@"view onError:%ld", code);
}	

// Callback function when the user operation ends, pass is false if the operation verification fails, pass is true if the operation verification passes.
- (void) onSuccess:(NSString *)rid pass:(BOOL)pass {
  NSLog(@"view onSuccess:%@", rid);
}

// Callback function when the verification code image is closed
- (void) onClose {
  NSLog(@"view onClose"); 
}
```

### Error Codes

| Return Code | Remarks                |
| ----------- | ---------------------- |
| 0           | Success                |
| 1001        | Input parameter is empty |
| 1002        | Organization not provided |
| 1003        | AppId not provided     |
| 1004        | Callback not set       |
| 1005        | Network issue          |
| 1006        | Error in returned result |
| 2003        | Input parameter error  |