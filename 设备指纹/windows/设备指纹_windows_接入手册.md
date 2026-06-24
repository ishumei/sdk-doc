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

## 2. 接口概览

### 2.1 C++ 包装类接口

C++ 包装类定义在 `namespace smantifraud` 中，内部调用 C ABI，但对 `std::string` 和 `std::vector<std::string>` 做了封装。

| 接口 | 参数 | 参数类型 | 返回类型 | 说明 | 同步/异步 |
|---|---|---|---|---|---|
| `smantifraud::SmAntiFraud::Create` | `options` | `const smantifraud::smOption&` | `bool` | 初始化 SDK，并启动后台采集和上传 | 接口同步返回，上传异步执行 |
| `smantifraud::SmAntiFraud::GetDeviceId` | 无 | 无 | `std::string` | 获取 deviceId；失败时返回空字符串 | 同步 |
| `smantifraud::SmAntiFraud::GetSDKVersion` | 无 | 无 | `std::string` | 获取 SDK 版本号；失败时返回空字符串 | 同步 |

### 2.2 Unity C# P/Invoke 接口

Unity 脚本底层通过 P/Invoke 调用 C ABI。

| C# 声明 | Native 导出名 | 参数 | 参数类型 | 返回类型 | 说明 |
|---|---|---|---|---|---|
| `SmAntiFraudBridge_win.Create` | `SmAntiFraudCreate_C` | `options`, `smOnSuccess`, `smOnError` | `SmAntiFraudBridge_win.Options`, `SmSuccessCallback`, `SmErrorCallback` | `int` | 初始化 SDK，并注册成功/失败回调 |
| `SmAntiFraudGetDeviceId_C` | `SmAntiFraudGetDeviceId_C` | `buffer`, `bufferSize` | `StringBuilder`, `int` | `int` | 获取 deviceId |
| `SmAntiFraudGetSDKVersion_C` | `SmAntiFraudGetSDKVersion_C` | `buffer`, `bufferSize` | `StringBuilder`, `int` | `int` | 获取 SDK 版本号 |

回调：

| 回调 | C/C++ 类型 | Unity C# 类型 | 说明 |
|---|---|---|---|
| `smOnSuccess` | `SmAntiSuccessCallback` | `SmSuccessCallback` | 服务端返回成功并下发 deviceId 后触发 |
| `smOnError` | `SmAntiErrorCallback` | `SmErrorCallback` | 参数、网络、加密、响应解析等失败时触发 |

注意：`smOnSuccess` 和 `smOnError` 在 SDK 工作线程中触发，不在 UI 主线程中触发。


## 3. 参数说明

### 3.1 C++ 参数：`smantifraud::smOption`

| 字段 | 参数类型 | 必填 | 默认值 | 说明 |
|---|---|---:|---|---|
| `organization` | `std::string` | 是 | 空字符串 | 数美分配的 organization |
| `appId` | `std::string` | 否 | `"default"` | 应用 ID |
| `publicKey` | `std::string` | 是 | 空字符串 | RSA 公钥 PEM 字符串 |
| `url` | `std::string` | 否 | 空字符串，使用 SDK 默认地址 | 服务端地址 |
| `extraInfo` | `std::string` | 否 | 空字符串 | 业务扩展信息，最大 1024 字节 |
| `channel` | `std::string` | 否 | 空字符串 | 渠道信息 |
| `https` | `bool` | 否 | `false` | 是否将默认 http 地址切换为 https |
| `notCollect` | `std::vector<std::string>` | 否 | 空数组 | 不采集字段，例如 `a9`、`a10` |
| `smOnSuccess` | `SmAntiSuccessCallback` | 否 | `0` | 成功回调 |
| `smOnError` | `SmAntiErrorCallback` | 否 | `0` | 失败回调 |
| `userData` | `void*` | 否 | `0` | 透传用户数据，SDK 只保存并回传，不解析 |

### 3.2 Unity C# 参数：P/Invoke `SmAntiOption`

C# 结构体需要与 Native `SmAntiOption` 字段顺序保持一致。字符串按 ANSI `LPStr` 传入，BOOL 字段使用 `int` 表达。

