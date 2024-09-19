# smauth-demo-android

smauth Mobile Number One-Click Login SDK Demo for Android

## SDK Location

`Demo/app/libs/smauth*.aar`

## Project Configuration

### Import SDK and Dependencies

``` java
implementation 'com.google.code.gson:gson:2.8.8'
implementation fileTree(dir: "libs", include: "smauth*.aar")
```

### Configure AndroidManifest

```xml
<application>
    <activity
            android:name="com.cmic.gen.sdk.view.GenLoginAuthActivity"
            android:configChanges="orientation|keyboardHidden|screenSize"
            android:launchMode="singleTop"
            android:screenOrientation="behind" />
</application>
```



## Start SDK

``` java
SmAuthHelper.create(context).register(applicationId, listener);
```



## API

### SmAuthHelper

#### Get SDK version
``` java
public static String sdkVersion()
```

#### Create SmAuthHelper instance
``` java
public static SmAuthHelper create(Context context)
```

* parameter:
    * context: Android context
* return:
    * SmAuthHelper instance


#### Get SmAuthHelper instance
``` java
public static SmAuthHelper getInstance()
```

* parameter:
    * -
* return:
    * SmAuthHelper instance

#### Destroy SmAuthHelper instance
``` java
public static void destroy()
```

* parameter:
    * -
* return:
    * -

#### Register service
``` java
public void register(SmAuthRegisterListener listener)
```
* parameter:
    * listener: Register callback
* return:
    * -

``` java
public void register(String applicationId, SmAuthRegisterListener listener)
```
* parameter:
    * applicationId: applicationId assigned after registering APP information on the console
    * listener: Register callback
* return:
    * -

#### Pre-fetch number
``` java
public void preloadAuthorization(SmAuthPreloadListener listener, int requestCode)
```

* parameter:
    * listener: Pre-fetch number callback
    * requestCode: Request code, the callback will return this request code
* return:
    * -

#### Custom authorization page UI configuration
``` java
public void setAuthThemeConfigure(ThemeConfig theme)
```

* parameter:
    * theme: UI configuration item
* return:
    * -

#### Number authorization (pop-up authorization page)
``` java
public void loginAuth(SmAuthTokenListener listener, int requestCode)
```

* parameter:
    * listener: Number authorization callback
    * requestCode: Request code, the callback will return this request code
* return:
    * -

#### Exit authorization page
``` java
public void quitLoginAuth()
```

* parameter:
    * -
* return:
    * -

#### Mobile number verification authorization
``` java
public void verifyMobile(SmAuthVerifyMobileListener listener, int requestCode)
```

* parameter:
    * listener: Mobile number verification authorization callback
    * requestCode: Request code, the callback will return this request code
* return:
    * -

#### Get current network status and operator information
``` java
public NetworkInfo getNetworkInfo()
```

* parameter:
    * -
* return:
    * Current network status and operator information

#### Set service interface timeout
``` java
public void setTimeout(long overtime)
```

* parameter:
    * overtime: Timeout duration (milliseconds)
* return:
    * -

