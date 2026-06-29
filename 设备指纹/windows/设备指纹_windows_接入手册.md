# windows端数美设备指纹SDK接入手册

## 1. 适用范围

本文说明 Windows Native SDK 的两类常见接入方式：

- 普通 C++ 程序接入
- Unity C# 程序通过 P/Invoke 接入

SDK 交付产物包括：

```text
include/SmAntiFraud.h
bin/<x86_md|x86_mt|x64_md|x64_mt>/SmAntiFraud.dll
lib/<x86_md|x86_mt|x64_md|x64_mt>/SmAntiFraud.lib
CSharp/SmAntiFraudBridge_win.cs
```

## 2. C++接入

### 2.1 配置项目

#### 2.1.1 拷贝 SDK 文件

以 x64 + MT 为例，对应 SDK 文件如下：

```text
include/SmAntiFraud.h
lib/x64_mt/SmAntiFraud.lib
bin/x64_mt/SmAntiFraud.dll
```

#### 2.1.2 配置 Visual Studio

项目属性中配置：

```text
C/C++ -> 常规 -> 附加包含目录:
  <YourApp>/include

链接器 -> 常规 -> 附加库目录:
  <YourApp>/lib

链接器 -> 输入 -> 附加依赖项:
  SmAntiFraud.lib
```

运行时需要保证 `SmAntiFraud.dll` 能被程序找到。常见方式：

- 放到 exe 同目录
- 放到系统 PATH 中
- 启动前把 DLL 所在目录加入 DLL 搜索路径

推荐放到 exe 同目录。

### 2.2 启动SDK
调用SDK的 `bool smantifraud::SmAntiFraud::Create(const smantifraud::smOption&)` 方法启动SDK  
Create 方法检测 ` smantifraud::smOption` 中的必传参数是否设置，并将检测结果同步返回  
若检测成功，则返回 `true`，SDK 会启动后台任务，异步完成设备数据采集、加密、网络请求，并缓存服务端下发的标识  
若检测失败，即返回 `false`，需要检查参数是否配置正确

以下是 `smantifraud::smOption` 中具体的配置项说明：

| 字段 | 参数类型 | 必填 | 默认值 | 说明 |
|---|---|---:|---|---|
| `organization` | `std::string` | 是 | 空字符串 | 数美分配的 organization |
| `appId` | `std::string` | 否 | `"default"` | 应用 ID |
| `publicKey` | `std::string` | 是 | 空字符串 | RSA 公钥，支持完整 PEM，也支持只传 BEGIN/END 中间的 base64 内容 |
| `url` | `std::string` | 否 | 空字符串，使用 SDK 默认地址 | 服务端地址 |
| `extraInfo` | `std::string` | 否 | 空字符串 | 业务扩展信息，最大 1024 字节 |
| `channel` | `std::string` | 否 | 空字符串 | 渠道信息 |
| `https` | `bool` | 否 | `false` | 是否将默认 http 地址切换为 https |
| `notCollect` | `std::vector<std::string>` | 否 | 空数组 | 不采集字段，例如 `a9`、`a10` |
| `smOnSuccess` | `SmAntiSuccessCallback` | 否 | `0` | 成功回调 |
| `smOnError` | `SmAntiErrorCallback` | 否 | `0` | 失败回调 |

### 2.3 获取标识
调用SDK的 `std::string smantifraud::SmAntiFraud::GetDeviceId()` 方法同步获取设备标识  
该方法首先会尝试获取设备缓存的'B'开头的boxId，若获取失败，则会**阻塞地**采集数据并返回'D'开头，长度较长的boxdata  

**注意：** 不要在启动SDK后立即获取标识，因为启动SDK后的异步任务需要时间，建议在启动SDK后2秒后再调用

### 2.4 代码示例

