# SmAuth Number Authentication iOS SDK Integration Guide

## I. Integration Instructions

Development Environment:
* Minimum deployment version: iOS 9.0 and above.
* Xcode development tool: 12.0 and above.

Integration Method:

* Drag `SmAuth.xcframework` into the Xcode project (make sure to check the Copy items if needed option).
* Click on the project target -> General -> Frameworks, Libraries, and Embedded Content -> SmAuth.xcframework, and under the Embed option, select `Embed & Sign`.

-------

**One-click Login Steps:**

* Import the header file
```oc
#import <SmAuth/SmAuth.h>
```

* Initialize the SDK
```oc
// AppId
[[SmAuthHelper shareInstance] registerWithAppId:@"xxxxx"];
// Enable logging
[[SmAuthHelper shareInstance] consolePrintLoggerEnable:YES];
// Set timeout
[[SmAuthHelper shareInstance] setTimeoutInterval:6000];
```

* Preload the authorization page (fetch number)
```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

* Load the authorization page
```oc
- (void)loadLoginAuthorizePageWithViewConfig:(UAuthViewConfig *)viewConfig completion:(void (^)(NSError *error, NSString *token))completion;
```

* Close the authorization page
```oc
- (void)unloadLoginAuthorizePageAnimated:(BOOL)animated completion:(void (^_Nullable)(void))completion;
```

-------

**Local Number Verification Steps:**

* Preload the authorization page (fetch number)
```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

* Phone number detection (get verification token)
```oc
- (void)verifyPhoneNumberCompletion:(void(^)(NSError * _Nullable, NSString * _Nullable))completion;
```

## II. SDK Interface Description

#### 2.1 Initialize the SDK

**Interface Description** 

>  Initialize the SDK.

**Interface Example**

```oc
[[SmAuthHelper shareInstance] registerWithAppId:@"xxxxx"];
```

**Parameter Description**

> appId: The ID provided by SmAuth Number Authentication.

-------

#### 2.2 Preload the Authorization Page (Fetch Number)

**Interface Description** 

>  Preload the authorization page. Before using, check carrier information, network information, etc.

**Interface Example**

```oc
- (void)preLoadLoginAuthorizePageWithCompletion:(void(^)(NSError * _Nullable))completion;
```

**Callback Description**

> Error codes: `200027`-->Cellular network not enabled; `-10003`-->Cellular network permission not enabled; other error codes refer to the carrier error code list.

-------

#### 2.3 Load the Authorization Page

**Interface Description** 

>  Load the authorization page.

**Interface Example**

```oc
- (void)loadLoginAuthorizePageWithViewConfig:(UAuthViewConfig *)viewConfig completion:(void (^)(NSError *error, NSString *token))completion;
```

**Parameter Description**

> viewConfig: Authorization page configuration model.

**Callback Description**

> Error codes: Refer to the carrier error code list.
> token: Fetch number token, use this token to exchange for the complete phone number in the developer's application backend.

-------

#### 2.4 Close the Authorization Page

**Interface Description** 

>  Close the authorization page.

**Interface Example**

```oc
- (void)unloadLoginAuthorizePageAnimated:(BOOL)animated completion:(void (^_Nullable)(void))completion;
```

**Parameter Description**

> animated: Whether to enable animation.

-------

#### 2.5 Local Number Verification Token Retrieval

**Interface Description** 

>  Retrieve the local number verification token.

**Interface Example**

```oc
- (void)verifyPhoneNumberCompletion:(void(^)(NSError * _Nullable, NSString * _Nullable))completion;
```

**Callback Description**

> Error codes: Refer to the carrier error code list.
> token: Local number verification token, use this token to verify if it is the local phone number in the developer's application backend.

-------

#### 2.6 Log Printing

**Interface Description** 

>  Log printing.

**Interface Example**

```oc
- (void)consolePrintLoggerEnable:(BOOL)enable;
```

**Parameter Description**

> enable: Whether to enable console log printing.

-------

#### 2.7 Timeout Setting

**Interface Description** 

>  Timeout setting.

**Interface Example**

```oc
- (void)setTimeoutInterval:(NSTimeInterval)interval;
```

**Parameter Description**

> interval: Unit in milliseconds, recommended to set to 8000 milliseconds.

-------

#### 2.8 Carrier Type

**Interface Description** 

