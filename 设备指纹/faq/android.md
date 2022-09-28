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
  // 只在主进程初始化 smsdk
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
// 大小写敏感，所有字母应为小写
notCollect.add("imei"); // 不采集 IMEI
notCollect.add("apps"); // 不采集 应用安装列表
option.setNotCollect(notCollect);
SmAntiFraud.create(ctx, option);
```

传入不采集字段名必须与下表 **字段名** 一致（字母升序）

| 字段名   | 含义                               | 系统关键 API                                                 | 删除后影响                                                   |
| -------- | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| abtmac   | 蓝牙 MAC 地址                      | BluetoothAdapter#getAddress                                  | 轻微                                                         |
| apps     | 应用列表，展示最近安装的 50 个 APP | ApplicationPackageManager#getInstalledPackages               | 影响所有积分墙的识别，影响部分安装黑产工具的识别             |
| aps      | 周边 WIFI 信息列表                 | WifiManager#getScanResults                                   | 影响风险设备聚集风险的识别                                   |
| bssid    | WIFI 热点的 MAC 地址               | WifiInfo#getBSSID                                            | 影响风险设备聚集风险的识别                                   |
| cell     | 基站信息                           | TelephonyManager#getCellLocation<br />GsmCellLocation#getCid<br />GsmCellLocation#getLac | 影响风险设备聚集风险的识别                                   |
| iccid    | SIM 卡唯一识别号码                 | TelephonyManager#getSimSerialNumber                          | 轻微                                                         |
| imei     | 手机 IMEI                          | TelephonyManager#getDeviceId                                 | 在 Android 9 及以下版本，可能影响设备标识稳定性              |
| imsi     | 国际移动用户识别码                 | TelephonyManager#getSubscriberId                             | 轻微                                                         |
| mac      | 本机 WIFI 网卡的 MAC 地址          | WifiInfo#getMacAddress                                       | 轻微                                                         |
| net      | 本机网络设备列表                   | NetworkInterface#getNetworkInterfaces<br />NetworkInterface#getHardwareAddress<br />WifiInfo#getMacAddress<br />InetAddress#getHostAddress | 在 Android 9 及以下版本，可能影响设备标识稳定性<br/>影响风险设备聚集风险的识别，影响网卡信息的校验 |
| network  | 手机网络链接方式                   | TelephonyManager#getNetworkType                              | 影响网络状态相关的逻辑校验                                   |
| operator | 运营商编码                         | TelephonyManager#getSimOperator                              | 影响网络状态的校验                                           |
| riskapp  | 黑产 APP 应用列表                  | PackageManager#getPackageInfo<br />旧版本 SDK 会调用以下两个方法：<br />ApplicationPackageManager#getInstalledApplications<br />ApplicationPackageManager#getInstalledPackages | 影响风险 APP 的识别                                          |
| sensor   | 传感器信息                         | SensorManager#getSensorList                                  | 影响与篡改识别相关的逻辑校验                                 |
| sn       | 硬件序列号                         | Build#getSerial<br />Build#SERIAL<br />                      | 在 Android 9 及以下版本，可能影响设备标识稳定性              |
| props_sn | 系统属性序列号                     | SystemProperties.get("sys.serialno")<br />SystemProperties.get("persist.radio.serialno")<br />SystemProperties.get("ro.ril.oem.sno")<br />SystemProperties.get("ro.ril.oem.psno")<br />SystemProperties.get("gsm.serial") | 在 Android 9 及以下版本，可能影响设备标识稳定性              |
| ssid     | WIFI 名称                          | WifiInfo#getSSID                                             | 影响风险设备聚集风险的识别                                   |
| wifiip   | 局域网 IP 地址                     | WifiInfo#getIpAddress                                        | 影响风险设备聚集风险的识别                                   |
| adid     | android_id                         | Secure#getString(Resolver, ANDROID_ID)                       | 影响设备标识稳定性                                           |

### 