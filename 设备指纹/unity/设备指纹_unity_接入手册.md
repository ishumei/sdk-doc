# 设备指纹Unity插件接入文档

## 1. 设备指纹插件结构

插件包含以下文件：

- `Plugins/Android/smsdk-x.x.x.aar` - Android 设备指纹SDK
- `Plugins/iOS/SmAntiFraud-x.x.x.xcframework` - iOS 设备指纹SDK
- `Plugins/iOS/SmAntiFraudExport.h` - iOS导出头文件
- `Plugins/iOS/SmAntiFraudExport.mm` - iOS导出实现
- `Scripts/SmAntiFraudBridge.cs` - Unity与原生SDK的桥接脚本

## 2. 工程配置

### 2.1 导入插件

将整个插件目录复制到Unity项目的 `Assets` 目录下，确保目录结构保持不变

### 2.2 iOS端依赖配置

iOS SDK依赖 `IOKit.framework`，因此需要在Unity中配置依赖：

在Unity中，选中iOS 设备指纹SDK `Assets > Plugins > iOS > SmAntiFraud`，勾选右侧的 `Inspector > Platform Settings > Rarely used frameworks > IOKit`

### 2.3 iOS端隐私文件配置

务必按照[iOS配置隐私文件](https://help.ishumei.com/docs/tw/sdk/iOS/developDoc/#15-%E9%85%8D%E7%BD%AE%E9%9A%90%E7%A7%81%E6%96%87%E4%BB%B6)中的1.5节配置 `PrivacyInfo.xcprivacy` 文件

### 2.4 Android端权限配置

声明权限，在 AndroidManifest.xml 中添加如下权限

```xml
<!-- 必选权限 -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

<!-- 建议权限 -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

### 2.5 Android端防混淆

向 proguard-rules.pro 文件中添加 smsdk 防混淆规则，如下

```properties
-keep class com.ishumei.** {*;}
```

## 3. 启动SDK

启动SDK会收集设备数据并进行网络请求，确保在用户同意隐私政策后再启动SDK

调用 `SmAntiFraudBridge.smCreate()` 方法启动SDK前需要先调用 `SmAntiFraudBridge.initSmoption()` 配置启动参数，参考如下代码：

```csharp
using UnityEngine;

public class ExampleClass : MonoBehaviour
{
    void smCreate()
    {
        // 1. 配置启动参数
        SmAntiFraudBridge.initSmoption();
        
        // 2. 必需项，设置组织ID
        SmAntiFraudBridge.optionSetOrganization("your_organization");
        
        // 3. 必需项，设置Android公钥
        SmAntiFraudBridge.optionSetAndroidPublicKey("android_public_key");
        
        // 4. 必需项，设置iOS公钥
        SmAntiFraudBridge.optionSetIOSPublicKey("ios_public_key");
        
        // 5. 必需项，设置App ID
        SmAntiFraudBridge.optionSetAppId("your_app_id");

      	// 6. 可选项，配置不采集字段
#if UNITY_ANDROID && !UNITY_EDITOR
    SmAntiFraudBridge.optionAddNotCollectAndroid("imei");
    SmAntiFraudBridge.optionAddNotCollectAndroid("wifiip");
#elif UNITY_IOS && !UNITY_EDITOR
    SmAntiFraudBridge.optionAddNotCollectIOS("idfa");
#endif
        
        // 7.启动SDK
        bool success = SmAntiFraudBridge.smCreate();
        if (success)
        {
            Debug.Log("设备指纹启动成功");
        }
        else
        {
            // 启动失败，须检查启动参数是否配置正确
            Debug.Log("设备指纹启动失败");
        }
    }
}
```

## 4. 获取标识
务必在启动SDK后再调用获取标识的方法，建议等待2秒后再调用，以便SDK收集数据和完成网络传输

客户端 `SmAntiFraudBridge.smGetDeviceId()` 正常情况下会获取到89位的 boxId。

若获取到非boxId，需要检查启动SDK的参数和时机是否正确。

```csharp
// 获取标识
string boxId = SmAntiFraudBridge.smGetDeviceId();
```

## 5. 其他可选配置

| 方法 | 说明 | 数据类型 |
|------|------|---------|
| `optionSetUrl` | 设置设备指纹URL<br />代理模式、私有化模式中须要设置设备数据上报地址 | string |
| `optionSetConfUrl` | 设置云配URL<br />代理模式、私有化模式中须要设置请求云配的地址 | string |
| `optionSetExtraInfo` | 设置额外信息 | string |
| `optionAddNotCollectAndroid` | 添加Android不采集项 | string |
| `optionAddNotCollectIOS` | 添加iOS不采集项 | string |
| `optionSetUsingShortBoxData` | 设置是否使用短boxdata，默认`false`。<br />如需使用短boxdata，须要设置为`true` | bool |
| `optionSetAutoUploadVData` | 设置是否自动上传微行为数据，默认`false`。<br />建议设置为`true`，开启微行为数据自动上传 | bool |

## 6. 检查接入是否成功

调用 `SmAntiFraudBridge.smCreate()` 方法获取返回值为 `true`。

调用 `SmAntiFraudBridge.smGetDeviceId()` 方法返回值为89位的 boxId，如 `Bm21V93t5QwTNdwyQxxxxxRYuSnOuwwylqZvz8Lixxxxx17lRMqcQ1jz9RwN6qW31/Z0YYmxN8KQnrya9xxxxxx==`。


## 7. 微行为接入

微行为插件的目录结构如下：
```
微行为/
├── Plugins/
│   ├── Android/
│   │   ├── SmScreenTouchDetector.java      # Android端触摸检测封装实现
│   │   ├── smsdk_screentouch-release.aar   # 安卓平台原生微行为SDK
│   │
│   ├── iOS/
│   │   ├── SmAntiFraudScreenTouch.xcframework  # iOS平台原生微行为SDK
│   │   ├── SmScreenTouchExport.h           # iOS端触摸检测封装头文件
│   │   ├── SmScreenTouchExport.mm          # iOS端触摸检测封装实现
│   │
├── Scripts/
│   ├── SmScreenTouchBridge.cs              # Unity与原生平台桥接代码
```

### 7.1 导入插件
将整个微行为插件目录复制到Unity项目的`Assets`目录下，确保目录结构保持不变

### 7.2 Android端配置
Android端接入须要在自定义Application中的`onCreate()`方法中调用插件的初始化方法`initDetector(Context context)`，如下：

```java
package com.yourcompany.yourapp;

import android.app.Application;
import com.ishumei.game.SmScreenTouchDetector;

// 自定义Application
public class CustomApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
      
        // 初始化触摸检测
        SmScreenTouchDetector.initDetector(this);
    }
}
```

**若Android端没有自定义 `Application` 类，则需要按照以下方式配置**：

1. 创建自定义Application类：

   - 在 `Plugins/Android/` 目录下创建一个Java文件，例如 `CustomApplication.java`
   - 继承自 `android.app.Application`
   - 在 `onCreate()` 方法中调用微行为SDK的初始化方法

2. 实现代码参考上面代码示例，务必在 `onCreate()` 方法中调用初始化方法 `initDetector(Context context)`

3. 参考[Unity Manual](https://docs.unity3d.com/2022.2/Documentation/Manual/overriding-android-manifest.html)，打开**Custom Main Manifest**开关

4. 指定自定义的application类名

   在 `Plugins/Android/AndroidManifest.xml` 文件的 `<application>` 标签中添加 `android:name` 属性，指定自定义的Application类


    ```xml
    <application
        android:name="com.yourcompany.yourapp.CustomApplication">
        <!-- 其他属性 -->
        <!-- 其他组件 -->
    </application>
    ```
### 7.3 开启自动上报
微行为数据默认**不会**自动上传，建议在 `SmAntiFraudBridge.smCreate()` 前调用 `SmAntiFraudBridge.optionSetAutoUploadVData(true)` 方法开启自动上传：

```csharp
// 开启微行为自动上传
SmAntiFraudBridge.optionSetAutoUploadVData(true);
```
若不开启自动上传功能，则可以通过设备指纹SDK的 `SmAntiFraudBridge.smGetVdata()` 方法获取微行为数据后，通过业务事件的 `data.microBehavior` 字段上报。

### 7.4 调用API
**下列的API务必在设备指纹SDK的 `SmAntiFraudBridge.smCreate()` 方法之后调用。**

调用 `smStartScreenTouchDetector()` 方法后，SDK开始收集用户的触摸行为数据

```csharp
// 开始触摸检测，收集用户触摸行为数据
SmScreenTouchBridge.smStartScreenTouchDetector();
```

调用 `smStopScreenTouchDetector()` 方法后，SDK停止收集用户的触摸行为数据

```csharp
// 停止触摸检测
SmScreenTouchBridge.smStopScreenTouchDetector();
```