>  Carrier type.

**Interface Example**

```oc
- (SmCarrierType)carrierType;
```

**Parameter Description**

> SmCarrierType: 

`SmCarrierTypeUnknown`-->Unknown;

`SmCarrierTypeMobile`-->China Mobile; 

`SmCarrierTypeUnicome`-->China Unicom;

`SmCarrierTypeTelecom`-->China Telecom.

-------

#### 2.9 Network Connection Type

**Interface Description** 

>  Current network connection type.

**Interface Example**

```oc
- (SmNetworkType)networkType;
```

**Parameter Description**

> SmNetworkType:

`SmNetworkTypeUnknown`-->Unknown

`SmNetworkTypeNotReachable`-->No network

`SmNetworkTypeCellular`-->Cellular data

`SmNetworkTypeWifi`-->WiFi

`SmNetworkTypeCellularAndWifi`-->Cellular data + WiFi

-------

#### 2.10 SDK Version

**Interface Description** 

>  Current SDK version.

**Interface Example**

```oc
- (NSString *)getSDKVersion;
```

## III. Other Notes

For the usage of other interfaces, please refer to the examples in the Demo.        

## IV. Carrier SDK Error Codes
> When calling the number authentication service for one-click login and local number verification functions, the corresponding carrier interface error codes will be returned. Common interface error codes and descriptions can be referred to below

### China Mobile

