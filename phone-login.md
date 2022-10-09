# 号码认证H5 一键登录接入文档

- [号码认证H5 一键登录接入文档](#号码认证h5-一键登录接入文档)
  - [1、准备](#1准备)
  - [2、引入](#2引入)
  - [3、SDK接入说明](#3sdk接入说明)
    - [3.1、初始化实例](#31初始化实例)
    - [3.2、获取Token](#32获取token)
    - [3.3、获取网络类型](#33获取网络类型)
    - [3.4、结束获取token](#34结束获取token)
    - [3.5、弹窗版自定义配置项(authPageType=2)](#35弹窗版自定义配置项authpagetype2)
      - [3.5.1、无蒙层](#351无蒙层)
      - [3.5.2、有蒙层](#352有蒙层)
      - [3.5.3、字段说明](#353字段说明)
    - [3.6、页面版自定义配置项(authPageType=3)](#36页面版自定义配置项authpagetype3)
      - [3.6.1、使用方式](#361使用方式)
      - [3.6.2、字段说明](#362字段说明)
      - [3.6.3、SDK字段调用说明](#363sdk字段调用说明)
  - [4、获取手机号码](#4获取手机号码)
  - [5、常见问题](#5常见问题)
  - [6、错误码](#6错误码)
    - [6.1、移动取号](#61移动取号)
    - [6.2、电信取号](#62电信取号)
    - [6.3、联通取号](#63联通取号)
    - [6.4、获取token](#64获取token)
    - [6.5、服务端token校验](#65服务端token校验)

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

- 直接下载源码，静态资源引入，[源码地址](./unvs-h5-sdk.min.js)

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

如果使用弹窗版本（authPageType值为1/2的情况）需要引入css文件

```
<link type="text/css" href="http://xxx/unvs-h5-sdk.css">
```

<link rel="stylesheet" type="text/css" href="xxx/css/ydrz-layer.css">


<a id="usage"></a>
## 3、SDK接入说明

<a id="usage51"></a>
### 3.1、初始化实例

```js
// 静态资源方式引入
const { LibInfo, PhoneNumberLogin } = window.UNVS_H5_SDK

// npm方式引入
const { PhoneNumberLogin } from "@ucloud-sdks/unvs-h5-sdk"

// 初始化实例
const phoneNumberServer = new PhoneNumberLogin({
    ApplicationId: "ApplicationId",
    PackageName: "One of originUrl",
})
```

| 参数名称          | 必选  | 类型     | 说明         |
| ------------- | --- | ------ | ------------------------------------------- |
| ApplicationId | 是   | string | 在UCloud控制台申请的H5类型的[应用ID](https://console.ucloud.cn/unvs/application) |
| PackageName   | 是   | string | 在UCloud控制台申请的H5类型的[源地址的其中一个](https://console.ucloud.cn/unvs/application)


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
    authPageType: "0"
    traceId: LibInfo.randomString()
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
| authPageType  | 是   | string | "0" 标准页面(默认值)<br/>"1" 时展示弹窗<br/>"2" 时展示自定义弹窗版<br/>"3" 时展示自定义页面版<br/>若为其它值则展示页面 |
| traceId       | 是   | string | 业务方生成唯一标识，字母数字组成32位随机，如HNNN2E3TH3S77LFUFUFTTP5C9J77GKER数                                                                     |
| expandParams  | 否   | string | 扩展参数 格式：参数名=值 多个时使用 \| 分割                                                      |                                        |
| success       | 否   | Function | 成功的回调                                          |
| error         | 否   | Function | 失败的回调                                          |
| layerCallback | 否   | Function | authPageType等于2时可以通过该回调方法监听，用户输入中间四位号码并勾选协议后触发 |


**成功回调**

1）authPageType不为2

```json
{
    "code": "103000",
    "message": "token获取成功",
    "token": "",
    "userInformation": "浏览器指纹"
}
```

2）authPageType=2（无蒙层）时，phoneNumberServer.getTokenInfo方法在弹窗渲染完成会回调弹窗加载成功

```json
{ 
    "code": "103000",
    "msgId": "HNNN2E3TH3S77LFUFUFTTP5C9J77GKER",
    "message": ""
}
```

**失败回调**

- 取号失败

```json
{
    "msgId": "",
    "YDData": {
        "code": -1,
        "message": "移动取号失败描述, 详见表格"
    },
    "CTData": {
        "code": -1,
        "message": "电信取号失败描述，详见表格"
    },
    "CUData": {
        "code": -1,
        "message": "联通取号失败描述，详见表格"
    }
}
```

- 获取token失败

```json
{
    "msgId": "", 
    "code": "错误码",
    "message": "错误描述，详见表格"
}
```

- ayerCallback回调

```json
{
    "code": "103000",
    "msgId": "HNNN2E3TH3S77LFUFUFTTP5C9J77GKER",
    "message":"用户已输入中间四位号码并勾选协议"
}

```
<a id="usage53"></a>
### 3.3、获取网络类型

**注：此方法为可选，可以获取也可不获取。iOS系统下由于兼容问题，netType基本返回结果为unknown 未知。**

定义connection变量通过phoneNumberServer.getConnection方法获取当前手机网络状态

示例：

```
const connection = phoneNumberServer.getConnection();
```
返回对象：
```
{
    appid:"appid",
    msgid:"唯一标识",
    netType:（cellular 数据流量、unknown 未知、wifi）3种状态
}
```
可根据netType状态自行选择后续流程。

<a id="usage54"></a>
### 3.4、结束获取token

**注：此方法当业务侧希望停止继续获取token时使用。调用此方法后，getTokenInfo返回503：获取token结束。**

```
phoneNumberServer.endGetToken();
```

### 3.5、弹窗版自定义配置项(authPageType=2)
#### 3.5.1、无蒙层

1）在您HTML代码需要引入弹窗的地方引入div；

```html
<html>
  <head><!-- 您的代码 --></head>
  <body>
    <!-- 您的代码 -->
    <div id="ydrzCustomControls"></div>
    <!-- 您的代码 -->
  </body>
</html>

```

2）在调用phoneNumberServer.getTokenInfo方法之前先初始化配置项；

```js
var Options = {
  layerStyle: {
    width:"",
    height:"240px",
    bgColor:"#236",
    borderRadius:"23px"
  },
  phoneStyle:{
    fontSize:"",
    fontColor:"#F829FF",
    high:"",
    left:"20px",
  },
  agreeStyle: {
    fontSize:"",
    textalign:"",
    fontColor:"",
    hrefColor:"",
    high:"",
    left:"",
    agreeArr:[
      {name:"《号码认证服务协议》",url:"协议链接"}
    ],
  },
  closeBtnStyle:{
    ifShowBtn:true,
    btnImage:"",
    top:"",
    right:"",
    width:"",
    height:""
  },
  customControlStyle:{
    ifShow:true,
    width:"",
    height:"24px",
    high:"center",
    left:"center",
    bgColor:"#fff",
    border:"0",
    borderRadius:"",
    url:"控件链接",
    name:"其他登录方式",
    fontSize:"16px",
    fontColor:"#392211",
    textAlign:"center",
    textDecoration:''
  }
}

phoneNumberServer.customControlsInit("ydrzCustomControls",options);

```
3）	调用phoneNumberServer.getTokenInfo方法取号，详见[3.2、获取Token](#32获取token)

4）	在用户完成输入之后调用phoneNumberServer.authGetTokenByLayer方法获取token

```js
phoneNumberServer.authGetTokenByLayer(function(res) { 
   // 成功回调 
}, function(res) {
   // 错误回调 
})
```

#### 3.5.2、有蒙层

1）在调用phoneNumberServer.getTokenInfo方法之前先初始化配置项；

```js
var Options = {
  layerStyle: {
    width:"",
    height:"240px",
    bgColor:"#236",
    borderRadius:"23px"
  },
  maskStyle: {
    ifShowMask:true,
    bgColor:"",
    opacity:""
  },
  phoneStyle:{
    fontSize:"",
    fontColor:"#F829FF",
    high:"",
    left:"20px",
  },
  agreeStyle: {
    fontSize:"",
    textalign:"",
    fontColor:"",
    hrefColor:"",
    high:"",
    left:"",
    agreeArr:[
      {name:"《号码认证服务协议》",url:"协议链接"}
    ],
  },
  closeBtnStyle:{
    ifShowBtn:true,
    btnImage:"",
    top:"",
    right:"",
    width:"",
    height:""
  },
  customControlStyle:{
    ifShow:true,
    width:"",
    height:"24px",
    high:"center",
    left:"center",
    bgColor:"#fff",
    border:"0",
    borderRadius:"",
    url:"控件链接",
    name:"其他登录方式",
    fontSize:"16px",
    fontColor:"#392211",
    textAlign:"center",
    textDecoration:''
  },
}
phoneNumberServer.customControlsInit("ydrzCustomControls",options);

```
#### 3.5.3、字段说明

<table>
   <tr>
      <td>配置项</td>
      <td>字段</td>
      <td>字段含义</td>
      <td>值</td>
      <td>说明</td>
   </tr>
   <tr>
      <td rowSpan="4">layerStyle</td>
      <td>width</td>
      <td>弹窗宽度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>弹窗高度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>弹窗背景颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>borderRadius</td>
      <td>弹窗圆角</td>
      <td>支持百分比或者数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="3">maskStyle</td>
      <td>ifShowMask</td>
      <td>是否显示蒙层</td>
      <td>Boolean值，默认值为FALSE</td>
      <td>如果要展示蒙层，则必选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>蒙层颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>opacity</td>
      <td>透明度</td>
      <td>从 0.0 （完全透明）到 1.0（完全不透明）</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="4">phoneStyle</td>
      <td>fontSize</td>
      <td>字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>距离弹窗左边框边距</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="7">agreeStyle</td>
      <td>fontSize</td>
      <td>字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textalign</td>
      <td>文本对齐选项</td>
      <td>“center/left/right”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>hrefColor</td>
      <td>协议链接颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>协议文案距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>协议文案距离弹窗左边框边距</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>agreeArr</td>
      <td>自定义协议数组</td>
      <td>如：[{name:"协议名称",url:"协议链接"}]</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="6">closeBtnStyle</td>
      <td>ifShowBtn</td>
      <td>是否展示关闭按钮</td>
      <td>Boolean值，默认值为TRUE</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>btnImage</td>
      <td>关闭按钮图片</td>
      <td>Url链接</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>top</td>
      <td>关闭按钮距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“12px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>right</td>
      <td>关闭按钮距离弹窗左边框边距</td>
      <td>支持百分比或者数值，如“12px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>关闭按钮宽度</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>关闭按钮高度</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="14">customControlStyle</td>
      <td>ifShow</td>
      <td>是否展示自定义控件</td>
      <td>Boolean值，默认值为FALSE</td>
      <td>如要展示自定义控件，则为必选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>自定义控件宽度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>自定义控件高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>自定义控件距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>自定义控件距离弹窗左边框边距</td>
      <td>支持百分比或者数值，如“20px</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>自定义控件背景颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>border</td>
      <td>自定义控件边框</td>
      <td>可以设置边框粗细，样式，颜色，如”1px solid #FFFFF”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>borderRadius</td>
      <td>自定义控件圆角</td>
      <td>支持百分比或者数值，如“20px”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>url</td>
      <td>自定义控件跳转URL</td>
      <td>Url链接</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>name</td>
      <td>自定义控件显示文案</td>
      <td>字符串</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>自定义控件字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>自定义控件字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>textAlign</td>
      <td>自定义控件文本对齐选项</td>
      <td>“center/left/right”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>textDecoration</td>
      <td>自定义控件下划线</td>
      <td>“none/underline”</td>
      <td>必选</td>
   </tr>
   <tr>
    <td rowSpan="10">submitBtnStyle（仅限level=1）</td>
      <td>name</td>
      <td>确认按钮显示文案</td>
      <td>字符串</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>确认按钮字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>确认按钮字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textAlign</td>
      <td>确认按钮文本对齐选项</td>
      <td>“center/left/right”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>确认按钮背景颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>确认按钮宽度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>确认按钮高度</td>
      <td>支持者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>borderRadius</td>
      <td>确认按钮圆角</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>确认按钮距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>确认按钮距离弹窗左边框高度</td>
      <td>支持百分比或者数值，如“200px”</td>
      <td>可选</td>
   </tr>
</table>


2）调用phoneNumberServer.getTokenInfo方法取号，详见[3.2、获取Token](#32获取token)

### 3.6、页面版自定义配置项(authPageType=3)

#### 3.6.1、使用方式

1、在调用phoneNumberServer.getTokenInfo方法之前先通过phoneNumberServer.authPageInit
方法初始化配置项；


```js
var Options = {
    "bgColor": "#FFFFFF",
    "titleStyle": {
        "name": "本机号码登录",
        "fontFamily": "PingFangSC-Medium, PingFang SC",
        "fontSize": "1.33rem",
        "fontColor": "#444444",
        "width": "70%",
        "height": "1.83rem",
        "left": "center",
        "high": "1rem",
        "textAlign": "center"
    },
    "logoStyle": {
        "url": "https://xxx/h5/js/jssdk_auth/image/logo.png",
        "width": "6.96rem",
        "height": "7.32rem",
        "high": "7.9rem",
        "left": "center"
    },
    "authTextStyle": {
        "fontFamily": "PingFangSC-Medium, PingFang SC",
        "fontSize": "1.08rem",
        "fontColor": "#444444",
        "appNameColor": "#444444",
        "width": "100%",
        "textAlign": "center",
        "high": "22.75rem",
        "left": "center",
        "fontWeight": "500"
    },
    "phoneNumStyle": {
        "fontFamily": "PingFangSC-Semibold, PingFang SC",
        "fontSize": "2.08rem",
        "fontColor": "#444444",
        "bgColor": "#FFFFFF",
        "fontWeight": "600",
        "width": "15.42rem",
        "left": "center",
        "high": "19.58rem",
        "inputStyle": {
            "width": "1.83rem",
            "height": "2.17rem"
        }
    },
    "agreeStyle": {
        "fontFamily": "PingFangSC-Regular, PingFang SC",
        "fontSize": "1rem",
        "fontColor": "#999999",
        "high": "30.58rem",
        "left": "center",
        "checkedButton": {
            "width": "1.33rem",
            "height": "1.33rem",
            "uncheckColor": "#cccccc",
            "checkedColor": "#1E82EB",
            "uncheckUrl": "",
            "checkedUrl": ""
        },
        "hrefStyle": {
            "fontColor": "#1E82EB",
            "agreeArr": []
        }
    },
    "tipStyle": {
        "fontFamily": "PingFangSC-Regular, PingFang SC",
        "fontSize": "0.92rem",
        "fontColor": "#999999",
        "high": "27rem",
        "left": "center"
    },
    "returnBtnStyle": {
        "width": "0.65rem",
        "height": "1.1rem",
        "left": "1rem",
        "high": "1rem",
        "url": "https://xxx/returnIcon.png"
    },
    "customControlStyle": {
        "ifShow": "ture",
        "width": "120px",
        "height": "24px",
        "high": "450px",
        "left": "center",
        "bgColor": "#fff",
        "border": "0",
        "borderRadius": "",
        "url": "https://www.ucloud.com",
        "name": "其他登录方式",
        "fontSize": "16px",
        "fontColor": "#392211",
        "textAlign": "center",
        "textDecoration": ""
    }
}

phoneNumberServer.authPageInit(options);

```

#### 3.6.2、字段说明

<table>
   <tr>
      <td>说明</td>
      <td>配置项</td>
      <td>字段</td>
      <td>字段含义</td>
      <td>值</td>
      <td>说明</td>
   </tr>
   <tr>
      <td>背景</td>
      <td>bgColor</td>
      <td></td>
      <td>背景颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="9">页面标题</td>
      <td rowSpan="9">titleStyle</td>
      <td>name</td>
      <td>页面标题的文案，默认“本机号码登录”</td>
      <td>string</td>
      <td>可选，标题支持为空，为空传“”</td>
   </tr>
   <tr>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>文案距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>文案距离页面左边框距离</td>
      <td>支持数值；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>标题宽度</td>
      <td>支持数值</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>标题高度</td>
      <td>支持数值；</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textAlign</td>
      <td>标题文案位置</td>
      <td>left/right/center</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="3">Logo</td>
      <td rowSpan="3">logoStyle</td>
      <td>url</td>
      <td>logo链接</td>
      <td></td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>logo距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>logo距离页面左边框距离</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="6">授权栏</td>
      <td rowSpan="6">authTextStyle</td>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>appNameColor</td>
      <td>APP名称文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>文案距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>文案距离页面左边框距离</td>
      <td>支持数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="8">号码栏</td>
      <td rowSpan="8">phoneNumStyle</td>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>配置号码栏的底色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>文案距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>文案距离页面左边框距离</td>
      <td>支持数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>配置号码栏的宽度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>inputStyle</td>
      <td>号码栏输入框配置</td>
      <td>Object，详细见下表</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="5">提示栏</td>
      <td rowSpan="5">tipStyle</td>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>文案距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>文案距离页面左边框距离</td>
      <td>支持数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="7">协议栏</td>
      <td rowSpan="7">agreeStyle</td>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>checkedButton</td>
      <td>协议勾选按钮配置项</td>
      <td>Object，详细见下表</td>
      <td></td>
   </tr>
   <tr>
      <td>hrefStyle</td>
      <td>协议名称配置项</td>
      <td>Object，详细见下表</td>
      <td></td>
   </tr>
   <tr>
      <td>left</td>
      <td>文案距离页面左边框距离</td>
      <td>支持数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>文案距离页面上边框高度</td>
      <td>支持数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="15">自定义按钮控件</td>
      <td rowSpan="15">customControlStyle</td>
      <td>ifShow</td>
      <td>是否展示自定义控件</td>
      <td>Boolean值，默认值为FALSE</td>
      <td>如要展示自定义控件，则为必选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>自定义控件宽度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>自定义控件高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>自定义控件距离弹窗上边框高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>自定义控件距离弹窗左边框边距</td>
      <td>支持百分比或者数值，如“20px”；居中可传入“center”</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>自定义控件背景颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>border</td>
      <td>自定义控件边框</td>
      <td>可以设置边框粗细，样式，颜色，如”1px solid #FFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>borderRadius</td>
      <td>自定义控件圆角</td>
      <td>支持百分比或者数值，如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>url</td>
      <td>自定义控件跳转URL</td>
      <td>Url链接</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>name</td>
      <td>自定义控件显示文案</td>
      <td>字符串</td>
      <td>必选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>自定义控件字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>自定义控件字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textAlign</td>
      <td>自定义控件文本对齐选项</td>
      <td>“center/left/right”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textDecoration</td>
      <td>自定义控件下划线</td>
      <td>“none/underline”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="5">返回键</td>
      <td rowSpan="5">returnBtnStyle</td>
      <td>width</td>
      <td>返回键图片宽度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>返回键图片高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>url</td>
      <td>配置返回键图片地址</td>
      <td>url</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>控件距离弹窗上边框高度</td>
      <td>数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>控件距离弹窗左边框边距</td>
      <td>数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td rowSpan="11">登录按钮（仅限level=1）</td>
      <td rowSpan="11">submitBtnStyle</td>
      <td>name</td>
      <td>按钮文案</td>
      <td>String</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>按钮文案字体大小</td>
      <td>支持数值，比如“20px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>按钮文案字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontFamily</td>
      <td>按钮文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>bgColor</td>
      <td>按钮的底色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>high</td>
      <td>控件距离弹窗上边框高度</td>
      <td>数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>left</td>
      <td>控件距离弹窗左边框边距</td>
      <td>数值，如“20px”；居中可传入“center”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>textAlign</td>
      <td>按钮文案位置</td>
      <td>left/right/center</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>borderRadius</td>
      <td>按钮圆角</td>
      <td>数值，如“20px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>width</td>
      <td>按钮宽度</td>
      <td>数值，如“20px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>按钮高度</td>
      <td>数值，如“20px</td>
      <td>可选</td>
   </tr>
</table>

号码栏输入框配置项如下：

<table>
   <tr>
      <td></td>
      <td>配置项</td>
      <td>字段</td>
      <td>字段含义</td>
      <td>值</td>
      <td>说明</td>
   </tr>
   <tr>
      <td rowSpan="2">号码栏输入框</td>
      <td rowSpan="2">inputStyle</td>
      <td>width</td>
      <td>宽度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
</table>

协议勾选按钮配置项如下：

<table>
   <tr>
      <td></td>
      <td>配置项</td>
      <td>字段</td>
      <td>字段含义</td>
      <td>值</td>
      <td>说明</td>
   </tr>
   <tr>
      <td rowSpan="3">协议勾选按钮</td>
      <td rowSpan="3">checkedButton</td>
      <td>width</td>
      <td>宽度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>height</td>
      <td>高度</td>
      <td>支持百分比或者数值，如“200px</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>url</td>
      <td>图片地址</td>
      <td>url</td>
      <td>可选</td>
   </tr>
</table>

协议名称配置项如下：

<table>
   <tr>
      <td></td>
      <td>配置项</td>
      <td>字段</td>
      <td>字段含义</td>
      <td>值</td>
      <td>说明</td>
   </tr>
   <tr>
      <td rowSpan="4">协议名称</td>
      <td rowSpan="4">hrefStyle</td>
      <td>fontFamily</td>
      <td>文案的字体</td>
      <td>string</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontSize</td>
      <td>文案的字体大小</td>
      <td>支持数值，如“200px”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>fontColor</td>
      <td>文案的字体颜色</td>
      <td>十六进制颜色码，如“#FFFFFF”</td>
      <td>可选</td>
   </tr>
   <tr>
      <td>agreeArr</td>
      <td>自定义协议数组</td>
      <td>如：[{name:"协议名称",url:"协议链接"}]</td>
      <td>可选</td>
   </tr>
</table>

数值支持传入rem，根元素`<html>`的fontsize值为12px；

调用phoneNumberServer.getTokenInfo方法取号，authPageType值为“3”，详见[3.2、获取Token](#32获取token)


#### 3.6.3、SDK字段调用说明

![SDK字段说明](https://github.com/umcloud-org/static/blob/master/unvs/sdk-api-detail.jpg?raw=true)

## 4、获取手机号码

调用接口[GetMobile](https://docs.ucloud.cn/api/unvs-api/get_mobile)获取用户手机号

为安全起见（保证用户PublicKey等参数不泄露），需服务端封装该接口，请求如下必要的参数：
- 1、Token
- 2、userInformation

这些参数上一步成功后可以获取，然后调用自己封装的接口校验结果

## 5、常见问题

1. 是否支持wifi环境下使用？
- 仅支持数据流量下使用，不支持wifi、热点环境下使用。请提醒用户请勿在热点环境下使用。

2. 授权页输入错误号码是否有限制次数？
- 同一号码连续输入3次错误，将会被锁定。为避免此问题，请开发者注意在联调过程中请勿连续输错3次。

3. jssdk和token校验接口中的sign是否相同？
- 不相同。两者的生成工具也不同，生成jssdk中的sign请使用附录1中MD5签名工具类；生成token校验接口的sign请使用附录2中“SHA256withRSA签名方法”。

4. 返回“referer校验失败”是什么原因？
- 请检查使用的referer与报备的集成页面地址不完全一致，请使用报备的referer或联系移动认证运营同事更换报备地址

5. token校验接口返回“验证签名失败”是什么原因？
- ①使用的公钥与报备内容是否一致；
- ②未使用附录2提供的工具类生成签名。

6. token校验接口返回“userinfomation 校验失败”是什么原因？
- 请检查token校验接口使用的userinfomation与jssdk返回是否完全一致。由于token校验接口的userinfomation 是业务方前端传给业务方后端的，可能由于该字段作为 url parameter，服务端收到后可能会解码导致出错。请业务方收到userinfomation不要解码（案例：jssdk返回的userinfomation中部分“%3d”，经服务端解码后变成了“=”）。

7. token校验接口返回“token不存在”是什么原因？
- 可能为token失效，token有效期2分钟。

8. token校验接口返回“inner error”是什么原因？
- ①请检查签名是否正确，若直接使用curl签名填写，会导致报inner error；
- ②接口 post json（-H 'Content-Type: application/json'（否则也会报inner error）），header是http的请求头。若确认无误，请联系UCloud认证侧协助排查。

## 6、错误码

### 6.1、移动取号

| 返回码    | 描述语              |
| ------ | ---------------- |
| 103000 | 成功               |
| 500    | 网络异常，请检查网络设置     |
| 503    | 获取token结束        |
| 130010 | 参数为空             |
| 105002 | 移动网关取号失败         |
| 105112 | 时间戳非法            |
| 105113 | APPID非法或为空       |
| 103101 | 错误的请求签名          |
| 103134 | WAP取号为空          |
| 110024 | businessType配置错误 |
| 110023 | 应用没有权益           |
| 110025 | 权益已失效            |
| 110026 | Origin校验失败       |
| 103111 | WAP网关IP错误        |
| 111002 | 黑名单号码用户          |
| 103609 | 物联网IP不允许取号       |
| 105002 | 移动网关取号失败         |
| 170001 | referer校验失败      |
| 103211 | 其他错误             |

### 6.2、电信取号

| 返回码    | 描述语                                |
| ------ | ---------------------------------- |
| 103000 | 成功                                 |
| 301    | 参数错误                               |
| 500    | 网络异常，请检查网络设置                       |
| 502    | 电信/联通取号能力关闭                        |
| 503    | 参数缺失                               |
| 103101 | 错误的请求签名                            |
| 105113 | APPID非法或为空                         |
| 301    | 参数错误                               |
| 130030 | 参数为空                               |
| 105117 | TRACEID为空                          |
| 108001 | businessType is null（电信预取号接口）      |
| 104001 | businessType config error          |
| 110025 | 权益已失效                              |
| 105113 | APPID非法或为空                         |
| 108001 | businessType is null（电信回调接口）       |
| 105003 | 电信网关取号失败                           |
| 110023 | 应用没有权益                             |
| 170001 | referer校验失败                        |
| 110026 | Origin校验失败                         |
| 130035 | 其他错误                               |
| 0      | 处理结果正常                             |
| -10000 | 取号异常                               |
| -10001 | 取号失败                               |
| -10002 | 参数错误                               |
| -10003 | 解密失败                               |
| -10004 | 无效的IP                              |
| -10005 | 异网授权回调参数异常                         |
| -10006 | 授权失败，且属于电信网络                       |
| -10007 | 重定向到异网取号                           |
| -10008 | 超过预设取号阀值                           |
| -10009 | 时间超期                               |
| -10010 | 号码识别异常                             |
| -10011 | 运营商不匹配                             |
| -10012 | 区域不匹配                              |
| -10013 | 业务类型不支持该运营商                        |
| -10014 | AES解密失败                            |
| -10015 | Ipv6取号失败                           |
| -10016 | 安全校验失败                             |
| -10017 | redirect方式需要https的callback地址       |
| -20005 | 签名非法                               |
| -20006 | 应用不存在                              |
| -20007 | 公钥数据不存在                            |
| -20100 | 内部解析错误                             |
| -20102 | 加密参数解析失败                           |
| -30001 | 时间戳非法                              |
| -30003 | topClass失效                         |
| -99999 | 服务内部错误                             |
| 51002  | 参数为空                               |
| 51114  | 无法获取手机号数据                          |
| 51207  | 获取accessCode使用的appid与本次操作的appid不一致 |
| 51208  | 无效的accessCode,该accessCode无法在该业务中使用 |

### 6.3、联通取号

| 返回码    | 描述语                           |
| ------ | ----------------------------- |
| 103000 | 成功                            |
| 500    | 网络异常，请检查网络设置                  |
| 502    | 电信/联通取号能力关闭                   |
| 503    | 参数缺失                          |
| 103101 | 错误的请求签名                       |
| 105113 | APPID非法或为空                    |
| 301    | 参数错误                          |
| 130030 | 参数为空                          |
| 105117 | TRACEID为空                     |
| 108001 | businessType is null（电信预取号接口） |
| 104001 | businessType config error     |
| 110025 | 权益已失效                         |
| 105001 | 联通网关取号失败                      |
| 105113 | APPID非法或为空                    |
| 108001 | businessType is null（电信回调接口）  |
| 105003 | 联通网关取号失败                      |
| 110023 | 应用没有权益                        |
| 110025 | 权益已失效                         |
| 170001 | referer校验失败                   |
| 110026 | Origin校验失败                    |
| 130035 | 其他错误                          |

### 6.4、获取token

| 返回码    | 描述语              |
| ------ | ---------------- |
| 501    | 用户取消授权           |
| 507    | 用户未补齐4位号码        |
| 508    | 用户未勾选协议          |
| 103002 | 没有填写必传参数         |
| 104000 | app不存在           |
| 104001 | businessType校验失败 |
| 104003 | 应用没有权益           |
| 104004 | 权益已失效            |
| 104005 | referer校验失败      |
| 104006 | origin校验失败       |
| 104007 | accessToken不存在   |
| 104008 | accessToken校验失败  |
| 104009 | 超过失败次数限制获取Token  |
| 104010 | 配置错误             |
| 104011 | 手机号不能为空          |
| 104012 | 本机号码校验失败         |
| 104014 | busineseTYpe校验失败 |
| 104015 | 运管配置错误           |
| 104016 | 开放配置错误           |
| 104018 | 正在处理，请稍后         |
| 104020 | 电信取号异常           |
| 104021 | 联通取号异常           |

### 6.5、服务端token校验

| 返回码    | 返回码描述                    |
| ------ | ------------------------ |
| 103000 | 成功                       |
| 103001 | fail                     |
| 103002 | 没有填写必传参数                 |
| 103003 | forbidden                |
| 103004 | resource not found       |
| 103005 | inner error              |
| 104000 | app不存在                   |
| 104001 | businessType校验失败         |
| 104003 | 应用没有权益                   |
| 104004 | 权益已失效                    |
| 104007 | token不存在(token 只有2分钟有效期) |
| 104008 | token校验失败                |
| 104010 | 配置错误                     |
| 104013 | 验证签名失败                   |
| 104014 | busineseType校验失败         |
| 104015 | 运管配置错误                   |
| 104016 | 开放平台配置错误                 |
| 104017 | IP校验失败（接口设置了ip白名单）       |
| 104019 | token校验失败[MSISDN_OWNERS] |

