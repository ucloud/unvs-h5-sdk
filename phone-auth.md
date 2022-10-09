# 号码认证H5 号码认证接入文档

- [号码认证H5 号码认证接入文档](#号码认证h5-号码认证接入文档)
  - [1、准备](#1准备)
  - [2、引入](#2引入)
  - [3、SDK接入说明](#3sdk接入说明)
    - [3.1、初始化实例](#31初始化实例)
    - [3.2、获取Token](#32获取token)
  - [4、手机号校验](#4手机号校验)
  - [5、运营商错误码描述](#5运营商错误码描述)

<a id="ready"></a>
## 1、准备
- 确保您已开通了号码认证服务，并成功创建了对应的认证方案。详情请参见[H5本机号码校验使用流程](https://docs.ucloud.cn/unvs/README)。
- 确保您的终端设备已关闭Wi-Fi连接且开启了SIM卡的4G移动数据网络（⽀持中国联通、中国移动的3G⽹络，但接⼝耗时会增加）。


<a id="install"></a>
## 2、引入

支持一下几种安装方式

- 引入sdk之前先引入crypto-js.js（3.x/4.x都可以，支持本地引入）

```js
// 这里举个例子
<script src="https://cdn.bootcdn.net/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
```

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

```js
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
    }
})
```

**getTokenInfo参数**

| 参数名称          | 必选  | 类型       | 说明                                             |
| ------------- | --- | -------- | ---------------------------------------------- |
| traceId       | 是   | string | 业务方生成唯一标识，字母数字组成32位随机，如HNNN2E3TH3S77LFUFUFTTP5C9J77GKER数                                                                     |
| expandParams  | 否   | string | 扩展参数 格式：参数名=值 多个时使用 \| 分割                                                      |                                        |
| success       | 否   | Function | 成功的回调                                          |
| error         | 否   | Function | 失败的回调                                          |

## 4、手机号校验

调用接口[VerifyMobile](https://docs.ucloud.cn/api/unvs-api/verify_mobile)进行校验

为安全起见（保证用户PublicKey等参数不泄露），需服务端封装该接口，请求如下必要的参数：
- 1、Token
- 2、Phone
- 3、userInformation

这些参数上一步成功后可以获取，然后调用自己封装的接口校验结果
## 5、运营商错误码描述

**移动取号：错误码及描述语**

返回码 | 描述语
---|---
130010 | 参数为空 
105112 | 时间戳非法 
 103101 | 错误的请求签名
110024 | businessType 配置错误
110025 | businessType 错误
110023 | 应用没有权益 
110025 | 权益已失效
103111 | WAP网关IP错误 
111002 | 黑名单号码用户 
103609 | 物联网 IP 不允许取号 
105002 | 移动网关取号失败 
103211 | 其他错误



**联通取号：错误码及描述语**

错误码| 错误描述语
---|---
100001 | 应用鉴权错误(clientID 或 clientSecret 错误)
200001 | 初始化失败
102 | 客户端类型错误
104 | clientId 错误
106 | 请求时间超时
107 | 鉴权信息错误
108 | 应用签名错误
110 | Referer 未报备
111 | 网络环境错误
113 | 客户端无权限
1002 | 网关错误
1003 | 预取号错误
1004 | AccessCode 错误
1011 | 数据解析错误
1012 | 网络环境错误
1013 | 网络环境错误


**电信取号:错误码及描述**

错误码 | 错误描述
---|---
130032 | 解析参数错误 
130010 | 参数为空
121016 | 电信取号功能已关闭 
130018 | 验证签名失败
110022 | 应用 ID 不存在
110023 | 应用没有权益
110025 | 权益已失效
170001 | 无效的请求 
999999 | 系统异常