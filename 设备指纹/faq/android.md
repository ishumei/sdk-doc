### 1 Android 屏蔽采集部分字段

在调用 `SmAntiFraud.create(...)` 方法时，传入 `SmOption` 字段名屏蔽系统 API 调用

```java
SmAntiFraud.SmOption option = new SmAntiFraud.SmOption(); 
... // 其它设置
Set<String> notCollect = new HashSet<>(); 
notCollect.add("oaid"); // 不采集 oaid
option.setNotCollect(notCollect);
SmAntiFraud.create(ctx, option);
```

传入不采集字段名必须与下表 **字段名** 一致

| 字段名   | 含义                        | 系统关键 API                                                 | 删除后影响                                      | 版本 |
| -------- | --------------------------- | ------------------------------------------------------------ | ----------------------------------------------- | -------- |
| adid     | android_id                  | Secure#getString(Resolver, ANDROID_ID)                       | 影响设备标识稳定性                              | 2.9.4/3.0.4 |
| bssid    | WIFI 热点的 MAC 地址        | WifiManager#getConnectionInfo<br />WifiInfo#getBSSID         | 影响风险设备聚集风险的识别                      | 2.9.4/3.0.4 |
| cell     | 基站信息                    | TelephonyManager#getCellLocation<br />GsmCellLocation#getCid<br />GsmCellLocation#getLac | 影响风险设备聚集风险的识别                      | 2.9.4/3.0.4 |
| network  | 手机网络链接方式            | TelephonyManager#getNetworkType                              | 影响网络状态相关的逻辑校验                      | 2.9.4/3.0.4 |
| ssid | WIFI 名称 | WifiManager#getConnectionInfo<br />WifiInfo#getSSID | 影响风险设备聚集风险的识别 | 2.9.4/3.0.4 |
| wifiip | 局域网 IP 地址 | WifiManager#getConnectionInfo<br />WifiInfo#getIpAddress | 影响风险设备聚集风险的识别 | 2.9.4/3.0.4 |
| operator | 运营商编码                  | TelephonyManager#getSimOperator                              | 影响网络状态的校验                              | 2.9.4/3.0.4 |
| sensor   | 传感器信息                  | SensorManager#getSensorList                                  | 影响与篡改识别相关的逻辑校验                    | 2.9.4/3.0.4 |
| oaid | Android开发中匿名设备标识符 | 各手机厂商关键 API 不同 | 暂无 | 2.10.0/3.0.6 |
| sensorsData | 传感器数值信息 | SensorManager#registerListener<br />SensorManager#unregisterListener | 暂无影响，后续可能会影响聚集类的策略 | 2.13.0/3.1.0 |
| sdCacheLimit | SD 卡缓存 |FileInputStream、FileOutputStream|低版本系统上会影响全局标识关联能力|  |
| props_sn | 系统属性序列号              | SystemProperties.get("sys.serialno")<br />SystemProperties.get("persist.radio.serialno")<br />SystemProperties.get("ro.ril.oem.sno")<br />SystemProperties.get("ro.ril.oem.psno")<br />SystemProperties.get("gsm.serial") | 在 Android 9 及以下版本，可能影响设备标识稳定性 | 2.13.0/3.2.2 |
| locationCls  | 获取 Location 对象          | Location#getLongitude<br />Location#getLatitude              | 影响与篡改地理位置类的识别<br />此数据非获取真实地理位置，但是可能会被三方检测误检为获取地理位置，如果无申诉途径可以将此字段屏蔽。 | 2.13.2/3.5.2 |

### 2 手机弹窗获取“位置”权限

在部分低版本 Android 系统中，初始化设备指纹会有弹窗现象，提示授权位置权限，原因是某些 ROM 修改了授权弹窗机制，只要调用 `WifiManager.getConnectionInfo` 方法，系统会自动弹窗，可以通过不采集方式，屏蔽 ssid, bssid, wifiip 字段避免调用 `WifiManager.getConnectionInfo` 方法。

