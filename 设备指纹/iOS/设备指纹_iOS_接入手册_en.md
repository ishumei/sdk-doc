Shumei Device Fingerprint SDK (i.e., 'smsdk') supports a minimum of iOS 9.

## 1 Project Configuration

### 1.1 Import smsdk Package
The new version of smsdk uses the .xcframework format package, and the import method is as follows:

  1. Copy SmAntiFraud.xcframework into the project

  2. Open the project with Xcode, click on the project, click on the corresponding target, and select the General page.

  3. In Frameworks, Libraries, and Embedded Content, click the + sign -> Add Other -> Add Files -> select SmAntiFraud.xcframework, and choose `Do Not Embed` for Embed.

  4. After importing the smsdk package, reference it in the code via `#import`:

     ```objective-c
     #import <SmAntiFraud/SmAntiFraud.h>
     ```

### 1.2 Add smsdk Dependencies

In TARGETS -> General -> Frameworks, Libraries, and Embedded Content, import the following dependency library

   - `IOKit.framework`

### 1.3 HTTP Settings
smsdk uses HTTP requests by default. According to Apple's ATS standards, you need to configure Info.plist:

   1. Click on the project's Info.plist, click the + sign, and select App Transport Security Settings.
   2. In the App Transport Security Settings configuration item, click the + sign, select Allow Arbitrary Loads, and set it to YES.

### 1.4 IDFA Configuration
If the APP has not previously collected `IDFA`, when submitting to the App Store, you need to specify the use of `IDFA` in the app and explain the reason according to App Connect's policy.

   If you do not want to use `IDFA`, you can refer to the SDK startup section below and set `notCollect` to not collect `IDFA`.

   If the app belongs to the children's category, contact Shumei operations to provide an SDK that does not include `IDFA` collection code.

### 1.5 Configure Privacy File

