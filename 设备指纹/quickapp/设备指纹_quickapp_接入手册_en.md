# Quick App Integration with Shumei Device Fingerprint SDK Instructions

> Note! The simulator in the Quick App local development tool may lack some device information, it is recommended to [debug on a real device]

1. Place the sdk js (fp-1.0.3.min.js) in the src/libs/ folder

2. Introduce the device fingerprint SDK in app.ux and initialize it

```javascript
<script>
// 1. Import SMSdk
const SMSdk = require('./libs/fp-1.0.2.min.js')

// 2. Inject SMSdk into global
const hook2global = global.__proto__ || global
hook2global.SMSdk = SMSdk;

// 3. Fill in the SMSdk configuration
const conf = {
    organization: 'xxxxxx',  // Organization identifier, required
    publicKey: 'xxxxx',   // Public key, required

    // apiHost: 'fp-it.fengkongcloud.com',  // Optional, can be modified according to actual deployment host for private deployment
    // apiPath: '/deviceprofile/v4', // Optional, can be modified according to actual deployment path for private deployment

    // notCollect: {
        // quickApp supports customizing whether to collect certain fields, you can contact Shumei staff to get a Demo
    // }
}

export default {
    onCreate() {
        // 4. Initialize SMSdk
        SMSdk.Create(conf);
    },
}
</script>

```

3. Call the getDeviceId method to get the device fingerprint when using;
```javascript
<template>
  <div class="wrapper">
    <text class="title">{{ title }}</text>
    <input
      class="btn"
      type="button"
      value="get Device Id"
      onclick="onDetailBtnClick"
    />
  </div>
</template>

<script>
export default {
  // 5. Call the getDeviceId method to get the device identifier
  async onDetailBtnClick() {
    const id = SMSdk.getDeviceId();
    console.log('deviceId: ', id)
  },
}
</script>
```