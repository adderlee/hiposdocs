# HiPOS App 接口文档

基于**OAuth2**对业务接口进行调用授权，回发数据统一为**JSON**格式，业务请求数据需要使用商户的**App Secret**签名。

* [`基本API接口`](#基本API接口)
* [`API接口签名规范`](#API接口签名规范)
* [`业务API接口`](#业务API接口)
* [`数据API接口`](#数据API接口)

<a name="基本API接口" />
## 基本API接口
调用业务接口前需要获得设备授权及AccessToken，本组接口向App提供这种能力。

### 设备授权
新安装 App 端的设备需要通过平台管理员的授权获得接口API接口的关键凭证。
> 接口：/device/authorize（**提示：此接口调用时不需要OAuth2授权及签名**）

调用参数：

| 名称        | 类型        | 说明         | 示例                                 |
| :---------- | :---------- | :----------- | :----------------------------------- |
| auth_code   | string      | 授权码       |  12f8b2b9a74c4591a67a44b9ba0647e9    |
| device_id   | string      | 设备唯一标识 |  3826ef14abfa52ca                    |
| version     | string      | App版本      |  1.0.0.1                             |

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
<a name="API接口签名规范" />
## API接口签名规范
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

<a name="业务API接口" />
## 业务API接口
完成与商户业务相关的API接口，如交易、结算等。
>所有业务接口设调用时均需要提供 Bearer Token（即 Access Token）

### 发起收款交易
向平台发起收款交易，获得全平台唯一交易号。
> 接口：/transaction/create

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| merchant_id | string      | 商户ID        | 1                                 |
| store_id    | string      | 门店ID        | 5642e198a4826ee461311319          |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
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

### 交班结算
结算当前帐单
> 接口：/transaction/settle

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| store_id    | string      | 门店ID        | 5642e198a4826ee461311319          |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| job_id      | string      | 工号          | 1001                              |

正确返回数据：
```
{
  "result": "ok"
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

<a name="数据API接口" />
## 数据API接口
调用数据接口前需要获得设备授权及AccessToken。

### 一日交易详情
返回当日的所有收款通道（现金、微信、支付宝）的交易统计，包括交易金额和笔数。
> 接口：/data/daily

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| on          | string      | 交易日期      | 20151126                          |

正确返回数据：
```
{
    "result": {
        "items": {
            "cash": {
                "amount": 5.7,
                "trades": 10
            },
            "alipay": {
                "amount": 12.05,
                "trades": 50
            },
            "weixin": {
                "amount": 82.25,
                "trades": 40
            }
        },
        "total": {
            "amount": 100,
            "trades": 100
        }
    }
}
```

### 一周交易详情
返回当天起往前一周的的所有收款通道（现金、微信、支付宝）的交易统计，包括交易金额和笔数。
> 接口：/data/weekly

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| on          | string      | 交易日期      | 20151126                          |

正确返回数据：
```
{
    "result": {
        "amount": {
            "cash": [
                102.5,
                195.1,
                140,
                110.3,
                208,
                161,
                164
            ],
            "alipay": [
                700.5,
                565.1,
                580,
                610.3,
                804,
                761,
                828
            ],
            "weixin": [
                630.5,
                475.1,
                420,
                560.3,
                704,
                651,
                681
            ]
        },
        "trades": {
            "cash": [
                20,
                23,
                14,
                19,
                21,
                16,
                19
            ],
            "alipay": [
                43,
                56,
                58,
                61,
                80,
                76,
                82
            ],
            "weixin": [
                63,
                47,
                42,
                56,
                74,
                61,
                64
            ]
        }
    }
}
```

### 帐单列表
返回指定日期范围内的结算帐单列表，含金额与交易笔数及小计。
> 接口：/data/bill/list

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| from        | string      | 起始日期      | 20151101                          |
| to          | string      | 截止日期      | 20151130                          |

正确返回数据：
```
{
    "result": [
        {
            "on": "2015-11-23",
            "amount": 651.5,
            "trades": 143,
            "settlements": [
                {
                    "stm_id": "86000001000001",
                    "created_at": "2015-11-23 08:03:49",
                    "settled_at": "2015-11-23 11:58:30",
                    "amount": 123.5,
                    "trades": 21
                },
                {
                    "stm_id": "86000001000002",
                    "created_at": "2015-11-23 11:59:00",
                    "settled_at": "2015-11-23 15:58:30",
                    "amount": 160.9,
                    "trades": 35
                },
                {
                    "stm_id": "86000001000003",
                    "created_at": "2015-11-23 15:59:00",
                    "settled_at": "2015-11-23 22:58:30",
                    "amount": 367.1,
                    "trades": 87
                }
            ]
        },
        {
            "on": "2015-11-24",
            "amount": 651.5,
            "trades": 143,
            "settlements": [
                {
                    "stm_id": "86000001000001",
                    "created_at": "2015-11-24 08:03:49",
                    "settled_at": "2015-11-24 11:58:30",
                    "amount": 123.5,
                    "trades": 21
                },
                {
                    "stm_id": "86000001000002",
                    "created_at": "2015-11-24 11:59:00",
                    "settled_at": "2015-11-24 15:58:30",
                    "amount": 160.9,
                    "trades": 35
                },
                {
                    "stm_id": "86000001000002",
                    "created_at": "2015-11-24 15:59:00",
                    "settled_at": "2015-11-24 22:58:30",
                    "amount": 367.1,
                    "trades": 87
                }
            ]
        },
        {
            "on": "2015-11-25",
            "amount": 651.5,
            "trades": 143,
            "settlements": [
                {
                    "stm_id": "86000001000003",
                    "created_at": "2015-11-25 08:03:49",
                    "settled_at": "2015-11-25 11:58:30",
                    "amount": 123.5,
                    "trades": 21
                },
                {
                    "stm_id": "86000001000002",
                    "created_at": "2015-11-25 11:59:00",
                    "settled_at": "2015-11-25 15:58:30",
                    "amount": 160.9,
                    "trades": 35
                },
                {
                    "stm_id": "86000001000003",
                    "created_at": "2015-11-25 15:59:00",
                    "settled_at": "2015-11-25 22:58:30",
                    "amount": 367.1,
                    "trades": 87
                }
            ]
        }
    ]
}
```

### 帐单明细
返回指定结算帐单及相关交易详情信息。
> 接口：/data/bill/detail

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| stm_id      | string      | 结算帐单ID    | 86000001000003                    |

正确返回数据：
```
{
    "result": {
        "stm_id": "86000001000002",
        "created_at": "2015-11-25 15:59:00",
        "settled_at": "2015-11-25 22:58:30",
        "amount": 0.18,
        "trades": 3,
        "total": {
            "cash": {
                "amount": 0.12,
                "trades": 1
            },
            "weixin": {
                "amount": 0.01,
                "trades": 1
            },
            "alipay": {
                "amount": 0.05,
                "trades": 1
            }
        }
        "transactions": [
            {
                "tid": "8600000100000100000009",
                "paid_at": "2015-11-25 22:58:30",
                "amount": 121.5,
                "method": "weixin"
            },
            {
                "tid": "8600000100000100000010",
                "paid_at": "2015-11-25 23:01:12",
                "amount": 98.6,
                "method": "alipay"
            },
            {
                "tid": "8600000100000100000011",
                "paid_at": "2015-11-25 23:11:34",
                "amount": 147,
                "method": "cash"
            }
        ]
    }
}
```

### 当前帐单明细
返回当前正在进行中结算帐单及相关交易详情信息。
> 接口：/data/bill/current

调用参数：

| 名称        | 类型        | 说明          | 示例                              |
| :---------- | :---------- | :------------ | :-------------------------------- |
| app_id      | string      | App ID        | c4ca4238a0b923820dcc509a6f75849b  |
| store_id    | string      | 门店ID        | 5642e198a4826ee461311319          |
| device_id   | string      | 设备唯一标识  | 3826ef14abfa52ca                  |
| job_id      | string      | 工号          | 1001                              |

正确返回数据：
```
{
    "result": {
        "stm_id": "86000001000002",
        "created_at": "2015-11-25 15:59:00",
        "settled_at": "2015-11-25 22:58:30",
        "amount": 0.18,
        "trades": 3,
        "total": {
            "cash": {
                "amount": 0.12,
                "trades": 1
            },
            "weixin": {
                "amount": 0.01,
                "trades": 1
            },
            "alipay": {
                "amount": 0.05,
                "trades": 1
            }
        }
        "transactions": [
            {
                "tid": "8600000100000100000009",
                "paid_at": "2015-11-25 22:58:30",
                "amount": 121.5,
                "method": "weixin"
            },
            {
                "tid": "8600000100000100000010",
                "paid_at": "2015-11-25 23:01:12",
                "amount": 98.6,
                "method": "alipay"
            },
            {
                "tid": "8600000100000100000011",
                "paid_at": "2015-11-25 23:11:34",
                "amount": 147,
                "method": "cash"
            }
        ]
    }
}
```