| Status Code | Status Code Description                                                                                                                                                                                                                                                                                                      |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 103000      | Success                                                                                                                                                                                                                                                                                                                      |
| 102101      | No network                                                                                                                                                                                                                                                                                                                   |
| 102102      | Network exception (network request error, generally occurs when network security policy restricts the use of http, device has proxy enabled, or connected WiFi has network link security policy restrictions, it is recommended to analyze specifically with SDK logs)                                                                                             |
| 102103      | Data network not enabled                                                                                                                                                                                                                                                                                                     |
| 102121      | User canceled login                                                                                                                                                                                                                                                                                                          |
| 102203      | Input parameter error                                                                                                                                                                                                                                                                                                        |
| 102223      | Data parsing exception                                                                                                                                                                                                                                                                                                       |
| 102507      | Login timeout (when clicking the login button on the authorization page)                                                                                                                                                                                                                                                     |
| 102508      | Data network switch failed                                                                                                                                                                                                                                                                                                   |
| 103101      | Request signature error (request signature error (if it occurs on the client side, it may be that the appkey is wrong, check if it is confused with appsecret, or there is a space. If it occurs on the server-side interface, check whether the verification method is MD5 or RSA, if it is MD5, check the signType field, if it is appsecret, confirm whether the appkey is mistakenly used for signing. If it is RSA, check whether the private key used corresponds to the reported public key and whether the message splicing meets the document requirements.)) |
| 103102      | Package signature Bundle ID error (the reported one does not match the actual one used)                                                                                                                                                                                                                                      |
| 103103      | User does not exist                                                                                                                                                                                                                                                                                                          |
| 103111      | Gateway IP error (check if VPN is enabled or if it is a foreign IP)                                                                                                                                                                                                                                                          |
| 103119      | AppID does not exist. (Check if the appid passed is correct or if there is a space)                                                                                                                                                                                                                                          |
| 103211      | Other errors (commonly seen in incorrect message format, first check if it meets these three requirements: a. JSON format message interaction must be standard JSON format; b. When sending, set content type to `application/json`; c. Parameter types are all String)                                                                                              |
| 103412      | Invalid request with encryption method error, non-JSON format, and empty request                                                                                                                                                                                                                                             |
| 103414      | Parameter validation exception                                                                                                                                                                                                                                                                                               |
| 103511      | Server IP whitelist validation failed                                                                                                                                                                                                                                                                                        |
| 103811      | Token is empty                                                                                                                                                                                                                                                                                                               |
| 103902      | Repeated login in a short time, causing script failure                                                                                                                                                                                                                                                                       |
| 103911      | Token request too frequent, no more than 30 tokens obtained within 10 minutes and not used                                                                                                                                                                                                                                   |
| 104201      | Token repeated validation failed, invalid or does not exist                                                                                                                                                                                                                                                                  |
| 105002      | Mobile number fetching failed (generally IoT card)                                                                                                                                                                                                                                                                           |
| 105013      | Unicom number fetching not supported                                                                                                                                                                                                                                                                                         |
| 105018      | Used local number verification token to fetch number, causing insufficient token permissions                                                                                                                                                                                                                                 |
| 105019      | Application not authorized                                                                                                                                                                                                                                                                                                   |
| 105021      | Reached daily number fetching limit                                                                                                                                                                                                                                                                                          |
| 105302      | AppID not in whitelist                                                                                                                                                                                                                                                                                                       |
| 105313      | Illegal request                                                                                                                                                                                                                                                                                                              |
| 200002      | Phone does not have a SIM card installed                                                                                                                                                                                                                                                                                      |
| 200005      | User not authorized (READ_PHONE_STATE)                                                                                                                                                                                                                                                                                       |
| 200006      | User not authorized (SEND_SMS)                                                                                                                                                                                                                                                                                               |
| 200007      | authType only uses SMS verification code authentication                                                                                                                                                                                                                                                                      |
| 200008      | 1. authType parameter is empty; 2. authType parameter is invalid;                                                                                                                                                                                                                                                            |
| 200009      | Application legality verification failed (package name and package signature not filled correctly)                                                                                                                                                                                                                           |
| 200010      | Unable to recognize SIM card or no SIM card (Android)                                                                                                                                                                                                                                                                        |
| 200012      | Number fetching failed, switch to SMS verification login                                                                                                                                                                                                                                                                     |
| 200013      | SMS uplink sending SMS failed (SMS uplink)                                                                                                                                                                                                                                                                                   |
| 200014      | Phone number format error (SMS verification)                                                                                                                                                                                                                                                                                 |
| 200015      | SMS verification code format error                                                                                                                                                                                                                                                                                           |
| 200016      | KS update failed                                                                                                                                                                                                                                                                                                             |
| 200017      | Non-mobile card does not support SMS uplink                                                                                                                                                                                                                                                                                  |
| 200018      | Gateway login not supported                                                                                                                                                                                                                                                                                                  |
| 200019      | SMS verification login not supported                                                                                                                                                                                                                                                                                         |
| 200020      | User canceled login                                                                                                                                                                                                                                                                                                          |
| 200021      | Data parsing exception                                                                                                                                                                                                                                                                                                       |
| 200022      | No network                                                                                                                                                                                                                                                                                                                   |
| 200023      | Request timeout                                                                                                                                                                                                                                                                                                              |
| 200024      | Data network switch failed                                                                                                                                                                                                                                                                                                   |
| 200025      | Location error (generally thread capture exception, socket, system not authorized mobile data network permission, etc., please submit a ticket to contact the engineer)                                                                                                                                                       |
| 200026      | Input parameter error.                                                                                                                                                                                                                                                                                                       |
| 200027      | Mobile data network not enabled or network unstable                                                                                                                                                                                                                                                                          |
| 200028      | Network request error                                                                                                                                                                                                                                                                                                        |
| 200029      | Request error, last request not completed                                                                                                                                                                                                                                                                                    |
| 200030      | No startup parameters                                                                                                                                                                                                                                                                                                        |
| 200031      | Token generation failed                                                                                                                                                                                                                                                                                                      |
| 200032      | KS cache does not exist                                                                                                                                                                                                                                                                                                      |
| 200033      | Middleware reuse token retrieval failed                                                                                                                                                                                                                                                                                      |
| 200034      | Pre-fetch token invalid                                                                                                                                                                                                                                                                                                      |
| 200035      | KS negotiation failed                                                                                                                                                                                                                                                                                                        |
| 200036      | Pre-fetch number failed                                                                                                                                                                                                                                                                                                      |
| 200037      | Unable to get openid                                                                                                                                                                                                                                                                                                         |
| 200038      | Cross-network number fetching network request failed                                                                                                                                                                                                                                                                         |
| 200039      | Cross-network number fetching gateway number fetching failed                                                                                                                                                                                                                                                                 |
| 200040      | UI resource loading exception                                                                                                                                                                                                                                                                                                |
| 200042      | Authorization page popup exception                                                                                                                                                                                                                                                                                           |
| 200048      | User does not have a SIM card installed                                                                                                                                                                                                                                                                                      |
| 200050      | EOF exception                                                                                                                                                                                                                                                                                                                |
| 200061      | Authorization page exception                                                                                                                                                                                                                                                                                                 |
| 200064      | Server returned data exception                                                                                                                                                                                                                                                                                               |
| 200072      | CA root certificate validation failed                                                                                                                                                                                                                                                                                        |
| 200082      | Server busy                                                                                                                                                                                                                                                                                                                  |
| 200086      | ppLocation is empty                                                                                                                                                                                                                                                                                                          |
| 200087      | Only used to monitor successful authorization page launch                                                                                                                                                                                                                                                                    |
| 200096      | Current network does not support number fetching                                                                                                                                                                                                                                                                             |

