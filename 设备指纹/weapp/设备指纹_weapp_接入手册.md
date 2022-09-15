## 设备指纹WEAPP(微信/QQ/字节跳动/支付宝小程序)sdk接入手册

以下以微信示例，其他小程序和微信小程序使用方式基本一致，可以参照下述例子使用

### 导入SDK文件
将产出js文件拷贝到小程序的libs目录

### 初始化
在app.js 引入如下所示代码：

```javascript
const SMSdk = require('./libs/weapp-fp'); // 注意：require之后要立即初始化配置

// 更多参数请看下一章节详细说明
SMSdk.initConf({
    // 1. 通用配置项
    organization: 'xxxxxxx', // 必填，公司标识，邮件中organization项
    publicKey: 'xxxxxxx', // publicKey是v3.0.0版本后必填项，是加密公钥，邮件中publicKey项

    // 2.连接海外机房特殊配置项，仅供设备数据上报海外机房客户使用
    // 2.1 业务机房在国内
    // 1) 用户分布：国内（默认设置）
    // apiHost:'fp-it.fengkongcloud.com'
    // 2) 用户分布：全球
    // apiHost:'fp-it-acc.fengkongcloud.com'

    // 2.2 业务机房在欧美（弗吉尼亚）
    // 1) 用户分布：欧美
    // apiHost: 'fp-na-it.fengkongcloud.com'
    // 2) 用户分布：全球
    // apiHost: 'fp-na-it-acc.fengkongcloud.com'

    // 2.3 业务机房在欧美（法兰克福）
    // apiHost: 'api-device-eur.fengkongcloud.com'

    // 2.4 业务机房在东南亚
    // 1) 用户分布：东南亚
    // apiHost:'fp-sa-it.fengkongcloud.com'
    // 2) 用户分布：全球
    // apiHost:'fp-sa-it-acc.fengkongcloud.com'

    // 2.5 私有化特殊配置的服务端接口域名
    // apiHost: 'xxxxxx';  // 私有化部署的服务域名
});

App({
    /**
     * Global shared
     * 可以定义任何成员，用于在整个应用中共享
     */
    data: {},
    /**
     * 注入数美SMSdk方法，后续页面可以引入
     */
    SMSdk: SMSdk,
    /**
     * 生命周期函数--监听小程序初始化
     * 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）
     */
    onLaunch() {
    }
});
```

### 使用设备标识

```javascript
// 首先获取数美设备指纹sdk对象
const {SMSdk} = getApp();

Page({
    data: {
        deviceId: '',
    },

    /**
     * 方式一 可以在自定义的方法中调用smSdk.getDeviceId获取数据，该方法返回一个Promise对象，所以可以通过async/await语法获取设备指纹
     */
    async fetchData() {
        const deviceId = await SMSdk.getDeviceId();
        // 后面是业务代码
        // ...
    },

    /**
     * 方式二 SMSdk.getDeviceId方法在初始化之后即可调用
     * 例如 可以在onReady生命周期函数中调用，也可以在小程序其他生命周期中调用
     */
    onReady() {
        SMSdk.getDeviceId().then(deviceId => {
            this.setData({ deviceId });
        });
    },
});
```

### 参数

| **字段** | **参数类型**  | **是否必填** | **默认值** | **字段说明** |
| -- | -- | -- | -- | -- |
| organization | string | 是 | 无 | 数美分配的公司标识，数美后台可以查看看 |
| publicKey | string | 是 | 无 | 私钥标识，邮件中publicKey项 |
| apiProtocol | string  | 否 | https | 如果使用http，则设置成 http |
| apiHost  | string  | 否 | fp-it.fengkongcloud.com  | 数据上报的域名，详见`APIHOST`枚举值，然后微信账号下要将域名设置到白名单中，例如 https://fp-it.fengkongcloud.com/  |

#### `APIHOST`枚举值

| **业务机房所在区域** | **用户分布**  | **apiHost值** |
| -- | -- | -- |
| 国内 | 国内（默认设置） | fp-it.fengkongcloud.com |
| 国内 | 全球 | fp-it-acc.fengkongcloud.com |
| 欧美（弗吉尼亚）| 欧美 | fp-na-it.fengkongcloud.com |
| 欧美（弗吉尼亚）| 全球 | fp-na-it-acc.fengkongcloud.com |
| 欧美（法兰克福）| - | api-device-eur.fengkongcloud.com |
| 东南亚 | 东南亚 | fp-sa-it.fengkongcloud.com |
| 东南亚 | 全球 | fp-sa-it-acc.fengkongcloud.com |

### 获取deviceId的方法

| **方法名** | **参数** | **返回值**  | **说明** |
| -- | -- | -- | -- |
| getDeviceId | 无 | Promise | 异步方法，返回一个 promise，以便在未来某个时候把deviceId交给使用者。该方法可以在初始化之后需要的地方都可以调用 |