### Custom authorization page configuration item (ThemeConfig.Builder)
``` java
// Set status bar color
public Builder setStatusBar(int statusBarColor, boolean isLightColor)
// Set custom ContentView
public Builder setAuthContentView(View contentView)
// Set phone number font size
public Builder setNumberSize(int numberSize, boolean isBold)
// Set phone number font color
public Builder setNumberColor(int numberColor)
// Set horizontal offset of phone number
public Builder setNumberOffsetX(int numberOffsetX)
// Set vertical offset of phone number relative to the top
public Builder setNumFieldOffsetY(int numFieldOffsetY)
// Set vertical offset of phone number relative to the bottom
public Builder setNumFieldOffsetY_B(int numFieldOffsetY_B)
// Set authorization button text
public Builder setLogBtnText(String logBtnText)
// Set authorization button text and font information
public Builder setLogBtnText(String logBtnText, int logBtnTextColor, int logBtnTextSize, boolean isBold)
// Set authorization button font color
public Builder setLogBtnTextColor(int logBtnTextColor)
// Set authorization button background resource name (if btn.png, just fill in btn)
public Builder setLogBtnImgPath(String logBtnBackgroundPath)
// Set authorization button size
public Builder setLogBtn(int width, int height)
// Set horizontal margin of authorization button
public Builder setLogBtnMargin(int marginLeft, int marginRight)
// Set vertical offset of authorization button relative to the top
public Builder setLogBtnOffsetY(int logBtnOffsetY)
// Set vertical offset of authorization button relative to the bottom
public Builder setLogBtnOffsetY_B(int logBtnOffsetY_B)
// Set the prompt text when the protocol checkbox is not selected
public Builder setCheckTipText(String checkTipText)
// Set the image resource name for the selected state of the protocol checkbox
public Builder setCheckedImgPath(String checkedImgPath)
// Set the image resource name for the unselected state of the protocol checkbox
public Builder setUncheckedImgPath(String uncheckedImgPath)
// Set the details of the protocol checkbox
public Builder setCheckBoxImgPath(String checkedImgPath, String uncheckedImgPath, int width, int height)
// Set the default selection state of the protocol
public Builder setPrivacyState(boolean privacyState)
// Set protocol font color
public Builder setClauseColor(int clauseBaseColor, int clauseColor)
// Set horizontal margin of the protocol
public Builder setPrivacyMargin(int privacyMarginLeft, int privacyMarginRight)
// Set vertical offset of the protocol relative to the top
public Builder setPrivacyOffsetY(int privacyOffsetY)
// Set vertical offset of the protocol relative to the bottom
public Builder setPrivacyOffsetY_B(int privacyOffsetY_B)
// Set whether the protocol name displays book symbols
public Builder setPrivacyBookSymbol(boolean haveBookSymbol)
// Set the checkbox relative to the right protocol text to be top-aligned or center-aligned, default is top-aligned. 0-top-aligned, 1-center-aligned
public Builder setCheckBoxLocation(int checkBoxLocation)
// Set the callback for the selection state change of the protocol policy checkbox
public Builder setPrivacyCheckedChangeListener(SmAuthPrivacyCheckedChangeListener listener)
// Set the entrance animation of the authorization page
public Builder setAuthPageActIn(String authPageActIn, String activityOut)
// Set the exit animation of the authorization page
public Builder setAuthPageActOut(String activityIn, String authPageActOut)
// Set the width and height ratio of the authorization page window
public Builder setAuthPageWindowMode(int windowWidth, int windowHeight)
// Set the X-axis and Y-axis offset of the authorization page window
public Builder setAuthPageWindowOffset(int windowX, int windowY)
// Set whether the authorization page is at the bottom, 0=centered; 1=bottom, setting to 1 invalidates the Y-axis offset
public Builder setWindowBottom(int windowBottom)
// Set the theme of the authorization page popup, can also be set in the Manifest
public Builder setThemeId(int themeId)
// 0. Simplified Chinese 1. Traditional Chinese 2. English
public Builder setAppLanguageType(int appLanguageType)
/** Enable Android bottom navigation bar adaptation. After enabling, the elements of the authorization page will also change relatively when the navigation bar is invoked;
  * If adaptation is not enabled, custom content can fill the full screen. After setting the status bar to transparent, an immersive display effect can be achieved.
  * 0-enable adaptation, 1-disable adaptation, default is enabled.
**/
public Builder setFitsSystemWindows(boolean isFitsSystemWindows)
```

### SmAuthRegisterListener

``` java
// Registration completed
void onRegisted(boolean isSuccess, Exception exception);
```

### SmAuthPreloadListener

``` java
// Pre-fetch number completed
void onPreloaded(int requestCode, PreloadResultBean token);
// Pre-fetch number failed
void onPreloadFailed(int requestCode, Exception exception);
```