### China Telecom

| Status Code | Status Code Description                                                         |
| ----------- | ------------------------------------------------------------------------------- |
| 0           | Request successful                                                              |
| -64         | Permission-denied (no permission to access)                                     |
| -65         | API-request-rates-Exceed-Limitations (API call limit exceeded)                  |
| -10001      | Number fetching failed, mdn is empty                                            |
| -10002      | Parameter error                                                                 |
| -10003      | Decryption failed                                                               |
| -10004      | IP restricted                                                                   |
| -10005      | Cross-network number fetching callback parameter exception                      |
| -10006      | mdn number fetching failed, and belongs to Telecom network                      |
| -10007      | Redirect to cross-network number fetching                                       |
| -10008      | Exceeded preset number fetching threshold                                       |
| -10009      | Timestamp expired                                                               |
| -10013      | Perator_unsupported, submit a ticket to contact the engineer                    |
| -20005      | Sign-invalid (signature error)                                                  |
| -20006      | Application does not exist                                                      |
| -20007      | Public key data does not exist                                                  |
| -20100      | Internal parsing error                                                          |
| -20102      | Encryption parameter parsing failed                                             |
| -30001      | Illegal timestamp                                                               |
| -30003      | topClass-invalid, topclass invalid                                              |
| 51002       | Parameter is empty                                                              |
| 51114       | Unable to get phone number data                                                 |
| -8001       | Network exception, request failed                                               |
| -8002       | Request parameter error                                                         |
| -8003       | Request timeout                                                                 |
| -8004       | Mobile data network not enabled                                                 |
| -8010       | No network connection (network error)                                           |
| -720001     | Wi-Fi switch to 4G request exception                                            |
| -720002     | Wi-Fi switch to 4G timeout                                                      |
| 80000       | Request timeout                                                                 |
| 80001       | Network connection failed, network link interrupted, Invalid argument, data connection not allowed currently |
| 80002       | Response code error 404                                                         |
| 80003       | No network connection                                                           |
| 80005       | Socket timeout exception                                                        |
| 80007       | IO exception                                                                    |
| 80008       | No route to host                                                                |
| 80009       | Nodename nor servname provided, or not known                                    |
| 80010       | Socket closed by remote peer                                                    |
| 80800       | Wi-Fi switch timeout                                                            |
| 80801       | Wi-Fi switch exception                                                          |
| -9999       | Network failure (networkauth-fail)                                              |
| -720001     | Switch exception, network unstable when switching data card                     |
| -720002     | Switch exception timeout                                                        |

### China Unicom

| **Status Code**              | **Status Code Description**            |
| ---------------------------- | -------------------------------------- |
| 0                            | Request successful                     |
| -10008                       | JSON conversion failed                 |
| 1, request timeout           | Request timeout                        |
| 1, private network IP number fetching failed | Private network IP number fetching failed |
| 1, private network IP validation error | Private network IP validation error |
| 1, source IP authentication failed | Source IP authentication failed   |
| 1, failed to get authentication information | Failed to get authentication information |
| 1, failed to get phone authorization code | Generally due to SDK request timeout |
| 1, gateway number fetching failed | Gateway number fetching failed     |
| 1, network request failed    | Network request failed                 |
| 1, signature validation failed | Signature validation failed           |
| 1, passed code and AppID do not match | Passed code and AppID do not match |
| 1, seems to have disconnected from the internet | Network instability causing disconnection |
| 1, connect address error     | Connection address error, generally due to timeout |
| 1, select socket error       | Select socket error                    |
| 1, handshake failed          | Handshake failed                       |
| 1, decode ret_url fail       | URL decoding failed                    |
| 1, connect error             | Connection error                       |
| 2, request timeout           | Interface request time exceeds timeout setting |
