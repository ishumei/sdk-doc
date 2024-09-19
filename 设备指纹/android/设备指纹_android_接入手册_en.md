Shumei Device Fingerprint SDK (i.e., 'smsdk') supports a minimum API Level of 14.

## 1 Project Configuration

This section provides the configuration steps for Android Studio projects. Developers using other IDEs, such as IntelliJ IDEA, Eclipse, etc., need to convert based on the description in this section. For cross-platform configurations, such as Flutter, uniapp, unity3d, etc., you can add plugins for integration or contact us for corresponding plugin demos.

1. Copy smsdk*.aar to the libs directory of the Module (e.g., app module).

2. Add aar reference in build.gradle

   ```groovy
   android {
     ……
     defaultConfig {
       ……
       ndk {
         // Select the actual required CPU architecture
         abiFilters 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64' 
       }
     }
   }
   
   dependencies {
     implementation fileTree(dir: "libs", include: ["smsdk*.aar"])
     ...
   }
   ```

3. Declare permissions by adding the following permissions in AndroidManifest.xml

   ```xml
   <!-- Required permissions -->
   <uses-permission android:name="android.permission.INTERNET" />
   <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
   
   <!-- Recommended permissions -->
   <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
   <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
   <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
   <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
   ```

   Permission purposes

   | Permission                     | Purpose                                                      |
   | ------------------------------ | ------------------------------------------------------------ |
   | INTERNET (required)            | Send collected data to the server via the network            |
   | ACCESS_NETWORK_STATE (required)| Determine if the network is connected; <br/>Get network connection status information, such as 2g, 3g, 4g, wifi, etc., and carrier information |
   | ACCESS_COARSE_LOCATION         | Get cell (base station) information                          |
   | ACCESS_FINE_LOCATION           | Get cell (base station) information                          |
   | ACCESS_WIFI_STATE              | Get bssid (wifi mac), ssid information                       |
   | WRITE_EXTERNAL_STORAGE         | Write Shumei device identifier to SD card                    |
   | READ_EXTERNAL_STORAGE          | Read Shumei device identifier stored on SD card              |

4. HTTP settings, smsdk uses HTTP requests by default. When targetSdkVersion > 27, add the following configuration in AndroidManifest.xml

   ```xml
   <application
   	......
   	android:usesCleartextTraffic="true"
   	......
   >
   ......
   </application>
   ```

   If the `application` node has the `android:networkSecurityConfig` attribute set, such as

   ```xml
   <application
   	......
   	android:usesCleartextTraffic="true"
   	android:networkSecurityConfig="@xml/network_security_config"
   	......>
   ......
   </application>
   ```

   Then add the smsdk domain configuration to the `xml/network_security_config.xml` file (add the corresponding domain for proxy or privatization mode)

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <network-security-config>
       <!-- smsdk domain configuration -->
       <domain-config cleartextTrafficPermitted="true">
           <domain includeSubdomains="true">fengkongcloud.com</domain>
       </domain-config>
   </network-security-config>
   ```

5. Code obfuscation

   Add smsdk obfuscation rules to the proguard-rules.pro file as follows

    ```sh
    -keep class com.ishumei.** {*;}
    ```

## 2 Standard Integration

smsdk is a terminal in the Shumei risk control system, mainly used for collecting device information and generating device identifiers. When the product needs to perform risk control analysis on related businesses, the `create` method of the `SmAntiFraud` class can be used for risk control. Calling the `create` method will immediately collect device information, so it **must** be called after agreeing to the privacy policy and in scenarios that require risk control to avoid collecting unnecessary information in unreasonable scenarios.

The usage of the `create` method is as follows:

```java
// smsdk parameter object
SmAntiFraud.SmOption option = new SmAntiFraud.SmOption();
// Required, organization identifier
option.setOrganization("YOUR_ORGANIZATION");
// Required, application identifier, view in Shumei backend application management, if no suitable value, you can write "default"
option.setAppId("YOUR_APP_ID"); 
// Required, encryption KEY, content of android_public_key attachment in the email
option.setPublicKey("YOUR_PUBLICK_KEY"); 

