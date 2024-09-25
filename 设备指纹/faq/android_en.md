### 1 How to Determine Isolated Process in Android

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
  // Do nothing if it's an isolated process
  if(isIsolated(Process.myUid())){
    return;
  }
}
```

### 2 How to Determine Main Process in Android

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
  // Only start smsdk in the main process
  if (getPackageName().equals(getCurProcessName(context))) {
    ...
    SmAntiFraud.create(context, option);
    ...
  }
  ...
}
```

### 3 How to Block Collection of Certain Fields in Android

When calling the `SmAntiFraud.create(...)` method, pass the `SmOption` field name to block system API calls

```java
SmAntiFraud.SmOption option = new SmAntiFraud.SmOption(); 
... // Other settings
Set<String> notCollect = new HashSet<>(); 
notCollect.add("oaid"); // Do not collect oaid
option.setNotCollect(notCollect);
SmAntiFraud.create(ctx, option);
```

The field names to be excluded must match the **Field Name** in the table below (in alphabetical order)

| Field Name | Meaning                    | Key System API                                               | Impact After Removal                              |
| ---------- | -------------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| adid       | android_id                 | Secure#getString(Resolver, ANDROID_ID)                       | Affects device identifier stability              |
| bssid      | MAC address of WIFI hotspot | WifiManager#getConnectionInfo<br />WifiInfo#getBSSID         | Affects the identification of risk device clusters |
| cell       | Cell tower information     | TelephonyManager#getCellLocation<br />GsmCellLocation#getCid<br />GsmCellLocation#getLac | Affects the identification of risk device clusters |
| network    | Mobile network connection type | TelephonyManager#getNetworkType                              | Affects logic verification related to network status |
| oaid       | Anonymous device identifier in Android development | Key APIs of various phone manufacturers                      | None                                              |
| operator   | Operator code              | TelephonyManager#getSimOperator                              | Affects network status verification              |
| sensor     | Sensor information         | SensorManager#getSensorList                                  | Affects logic verification related to tampering identification |
| sensorsData | Sensor value information | SensorManager#registerListener<br />SensorManager#unregisterListener | No impact for now, may affect clustering strategies later |
| sdCacheLimit | SD card cache |FileInputStream, FileOutputStream|Affects global identifier association capability on lower version systems|
| props_sn   | System property serial number | SystemProperties.get("sys.serialno")<br />SystemProperties.get("persist.radio.serialno")<br />SystemProperties.get("ro.ril.oem.sno")<br />SystemProperties.get("ro.ril.oem.psno")<br />SystemProperties.get("gsm.serial") | May affect device identifier stability on Android 9 and below |
| ssid       | WIFI name                  | WifiManager#getConnectionInfo<br />WifiInfo#getSSID          | Affects the identification of risk device clusters |
| wifiip     | Local area network IP address | WifiManager#getConnectionInfo<br />WifiInfo#getIpAddress     | Affects the identification of risk device clusters |

### 4 Phone Popup to Get "Location" Permission or Show Location Icon in Notification Bar

In some Android systems, initializing the smsdk device fingerprint will cause a popup (or show a location icon in the notification bar) prompting for location permission. The reason is that some ROMs have modified the authorization popup mechanism, and as long as the `WifiManager.getConnectionInfo` method is called, the system will automatically popup (or show the location icon). You can avoid calling the `WifiManager.getConnectionInfo` method by blocking the ssid, bssid, wifiip fields.