```cpp
#include "SmAntiFraud.h"

#include <chrono>
#include <iostream>
#include <string>
#include <thread>

void SMANTI_CALL smOnSuccess(const char* deviceId) {
    std::cout << "SmAntiFraud success: "
              << (deviceId == 0 ? "" : deviceId)
              << std::endl;
}

void SMANTI_CALL smOnError(int errorCode) {
    std::cout << "SmAntiFraud error: " << errorCode << std::endl;
}

int main() {
    smantifraud::smOption option;
    option.organization = "your_organization";  // 必填参数
    option.appId = "default";
    option.publicKey = "your_public_key";       // 必填参数
    option.url = "https://your_server/deviceprofile/v4";
    option.channel = "default";
    option.extraInfo = "{\"scene\":\"login\"}";
    option.notCollect.push_back("a01");
    option.smOnSuccess = &smOnSuccess;
    option.smOnError = &smOnError;

    smantifraud::SmAntiFraud sdk;
    if (!sdk.Create(option)) {
        std::cout << "Create failed" << std::endl;
        return 1;
    }

    // 2秒后再调用获取设备标识的方法
    std::this_thread::sleep_for(std::chrono::seconds(2));

    std::string deviceId = sdk.GetDeviceId();
    std::cout << "GetDeviceId: " << deviceId << std::endl;

    return 0;
}
```

### 2.5 C++ 线程注意事项
SDK 的成功和失败回调不在主线程触发，不要在回调中直接操作 UI 控件；如需更新 UI，应切回应用主线程。

## 3. Unity C#接入

### 3.1 配置项目

#### 3.1.1 拷贝 C# 脚本

将 SDK 交付包中的 `SmAntiFraudBridge_win.cs` 脚本复制到 Unity 工程中

```text
Assets/Scripts/SmAntiFraudBridge_win.cs
```

`SmAntiFraudBridge_win.cs` 暴露以下接口：

| 类型 | 说明 |
|---|---|
| `SmAntiFraudBridge_win` | 静态封装类，内部声明并调用 Native C ABI |
| `SmAntiFraudBridge_win.SmOption` | 面向 Unity 的 C# 封装参数结构，脚本内部会转换为 Native C ABI 结构 |
| `SmAntiFraudBridge_win.Create` | 初始化 SDK，并注册成功/失败回调 |
| `SmAntiFraudBridge_win.GetDeviceId` | 获取 deviceId |
| `SmAntiFraudBridge_win.GetSDKVersion` | 获取 SDK 版本号 |

#### 3.1.2 拷贝 Native DLL

根据 Unity Player 架构选择对应的 DLL。推荐优先使用 MT 版本，减少客户工程对 VC++ 运行库的额外依赖。

| Unity Player 架构 | SDK DLL | Unity 目标路径 |
|---|---|---|
| Windows x64 | `bin/x64_mt/SmAntiFraud.dll` | `Assets/Plugins/x64/SmAntiFraud.dll` |
| Windows x86 | `bin/x86_mt/SmAntiFraud.dll` | `Assets/Plugins/x86/SmAntiFraud.dll` |

`SmAntiFraudBridge_win.cs` 中通过 `DllImport("SmAntiFraud")` 调用 Native 接口，DLL 文件名必须保持为 `SmAntiFraud.dll`。

### 3.2 启动SDK
调用SDK的 `int SmAntiFraudBridge_win.Create(SmAntiFraudBridge_win.SmOption)` 方法启动SDK  
`Create` 方法检测 `SmAntiFraudBridge_win.SmOption` 中的必传参数是否设置，并将检测结果同步返回  
若检测成功，则返回 `0`，SDK 会启动后台任务，异步完成设备数据采集、加密、网络请求，并缓存服务端下发的标识  
若检测失败，即返回非 0，需要检查参数是否配置正确

以下是 `SmAntiFraudBridge_win.SmOption` 中具体的配置项说明：

