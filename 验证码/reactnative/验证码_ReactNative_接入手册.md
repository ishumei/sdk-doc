数美验证码 ReactNative 插件目前支持 Android / iOS 平台，验证码功能/限制与原生验证码 SDK 一致，原生验证码 SDK 接入文档见：

1. [Android 验证码接入手册](https://help.ishumei.com/docs/tw/captcha/android/developDoc)；
2.  [iOS 验证码接入手册](https://help.ishumei.com/docs/tw/captcha/iOS/developDoc)。

## ReactNative 接入手册

### 1 工程配置

本文以非框架模式创建的 React Native 项目为例介绍如何接入数美验证码 SDK（以下称 smcaptcha），如果项目使用框架比如 expo pro 等，需要根据实际情况进行调整。

文中使用到的文件可以直接从测试 Demo 中获取，如：

1. Android 相关文件
   1. 原生 SDK：`captcha*.jar`
   2. 插件类：`SmCaptchaModule.java` 和 `SmCaptchaRnPackage.java`
2. iOS 相关文件
   1. 原生 SDK：`SmCaptcha.xcframework`
   2. 插件类：`SmCaptchaModule.h` 和 `SmCaptchaModule.m` 

#### 1.1 导入 Android 原生 SDK 及 插件

导入 SDK：在 `./android/app` 目录下创建 libs 目录并将 `captcha*.jar` 文件拷贝到此目录，修改 `./android/app/build.gradle` 文件，增加如下代码

```groovy
dependencies {
  ... 
  implementation fileTree(include: ['captcha*.jar'], dir: 'libs') // 增加此项，注意文件名称是否匹配
}
```

导入插件：

1. 在 `./android/app/src/main/java/com` 目录下创建 `ishumei/sm_captcha_rn` 目录，并将 `SmCaptchaModule.java` 和 `SmCaptchaRnPackage.java` 文件放入 `sm_captcha_rn` 目录

2. 在 `MainApplication.kt`（此处为 kotlin 文件，java 文件类似）中注册 `SmCaptchaRnPackage`

   ```kotlin
   // 导入类
   import com.ishumei.sm_captcha_rn.SmCaptchaRNPackage
   
   class MainApplication : Application(), ReactApplication {
     override val reactNativeHost: ReactNativeHost =
         object : DefaultReactNativeHost(this) {
           override fun getPackages(): List<ReactPackage> =
               PackageList(this).packages.apply {
                 add(SmCaptchaRNPackage()) // 注册类
               }
           ...
         }
     ...
   }
   ```

#### 1.2 导入 iOS 原生 SDK 及 插件

导入 SDK：

1. 将 `SmCaptcha.xcframework 拷贝到` `./ios` 目录，并在此目录增加 `sm_captcha.podspec` 文件，文件内容如下

   ```podspec
   Pod::Spec.new do |s|
     s.name             = 'sm_captcha'
     s.version          = '0.0.1'
     s.summary          = 'Sm captcha plugin'
     s.description      = <<-DESC
   Sm captcha plugin
                          DESC
     s.homepage         = 'http://example.com'
     s.author           = { 'Your Company' => 'email@example.com' }
     s.source           = { :path => '.' }
     s.platform = :ios, '9.0'
     s.static_framework = true
     s.vendored_frameworks = 'SmCaptcha.xcframework'
     s.pod_target_xcconfig = { 'DEFINES_MODULE' => 'YES', 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'i386' }
   end
   ```
   
2. 修改 `./ios/Podfile`，增加依赖

   ```pod
   pod 'sm_captcha', :path => './sm_captcha.podspec'
   ```
   
   在 `./ios` 目录执行 `pod install`

导入插件：

xcode 打开 ios 项目，将 `SmCaptchaModule.h` 和 `SmCaptchaModule.m` 拷贝到项目中

#### 1.3 使用插件

打开需要展示验证码的页面，比如 `App.tsx`

1. 导入组件

   ```tsx
   import {
   	NativeModules,
   	NativeEventEmitter
   } from 'react-native';
   ```

2. 声明组件及回调方法

   ```tsx
   const { SmCaptchaModule } = NativeModules;
   const eventEmitter = new NativeEventEmitter(SmCaptchaModule);
   
   eventEmitter.addListener('smcaptchaOnInitWithContent', (message) => {
     // web 参数初始化完成时，回调此方法
     // message 为 json 字符串
     const result = JSON.parse(message)
     console.log('smcaptchaOnInitWithContent', result);
   });
   
   eventEmitter.addListener('smcaptchaOnReadyWithContent', (message) => {
     // 验证码图片加载成功时回调此方法
     // message 为 json 字符串
     const result = JSON.parse(message)
     console.log('smcaptchaOnReadyWithContent', result);
   });
   
   eventEmitter.addListener('smcaptchaOnSuccessWithContent', (message) => {
     // 验证结束时回调此方法
     // message 为 json 字符串
     const result = JSON.parse(message)
     console.log('smcaptchaOnSuccessWithContent', result);
     if (result['pass'] == true) { 
       // 代表验证通过
       SmCaptchaModule.dismiss()
     }
   });
   
   eventEmitter.addListener('smcaptchaOnCloseWithContent', (message) => {
     // 带关闭按钮样式的验证码点击关闭按钮时回调此方法
     // message 为 json 字符串
     const result = JSON.parse(message)
     console.log('smcaptchaOnCloseWithContent', result);
     SmCaptchaModule.dismiss();
   });
   
   eventEmitter.addListener('smcaptchaOnErrorWithContent', (message) => {
     // 加载或验证过程中出现错误时，回调此方法，code 码见原生接入文档
     // message 为 json 字符串
     const result = JSON.parse(message)
     console.log('smcaptchaOnErrorWithContent', result);
   });
   
   eventEmitter.addListener('smcaptchaOnError', (message) => {
     // 初始化错误会回调此方法
     // 注意：此处 message 为 字符串 类型
     console.log('smcaptchaOnError', message);
   });
   ```

3. 插件支持的方法

   1. 加载验证码

      ```tsx
      SmCaptchaModule.load({
        organization: YOUR-ORGANIZATION, // 填写数美组织标识
        appId: "default",
        // 支持样式自定义，具体参数见原生文档
      })
      ```

   2. 展示验证码

      ```tsx
      SmCaptchaModule.show({
        canceledOnTouchOutside: true, // 点击弹窗外部是否会取消弹唱
        viewSizeRatio: 0.8 // 设置弹窗相对于屏幕比例，如 1 代表与屏幕等宽
      })
      ```

   3. 取消验证码

      ```tsx
      SmCaptchaModule.dismiss()
      ```

### 2 验证

执行 npm start 启动项目，选择 a 运行 Android 程序，选择 i 运行 iOS 程序。

执行如下操作：

1. 点击 load 按钮加载验证码

2. 点击 show 按钮显示验证码

3. 调用 dismiss 方法隐藏验证码

行为无异常，回调无异常即验证通过。



