# 快应用集成数美设备指纹 SDK 说明

> 注意！快应用本地开发工具的模拟器中可能缺少部分设备信息，建议【真机调试】

1. sdk js (fp-1.0.3.min.js) 放置在 src/libs/ 文件夹下面

2. 在 app.ux 引入设备指纹 sdk，并初始化

```javascript
<script>
// 1.引入 SMSdk
const SMSdk = require('./libs/fp-1.0.2.min.js')

// 2.将 SMSdk 注入全局
const hook2global = global.__proto__ || global
hook2global.SMSdk = SMSdk;

// 3.填写 SMSdk 配置
const conf = {
    organization: 'xxxxxx',  //组织标识  必填项
    publicKey: 'xxxxx',   //公钥 必填项

    // apiHost: 'fp-it.fengkongcloud.com',  // 可选项，私有化部署时可根据实际部署 host 修改
    // apiPath: '/deviceprofile/v4', // 可选项，私有化部署时可根据实际部署 path 修改

    // notCollect: {
        // quickApp 支持对部分字段自定义是否采集，可联系数美工作人员获取 Demo
    // }
}

export default {
    onCreate() {
        // 4.初始化 SMSdk
        SMSdk.Create(conf);
    },
}
</script>

```

3. 使用时调用 getDeviceId 方法获取设备指纹；
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
  // 5.调用 getDeviceId 方法，获取设备标识
  async onDetailBtnClick() {
    const id = SMSdk.getDeviceId();
    console.log('deviceId: ', id)
  },
}
</script>
```