// Optional, use this method to block some data collection, here taking oaid as an example, other controllable fields can be found in the compliance guide "Controllable Configuration Items -> Android End"
Set<String> notCollect = new HashSet<>(); 
notCollect.add("oaid"); // Indicate not to collect oaid
option.setNotCollect(notCollect);

// Collect device data
boolean isOk = SmAntiFraud.create(context, option);
```

| SmOption Method   | Parameter Type | Required | Default Value        | Description                                                  |
| ----------------- | -------------- | -------- | -------------------- | ------------------------------------------------------------ |
| setOrganization   | String         | Yes      | None                 | Organization identifier assigned by Shumei, viewable in Shumei backend |
| setAppId          | String         | Yes      | default              | Application identifier, distinguishes different applications, viewable in Shumei backend |
| setPublicKey      | String         | Yes      | None                 | Public key identifier, sent in the email when the account is opened |
| setUrl            | String         | No       | None                 | Set device data reporting address in proxy mode or privatization mode |
| setConfUrl        | String         | No       | None                 | Set cloud configuration information retrieval address in proxy mode or privatization mode |
| setCloudConf      | boolean        | No       | true                 | Whether to enable cloud configuration, if set to false, no cloud network requests will be made |
| setArea           | String         | No       | SmAntiFraud.AREA_BJ  | In standard mode, set the data reporting request area, with the following values:<br />AREA_BJ: Business in China (default value)<br />AREA_XJP: Business in Southeast Asia<br />AREA_FJNY: Business in Europe and America |
| setNotCollect     | `Set<String>`  | No       | No                   | Block some collection fields                                   |
| setUsingHttps     | boolean        | No       | false                | Whether to use HTTPS protocol for network requests            |
| addSubCollectors  | SubCollector   | No       | None                 | Subpackage collection data, no subpackage provided by default, provided separately for customers with needs, currently supports:<br />smsdk_ids: Collect device identifier information |

| SmAntiFraud Method         | Parameter Type            | Required | Default Value | Description                                                  |
| -------------------------- | ------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| create                     | Context, SmOption         | Yes      | None          | smsdk initialization method, calling this field will collect data |
| getDeviceId                | None                      | None     | None          | Get smsdk identifier, prioritize cached data, if no cache, wait for collection to complete, this process will block the current thread, be careful to avoid ANR issues. |
| registerServerIdCallback   | IServerSmidCallback       | No       | None          | smsdk callback method, can get identifier through onSuccess callback, callback method is not on the main thread, be careful to avoid UI operations in the callback. |
| startDetector              | AbsDetector               | No       | None          | Subpackage collection of micro-behavior data, no subpackage provided by default, provided separately for customers with needs, currently supports:<br />smsdk_screentouch: Machine operation detection |

When calling the `create` method in a risk control scenario, smsdk will check if the passed parameters are valid. If the return value is `false`, you need to filter the `Smlog` in the logcat log for self-check. Note that versions before smsdk 3.3.0 do not have a return value.

The `create` method takes about 1 second to collect data (low-end models may exceed 2 seconds). The collection process occurs in a sub-thread and will not block the current thread. After the `create` method returns `true`, you can call the `SmAntiFraud.getDeviceId()` method to get the identifier.

Timing for getting the identifier: Use it when triggering business events that require risk control, such as reporting the string returned by `getDeviceId` in key events like login and registration.

Some developers put `getDeviceId` in the Header of all network requests. This can cause some issues that need to be handled by the developer. The reasons and solutions will be explained with the smsdk sequence diagram.

![smsdk-flow](./res/smsdk-flow.png)

Scenario 1: After calling the `create` method, smsdk has boxId in the cache, as shown in "7. Application Event". At this time, calling `getDeviceId` will immediately return boxId, which can be directly placed in the business request Header. This scenario generally occurs when smsdk has completed a service interaction when calling the `create` method, obtained boxId from the server, and cached it successfully.

Scenario 2: After calling the `create` method, smsdk does not have boxId in the cache but has boxData, as shown in "9. Application Event". At this time, calling `getDeviceId` will immediately return boxData. Since boxData is relatively long, directly placing it in the business request Header may encounter a 4K limit, causing boxData to be truncated. The solution is described below. This scenario generally occurs when calling the `getDeviceId` method, smsdk has completed collection, generated boxData, but has not completed a service interaction, and has not updated boxData to boxId.

Scenario 3: After calling the `create` method, smsdk does not have boxId or boxData in the cache, as shown in "12. Application Event". At this time, calling the `getDeviceId` method will wait for the collection to complete and return the collected data encrypted as boxData. The waiting time is related to the collection speed and the timing of `create`. Assuming the collection takes 2 seconds, if `getDeviceId` is called immediately after `create`, the `getDeviceId` method will block the current thread for 2 seconds; if `getDeviceId` is called 1 second after `create`, `getDeviceId` will block the current thread for 1 second. Developers need to ensure that blocking does not cause ANR. The boxData truncation issue also exists here. This scenario generally occurs when calling the `create` method, smsdk does not have boxId in the cache, and smsdk is collecting when calling the `getDeviceId` method.

Solution to the boxData truncation issue described in Scenario 2 and Scenario 3: Coordinate with business colleagues to adjust the business request Header length limit, recommended to adjust to 24KB; if the business side cannot modify, you can configure smsdk to adjust the boxData length, this method will reduce the security of boxData, please execute as a backup plan

```java
option.usingShortBoxData(true); 
```

Solution to the blocking issue described in Scenario 3: Do not call the `getDeviceId` method immediately when starting the SDK, it is recommended to call it after a delay of 1 to 2 seconds. If you need to call it at the earliest time, you can use the callback method to listen for boxId

```java
// This method must be called before the create method, otherwise, the callback may not be triggered
SmAntiFraud.registerServerIdCallback(new SmAntiFraud.IServerSmidCallback() {
  @Override
  public void onSuccess(String boxId) {
    // Server issued successfully or cache has available boxId
    // If there is a boxId in the cache, this method will be triggered twice, the second time will update the boxId in the cache
  }

  @Override
  public void onError(int errCode) {
    // Error code meaning see the table below
    // After this method is triggered, you can get boxId or boxData through SmAntiFraud.getDeviceId(), it is recommended to call SmAntiFraud.getDeviceId() in a sub-thread.
  }
});
```

| errCode | Meaning                                                      |
| ------- | ------------------------------------------------------------ |
| -1      | No network, common reason: device has no network             |
| -2      | Network exception, network connection exception (conn.getResponseCode() throws exception) or HTTP status is not 200, common reason: proxy or privatization server configuration error |
| -3      | Business exception, business status code issued is not 1100, server did not return deviceId, common reason: parameter configuration error, qps limit exceeded, server exception |
| -4      | Other exceptions                                             |

boxId and boxData cannot be directly used as device identifiers, but you can use boxId or boxData to directly query device risks.

At this point, the standard integration of smsdk in China has been fully completed. If there are no customization requirements, the integration is complete. It is recommended to refer to the "Testing" section to self-check if the integration is successful.

Overseas standard integration (due to data compliance requirements, all overseas integrations need to use proxy mode, this is for special customers who cannot provide proxy services), requires using the overseas special package and switching the device fingerprint data center, set as follows

1. Business data center in Europe and America (Virginia data center)

   ```java
   // User distribution range is Europe and America
   option.setArea(SmAntiFraud.AREA_FJNY);
   // If the user distribution range is global, acceleration needs to be enabled, configure as follows
   // option.setArea(SmAntiFraud.AREA_FJNY);
   // String host = "http://fp-na-it-acc.fengkongcloud.com";
   // option.setUrl(host + "/deviceprofile/v4");
   // option.setConfUrl(host + "/v3/cloudconf");
   ```

2. Business data center in Europe and America (Frankfurt data center)

   ```java
   option.setArea("flkf");
   String host = "http://api-device-eur.fengkongcloud.com";
   option.setUrl(host + "/deviceprofile/v4");
   option.setConfUrl(host + "/v3/cloudconf");
   ```

3. Business data center in Southeast Asia (Singapore data center)

   ```java
   // User distribution range is Southeast Asia
   option.setArea(SmAntiFraud.AREA_XJP)
   // If the user distribution range is global, acceleration needs to be enabled, configure as follows
   // option.setArea(SmAntiFraud.AREA_XJP);
   // String host = "http://fp-sa-it-acc.fengkongcloud.com";
   // option.setUrl(host + "/deviceprofile/v4");
   // option.setConfUrl(host + "/v3/cloudconf");
   ```

## 3 Privatization Integration

The main steps are similar to standard integration, with the following additional configurations

```java
// Set area, this value is the organization identifier
option.setArea("YOUR-ORGANIZATION");
// Set private address, replace private-host with the privatized hostname (domain name)
String host = "https://private-host"; 
option.setUrl(host + "/deviceprofile/v4"); // Example path, needs to be consistent with the real scenario
option.setConfUrl(host + "/v3/cloudconf"); // Example path, needs to be consistent with the real scenario
```

Note, if the passed host is an HTTP request, such as `http://private-host`, ensure that the APP can send HTTP requests, refer to the HTTP settings part of the "Project Configuration" section. After the privatization integration is completed, self-check according to the "Testing" section.