| 字段 | C# 参数类型 | Native 参数类型 | 必填 | 默认值 | 说明 |
|---|---|---|---:|---|---|
| `organization` | `string` | `const char*` | 是 | `null` | 数美分配的 organization |
| `appId` | `string` | `const char*` | 否 | `null`，SDK 内部按 `default` 处理 | 应用 ID |
| `publicKey` | `string` | `const char*` | 是 | `null` | RSA 公钥 PEM 字符串 |
| `url` | `string` | `const char*` | 否 | `null`，使用 SDK 默认地址 | 服务端地址 |
| `extraInfo` | `string` | `const char*` | 否 | `null` | 业务扩展信息，最大 1024 字节 |
| `channel` | `string` | `const char*` | 否 | `null` | 渠道信息 |
| `https` | `int`，按 BOOL 使用 | `int` | 否 | `0`，表示 `false` | `0=false`，非 `0=true` |
| `notCollectCsv` | `string` | `const char*` | 否 | `null` | 不采集字段，逗号分隔 |
| `smOnSuccess` | `SmSuccessCallback` | `SmAntiSuccessCallback` | 否 | `null` | 成功回调；需要在 C# 侧持有委托引用，避免被 GC 回收 |
| `smOnError` | `SmErrorCallback` | `SmAntiErrorCallback` | 否 | `null` | 失败回调；需要在 C# 侧持有委托引用，避免被 GC 回收 |
| `userData` | `IntPtr` | `void*` | 否 | `IntPtr.Zero` | 透传用户数据，SDK 只保存并回传，不解析 |


## 4. 普通 C++ 接入

### 4.1 拷贝 SDK 文件

以 x64 + MT 为例，对应 SDK 文件如下：

```text
include/SmAntiFraud.h
lib/x64_mt/SmAntiFraud.lib
bin/x64_mt/SmAntiFraud.dll
```

### 4.2 配置 Visual Studio

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

### 4.3 使用 C++ 包装类接入

```cpp
#include "SmAntiFraud.h"

#include <iostream>
#include <string>

void SMANTI_CALL smOnSuccess(const char* deviceId, void* userData) {
    std::cout << "SmAntiFraud success: "
              << (deviceId == 0 ? "" : deviceId)
              << std::endl;
}

void SMANTI_CALL smOnError(int errorCode, void* userData) {
    std::cout << "SmAntiFraud error: " << errorCode << std::endl;
}

int main() {
    smantifraud::smOption options;
    options.organization = "your_organization";
    options.appId = "default";
    options.publicKey = "your_public_key";
    options.url = "https://your_server/deviceprofile/v4";
    options.channel = "default";
    options.extraInfo = "{\"scene\":\"login\"}";
    options.notCollect.push_back("a01");
    options.smOnSuccess = &smOnSuccess;
    options.smOnError = &smOnError;

    smantifraud::SmAntiFraud sdk;
    if (!sdk.Create(options)) {
        std::cout << "Create failed" << std::endl;
        return 1;
    }

    std::string deviceId = sdk.GetDeviceId();
    std::cout << "GetDeviceId: " << deviceId << std::endl;

    return 0;
}
```

### 4.4 使用 C ABI 接入

如果不使用 C++ 包装类，可以直接使用 C ABI。

```cpp
#include "SmAntiFraud.h"

#include <iostream>
#include <string>
#include <vector>
#include <cstring>

void SMANTI_CALL smOnSuccess(const char* deviceId, void* userData) {
    std::cout << "success: " << (deviceId ? deviceId : "") << std::endl;
}

void SMANTI_CALL smOnError(int errorCode, void* userData) {
    std::cout << "error: " << errorCode << std::endl;
}

std::string ReadDeviceId() {
    int required = SmAntiFraudGetDeviceId_C(0, 0);
    if (required <= 0) {
        return std::string();
    }

    std::vector<char> buffer(static_cast<size_t>(required) + 1);
    int written = SmAntiFraudGetDeviceId_C(&buffer[0], static_cast<int>(buffer.size()));
    if (written < 0) {
        return std::string();
    }
    return std::string(&buffer[0], static_cast<size_t>(written));
}

int main() {
    SmAntiOption option;
    memset(&option, 0, sizeof(option));

    option.organization = "your_organization";
    option.appId = "default";
    option.publicKey = "your_public_key";
    option.url = "https://your_server/deviceprofile/v4";
    option.https = 1;
    option.notCollectCsv = "a01,a02,a03";
    option.smOnSuccess = &smOnSuccess;
    option.smOnError = &smOnError;

    int ret = SmAntiFraudCreate_C(&option);
    if (ret != SMANTI_SUCCESS) {
        std::cout << "SmAntiFraudCreate_C failed: " << ret << std::endl;
        return 1;
    }

    std::cout << "deviceId: " << ReadDeviceId() << std::endl;
    return 0;
}
```

