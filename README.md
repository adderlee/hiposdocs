# HiPOS App 接口文档

基于**OAuth2**对业务接口进行调用授权，回发数据统一为**JSON**格式，业务请求数据需要使用商户的**App Secret**签名。

## 基本API接口
调用业务接口前需要获得设备授权及AccessToken。

### 设备授权
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
  "merchant_name": "海商咖啡",
  "merchant_logo": "http://him.hishop.com.cn:10800/assets/images/merchants/1.jpg",
  "store_id": "5642e198a4826ee461311319",
  "store_name", "大剧院旗舰店",
  "app_id": "c4ca4238a0b923820dcc509a6f75849b",
  "app_secret": "c40966e8f40a455425610606561819fa2a578c32",
  "expires": "2016-11-11 23:59:59",
}
```

### 获取Access Token
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

## 业务API接口
完成与商户业务相关的API接口，如交易、结算等。
>所有业务接口设调用时均需要提供 Bearer Token（即 Access Token）

### 业务接口签名规范
原始业务数据如：
>a=1&b=2&z=3

加入随机时间码：
>rnd=1447386720779

按字母序形成待签名字符串：
>a=1&b=2&rnd=1447386720779&z=3

将"sign=[App密钥]"拼接到此字符串的最后面，形成：
>a=1&b=2&rnd=1447386720779&z=3&sign=c40966e8f40a455425610606561819fa2a578c32

将此字符串进行SHA1运算，得到签名：
>sign = SHA1('a=1&b=2&rnd=1447386720779&z=3&sign=c40966e8f40a455425610606561819fa2a578c32')

最后向接口POST的数据如下：
>a=1&b=2&rnd=1447386720779&z=3&sign=cf81e8e0d756db9f0e301bf0b64d8525ef3edf85

### 发起收款交易
向平台发起收款交易，获得全平台唯一交易号。
> 接口：/transaction/create

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| merchant_id | string      | 商户ID        | 1                                 |
| store_id    | string      | 门店ID        | 5642e198a4826ee461311319          |
| device_id   | string      | 设备唯一标识  | f5bb0c8de146c67b44babbf4e6584cc0  |
| job_id      | string      | 工号          | 1001                              |
| amount      | number      | 金额          | 16.8                              |

正确返回数据：
```
{
  "result": {
    "__v": 0,
    "_id": "8600000100000200000011",
    "pay_expire": "2015-11-13T04:07:04.512Z",
    "created_at": "2015-11-13T03:37:04.544Z",
    "amount": 0.01,
    "clerk_id": "5642e198a4826ee46131131a",
    "store_id": "5642e198a4826ee461311319",
    "stm_id": "86000001000002"
  }
}
```

### 确认现金收款交易
向平台确认正在进行中的现金收款交易，操作成功后，此交易即会关闭。
> 接口：/transaction/confirm/cash

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| tid         | string      | 交易号        | 8600000100001800000040            |

正确返回数据：
```
{
  "result": "ok"
}
```

### 请求微信扫码支付
生成微信扫码支付的二维条形码。
> 接口：/transaction/weixin/qr/get

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| tid         | string      | 交易号        | 8600000100001800000040            |

正确返回数据：
用于生成微信扫码付的二维码字符串。
```
{
    "result": "weixin://wxpay/bizpayurl?sign=FBA2E678AC50D9FEB6250413079FED43&appid=wx6f38b684087b31c2&mch_id=127764401&product_id=8600000100001800000092&time_stamp=1447406069&nonce_str=37sA8E"
}
```

### 微信刷卡支付
扫描用户微信钱包中刷卡条码（二维码）完成微信支付交易。
> 接口：/transaction/weixin/micropay

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| tid         | string      | 交易号        | 8600000100001800000040            |
| auth_code   | string      | 条码或二维码  | 120061098828009406                |

正确返回数据：
```
{
    "result": "ok"
}
```

### 请求支付宝扫码支付
将支付宝返回的二维码信息展示给用户，由用户扫描二维码完成订单支付。
> 接口：/transaction/alipay/qr/get

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| tid         | string      | 交易号        | 8600000100001800000040            |

正确返回数据：
用于生成支付宝扫码付的二维码字符串。
```
{
    "result": "https://qr.alipay.com/baxcwo4myomqqelrb4"
}
```

### 支付宝条码支付
收银员使用扫码设备读取用户手机支付宝“付款码”后，将二维码或条码信息通过本接口上送至支付宝发起支付。
> 接口：/transaction/alipay/barcode

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| tid         | string      | 交易号        | 8600000100001800000040            |
| auth_code   | string      | 条码或二维码  | 120061098828009406                |

正确返回数据：
```
{
    "result": "ok"
}
```
