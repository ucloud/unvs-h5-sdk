# 号码认证H5 号码认证接入文档

- [号码认证H5 号码认证接入文档](#号码认证h5-号码认证接入文档)
  - [1、准备](#1准备)
  - [2、引入](#2引入)
  - [3、SDK接入说明](#3sdk接入说明)
    - [3.1、初始化实例](#31初始化实例)
    - [3.2、获取Token](#32获取token)

<a id="ready"></a>
## 1、准备
- 确保您已开通了号码认证服务，并成功创建了对应的认证方案。详情请参见[H5本机号码校验使用流程](https://docs.ucloud.cn/unvs/README)。
- 确保您的终端设备已关闭Wi-Fi连接且开启了SIM卡的4G移动数据网络（⽀持中国联通、中国移动的3G⽹络，但接⼝耗时会增加）。


<a id="install"></a>
## 2、引入

支持一下几种安装方式

- 直接下载源码，静态资源引入，[源码地址](./unvs-h5-sdk.js)

```
<script type="text/javascript" charset="utf-8" src="xxx/unvs-h5-sdk.js"></script>
```
通过 script 标签引入该文件，会在全局生成名为 UNVS_H5_SDK 的对象

- 使用NPM引入

```
npm i @ucloud-sdks/unvs-h5-sdk
```

- 使用yarn引入

```
yarn add @ucloud-sdks/unvs-h5-sdk
```

**注意**

生产上有 https 访问的，https 请求跨域，会导致上报的 referer 为空，请在 head 中添加代码：解决此问题。

```
<meta content="always" name="referrer">
```

<a id="usage"></a>
## 3、SDK接入说明

<a id="usage51"></a>
### 3.1、初始化实例

```js
// 静态资源方式引入
const { LibInfo, PhoneNumberAuth } = window.UNVS_H5_SDK

// npm方式引入
const { PhoneNumberAuth } from "@ucloud-sdks/unvs-h5-sdk"

// 初始化实例
const phoneNumberServer = new PhoneNumberLogin({
    ApplicationId: "UCloudAppId",
    PackageName: "UCloudAppName",
})
```

| 参数名称          | 必选  | 类型     | 说明                                                                             |
| ------------- | --- | ------ | ------------------------------------------------------------------------------ |
| ApplicationId | 是   | string | 在UCloud控制台申请的AppId                                                             |
| PackageName   | 是   | string | 在UCloud控制台申请的包名   


**LibInfo 提供方法，可自行实现**

```
// 判断当前环境是否为wifi
LibInfo.isWifi()
// 判断当前环境是否为PC
LibInfo.osIsPc()
// 生成32位随机码，由字母和数字组成
LibInfo.randomString()
```

<a id="usage52"></a>
### 3.2、获取Token

```js
phoneNumberServer.getTokenInfo({
    traceId: Lib.randomString(),
    success: function(res) {
        console.log("success", res)
    },
    error: function(res) {
        console.log("error", res)
    },
    layerCallback: function(res) {
        console.log("callback", res)
    }
})
```

| 参数名称          | 必选  | 类型       | 说明                                             |
| ------------- | --- | -------- | ---------------------------------------------- |
| traceId       | 是   | string | 业务方生成唯一标识，字母数字组成32位随机，如HNNN2E3TH3S77LFUFUFTTP5C9J77GKER数                                                                     |
| expandParams  | 否   | string | 扩展参数 格式：参数名=值 多个时使用 \| 分割                                                      |                                        |
| success       | 否   | Function | 成功的回调                                          |
| error         | 否   | Function | 失败的回调                                          |
| layerCallback | 否   | Function | authPageType等于2时可以通过该回调方法监听，用户输入中间四位号码并勾选协议后触发 |

