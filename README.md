# HiPOS App 接口文档

基于**OAuth2**对业务接口进行调用授权，回发数据统一为**JSON**格式，业务请求数据需要使用商户的**App Secret**签名。


## 设备授权
新安装 App 端的设备需要通过平台管理员的授权获得接口API接口的关键凭证。
> 接口：/device/authorize（**提示：此接口调用时不需要OAuth2授权及签名**）

调用参数：

| 名称        | 类型        | 说明         | 示例                                |
| :---------- | :---------- | :----------- | :---------------------------------- |
| auth_code   | string      | 授权码       |  12f8b2b9a74c4591a67a44b9ba0647e9   |
| device_id   | string      | 设备唯一标识 |  f5bb0c8de146c67b44babbf4e6584cc0   |
| version     | string      | App版本      |  1.0.0.1                            |

正确返回数据：
```
{
  "merchant_id": "1",
  "app_id": "c4ca4238a0b923820dcc509a6f75849b",
  "app_secret": "c40966e8f40a455425610606561819fa2a578c32",
  "store_id": "5642e198a4826ee461311319"
}
```

## 获取Access Token
在正式调用业务接口前需要通过OAuth2获得**Access Token**，并且具有一定的有效期，再次调用业务接口前需要检查**Access Token**是否已经**过期**，否则需要重新获取。
> 接口：/token

调用参数：

HTTP请求头中将 app_id 与 app_secret使用“:”拼接后使用**Base64**编码，使用**[HTTP基本认证](https://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)**

```
Authorization: Basic YzRjYTQyMzhhMGI5MjM4MjBkY2M1MDlhNmY3NTg0OWI6YzQwOTY2ZThmNDBhNDU1NDI1NjEwNjA2NTYxODE5ZmEyYTU3OGMzMg==
```

正确返回数据：
```
{
  "token_type": "bearer",
  "access_token": "f7420dad8e471ab7df0f6b4b646f9010aea3e131",
  "expires_in": 3600
}
```