## 5. Unity C# 接入

### 5.1 拷贝 C# 脚本

将 SDK 交付包中的 `SmAntiFraudBridge_win.cs` 脚本复制到 Unity 工程，例如：

```text
Assets/Scripts/SmAntiFraudBridge_win.cs
```

`SmAntiFraudBridge_win.cs` 包含：

| 类型 | 说明 |
|---|---|
| `SmAntiFraudBridge_win` | 静态封装类，内部声明并调用 Native C ABI |
| `SmAntiFraudBridge_win.Options` | 对应 Native `SmAntiOption` 的 C# 参数结构 |
| `SmAntiFraudBridge_win.Create` | 初始化 SDK，并注册成功/失败回调 |
| `SmAntiFraudBridge_win.GetDeviceId` | 获取 deviceId |
| `SmAntiFraudBridge_win.GetSDKVersion` | 获取 SDK 版本号 |

### 5.2 拷贝 Native DLL

根据 Unity Player 架构选择 DLL。推荐优先使用 MT 版本，减少客户工程对 VC++ 运行库的额外依赖。

| Unity Player 架构 | SDK DLL | Unity 目标路径 |
|---|---|---|
| Windows x64 | `bin/x64_mt/SmAntiFraud.dll` | `Assets/Plugins/x64/SmAntiFraud.dll` |
| Windows x86 | `bin/x86_mt/SmAntiFraud.dll` | `Assets/Plugins/x86/SmAntiFraud.dll` |

`SmAntiFraudBridge_win.cs` 中通过 `DllImport("SmAntiFraud")` 调用 Native 接口，DLL 文件名必须保持为 `SmAntiFraud.dll`。


### 5.3 Unity 调用示例

```csharp
using System;
using System.Runtime.InteropServices;
using UnityEngine;

public class YourSmAntiFraudBehaviour : MonoBehaviour
{
    private void Start()
    {
        var options = new SmAntiFraudBridge_win.Options
        {
            organization = "your_organization",
            appId = "default",
            publicKey = "your_public_key",
            url = "https://your_server/deviceprofile/v4",
            extraInfo = "{\"scene\":\"unity\"}",
            channel = "unity",
            https = 1, // 0=false, non-zero=true
            notCollectCsv = "a01,a02",
            userData = IntPtr.Zero
        };

        int ret = SmAntiFraudBridge_win.Create(options, smOnSuccess, smOnError);
        Debug.Log("SmAntiFraudBridge_win.Create ret=" + ret);
        Debug.Log("SmAntiFraudBridge_win.GetDeviceId=" + SmAntiFraudBridge_win.GetDeviceId());
    }

    private static void smOnSuccess(IntPtr deviceIdPtr, IntPtr userData)
    {
        string deviceId = Marshal.PtrToStringAnsi(deviceIdPtr) ?? string.Empty;
        Debug.Log("SmAntiFraud success: " + deviceId);
    }

    private static void smOnError(int errorCode, IntPtr userData)
    {
        Debug.LogError("SmAntiFraud error: " + errorCode);
    }
}
```

### 5.4 Unity 线程注意事项

SDK 回调不保证在 Unity 主线程执行。Unity API 大多只能在主线程调用。

更稳妥的方式是：

- 回调里只保存数据或投递事件
- 在 `Update()` 中从队列取出结果，再操作 Unity 对象

示例：

```csharp
private readonly object _lock = new object();
private readonly System.Collections.Generic.Queue<string> _events =
    new System.Collections.Generic.Queue<string>();

private void smOnSuccess(IntPtr deviceIdPtr, IntPtr userData)
{
    string deviceId = Marshal.PtrToStringAnsi(deviceIdPtr) ?? string.Empty;
    lock (_lock)
    {
        _events.Enqueue("success:" + deviceId);
    }
}

private void Update()
{
    lock (_lock)
    {
        while (_events.Count > 0)
        {
            Debug.Log(_events.Dequeue());
        }
    }
}
```