| 字段 | C# 参数类型 | 必填 | 默认值 | 说明 |
|---|---|---:|---|---|
| `organization` | `string` | 是 | `null` | 数美分配的 organization |
| `appId` | `string` | 否 |  `default` | 应用 ID |
| `publicKey` | `string` | 是 | `null` | RSA 公钥，支持完整 PEM，也支持只传 BEGIN/END 中间的 base64 内容 |
| `url` | `string` | 否 | `null`，使用 SDK 默认地址 | 服务端地址 |
| `extraInfo` | `string` | 否 | `null` | 业务扩展信息，最大 1024 字节 |
| `channel` | `string` | 否 | `null` | 渠道信息 |
| `https` | `int`，按 BOOL 使用 | 否 | `0`，表示 `false` | `0=false`，非 `0=true`；为兼容 Native C ABI 保持 `int` |
| `notCollectCsv` | `string` | 否 | `null` | 不采集字段，逗号分隔 |
| `smOnSuccess` | `SmSuccessCallback` | 否 | `null` | 成功回调，签名为 `void Callback(string deviceId)`；由 `SmAntiFraudBridge_win` 内部持有委托引用，避免被 GC 回收 |
| `smOnError` | `SmErrorCallback` | 否 | `null` | 失败回调，签名为 `void Callback(int errorCode)`；由 `SmAntiFraudBridge_win` 内部持有委托引用，避免被 GC 回收 |

### 3.3 获取标识
调用SDK的 `string SmAntiFraudBridge_win.GetDeviceId()` 方法同步获取设备标识  
该方法首先会尝试获取设备缓存的'B'开头的boxId，若获取失败，则会**阻塞地**采集数据并返回'D'开头，长度较长的boxdata  

**注意：** 不要在启动SDK后立即获取标识，因为启动SDK后的异步任务需要时间，建议在启动SDK后2秒后再调用

### 3.4 Unity 调用示例

```csharp
using System;
using System.Collections;
using UnityEngine;

public class YourSmAntiFraudBehaviour : MonoBehaviour
{
    private void Start()
    {
        var option = new SmAntiFraudBridge_win.SmOption
        {
            organization = "your_organization", // 必填参数
            appId = "default",
            publicKey = "your_public_key",      // 必填参数
            url = "https://your_server/deviceprofile/v4",
            extraInfo = "{\"scene\":\"unity\"}",
            channel = "unity",
            https = 1, // 0=false, non-zero=true
            notCollectCsv = "a01,a02",
            smOnSuccess = smOnSuccess,
            smOnError = smOnError
        };

        int ret = SmAntiFraudBridge_win.Create(option);
        Debug.Log("SmAntiFraudBridge_win.Create ret=" + ret);

        if (ret == SmAntiFraudBridge_win.Success)
        {
            // 2秒后再调用获取设备标识的方法
            StartCoroutine(GetDeviceIdAfterDelay());
        }
    }

    private IEnumerator GetDeviceIdAfterDelay()
    {
        yield return new WaitForSecondsRealtime(2f);

        Debug.Log("SmAntiFraudBridge_win.GetDeviceId=" + SmAntiFraudBridge_win.GetDeviceId());
    }

    private static void smOnSuccess(string deviceId)
    {
        // 保存 deviceId
    }

    private static void smOnError(int errorCode)
    {
        // errorCode错误码
    }
}
```

### 3.5 Unity 线程注意事项
SDK 的回调不在 Unity 主线程触发，而 Unity API 大多只能在主线程调用。

因此，`smOnSuccess` 和 `smOnError` 回调中不要直接操作场景对象、UI 组件、`GameObject`、`Transform` 等 Unity 对象。如果需要更新界面或游戏对象，建议在回调中只保存结果或错误码，然后在 `Update()`、协程或项目已有的主线程派发器中处理。

## 4. 返回码和错误码说明

### 4.1 回调错误码

| 返回码/错误码 |  说明 |
|---|---|
| `1903` |  服务端响应异常或响应内容无法解析 |
| `2001` |  网络请求失败 |
| `2002` |  设备信息封装或加密失败 |

以上错误码会通过 `smOnError` 回调返回。C++ 接入时对应 `smOption.smOnError`，Unity 接入时对应 `SmOption.smOnError`。
