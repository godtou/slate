---
title: 方便面面试-接口文档

language_tabs:
  - shell: ""

includes:
  - errors

search: true

code_clipboard: true
---

# 统一说明

* 方便面面试API设计基于REST，使用标准的HTTP协议如HTTP方法、HTTP状态码来提供接入。所有的接口数据都通过JSON格式传输（包括错误信息），并使用Unicode UTF-8进行编码。当有字段值为空的时候，若为对象则返回为null，若为数组型，返回的结果为空数组[]，其他则为对应默认值

* 为了提供更好的体验，方便面面试API在不断完善升级中。当更新中包含不向后兼容的接口时，我们会升级API版本，并保留旧的API正常运行。这意味着在接入方便面面试API时，你的代码需要能够处理一些如返回数据字段增加等向后兼容的变动。目前的API版本是v0.1。

* 所有的请求都必须通过HTTPS发送。
## 接口地址

* 生产环境：`https://open.fbmms.cn`

* 开发环境: `https://open-dev.fbmms.cn`

## 返回说明

> 返回样例

```json
{
    "code": 200,
    "msg": "OK"
    "data": {
    }
}
```

* 所有接口基于REST风格，HTTP状态码返回，200 代表请求成功，非200代表请求异常。

* **除绑定认证/OAuth2相关接口外**, 为了业务的向后兼容性，响应body中会增加`code-业务状态码`, `msg-业务信息` 两个字段来扩展业务信息返回, 暂时这两个字段并无特殊意义，和HTTP响应状态码和状态相同。真实响应内容在`data - 字段中`

## 时间格式

当前API版本返回的时间类型共有以下几种格式：

类型 | 格式 | 例子 | 例子的含义
---- | ---- | ---- | ----
精确到年的日期 | yyyy | 2020 | 例如毕业年份
精确到月的日期 | yyyy-MM | 2020-01 | 例如毕业日期
精确到日的日期 | yyyy-MM-dd | 2020-01-01 | 例如入职日期
精确到时、分、秒的日期 | yyyy-MM-ddTHH:mm:ss.sssZ | 2018-09-27T02:00:00.000Z | 例如面试开始时间

其中，`yyyy-MM-ddTHH:mm:ss.sssZ`的格式遵循ISO8601标准.
# 绑定认证/OAuth

## 流程

> OAuth2交互流程

```
+--------+                                          +-------------+
|        |--(A)------- Authorization Grant -------->|             |
|        |                                          |             |
|        |<-(B)----------- Access Token ------------|             |
|        |               & Refresh Token            |             |
|        |                                          |             |
|        |                            +----------+  |             |
|        |--(C)---- Access Token ---->|          |  |             |
|        |                            |          |  |             |
|        |<-(D)- Protected Resource --| Resource |  |Authorization|
| Client |                            |  Server  |  |    Server   |
|        |--(E)---- Access Token ---->|          |  |             |
|        |                            |          |  |             |
|        |<-(F)- Invalid Token Error -|          |  |             |
|        |                            +----------+  |             |
|        |                                          |             |
|        |--(G)----------- Refresh Token ---------->|             |
|        |                                          |             |
|        |<-(H)----------- Access Token ------------|             |
+--------+           & Optional Refresh Token       +-------------+
```

采用OAuth2.0 进行授权, 参考<a href="https://oauth.net/2/">OAuth2官网</a>

说明：

1. 第三方供应商向方便面面试提供授权回调地址, 基本注册信息进行注册, 方便面面试向第三方供应商发放clientId与clientSecret。

3. 第三方供应商带着clientId与redirectUri将用户导向至方便面面试OAuth授权认证页面。

4. 授权验证成功后方便面面试内部绑定成功，然后带着一次性code将跳转回redirectUri。

5. 第三方供应商用code与clientSecret等请求方便面面试提供的API得到accessToken、refreshToken、appId（每次绑定产生唯一值，accessToken与方便面面试用户绑定，appId与第三方供应商用户绑定）等信息，第三方供应商绑定成功。

**补充**

第三方供应商通过refreshToken维护accessToken长时间有效，如过期，需要用户重新授权。

## 跳转OAuth授权页面
> 请求URL示例:

```shell
https://open.fbmms.cn/oauth/authorize?clientId=YOUR_CLIENT_ID&redirectUri=YOUR_URL_ENCODE_REDIRECT_URI&scope=YOUR_SCOPE&state=YOUR_STATE
```

### 请求地址：
`GET /oauth/authorize`

