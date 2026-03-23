# windows端数美设备指纹SDK接入手册

## 1. 工程配置

### 1.1 导入SDK包

在Visual Studio中，选择需要接入SDK的工程

1. 右键工程的“依赖项”选项，选择“添加项目引用”，弹出“引用管理器”
2. 在“引用管理器”的底部，选择"浏览"项，选择数美的.dll库，SmAntiFraud.dll

### 1.2 添加依赖

数美的库文件依赖如下库，需要使用 NuGet 搜索并添加以下项目引用

1. System.Text.Json， 6.0.0版本
2. System.Management，8.0.0版本
3. BouncyCastle.Cryptography，2.4.0版本

## 2. 标准接入

### 2.1 启动SDK

1. 在需要的文件中，使用 `using` 关键字，导入数美命名空间。

   ```c#
   using SmAntiFraud;
   ```

2. 启动SDK需要 `Smoption` 实例，具体参数如下

   | 字段           | 数据类型              | 是否必填 | 字段说明                                                     |
   | -------------- | --------------------- | -------- | ------------------------------------------------------------ |
   | organization   | string                | 是       | 数美分配的公司标识                                           |
   | appid          | string                | 是       | 应用标识                                                     |
   | publicKey      | string                | 是       | 公钥                                                         |
   | url            | string                | 否       | 设备数据上传的服务器url                                      |
   | extraInfo      | string                | 否       | 额外信息                                                     |
   | channel        | string                | 否       | 渠道                                                         |
   | https          | bool                  | 否       | 是否使用https，若使用https须配置此项为true<br />默认使用http请求 |
   | notCollect     | `List<string>`        | 否       | 不采集项                                                     |
   | smAntiDelegate | object:SmAntiDelegate | 否       | 使用回调方式获取设备标识或错误码时，需要设置此参数。<br />该参数需要继承自SmAntiDelegate类，并实现SmOnSuccess和SmOnError方法 |

3. 调用SDK的 `SmAntiFraud.ShareInstance().Create(Smoption)` 方法启动SDK

   ```C#
   bool createRes = SmAntiFraud.SmAntiFraud.ShareInstance().Create(smoption);
   ```

### 2.2 获取标识

客户端获取到的标识分为 `boxId` 和 `boxData` ，`boxId` 的长度是89，`boxData`  长度不固定，约为 1.5 KB。

客户端有同步和异步的方式，其中异步获取的方式只会获取到 `boxId` 

- 同步获取标识

  调用SDK的 `SmAntiFraud.ShareInstance().GetDeviceId()` 方法获取

  ```c#
  string did = SmAntiFraud.SmAntiFraud.ShareInstance().GetDeviceId();
  ```

- 异步获取标识

  1. 创建一个类，实现 SmAntiDelegate 接口的`SmOnSuccess(string)`和`SmError(int)`回调方法
  2. 初始化一个上述创建类的实例，在初始化数美 SDK 的时候，设置其为 Smoption 的 `smAntiDelegate` 属性即可

### 2.3 代码示例

```c#
using SmAntiFraud;

class ExampleClass : SmAntiDelegate
{
    static void initSm(){
        // 1. 设置 Smoption 参数
        Smoption opt = new Smoption
        {
            organization = "xxxxxx",
            appid = "default",
            publicKey = @"------BEGIN PUBLIC KEY------
xxxxxxxxxx
------END PUBLIC KEY------
",
            smAntiDelegate = this,
            https = true,
        };
        // 2. 启动SDK
        bool createRes = SmAntiFraud.SmAntiFraud.ShareInstance().Create(opt);
        
        // 3. 主动获取标识
        string did = SmAntiFraud.SmAntiFraud.ShareInstance().GetDeviceId();
    }
    
    // 数美成功回调方法
    public void SmOnSuccess(string boxid)
    {
        Debug.WriteLine(boxid);
    }

    // 数美失败回调方法
	public void SmOnError(int errorCode)
    {
        Debug.WriteLine(errorCode);
    }
}
```

### 2.4 测试

通过以下判断，检测SDK是否接入成功

1. 调用 `SmAntiFraud.SmAntiFraud.ShareInstance().Create(opt)` 方法获取的返回值为 `true`
2. 调用 `SmAntiFraud.SmAntiFraud.ShareInstance().GetDeviceId()` 方法或者`SmOnSuccess(string)`回调中获取到长度为 89 位的 `boxid`

### 2.5 错误码

| 错误码 | 原因                 | 排查                               |
| ------ | -------------------- | ---------------------------------- |
| 1901   | qps 超限             |                                    |
| 1902   | 服务端解密错误       | 排查 org 和 publickey 是否填写正确 |
| 1903   | 服务端错误           | 联系数美人员                       |
| 9101   | org 错误或服务未开通 | 联系数美人员                       |
| 2001   | 网络错误             | 检查网络                           |
| 2002   | SDK 内部加密错误     | 排查 publickey 是否填写正确        |