According to [Apple's privacy policy regulations](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files?language=objc), apps embedding Shumei SDK need to complete the terms in the PrivacyInfo.xcprivacy of the Xcode project. If the project does not have it, you need to create a new PrivacyInfo.xcprivacy file using Xcode 15 or above according to the [official instructions](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files?language=objc).

Refer to either of the following two methods to add the privacy terms dependent on Shumei SDK:

- Using the default Property List configuration

  1. Select PrivacyInfo.xcprivacy in Xcode.

  2. Add the following terms to `Privacy Nutrition Label Types` and `Privacy Accessed API Types` in PrivacyInfo.xcprivacy, as shown in the figure below:

     ![privacy](./res/privacy.png)

- Using Source Code configuration

  1. Right-click PrivacyInfo.xcprivacy in Xcode, select Open As, and choose Source Code.

  2. Add NSPrivacyCollectedDataTypes related terms: paste the following text under the key value `NSPrivacyCollectedDataTypes` within the outermost `<dict>` and `</dict>`. If there is no such key value, you need to create it first.

     ```xml
     <key>NSPrivacyCollectedDataTypes</key>
     	<array>
     		<dict>
     			<key>NSPrivacyCollectedDataType</key>
     			<string>NSPrivacyCollectedDataTypeDeviceID</string>
     			<key>NSPrivacyCollectedDataTypeLinked</key>
     			<false/>
     			<key>NSPrivacyCollectedDataTypeTracking</key>
     			<true/>
     			<key>NSPrivacyCollectedDataTypePurposes</key>
     			<array>
     				<string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
     			</array>
     		</dict>
     		<dict>
     			<key>NSPrivacyCollectedDataType</key>
     			<string>NSPrivacyCollectedDataTypeFitness</string>
     			<key>NSPrivacyCollectedDataTypeLinked</key>
     			<false/>
     			<key>NSPrivacyCollectedDataTypeTracking</key>
     			<false/>
     			<key>NSPrivacyCollectedDataTypePurposes</key>
     			<array>
     				<string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
     			</array>
     		</dict>
     	</array>
     ```

  3. Add NSPrivacyAccessedAPITypes related terms: paste the following text under the key value `NSPrivacyAccessedAPITypes` within the outermost `<dict>` and `</dict>`. If there is no such key value, you need to create it first.

     ```xml
     <key>NSPrivacyAccessedAPITypes</key>
     	<array>
     		<dict>
     			<key>NSPrivacyAccessedAPITypeReasons</key>
     			<array>
     				<string>35F9.1</string>
     			</array>
           <key>NSPrivacyAccessedAPIType</key>
           <string>NSPrivacyAccessedAPICategorySystemBootTime</string>
     		</dict>
     		<dict>
     			<key>NSPrivacyAccessedAPITypeReasons</key>
     			<array>
     				<string>C617.1</string>
     			</array>
     			<key>NSPrivacyAccessedAPIType</key>
     			<string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
     		</dict>
     		<dict>
     			<key>NSPrivacyAccessedAPITypeReasons</key>
     			<array>
     				<string>CA92.1</string>
     			</array>
     			<key>NSPrivacyAccessedAPIType</key>
     			<string>NSPrivacyAccessedAPICategoryUserDefaults</string>
     		</dict>
     	</array>
     ```

## 2 Standard Integration

### 2.1 Start SDK

smsdk is the terminal in Shumei's risk control system, mainly used for collecting device information and generating device identifiers. When the product needs to perform risk control analysis on related businesses, it can use the `create` method of the `SmAntiFraud` class for risk control. **Note** that calling the `create` method will immediately collect device information, so it **must** be called after agreeing to the privacy policy and in scenarios that require risk control to avoid collecting unnecessary information in unreasonable scenarios.

Call the `-[SmAntiFraud create:]` method to start the SDK.

Starting the SDK does not block the current thread, it will collect data and transmit it over the network, caching the `deviceId`. The timing of the call is as follows:

1. The first time the APP starts, call it after agreeing to the privacy policy.
2. When the APP starts for the non-first time and the privacy policy has been agreed to, call it at startup.

If the `create` method returns `NO`, you need to check whether the parameters for starting the SDK are correct. You can filter the logs with `Smlog` for self-check.

Starting the SDK requires an `SmOption` instance, with specific parameters as follows:

| **Field**          | **Parameter Type**           | **Required** | **Default Value**                                      | **Field Description**                                                 |
| ----------------- | ---------------------- | ------------ | ----------------------------------------------- | ------------------------------------------------------------ |
| organization      | NSString*              | Yes           | null                                            | Company identifier assigned by Shumei, viewable in the Shumei backend                         |
| appId             | NSString*              | Yes           | null                                            | Application identifier, used to distinguish different applications, viewable in the Shumei backend                     |
| publicKey         | NSString*              | Yes           | null                                            | Public key identifier, found in the publicKey item of the account opening email                      |
| url               | NSString*              | No           | http://fp-it.fengkongcloud.com/deviceprofile/v4 | URL for uploading device data                                            |
| confUrl           | NSString*              | No           | http://fp-it.fengkongcloud.com/v3/cloudconf     | URL for requesting cloud configuration                                                |
| extraInfo         | NSString*              | No           | null                                            | Additional information                                                     |
| cloudConf         | BOOL                   | No           | YES                                             | Whether to enable cloud configuration, YES means enabled                                |
| transport         | BOOL                   | No           | YES                                             | Whether to enable device fingerprint function, YES means enabled                            |
| usingShortBoxData | BOOL                   | No           | NO                                              | Whether to use shorter boxData, NO means not using                          |
| delegate          | `id<ServerSmidProtocol>` | No           | null                                            | Object implementing `ServerSmidProtocol` to asynchronously obtain identifiers   |
| notCollect        | NSArray<NSString*>     | No           | null                                            | Set items not to be collected by the SDK, currently only supports "idfa"                            |
| area              | SmAntiFraudArea        | No           | AREA_BJ                                         | Server room address for data upload and cloud configuration.<br />AREA_BJ: Beijing server room<br />AREA_XJP: Singapore server room<br />AREA_FJNY: Virginia server room |
| useHttps | BOOL | No | NO | Whether to use HTTPS protocol for network requests |

### 2.2 Get Identifier

The client obtains the identifier as `boxId` and `boxData`, **both of which will change**. If a unique and unchanging identifier is needed, refer to the "Decryption Tool and Proxy Server Instructions Device Fingerprint Identifier Decryption" section to learn how to obtain the plaintext device identifier.

- Synchronous Method

  Call the `-[SmAntiFraud getDeviceId]` method to get the device identifier. The timing of the call is as follows:

  1. 1-2 seconds after calling the `create` method to start the SDK and it returns `YES`. This time allows the SDK to collect data and transmit it over the network.
  2. Use it during business events, such as reporting the string returned by `getDeviceId` during key events like login and registration.

- Asynchronous Method

  The asynchronous method can obtain the latest boxId issued by the server at the earliest time after starting the SDK.

  You need to call `-[SmOption setDelegate:]` to set an instance object that implements the `ServerSmidProtocol` interface, as follows:

  1. Inherit the `ServerSmidProtocol` interface, such as `@interface ViewController () <ServerSmidProtocol>`

  2. Implement the `smOnSuccess` and `smOnError` methods of the interface, such as:

     ```objective-c
     - (void)smOnSuccess:(NSString*) boxId {
       // Server issued successfully or there is a usable boxId in the cache
     }
     
     - (void)smOnError:(NSInteger) errCode {
       // Server failed to issue identifier, error code: errCode
       // Try using the `-[SmAntiFraud getDeviceId]` method to actively obtain the identifier
       // Shumei does not handle errCode, business parties need to record, handle, and analyze it themselves
     }
     ```

  3. Set the `delegate` in the `SmOption` object, such as `[option setDelegate:self]`

  The `errCode` error codes are listed as follows:
  
  | errCode | **Description**                                                     |
  | ------- | ------------------------------------------------------------ |
  | -1      | No network, common reason: device has no network                                 |
  | -2      | Network exception, network connection exception or HTTP status is not 200, common reason: proxy or private server configuration error |
  | -3      | Business exception, business status code issued is not 1100, server did not return deviceId, common reason: parameter configuration error, QPS limit exceeded, server exception |
  | -4      | Unknown error                                                     |

### 2.3 Code Example

```objective-c
// SDK startup parameter object
SmOption *option = [[SmOption alloc] init];
[option setOrganization: @"YOUR_ORGANIZATION"];// Required
[option setAppId:@"YOUR_APP_ID"];							 // Required
[option setPublicKey:@"YOUR_PUBLICK_KEY"];		 // Required

// Start SDK
BOOL isOk = [[SmAntiFraud shareInstance] create:option];
if (!isOk) {
	// Check if the option startup SDK parameters are set correctly
}
```

### 2.4 Integration Verification

Refer to the "Testing" section to self-check whether the SDK integration is successful.

## 3 Overseas Business Integration

Overseas business needs to be integrated in proxy mode. If proxy services cannot be provided, Shumei's overseas server nodes can be used. The main steps are the same as standard integration, with the following additional configurations:

1. Business server room in Europe and America (Virginia server room)

   ```objective-c
   // User distribution range is Europe and America
   [option setArea:AREA_FJNY];
   // If the user distribution range is global, acceleration needs to be enabled, configured as follows
   // [option setArea:AREA_FJNY];
   // NSString* host = @"http://fp-na-it-acc.fengkongcloud.com";
   // [option setUrl:[host stringByAppendingString:@"/deviceprofile/v4"]];
   // [option setConfUrl:[host stringByAppendingString:@"/v3/cloudconf"]];
   ```

2. Business server room in Europe and America (Frankfurt server room)

   ```objective-c
   NSString* host = @"http://api-device-eur.fengkongcloud.com";
   [option setUrl:[host stringByAppendingString:@"/deviceprofile/v4"]];
   [option setConfUrl:[host stringByAppendingString:@"/v3/cloudconf"]];
   ```

3. Business server room in Southeast Asia (Singapore server room)

   ```java
   // User distribution range is Southeast Asia
   [option setArea:AREA_XJP];
   // If the user distribution range is global, acceleration needs to be enabled, configured as follows
   // [option setArea:AREA_XJP];
   // NSString* host = @"http://fp-sa-it-acc.fengkongcloud.com";
   // [option setUrl:[host stringByAppendingString:@"/deviceprofile/v4"]];
   // [option setConfUrl:[host stringByAppendingString:@"/v3/cloudconf"]];
   ```

## 4 Private Integration

The main steps are similar to standard integration, with the following additional configurations:

```objective-c
// Set private address, replace private-host with the private host name (domain name)
NSString *host = @"http://private-host"; 
[option setUrl: [host stringByAppendingString:@"/deviceprofile/v4"]]; // Example path, needs to be consistent with the real scenario
[option setConfUrl:[host stringByAppendingString:@"/v3/cloudconf"]]; // Example path, needs to be consistent with the real scenario
```

Note that if the host passed in is an HTTP request, such as `http://private-host`, you need to ensure that the APP can send HTTP requests. Refer to the HTTP settings part of the "Project Configuration" section. After completing the private integration, you need to perform self-checks according to the "Testing" section.

## 5 Proxy Integration

The main steps are similar to standard integration, with the following additional configurations:

```java
// Set private address, replace host with the proxy server's host name (domain name)
String host = "https://proxy-host";
[option setUrl: [host stringByAppendingString:@"/deviceprofile/v4"]]; // Example path, needs to be consistent with the real scenario
[option setConfUrl:[host stringByAppendingString:@"/v3/cloudconf"]]; // Example path, needs to be consistent with the real scenario
```

Developers need to set up the proxy server themselves. For proxy server-related processing, refer to the "Proxy Server Instructions Proxy Integration" section.

## 6 Testing

1. Call the `[[SmAntiFraud shareInstance] create:option]` method and get a return value of `YES`.
2. Call the `[[SmAntiFraud shareInstance] getDeviceId]` method and get a return value of an 89-character boxId, such as `Bm21V93t5QwTNdwyQxxxxxRYuSnOuwwylqZvz8Lixxxxx17lRMqcQ1jz9RwN6qW31/Z0YYmxN8KQnrya9xxxxxx==`.
3. Check if there are any Smlog exception outputs in the console. If there are exception outputs, modify them according to the prompts.
4. In the Shumei management backend, select "Device Risk Trend" in the navigation bar, find the "Device Details" section, and check if there is data reporting (there may be a delay, generally no more than 30 minutes).
5. If the test cannot be passed, contact Shumei staff for troubleshooting.