### 请求参数:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
clientId | 是 | String | 方便面生产的Client唯一标识
redirectUri | 是 | String | 重定向地址，必须为开通时填写的一致, 需进行URL_ENCODE处理
scope | 是| String | 第三方巩营乡所拥有的权限: 
state | 否| String | 在回调时，会回传state参数给第三方供应商， 开发者可以验证state参数的有效性

### 响应:（绑定后，浏览器页面会跳转到回调地址）
redirectUri?code=xxxx

**例：http://www.xxx.com/redirect?code=xxx**

**注：**

1. code参数会拼接到回调地址url上

2. code有效期为五分钟

## 获取access_token
> 请求示例:

```shell
curl -X POST \
  https://open.fbmms.cn/oauth/token \
  --header 'Content-Type: application/json' \
  --data '{"clientId":"YOUR_CLIENT_ID","clientSecret":"YOUR_CLIENT_SECRET","code":"RETURN_CODE", "redirectUri":"YOUR_REDIECT_URI"}'
```

> 响应示例:

```json
{
 "access_token": "de6780bc506a0446309bd9362820ba8aed28aa506c71eedbe1c5c4f9dd350e54",
 "accessTokenExpiresAt": "2019-03-24T05:45:39.014Z" ,
 "refresh_token": "8257e65c97202ed1726cf9571600918f3bffb2544b26e00a61df9897668c33a1",
 "refreshTokenExpiresAt": "2019-03-24T05:45:39.014Z" ,
 "scope": "",
 "appId": "BDMxLbTWtIKvAuCCQH2ZcCquC73rZAMKTYpf"
}

```

### 请求地址: 
`POST /oauth/token`
### 请求参数:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
code | 是 | String | 上一步获得的Code
clientId | 是 | String | 开通时提供
clientSecret| 是| String | 开通是提供 
redirectUri | 否| String | 上一步填写的redirectUri

### 响应:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
accessToken | String | 访问方便面面试接口所需的Token
accessTokenExpiresAt | String | refreshToken的过期时间 \n 如:2019-03-24T05:45:39.014Z \n 默认为30天
refreshToken | String | 用于刷新accessToken的refreshToken
refreshTokenExpiresAt | String | refreshToken的过期时间 \n 如:2019-03-24T05:45:39.014Z \n 默认为60天
scope | String | 该Token所拥有的权限
appId | String | 每次绑定方便面面试生成的唯一值,建议与第三方供应商账号绑定

## 刷新access_token
> 请求示例:

```shell
curl -X POST \
  https://open.fbmms.cn/oauth/refreshToken \
  --header 'Content-Type: application/json' \
  --data '{"clientId":"YOUR_CLIENT_ID","clientSecret":"YOUR_CLIENT_SECRET", "refreshToken":"YOUR_REFRESH_TOKEN"}'

```

> 响应示例:

```json
{
 "access_token": "de6780bc506a0446309bd9362820ba8aed28aa506c71eedbe1c5c4f9dd350e54",
 "accessTokenExpiresAt": "2019-03-24T05:45:39.014Z" ,
 "refresh_token": "8257e65c97202ed1726cf9571600918f3bffb2544b26e00a61df9897668c33a1",
 "refreshTokenExpiresAt": "2019-03-24T05:45:39.014Z" ,
 "scope": ""
}

```

### 请求地址:
`POST /oauth/refreshToken`
### 请求参数:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
refreshToken | 是 | String | 刷新accessToken的refreshToken
clientId | 是 | String | 开通时提供
clientSecret| 是| String | 开通是提供 

### 响应:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
accessToken | String | 访问方便面面试接口所需的Token
accessTokenExpiresAt | String | refreshToken的过期时间 \n 如:2019-03-24T05:45:39.014Z \n 默认为30天
refreshToken | String | 用于刷新accessToken的refreshToken
refreshTokenExpiresAt | String | refreshToken的过期时间 \n 如:2019-03-24T05:45:39.014Z \n 默认为60天
appId | String | 每次绑定moka生成的唯一值,建议与第三方供应商账号绑定

## 删除绑定
> 请求示例:

```shell
curl -X POST \
  https://open.fbmms.cn/oauth/unbind \
  --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{"clientId":"YOUR_CLIENT_ID","clientSecret":"YOUR_CLIENT_SECRET","appId":"YOUR_APP_ID"}'
```

### 请求地址:
`DELETE /oauth/unbind`
### 请求参数:

字段 | 必填 | 类型 | 描述
--- | --- | --- | ---
appId | 是 | String | 每次绑定方便面面试生成的唯一值
clientId | 是 | String | 开通时提供
clientSecret| 是| String | 开通是提供 

# 岗位流程
