### 1 SDK v2、v3 版本差别

#### 设备标识形式不同

| SDK 版本 | v2          | v3        | 格式说明                                    |
| -------- | ----------- | --------- | ------------------------------------------- |
| boxId    | N/A         | Bxxxx==   | 以 'B' 开头的 89 位字符串                   |
| boxData  | N/A         | Dxxxxxx== | 以 'D' 开头的 8k 左右长度的字符串           |
| localId  | 20xxxx00xxx | N/A       | 62 位字符长度的字符串，第 48/49 位数字为 00 |
| serverId | 20xxxx01xxx | N/A       | 62 位字符长度的字符串，第 48/49 位数字为 01 |

app 首次启动，初始化 SDK 未完成数据上报时调用 `getDeviceId` 方法，v2 版本会返回 localId，v3 版本会返回 boxData，localId 不可以当做设备标识，卸载重装 lcoalId 会发生变化。

#### 代理服务器地址不同

v3 版本设置如“解密工具及代理服务器说明”章节所述，v2 版本地址如下（其他设置与 v3 相同）

| 设备指纹请求 | Android/iOS                                                  |
| ------------ | ------------------------------------------------------------ |
| 国内         | http://fp-proxy.fengkongcloud.com/v3/profile/android<br/>http://fp-proxy.fengkongcloud.com/v3/profile/ios |
| 东南亚       | http://fp-sa-proxy.fengkongcloud.com/v3/profile/android<br/>http://fp-sa-proxy.fengkongcloud.com/v3/profile/ios |
| 欧美         | http://fp-na-proxy.fengkongcloud.com/v3/profile/android<br/>http://fp-na-proxy.fengkongcloud.com/v3/profile/ios |

| 云配请求 | Android/iOS                                       |
| -------- | ------------------------------------------------- |
| 国内     | http://fp-proxy.fengkongcloud.com/v3/cloudconf    |
| 东南亚   | http://fp-sa-proxy.fengkongcloud.com/v3/cloudconf |
| 欧美     | http://fp-na-proxy.fengkongcloud.com/v3/cloudconf |



