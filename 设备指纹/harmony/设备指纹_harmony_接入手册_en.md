Shumei Device Fingerprint SDK (i.e., 'smsdk') compatibleSdkVersion 10

## 1 Project Configuration

This section provides the DevEco Studio project configuration steps.

1. Copy `smsdk_x.x.x.har` to the libs directory of the Module (e.g., entry module). If there is no libs directory, create a new libs folder, or place it in another location (just ensure the subsequent dependency path is consistent).

2. Add the har reference in the oh-package.json5 file in the entry directory.

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
       "@ohos/library": "file:./libs/smsdk_x.x.x.har" // The version number needs to be consistent with the actual imported smsdk version
     }
   }
   ```

3. Declare permissions by adding the following permissions in module.json5.

   ```xml
   "requestPermissions": [
       {
         // Internet permission, mandatory
         "name": "ohos.permission.INTERNET",
       },
       {
         // Accelerometer sensor, optional
         "name": "ohos.permission.ACCELEROMETER",
       },
       {
         // Gyroscope sensor, optional
         "name": "ohos.permission.GYROSCOPE",
       },
       {
         // WiFi status permission, optional
         "name": "ohos.permission.GET_WIFI_INFO",
       },
       {
         // Network status permission, optional
         "name": "ohos.permission.GET_NETWORK_INFO",
       },
       {
         // Advertising ID permission, optional
         "name": "ohos.permission.APP_TRACKING_CONSENT"
       }
     ]
   }
   ```

   Permission purposes

   | Permission                | Purpose                                                     |
   | ------------------------- | ----------------------------------------------------------- |
   | INTERNET (mandatory)      | Send collected data to the server via the network           |
   | GET_NETWORK_INFO (mandatory) | Determine if the network is connected; <br/>Get network connection status information, such as 2g, 3g, 4g, wifi, etc., and carrier information |
   | GET_WIFI_INFO             | Get bssid, ssid, band, wifiip information                   |
   | ACCELEROMETER             | Get accelerometer sensor information                        |
   | GYROSCOPE                 | Get gyroscope sensor information                            |
   | APP_TRACKING_CONSENT      | Get OAID information                                        |


## 2 Standard Integration

Minimal initialization. After successful initialization, data collection will start immediately, so developers need to ensure **not** to call the `create` method before agreeing to the privacy policy.

```java
// Initialize parameter object
let option = new SmOption();
// Mandatory, organization identifier
option.organization = "YOUR_ORGANIZATION"
// Mandatory, application identifier, view in Shumei backend application management, if no suitable value, you can write "default"
option.appId = "YOUR_APP_ID"
// Mandatory, encryption KEY, content of harmony_public_key attachment in the email
option.publicKey = "YOUR PUBLICK_KEY"
// Optional, cancel the collection of certain fields through notCollect, supports the following fields (case-sensitive): 'bssid', 'ssid', 'wifiIp', 'sensorsData', 'sensor', 'oaid', 'battery', 'band'
// For example, do not collect bssid and wifiIp fields
// option.notCollect = new Set(['bssid','wifiIp']) 
// option.usingHttp = true // Use https protocol network request, default false
// Initialization
SmAntiFraud.create(context as common.UIAbilityContext, option).then(boxId => {
  // Success callback, get boxId
}).catch(err => {
  // Failure callback, this method may never be triggered
  //   Get error code: err.code, see "Error Codes" section for corresponding handling methods
  //   Get error message: err.message
})
  
