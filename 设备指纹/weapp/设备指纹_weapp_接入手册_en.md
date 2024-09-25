## Device Fingerprint WEAPP (WeChat/QQ/ByteDance/Alipay Mini Program) SDK Integration Manual

The following example uses WeChat. The usage for other mini programs and WeChat mini programs is basically the same. You can refer to the example below.

### Import SDK File
Copy the generated js file to the libs directory of the mini program.

### Start
Normal use: Introduce the following code in app.js:

```javascript
const SMSdk = require('./libs/weapp-fp'); // Note: Configure parameters and start immediately after require

// For more parameters, please see the detailed description in the next chapter
// If you need to collect openId and unionId, get the openId and unionId after wx.login(). Then perform the SMSdk.initConf() startup operation.
SMSdk.initConf({
    // 1. General configuration items
    organization: 'xxxxxxx', // Required, company identifier, organization item in the email
    publicKey: 'xxxxxxx', // publicKey is a required item after version v3.0.0, it is the encryption public key, publicKey item in the email

    // 2. Special configuration items for connecting to overseas data centers, only for customers reporting device data to overseas data centers
    // 2.1 Business data center in China
    // 1) User distribution: China (default setting)
    // apiHost:'fp-it.fengkongcloud.com'
    // 2) User distribution: Global
    // apiHost:'fp-it-acc.fengkongcloud.com'

    // 2.2 Business data center in North America (Virginia)
    // 1) User distribution: North America
    // apiHost: 'fp-na-it.fengkongcloud.com'
    // 2) User distribution: Global
    // apiHost: 'fp-na-it-acc.fengkongcloud.com'

    // 2.3 Business data center in Europe (Frankfurt)
    // apiHost: 'api-device-eur.fengkongcloud.com'

    // 2.4 Business data center in Southeast Asia
    // 1) User distribution: Southeast Asia
    // apiHost:'fp-sa-it.fengkongcloud.com'
    // 2) User distribution: Global
    // apiHost:'fp-sa-it-acc.fengkongcloud.com'

    // 2.5 Special configuration for private deployment server interface domain name
    // apiHost: 'xxxxxx';  // Private deployment service domain name
});

App({
    /**
     * Global shared
     * You can define any member for sharing throughout the application
     */
    data: {},
    /**
     * Inject SMSdk method, subsequent pages can introduce it
     */
    SMSdk: SMSdk,
    /**
     * Lifecycle function--Listen to mini program startup
     * When the mini program startup is complete, onLaunch will be triggered (only once globally)
     */
    onLaunch() {
    }
});
```
### Collect openId and unionId
To ensure the stability of the device identifier and risk identification capability, you need to pass the openId and unionId to us through configuration. The specific access method is as follows:

1. After wx.login(), pass the obtained res.code to the backend to get the corresponding openId and unionId.

2. Get the openId and unionId. Call SMSdk.setOpenId and SMSdk.setUnionId for configuration.

3. Perform the SMSdk.initConf() startup operation.

```javascript
    SMSdk.setOpenId('The openId you obtained');
    SMSdk.setUnionId('The UnionId you obtained');
    SMSdk.initConf({}); // Same as normal startup access
```

### Use Device Identifier

```javascript
// First get the SMSdk object of the device fingerprint SDK
const {SMSdk} = getApp();

Page({
    data: {
        deviceId: '',
    },

    /**
     * Method 1: You can call smSdk.getDeviceId to get data in a custom method. This method returns a Promise object, so you can get the device fingerprint through async/await syntax.
     */
    async fetchData() {
        const deviceId = await SMSdk.getDeviceId();
        // The following is business code
        // ...
    },

    /**
     * Method 2: The SMSdk.getDeviceId method can be called after startup
     * For example, it can be called in the onReady lifecycle function, or in other lifecycle functions of the mini program.
     */
    onReady() {
        SMSdk.getDeviceId().then(deviceId => {
            this.setData({ deviceId });
        });
    },
});
```

### Notes
1. The mini program development management needs to add the server domain name to the whitelist. If the apiHost is configured privately, this private custom domain name needs to be whitelisted.

2. The device fingerprint SDK supports integration and use in cross-end frameworks taro2/3. However, note that in the taro2 framework, the SDK needs to be excluded from compilation. In the taro2 framework's config, configure as follows.
```javascript
mini: {
    compile: {
        exclude: [
            function (modulePath) {
                return modulePath.indexOf('fp-wx.min') >= 0
            },
        ],
    },
},
```

### Parameters

| **Field** | **Parameter Type**  | **Required** | **Default Value** | **Field Description** |
| -- | -- | -- | -- | -- |
| organization | string | Yes | None | Company identifier assigned by SMSdk, can be viewed in the SMSdk backend |
| publicKey | string | Yes | None | Private key identifier, publicKey item in the email |
| apiProtocol | string  | No | https | If using http, set it to http |
| apiHost  | string  | No | fp-it.fengkongcloud.com  | Domain name for data reporting, see `APIHOST` enumeration values, and set the domain name to the whitelist under the WeChat account, for example https://fp-it.fengkongcloud.com/  |

#### `APIHOST` Enumeration Values

| **Business Data Center Region** | **User Distribution**  | **apiHost Value** |
| -- | -- | -- |
| China | China (default setting) | fp-it.fengkongcloud.com |
| China | Global | fp-it-acc.fengkongcloud.com |
| North America (Virginia) | North America | fp-na-it.fengkongcloud.com |
| North America (Virginia) | Global | fp-na-it-acc.fengkongcloud.com |
| Europe (Frankfurt) | - | api-device-eur.fengkongcloud.com |
| Southeast Asia | Southeast Asia | fp-sa-it.fengkongcloud.com |
| Southeast Asia | Global | fp-sa-it-acc.fengkongcloud.com |

### Method to Get deviceId

| **Method Name** | **Parameter** | **Return Value**  | **Description** |
| -- | -- | -- | -- |
| getDeviceId | None | Promise | Asynchronous method, returns a promise to provide the deviceId to the user at some point in the future. This method can be called wherever needed after starting the SDK |

### Method to Collect openId and unionId

| **Method Name** | **Parameter** | **Return Value**  | **Description** |
| -- | -- | -- | -- |
| setOpenId | string | None | Add the openId obtained after login to the collection |
| setUnionId | string | None | Add the unionId obtained after login to the collection |