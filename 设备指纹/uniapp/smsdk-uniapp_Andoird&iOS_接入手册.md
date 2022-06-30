

# 数美设备指纹 SDK uniapp 离线插件接入说明文档

## 1 插件本地配置

将原生插件配置到 uni-app 项目的 nativeplugins 目录下，还需要在 manifest.json 文件的 “App原生插件配置” 项下点击 “选择本地插件”，在列表中选择需要打包生效的插件

![image-20220216095546654](./res/uniapp-img.png)

**注意：保存后，需要提交云端打包，（制作自定义基座也属于云端打包）后插件才会生效**



## 2 API 说明

### 2.1 平台支持

Android 最低支持 API LEVEL 16

iOS 最低支持 9.0

### 2.2 SmAntiFraud.create

初始化方法，调用此方法会执行信息采集操作，调用注意事项

- 程序首次启动时，需要保证此方法在用户同意隐私政策后再调用
- 同意隐私政策后，后续启动时，在启动时调用一次初始化方法，**避免**多次调用

### 2.3 SmAntiFraud.getDeviceId

获取设备标识，调用注意事项

- 要在 create 后调用此方法，否则获取数据不可靠，可能是异常信息

- getDeviceId 会检查 smsdk 内部是否有缓存 id，如果有会直接获取缓存数据，所以应用程序不需要缓存调用 getDeviceId 的结果，直接调用 getDeviceId 方法即可

- getDeviceId 会返回两种字符串

  - boxId: 首字母为 `B` 的字符串，长度为 89 位

  - boxData: 首字母为 `D` 的字符串，此数据为采集设备数据的加密数据，长度超过了 5K，需要应用处理字符串超长逻辑，避免字长限制导致截断现象

### 2.4 SmAntiFraud.getSdkVersion

获取原生 smsdk 版本号

## 3 接入示例

需要将示例代码中占位字符（`YOUR-xxx`）全部替换，如果没有相关参数，请删除对应键值。比如：`url: 'YOUR-URL'` 代表替换 smsdk 内部请求地址，标准接入不需要此值，因此可以删除 `url: 'YOUR-URL'` 代码。

页面路径：`project/pages/index/index.vue`

```vue
<template>
	<view>
		<view> SDK version: {{ sdkVer }}</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="org" v-model="org" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="appId" v-model="appId" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="url" v-model="url" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="confUrl" v-model="confUrl" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="publicKey" maxlength="-1" v-model="publicKey" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="notCollect(数组，用空格隔开非采集字段)" v-model="notCollect" />
		</view>
		<view class="uni-input-wrapper">
			<input class="uni-input" placeholder="area" v-model="area" />
		</view>

		<view class="uni-flex uni-row">
			<view class="flex-item">
				<button type="primary" plain="true" @click="create">Create</button>
			</view>
			<view class="flex-item">
				<button type="primary" plain="true" @click="getDeviceId">GetDeviceId</button>
			</view>
		</view>

		<view class="uni-padding-wrap uni-common-mt">
			<view class="text-box" scroll-y="true" scroll-x="false">
				<text>{{logInfo}}</text>
			</view>
		</view>

	</view>
</template>
<script>
	// #ifdef APP-PLUS
	const SmAntiFraud = uni.requireNativePlugin('SmUniPlugin-AntiFruadModule');
	// #endif

	export default {
		data() {
			return {
				sdkVer: this.getSdkVersion(), // 当前原生 smsdk 版本
				org: 'YOUR-ORGANIZATION', // 必填，组织标识，邮件中 organization 项
				appId: 'YOUR-APPID', //必填，应用标识，登录数美后台应用管理查看，没有合适值，可以写 default
				publicKey: this.getPublicKey(), // 必填，加密 KEY，邮件中 *_public_key 附件内容，Android 与 iOS public_key 不一致，需要隔离配置
				area: '', // 选填，选择地区，默认支持 bj, xjp, fjny 三个选项
				url: 'YOUR-URL', // 选填，选择机房，代理模式与私有化模式使用，私有化模式需要同时设置 area 属性，值为 YOUR-ORGANIZATION
				confUrl: 'YOUR-CONF-URL', // 选填，选择机房，代理模式与私有化模式使用
				notCollect: '', // 选填，不采集项，可根据不采集文档配置 smsdk 不采集某些信息，以空格隔开多个选项，比如 ”imei imsi“ 代表不采集 imei 和 imsi 信息
				logInfo: '' // 页面调试信息

			}
		},
		methods: {
			create: function(e) {
				// #ifdef APP-PLUS
				SmAntiFraud.create({
						'org': this.org,
						'appid': this.appId,
						'publicKey': this.publicKey,
						'url': this.url,
						'notCollect': this.notCollect,
						'confUrl': this.confUrl,
						'area': this.area
					},
					(ret) => {
						this.logInfo = this.logInfo + "\n" + JSON.stringify(ret);
					});
				this.logInfo = 'create'
				// #endif

				// #ifndef APP-PLUS
				this.logInfo = "platform error: only support Android and iOS";
				// #endif
			},
			getDeviceId: function(e) {
				// #ifdef APP-PLUS
				this.logInfo = this.logInfo + "\n" + SmAntiFraud.getDeviceId();
				// #endif

				// #ifndef APP-PLUS
				this.logInfo = this.logInfo + "\nplatform error: only support Android and iOS";
				// #endif
			},
			getPublicKey: function(e) {
				// #ifdef APP-PLUS
				return this.publicKey;
				// #endif

				// #ifndef APP-PLUS
				return "";
				// #endif
			},
			getSdkVersion: function(e) {
				// #ifdef APP-PLUS
				var platform = uni.getSystemInfoSync().platform;
				return SmAntiFraud.getSdkVersion();
				// #endif

				// #ifndef APP-PLUS
				return "error";
				// #endif
			}
		}
	}
</script>

<style>
</style>
```