### SmAuthTokenListener

``` java
// Authorization Token obtained
void onGetToken(int requestCode, TokenBean token);
// Authorization Token failed to obtain
void onGetTokenFailed(int requestCode, Exception exception);
```

### SmAuthVerifyMobileListener

``` java
// Verification completed
void onVerified(int requestCode, VerifyMobileBean result);
// Verification failed
void onVerifiedFailed(int requestCode, Exception exception);
```

## Operator SDK Error Codes

> When calling the number authentication service one-click login and local number verification function interfaces, the error codes of the corresponding operator interfaces will be returned. Common interface error codes and error descriptions can be referred to below

### China Mobile

| Status Code | Status Code Description                                                                                                                                                                                                                                                                                                      |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 103000      | Success                                                                                                                                                                                                                                                                                                                      |
| 102101      | No network                                                                                                                                                                                                                                                                                                                   |
| 102102      | Network exception (network request error, generally occurs when the network security policy restricts the use of http, the device has enabled a proxy, or the connected WiFi has network link security policy restrictions, it is recommended to analyze specifically in combination with the SDK log)                                                              |
| 102103      | Data network not enabled                                                                                                                                                                                                                                                                                                     |
| 102121      | User canceled login                                                                                                                                                                                                                                                                                                          |
| 102203      | Input parameter error                                                                                                                                                                                                                                                                                                        |
| 102223      | Data parsing exception                                                                                                                                                                                                                                                                                                       |
| 102507      | Login timeout (when clicking the login button on the authorization page)                                                                                                                                                                                                                                                     |
| 102508      | Data network switch failed                                                                                                                                                                                                                                                                                                   |
| 103101      | Request signature error (request signature error (if it occurs on the client side, it may be that the appkey is wrong, you can check whether it is confused with the appsecret, or there is a space. If it occurs on the server interface, you need to check whether the verification method is MD5 or RSA, if it is MD5, check the signType field, if it is appsecret, confirm whether the appkey is mistakenly used for signing. If it is RSA, check whether the private key used corresponds to the reported public key and whether the message splicing meets the document requirements.)) |
| 103102      | Package signature Bundle ID error (the reported one does not match the actual one used)                                                                                                                                                                                                                                       |
| 103103      | User does not exist                                                                                                                                                                                                                                                                                                          |
| 103111      | Gateway IP error (check if VPN is enabled or if it is a foreign IP)                                                                                                                                                                                                                                                          |
| 103119      | AppID does not exist. (Check if the appid passed is correct or if there is a space)                                                                                                                                                                                                                                          |
| 103211      | Other errors (commonly seen in incorrect message format, please check if it meets these three requirements: a. JSON format message interaction must be in standard JSON format; b. When sending, please set content type to `application/json`; c. Parameter types are all String)                                              |
| 103412      | Invalid request with encryption method error, non-JSON format, and empty request, etc.                                                                                                                                                                                                                                       |
| 103414      | Parameter validation exception                                                                                                                                                                                                                                                                                               |
| 103511      | Server IP whitelist validation failed                                                                                                                                                                                                                                                                                        |
| 103811      | Token is empty                                                                                                                                                                                                                                                                                                               |
| 103902      | Repeated login in a short time, causing script to fail                                                                                                                                                                                                                                                                       |
| 103911      | Token request too frequent, no more than 30 unused Tokens obtained within 10 minutes                                                                                                                                                                                                                                         |
| 104201      | Token repeated validation failed, invalid or does not exist                                                                                                                                                                                                                                                                  |
| 105002      | Mobile number retrieval failed (generally IoT card)                                                                                                                                                                                                                                                                          |
| 105013      | Does not support Unicom number retrieval                                                                                                                                                                                                                                                                                     |
| 105018      | Used a Token for local number verification to get the number, causing insufficient Token permissions                                                                                                                                                                                                                         |
| 105019      | Application not authorized                                                                                                                                                                                                                                                                                                   |
| 105021      | Reached the daily number retrieval limit                                                                                                                                                                                                                                                                                     |
| 105302      | AppID not in whitelist                                                                                                                                                                                                                                                                                                       |
| 105313      | Illegal request                                                                                                                                                                                                                                                                                                              |
| 200002      | Phone does not have a SIM card installed                                                                                                                                                                                                                                                                                     |
| 200005      | User not authorized (READ_PHONE_STATE)                                                                                                                                                                                                                                                                                       |
| 200006      | User not authorized (SEND_SMS)                                                                                                                                                                                                                                                                                               |
| 200007      | authType only uses SMS verification code authentication                                                                                                                                                                                                                                                                      |
| 200008      | 1. authType parameter is empty; 2. authType parameter is illegal;                                                                                                                                                                                                                                                            |
| 200009      | Application legality verification failed (package name and package signature not filled in correctly)                                                                                                                                                                                                                       |
| 200010      | Unable to recognize SIM card or no SIM card (Android)                                                                                                                                                                                                                                                                        |
| 200012      | Number retrieval failed, switch to SMS verification code login                                                                                                                                                                                                                                                               |
| 200013      | SMS uplink failed to send SMS (SMS uplink)                                                                                                                                                                                                                                                                                   |
| 200014      | Phone number format error (SMS verification)                                                                                                                                                                                                                                                                                 |
| 200015      | SMS verification code format error                                                                                                                                                                                                                                                                                           |
| 200016      | Update KS failed                                                                                                                                                                                                                                                                                                             |
| 200017      | Non-Mobile card does not support SMS uplink                                                                                                                                                                                                                                                                                  |
| 200018      | Does not support gateway login                                                                                                                                                                                                                                                                                               |
| 200019      | Does not support SMS verification code login                                                                                                                                                                                                                                                                                 |
| 200020      | User canceled login                                                                                                                                                                                                                                                                                                          |
| 200021      | Data parsing exception                                                                                                                                                                                                                                                                                                       |
| 200022      | No network                                                                                                                                                                                                                                                                                                                   |
| 200023      | Request timeout                                                                                                                                                                                                                                                                                                              |
| 200024      | Data network switch failed                                                                                                                                                                                                                                                                                                   |
| 200025      | Location error (generally thread capture exception, socket, system not authorized mobile data network permission, etc., please submit a ticket to contact the engineer)                                                                                                                                                       |
| 200026      | Input parameter error.                                                                                                                                                                                                                                                                                                       |
| 200027      | Data network not enabled or network unstable                                                                                                                                                                                                                                                                                 |
| 200028      | Network request error                                                                                                                                                                                                                                                                                                        |
| 200029      | Request error, previous request not completed                                                                                                                                                                                                                                                                                |
| 200030      | No startup parameters                                                                                                                                                                                                                                                                                                        |
| 200031      | Token generation failed                                                                                                                                                                                                                                                                                                      |
| 200032      | KS cache does not exist                                                                                                                                                                                                                                                                                                      |
| 200033      | Middleware reuse Token retrieval failed                                                                                                                                                                                                                                                                                      |
| 200034      | Pre-fetch number token invalid                                                                                                                                                                                                                                                                                               |
| 200035      | Negotiation ks failed                                                                                                                                                                                                                                                                                                        |
| 200036      | Pre-fetch number failed                                                                                                                                                                                                                                                                                                      |
| 200037      | Unable to get openid                                                                                                                                                                                                                                                                                                         |
| 200038      | Different network number retrieval network request failed                                                                                                                                                                                                                                                                    |
| 200039      | Different network number retrieval gateway number retrieval failed                                                                                                                                                                                                                                                           |
| 200040      | UI resource loading exception                                                                                                                                                                                                                                                                                                |
| 200042      | Authorization page pop-up exception                                                                                                                                                                                                                                                                                          |
| 200048      | User does not have a SIM card installed                                                                                                                                                                                                                                                                                      |
| 200050      | EOF exception                                                                                                                                                                                                                                                                                                                |
| 200061      | Authorization page exception                                                                                                                                                                                                                                                                                                 |
| 200064      | Server returned data exception                                                                                                                                                                                                                                                                                               |
| 200072      | CA root certificate validation failed                                                                                                                                                                                                                                                                                        |
| 200082      | Server busy                                                                                                                                                                                                                                                                                                                  |
| 200086      | ppLocation is empty                                                                                                                                                                                                                                                                                                          |
| 200087      | Only used to monitor the successful launch of the authorization page                                                                                                                                                                                                                                                         |
| 200096      | Current network does not support number retrieval                                                                                                                                                                                                                                                                            |

