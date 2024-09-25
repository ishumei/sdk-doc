# ShuMei Device Fingerprint uniapp Integration Manual

ShuMei Device Fingerprint SDK supports App native plugin local plugins. For related basic knowledge, refer to the uniapp official [plugin instructions](https://nativesupport.dcloud.net.cn/NativePlugin/README).

## 1 SDK Integration

### 1.1 Add Local Plugin

1. Place the `SmsdkUniPlugin` folder into the `YOUR_PROJECT/nativeplugins` directory.
2. In HBuilderX, select `manifest.json` and choose App native plugin configuration.
3. Click on local plugin `[Select Local Plugin]` and check SmsdkUniPlugin.

![image-20220919154526570](./res/image-20220919154526570.png)

### 1.2 Import Plugin

```js
// Example code location: SmsdkUniappDemo/pages/index/index.vue
const SmAntiFraud = uni.requireNativePlugin('SmsdkUniPlugin');
```

### 1.3 SDK API

#### Start smsdk

```js
SmAntiFraud.create({
    'org': 'YOUR_ORGANIZATION',                   // Organization identifier
    'appid': 'YOUR_APPID',                        // appId
    'publicKeyAndroid': YOUR_ANDROID_PUBLICK_KEY, // Android publicKey
    'publicKeyIOS': YOUR_IOS_PUBLICK_KEY,         // iOS publicKey
    'url': 'YOUR_URL',                            // Device fingerprint request address
    'confUrl': 'YOUR_CONF_URL',                   // Cloud configuration request address
  	'cloudConf': [false|true],					  // Whether to use cloud configuration, default is true
    'notCollect': 'imei imsi',                    // Android non-collection items (each item separated by space)
    'area': 'YOUR_AREA',                          // Area
    'usingHttps': [false|true]                    // Whether to use https for data transmission
  },
  (ret) => {
    console.log(JSON.stringify(ret));
});
```

ret structure

```json
{
  "smid": "xxx", // Device identifier, if there is no smid field, you can check the code
  "code": -1 // Error code, meaning refer to the native integration manual
}
```

#### Get Identifier

```js
SmAntiFraud.getDeviceId();
```

#### Other Methods

Get SDK version number

```js
SmAntiFraud.getSdkVersion();
```

## 2 Packaging

Use HBuilderX -> Release -> Native APP-Cloud Packaging to generate apk or ipa packages. For specific packaging methods, refer to the DCloud official instructions.

## 3 Precautions

### 3.1 Data Compliance

When starting the SDK, device data will be collected. To avoid data compliance issues, you need to add the ShuMei privacy policy in the app's privacy policy and start it after agreeing to the privacy policy.

Refer to the ShuMei privacy policy in the "ShuMei Device Risk SDK Integration Manual".

### 3.2 Verify Successful Integration

- After starting the SDK, calling the `SmAntiFraud.getDeviceId()` method to get the serverId or boxId means the integration is successful, otherwise it is a failure. Refer to the "ShuMei Device Risk SDK Integration Manual" for serverId and boxId examples;

- The callback method of `SmAntiFraud.create` returning `"smid": "xxx"` means the integration is successful. Refer to section 1.3 "Start smsdk" for the callback method.