## 4 Proxy Integration

Developers need to set up a proxy server themselves. The logic settings on the SDK side are similar to standard integration, with the following additional configurations

```java
// Set private address, replace host with the proxy server hostname (domain name)
String host = "https://proxy-host";
option.setUrl(host + "/deviceprofile/v4"); // Example path, needs to be consistent with the real scenario
option.setConfUrl(host + "/v3/cloudconf"); // Example path, needs to be consistent with the real scenario
```

## 5 Testing

1. Call the `SmAntiFraud.create(context, option)` method and get a return value of `true`
2. Call the `SmAntiFraud.getDeviceId()` method and get a return value of boxId, such as `Bm21V93t5QwTNdwyQxxxxxRYuSnOuwwylqZvz8Lixxxxx17lRMqcQ1jz9RwN6qW31/Z0YYmxN8KQnrya9xxxxxx==`
3. Execute `adb logcat | grep Smlog` with no abnormal output, if there is abnormal output, modify according to the prompt
4. In the Shumei management backend, select "Device Risk Trend" in the navigation bar, find the "Device Details" section, and check if there is data reporting (there may be a delay, generally no more than 30 minutes)
5. If testing fails, contact Shumei staff for troubleshooting

## 6 SDK Upgrade

This document is applicable for upgrading from Android smsdk v2 to Android smsdk v3. It is recommended that customers who have already integrated switch to smsdk v3 as soon as possible.

Upgrade steps and precautions

1. Delete the old version of smsdk, locate the smsdk package according to the location of the SmAntiFraud class, and delete it. If the project contains the `libsmsdk.so` dynamic library, delete it together.
2. Integrate the new version of smsdk according to the steps in the "Android SDK Project Configuration" section.
3. Some APIs have been deleted in the new version. After replacing the package, if compilation fails, directly delete the smsdk error methods.
4. Run the new smsdk and perform self-testing according to the "Android SDK Testing" section. If unsuccessful, modify according to the error prompt or contact Shumei staff for troubleshooting.
5. Proxy integration customers need to upgrade the proxy server according to the "Decryption Tool and Proxy Server Instructions Proxy Integration" section.
6. Privatization integration customers need to contact Shumei staff for overall upgrade.

The smsdk v3 version terminal no longer provides plaintext device identifiers. The business side cannot directly use boxId or boxData as identifiers. Refer to the "Decryption Tool and Proxy Server Instructions Device Fingerprint Identifier Decryption" for the method to