### China Telecom

| Status Code | Status Code Description                                                                 |
| ----------- | --------------------------------------------------------------------------------------- |
| 0           | Request successful                                                                      |
| -64         | Permission-denied (no permission to access)                                             |
| -65         | API-request-rates-Exceed-Limitations (API call limit exceeded)                          |
| -10001      | Number retrieval failed, mdn is empty                                                   |
| -10002      | Parameter error                                                                         |
| -10003      | Decryption failed                                                                       |
| -10004      | IP restricted                                                                           |
| -10005      | Different network number retrieval callback parameter exception                         |
| -10006      | mdn number retrieval failed, and belongs to Telecom network                             |
| -10007      | Redirect to different network number retrieval                                          |
| -10008      | Exceeded preset number retrieval threshold                                              |
| -10009      | Timestamp expired                                                                       |
| -10013      | Perator_unsupported, submit a ticket to contact the engineer                            |
| -20005      | Sign-invalid (signature error)                                                          |
| -20006      | Application does not exist                                                              |
| -20007      | Public key data does not exist                                                          |
| -20100      | Internal parsing error                                                                  |
| -20102      | Encryption parameter parsing failed                                                     |
| -30001      | Timestamp illegal                                                                       |
| -30003      | topClass-invalid, topclass invalid                                                      |
| 51002       | Parameter is empty                                                                      |
| 51114       | Unable to get phone number data                                                         |
| -8001       | Network exception, request failed                                                       |
| -8002       | Request parameter error                                                                 |
| -8003       | Request timeout                                                                         |
| -8004       | Mobile data network not enabled                                                         |
| -8010       | No network connection (network error)                                                   |
| -720001     | Wi-Fi switch to 4G request exception                                                    |
| -720002     | Wi-Fi switch to 4G timeout                                                              |
| 80000       | Request timeout                                                                         |
| 80001       | Network connection failed, network link interrupted, Invalid argument, data connection not currently allowed |
| 80002       | Response code error 404                                                                 |
| 80003       | No network connection                                                                   |
| 80005       | Socket timeout exception                                                                |
| 80007       | IO exception                                                                            |
| 80008       | No route to host                                                                        |
| 80009       | Nodename nor servname provided, or not known                                            |
| 80010       | Socket closed by remote peer                                                            |
| 80800       | Wi-Fi switch timeout                                                                    |
| 80801       | Wi-Fi switch exception                                                                  |
| -9999       | Network fault (networkauth-fail)                                                        |
| -720001     | Switch exception, network unstable when switching traffic card                          |
| -720002     | Switch exception timeout                                                                |

### China Unicom

| Status Code                  | Status Code Description                    |
| ---------------------------- | ------------------------------------------ |
| 0                            | Request successful                         |
| -10008                       | JSON conversion failed                     |
| 1, request timeout           | Request timeout                            |
| 1, private IP number lookup failed | Private IP number lookup failed      |
| 1, private IP verification error | Private IP verification error          |
| 1, source IP authentication failed | Source IP authentication failed      |
| 1, failed to get authentication information | Failed to get authentication information