// Get identifier, need to call after initialization (create)
SmAntiFraud.getDeviceId().then(box => {
	// Success callback, get boxId or boxData
}).catch(err => {
	// Failure callback
  //   Get error code: err.code, see "Error Codes" section for corresponding handling methods
  //   Get error message: err.message
})
```



Initialization timing

1. First launch of the APP, call after agreeing to the privacy policy
2. Non-first launch of the APP, and agreed to the privacy policy, call at startup
3. Each call to the create method will collect device data once, avoid frequent collection due to repeated calls

The `create` method takes about 200 milliseconds to collect data (test data), the collection process occurs in a sub-thread and will not block the current thread. After initialization, you can call `SmAntiFraud.getDeviceId` to get the identifier.

Timing to get the identifier:

1. Call after the `create` method
2. Use when reporting business events, such as reporting the string returned by `getDeviceId` in key events like login, registration, etc.

Within the same App activity cycle (from app launch to termination), the device identifier may change, such as from localId to serverId. The following is the main process of the sdk.

![smsdk-flow](./res/smsdk-flow-v3.png)

When the application is first launched, after initialization, if all settings and network are normal, steps 2, 3, 4, and 5 will be completed in sequence, and the boxId issued by the server will be obtained; if there is an abnormal situation and the boxId cannot be obtained, the identifier obtained during the App's activity cycle will always be boxData, which is the encrypted value of the full data and is relatively long. Pay attention to avoid the issue of boxData being truncated due to length restrictions when reporting.

The above is the standard integration of smsdk in China. If there are no customized requirements, the integration is complete at this point. It is recommended to refer to the "Testing" section to check whether the integration is successful.

Overseas standard integration (due to data compliance requirements, it is recommended that all overseas integrations use the proxy mode, this is for customers who cannot provide proxy services), switch the device fingerprint data center, set as follows

1. Business data center in Europe and America (Virginia data center)

   ```java
   // User distribution range is Europe and America
   option.area = Area.fjny
   ```

2. Business data center in Europe and America (Frankfurt data center)

   ```java
   option.area = Area.flkf
   ```

3. Business data center in Southeast Asia (Singapore data center)

   ```java
   // User distribution range is Southeast Asia
   option.area = Area.xjp
   ```

## 3 Private Integration

The main steps are similar to standard integration, with the following additional configuration

```java
// Set private address, replace private-host with the private host name (domain name)
option.url = "https://private-host/deviceprofile/v4" // Example path, needs to be consistent with the real scenario
```

Note, if the host passed in is an http request, such as `http://private-host`, ensure that the APP can send http requests.

## 4 Proxy Integration

The logic settings on the smsdk side are similar to standard integration, with the following additional configuration

```java
// Set proxy address, replace host with the proxy server's host name (domain name)
option.url = "https://host/deviceprofile/v4" // Example path, needs to be consistent with the real scenario
```

Developers need to set up the proxy server themselves. For proxy server related processing, refer to the "Decryption Tool and Proxy Server Instructions Proxy Integration" section.

## 5 Testing

1. Calling the `SmAntiFraud.create` method can trigger a successful callback and obtain the smid.
2. Calling the `SmAntiFraud.getDeviceId` method can obtain the identifier without any exceptions.
3. In the Shumei management backend, select "Device Risk Trend" in the navigation bar, find the "Device Details" section, and check if there is data reporting (there may be a delay, generally no more than 30 minutes).
4. If the test cannot be passed, contact Shumei staff for troubleshooting.

## 6 Error Codes

| Error Code | Meaning     | Explanation                                                 | Solution                                                     |
| :--------- | :---------- | :---------------------------------------------------------- | :----------------------------------------------------------- |
| -1         | Unknown     | This error theoretically should not occur                   | If found, provide the message to Shumei technical support for troubleshooting |
| 2102       | Network Issue | Timeout or other network issues like 404/503, Saas service will not have this issue, generally occurs in proxy or private integration | Troubleshoot yourself or send to Shumei technical support for troubleshooting |
| 2103       | Internal Error | System does not support the current operation, encryption/decryption failure | Provide device information and error message to Shumei technical support for troubleshooting |
| 2200       | Parameter Error | Parameter error detected by the client, causing data to not be reported to the server, such as key issues, mandatory parameters being empty, etc. | Check if the parameters are passed in according to the integration document |
| 1902       | Parameter Error | Parameter error detected by the server                   | Is there code obfuscation?                                    |
| 1903       | Service Failure | Server-side issue                                        | Retry or save the requestId and send to Shumei technical support for troubleshooting |
| 1901       | QPS Limit Exceeded |                                                      | Notify Shumei technical staff to increase QPS                 |
| 9101       | Service Unavailable | org error or service not enabled                     | Check if the initialization parameter organization has spelling errors, if no issues, send to Shumei support for troubleshooting. |