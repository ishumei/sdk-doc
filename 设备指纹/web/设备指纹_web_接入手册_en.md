## Device Fingerprint Web-SDK Integration Manual

- Embed the SDK on the pages where you need to use the Shumei anti-fraud service, such as registration and login pages. Other pages do not need to embed it.

- The WEB SDK does not require downloading additional SDK packages. You can integrate it directly by following the steps below.

### 1. Start the SDK
Include the following code at the bottom of the page:

Latest version: 3.0.0

```javascript
(function() {
    window._smReadyFuncs = [];
    window.SMSdk = {
        onBoxDataReady: function (boxData) { // Optional
            console.log('The data obtained at this time is boxData or boxId', boxData);
        },
        ready: function (fn) {
            fn && _smReadyFuncs.push(fn);
        }
    };

    // 1. General configuration items
    window._smConf = {
        organization: 'xxxxxxxxxx', // Required, organization identifier, organization item in the email
        appId : 'xxxxxxxxxx', // Required, application identifier, default value is "default", other application identifiers need to contact Shumei for assistance
        publicKey : 'xxxxxxxxxx', // Required, private key identifier, publicKey item in the email
        staticHost: 'static.portal101.cn', // Required, set the JS-SDK file domain, it is recommended to fill in static.portal101.cn
        protocol: 'https', // If using https, set it; if not, do not set this field

       // 2. Special configuration items for connecting to overseas data centers, only for customers reporting device data to overseas data centers
       // 2.1 Business data center in China
       // 1) User distribution: China (default setting)
       // apiHost:'fp-it.portal101.cn'
       // 2) User distribution: Global
       // apiHost:'fp-it-acc.portal101.cn'

       // 2.2 Business data center in North America (Virginia)
       // 1) User distribution: North America
       // apiHost: 'fp-na-it.portal101.cn'
       // 2) User distribution: Global
       // apiHost: 'fp-na-it-acc.portal101.cn'

       // 2.3 Business data center in Europe (Frankfurt)
       // apiHost: 'api-device-eur.portal101.cn'
      
       // 2.4 Business data center in Southeast Asia
       // 1) User distribution: Southeast Asia
       // apiHost:'fp-sa-it.portal101.cn'
       // 2) User distribution: Global
       // apiHost:'fp-sa-it-acc.portal101.cn'
      
       // 2.5 Special configuration for privatization
       // staticHost: 'xxxxxx' // Privatized customers introduce the online CDN address themselves, this item is required; if the customer introduces the js file locally, this item is not filled
       // apiHost: 'xxxxxx';  // Domain name of the privatized deployment service
    };

    var url = (function () {
        var isHttps = 'https:' === document.location.protocol;
        var protocol = isHttps ? 'https://' : 'http://';
        var fpJsPath = '/dist/web/v3.0.0/fp.min.js';
        var url =  protocol + _smConf.staticHost + fpJsPath;
        return url;
    })();
    var sm = document.createElement('script');
    var s = document.getElementsByTagName('script')[0];
    sm.src = url;
    s.parentNode.insertBefore(sm, s);
})();
```

### 2. Use Device Identifier

```javascript
/**
 * Bind event
 * @param element
 * @param eventType
 * @param func
 */
function bindEvent(element, eventType, func) {
    if (element.addEventListener) {
        element.addEventListener(eventType, func, false);
    }
    else if (element.attachEvent) {
        eventType = 'on' + eventType;
        element.attachEvent(eventType, func);
    }
    else {
        eventType = 'on' + eventType;
        element[eventType] = func;
    }
}

/**
 * cb business logic
 * Use Shumei device identifier logic
 */
function dealSmDeviceId(cb) {
    var smDeviceId = '';
    var smDeviceIdReady = false;

    SMSdk.ready(function () {
        if (SMSdk.getDeviceId) {
            smDeviceId = SMSdk.getDeviceId();
        }
        if (!smDeviceIdReady) {
            smDeviceIdReady = true;
            // Execute business logic
            cb && cb(smDeviceId);
        }
    });
}

// Customers choose the following scenarios according to the actual situation. If you have any questions, contact Shumei technical support for answers
// Scenario 1: Need to click a button (such as login, registration, coupon collection, etc.) for interactive scenarios
var buttonEl = document.getElementById('getDeviceId');
bindEvent(buttonEl, 'click', function () {
    dealSmDeviceId(function (deviceId) {
        console.log('Callback executed successfully, device identifier is: ' + deviceId);
    });
});

// Scenario 2: Use directly without interaction (such as browsing)
dealSmDeviceId(function (deviceId) {
    console.log('Callback executed successfully, device identifier is: ' + deviceId);
});
```

- If the obtained deviceId is requested to the backend interface via POST, it can be passed directly; if it is requested to the backend interface via GET, the URL parameter value needs to be encoded using encodeURIComponent, and the server uses the corresponding language's decode code to process it.
- Avoid multiple references on the page. After referencing, you can check the number of references to fp.min.js by packet capture.
  In addition to the two usage scenarios mentioned above, if there are other usage methods, it is recommended to contact Shumei technical support to discuss new solutions together.

### 3. Check if the integration is successful

- You can view the data uploaded by the SDK in the Shumei management backend. By checking the uploaded data, you can confirm whether the integration is successful. See Chapter 2 for the viewing method.

- Packet capture verification of successful integration. After the integration is completed, when the page loads, fp.min.js loads successfully, /deviceprofile/v4, as shown below:

![](./res/test.png)


### Parameters

| **Field** | **Parameter Type**  | **Required** | **Default Value** | **Field Description** |
| -- | -- | -- | -- | -- |
| organization | string | Yes | None | Company identifier assigned by Shumei, can be viewed in the Shumei backend |
| appId | string | Yes | None | Application identifier, distinguishes different applications, can be managed in the Shumei backend |
| publicKey | string | Yes | None | Private key identifier, publicKey item in the email |
| staticHost | string | Yes | None | Set the JS-SDK file domain |
| protocol | string  | No | http | If using https, set it to http; if not, do not set this field |
| apiHost  | string  | No | fp-it.portal101.cn  | Domain name for data reporting, see `APIHOST` enumeration values |

#### `APIHOST` Enumeration Values
| **Business Data Center Location** | **User Distribution**  | **apiHost Value** |
| -- | -- | -- |
| China | China (default setting) | fp-it.portal101.cn |
| China | Global | fp-it-acc.portal101.cn |
| North America (Virginia) | North America | fp-na-it.portal101.cn |
| North America (Virginia) | Global | fp-na-it-acc.portal101.cn |
| Europe (Frankfurt) | - | api-device-eur.portal101.cn |
| Southeast Asia | Southeast Asia | fp-sa-it.portal101.cn |
| Southeast Asia | Global | fp-sa-it-acc.portal101.cn |

### Methods
| **Method Name** | **Parameters** | **Return Value**  | **Description** |
| -- | -- | -- | -- |
| onBoxDataReady | boxData  | None | Callback function, executed at the moment when boxData is obtained, the parameter value is boxData or boxId |
| ready | fn | None | Callback function, executed after the front-end interface request is successful or failed |
| getDeviceId | - | string | Returns deviceId |