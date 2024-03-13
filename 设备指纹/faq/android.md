### 1 Android 如何判断隔离进程

```java
public static boolean isIsolated(int uid) {
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    return Process.isIsolated();
  }
  return uid % 100000 >= 90000;
}

// MyApplication.java
@Override
public void onCreate() {
  super.onCreate();
	// 隔离进程不进行任何操作
  if(isIsolated(Process.myUid())){
    return;
  }
}
```

### 2 Android 如何判断主进程

```java
String getCurProcessName(Context context) {
  int pid = android.os.Process.myPid();
  ActivityManager mActivityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
  for (ActivityManager.RunningAppProcessInfo appProcess : mActivityManager.getRunningAppProcesses()) {
    if (appProcess.pid == pid) {
      return appProcess.processName;
    }
  }
  return null;
}

// MyApplication.java
@Override
public void onCreate() {
  super.onCreate();
  // 只在主进程启动 smsdk
	if (getPackageName().equals(getCurProcessName(context))) {
    ...
    SmAntiFraud.create(context, option);
    ...
  }
  ...
}
```

### 3 Android 屏蔽采集部分字段

在调用 `SmAntiFraud.create(...)` 方法时，传入 `SmOption` 字段名屏蔽系统 API 调用

```java
SmAntiFraud.SmOption option = new SmAntiFraud.SmOption(); 
... // 其它设置
Set<String> notCollect = new HashSet<>(); 
notCollect.add("oaid"); // 不采集 oaid
option.setNotCollect(notCollect);
SmAntiFraud.create(ctx, option);
```

传入不采集字段名必须与下表 **字段名** 一致（字母升序）

| 字段名   | 含义                        | 系统关键 API                                                 | 删除后影响                                      |
| -------- | --------------------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| adid     | android_id                  | Secure#getString(Resolver, ANDROID_ID)                       | 影响设备标识稳定性                              |
| bssid    | WIFI 热点的 MAC 地址        | WifiManager#getConnectionInfo<br />WifiInfo#getBSSID         | 影响风险设备聚集风险的识别                      |
| cell     | 基站信息                    | TelephonyManager#getCellLocation<br />GsmCellLocation#getCid<br />GsmCellLocation#getLac | 影响风险设备聚集风险的识别                      |
| network  | 手机网络链接方式            | TelephonyManager#getNetworkType                              | 影响网络状态相关的逻辑校验                      |
| oaid     | Android开发中匿名设备标识符 | 各手机厂商关键 API 不同                                      | 暂无                                            |
| operator | 运营商编码                  | TelephonyManager#getSimOperator                              | 影响网络状态的校验                              |
| sensor   | 传感器信息                  | SensorManager#getSensorList                                  | 影响与篡改识别相关的逻辑校验                    |
| sensorsData | 传感器数值信息 | SensorManager#registerListener<br />SensorManager#unregisterListener | 暂无影响，后续可能会影响聚集类的策略 |
| sdCacheLimit | SD 卡缓存 |FileInputStream、FileOutputStream|低版本系统上会影响全局标识关联能力|
| props_sn | 系统属性序列号              | SystemProperties.get("sys.serialno")<br />SystemProperties.get("persist.radio.serialno")<br />SystemProperties.get("ro.ril.oem.sno")<br />SystemProperties.get("ro.ril.oem.psno")<br />SystemProperties.get("gsm.serial") | 在 Android 9 及以下版本，可能影响设备标识稳定性 |
| ssid     | WIFI 名称                   | WifiManager#getConnectionInfo<br />WifiInfo#getSSID          | 影响风险设备聚集风险的识别                      |
| wifiip   | 局域网 IP 地址              | WifiManager#getConnectionInfo<br />WifiInfo#getIpAddress     | 影响风险设备聚集风险的识别                      |

### 4 手机弹窗获取“位置”权限

在部分低版本 Android 系统中，启动SDK设备指纹会有弹窗现象，提示授权位置权限，原因是某些 ROM 修改了授权弹窗机制，只要调用 `WifiManager.getConnectionInfo` 方法，系统会自动弹窗，可以通过不采集方式，屏蔽 ssid, bssid, wifiip 字段避免调用 `WifiManager.getConnectionInfo` 方法。
