# 海外钱台接口文档

**版本: 2017-02-21**

* [0. 请求方式](#0-请求方式)
	* [0.0. 名词解释](#00-名词解释)
	* [0.1. 非 OAuth 2.0 接口](#01-非-oauth-20-接口)
	* [0.2. OAuth 2.0 接口](#02-oauth-20-接口)
* [1. 子商户注册/查询接口](#1-子商户注册查询接口)
	* [1.1. /mch/v1/signup 注册接口](#11-mchv1signup-注册接口)
	* [1.2. /mch/v1/uploadcert 补件](#12-mchv1uploadcert-补件)
	* [1.3. /mch/v1/query 查询用户信息](#13-mchv1query-查询用户信息)
	* [1.4. /tool/v1/mcc mcc 列表接口](#14-toolv1mcc-mcc-列表接口)
* [2. 支付接口](#2-支付接口)
	* [2.1. /trade/v1/payment 支付接口](#21-tradev1payment-支付接口)
	* [2.2. /trade/v1/close 关闭订单](#22-tradev1close-关闭订单)
	* [2.3. /trade/v1/refund 退款](#23-tradev1refund-退款)
	* [2.4. /trade/v1/reversal 撤销/冲正](#24-tradev1reversal-撤销冲正)
	* [2.5. /trade/v1/query 查询订单信息](#25-tradev1query-查询订单信息)
	* [2.6. 预下单结果通知.](#26-预下单结果通知)
	* [2.7. 支付宝线上支付重定向参数](#27-支付宝线上支付重定向参数)
* [3. OAuth 授权接口](#3-oauth-授权接口)
	* [3.1. /oauth/v2/authorize 用户授权](#31-oauthv2authorize-用户授权)
	* [3.2. /oauth/v2/access_token 获取/刷新 access_token](#32-oauthv2access_token-获取刷新-access_token)
* [4. OAuth 查询接口](#4-oauth-查询接口)
	* [4.1. /user/v1/baseinfo 获取用户信息](#41-userv1baseinfo-获取用户信息)
	* [4.2. /user/v1/tradelist 获取订单信息](#42-userv1tradelist-获取订单信息)
* [5. 清算查询接口](#5-清算查询接口)
	* [5.1. /settlement/v1/query](#51-settlementv1query)
* [6. 附录](#6-附录)
	* [6.1 respcd 列表](#61-respcd-列表)

---

## 0. 请求方式

### 0.0. 名词解释

- appcode: 开发者唯一标示.
- appkey: 开发者密钥, 调用除 SDK 接口外的其它接口时使用, 包括退款, 冲正及 OAuth 获取 access_token 等.

### 0.1. 非 OAuth 2.0 接口

1. 根据各接口文档, 使用 GET 或 POST 方式请求.
2. 对**注册**, **补件**等需要上传文件的接口, Content-Type 需设置为 `multipart/form-data`.
3. 如无特殊说明, 所有请求均应加上以下 HTTP 头:
    - X-QF-APPCODE: 分配给开发者的 `appcode`.
    - X-QF-SIGN: 数据签名.
4. 数据签名的计算方式: 将所有请求参数 (文件除外) 按照字典序排序, 并拼装成 `a=1&b=2` 的格式, 再在最后拼接上事先分配的 `appkey`, 并计算 MD5 得到签名. 如对于请求数据:

    ```
    {
        "username": "some_user",
        "mobile": "14401234567",
        "email": "email@example.com",
        "idnumber": "110000201608220123",
        "name": "测试企业",
        "country": "中国",
        "city": "成都",
        "address": "XX区XX路XX号",
        "shopname": "XX商店",
        "bankuser": "王",
        "bankaccount": "1234567890",
        "bankcountry": "中国",
        "bankcity": "成都",
        "bankname": "中国建设银行",
        "bankcode": "123",
        "mcc_id": "6125485337528623306",
        "channel_type": "1",
        "licenseactive_date": "2016-07-10",
        "licensenumber": "12345678",
        "bankaddr": "银行详细地址",
        "bankswiftcode": "001"
    }
    ```

    按照字典序拼好后为

    ```
    address=XX区XX路XX号&bankaccount=1234567890&bankaddr=银行详细地址&bankcity=成都&bankcode=123&bankcountry=中国&bankname=中国建设银行&bankswiftcode=001&bankuser=王&channel_type=1&city=成都&country=中国&email=email@example.com&idnumber=110000201608220123&licenseactive_date=2016-07-10&licensenumber=12345678&mcc_id=6125485337528623306&mobile=14401234567&name=测试企业&shopname=XX商店&username=some_user
    ```

    假设 `appkey` 为 789012, 则待签名字符串为

    ```
    address=XX区XX路XX号&bankaccount=1234567890&bankaddr=银行详细地址&bankcity=成都&bankcode=123&bankcountry=中国&bankname=中国建设银行&bankswiftcode=001&bankuser=王&channel_type=1&city=成都&country=中国&email=email@example.com&idnumber=110000201608220123&licenseactive_date=2016-07-10&licensenumber=12345678&mcc_id=6125485337528623306&mobile=14401234567&name=测试企业&shopname=XX商店&username=some_user789012
    ```

    计算出的签名为 `E019253E135863C335333C983DF05359`.

### 0.2. OAuth 2.0 接口

1. 按照标准 OAuth 流程, 将用户重定向到 `/oauth/v2/authorize` 接口, 并需带上 `response_type`, `client_id`, `scope` 和 `redirect_uri` 等参数. 目前 `client_id` 与 `appcode` 保持一致.
2. 用户确认授权后, 会将用户再重定向到 `redirect_uri`, 并附带 `code` 参数.
3. 使用 `code` 调用 `/oauth/v2/access_token` 接口获取 `access_token`.

---

## 1. 子商户注册/查询接口

### 1.1. /mch/v1/signup 注册接口

注册子商户接口, 调用成功后会返回子商户 id `mchid`, 通过审核后可以使用子商户进行收款.

- POST

    | 参数名             | 描述               | 是否必填 | 备注                                                                     | 示例                     |
    |--------------------|--------------------|----------|--------------------------------------------------------------------------|--------------------------|
    | username           | 用户名             | 必填     | 格式必须为: 1) 邮箱+数字后缀; 2) 11 位手机号 二者之一, 否则无法通过审核. | some_user@example.com001 |
    | idnumber           | 身份证号           | 必填     |                                                                          | 100000201608220123       |
    | name               | 企业名称           | 必填     |                                                                          | XX商店                   |
    | country            | 国家               | 必填     |                                                                          | 中国                     |
    | city               | 城市               | 必填     |                                                                          | 北京                     |
    | address            | 详细地址           | 必填     |                                                                          | XX区XX路XX号             |
    | shopname           | 店铺名称/收据名称  | 必填     |                                                                          | XX商店                   |
    | bankaccount        | 银行卡号           | 必填     |                                                                          | 1234567890               |
    | bankuser           | 开户人姓名         | 必填     |                                                                          | 王某某                   |
    | bankcountry        | 开户行所在国家     | 必填     |                                                                          | 中国                     |
    | bankcity           | 开户行所在市       | 必填     |                                                                          | 北京                     |
    | bankaddr           | 开户行详细地址     | 必填     |                                                                          | XX区XX路支行             |
    | bankname           | 开户行名称         | 必填     |                                                                          | 中国建设银行             |
    | bankcode           | 开户行编号         | 必填     |                                                                          | 123                      |
    | bankswiftcode      | 联行号             |          |                                                                          | 456                      |
    | mcc_id             | 商户类别 id        | 必填     | 通过调用 mcc 列表接口获取                                                | 6125485337528623306      |
    | legalperson        | 法人姓名           | 必填     |                                                                          | 王某某                   |
    | channel_type       | 渠道 ID            | 必填     |                                                                          |                          |
    | email              | 邮箱               | 必填     |                                                                          | mail@example.com         |
    | mobile             | 手机号             | 必填     |                                                                          | 14412345678              |
    | licensenumber      | 证书编号           | 必填     |                                                                          | 1234567890               |
    | licenseactive_date | 公司申报生效日期   | 必填     |                                                                          | 2016-08-01               |
    | idcardfront        | 法人身份证正面照片 | 必填     | 文件                                                                     |                          |
    | licensephoto       | 公司商业登记证照片 | 必填     | 文件                                                                     |                          |
    | goodsphoto         | 经营场所内景照片   | 必填     | 文件                                                                     |                          |
    | shopphoto          | 经营场所外景照片   | 必填     | 文件                                                                     |                          |
    | credit_front       | 公司注册证书照片   |          | 文件                                                                     |                          |
    | shoper             | 业务员在店铺照片   |          | 文件                                                                     |                          |
    | other_1            | 其他               |          | 文件                                                                     |                          |
    | other_2            | 其他               |          | 文件                                                                     |                          |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "mchid": "BvDtmKJA5mx7GpN0"  // 子商户 id
    }
    ```

### 1.2. /mch/v1/uploadcert 补件

补充上传注册接口中未传的文件.

- POST

    | 参数名       | 描述                    | 是否必填 | 备注 |
    |--------------|-------------------------|----------|------|
    | mchid        | signup 接口返回的 mchid | 必填     |      |
    | idcardfront  | 法人身份证正面照片      |          | 文件 |
    | licensephoto | 公司商业登记证照片      |          | 文件 |
    | goodsphoto   | 经营场所内景照片        |          | 文件 |
    | shopphoto    | 经营场所外景照片        |          | 文件 |
    | credit_front | 公司注册证书照片        |          | 文件 |
    | shoper       | 业务员在店铺照片        |          | 文件 |
    | other_1      | 其他                    |          | 文件 |
    | other_2      | 其他                    |          | 文件 |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": ""
    }
    ```

### 1.3. /mch/v1/query 查询用户信息

- GET

    | 参数名 | 描述                    | 是否必填 | 备注         |
    |--------|-------------------------|----------|--------------|
    | mchid  | signup 接口返回的 mchid | 必填     | 子商户 mchid |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "data": {
            "username": "",       // 用户名
            "name": "",           // 企业名称
            "email": "",          // 邮箱
            "mobile": "",         // 手机号
            "idnumber": "",       // 身份证号
            "legalperson": "",    // 法人姓名
            "shopname": "",       // 店铺名
            "country": "",        // 国家
            "city": "",           // 城市
            "address": "",        // 地址
            "businessaddr": "",   // 营业地址
            "bankaccount": "",    // 银行卡号
            "bankuser": "",       // 开户人
            "bankcountry": "",    // 开户行国家
            "bankcity": "",       // 开户行城市
            "bankaddr": "",       // 开户行地址
            "bankname": "",       // 银行名称
            "bankswiftcode": "",  // 联行号
            "state": 1            // 用户状态
        }
    }
    ```

### 1.4. /tool/v1/mcc mcc 列表接口

该接口可以直接 HTTP GET 访问, 无需附加参数及签名.

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "data": {
            "mcc_list": [{
                "mcc_id": 6128181289566687125,
                "mcc_name": "XXX"
            },
            ...
            ]
        }
    }
    ```

---

## 2. 支付接口

### 2.1. /trade/v1/payment 支付接口

- POST

    | 参数名       | 描述                                                              | 是否必填 | 备注          | 示例                |
    |--------------|-------------------------------------------------------------------|----------|---------------|---------------------|
    | mchid        | signup 接口返回的 mchid                                           | 必填     | 子商户 mchid  | BvDtmKJA5mx7GpN0    |
    | pay_type     | 交易类型, 必填                                                    | 必填     |               | 800208              |
    | out_trade_no | 外部订单号, 必填                                                  | 必填     |               | 1470020842103       |
    | txdtm        | 外部订单生成时间, 必填                                            | 必填     |               | 2016-08-01 11:07:22 |
    | txamt        | 交易金额, 必填, 单位为分, 币种为商户绑定的币种                    | 必填     |               | 10                  |
    | product_name | 商品名称                                                          |          |               |                     |
    | valid_time   | 订单有效期, 单位为秒                                              |          | 最少为 300 秒 |                     |
    | udid         | 设备唯一标示                                                      |          |               |                     |
    | openid       |                                                                   |          |               |                     |
    | sub_openid   |                                                                   |          |               |                     |
    | auth_code    | 反扫消费者二维码得到的 auth_code                                  |          |               |                     |
    | lnglat       | 经纬度, 格式为 "12.34 56.78", 精度在前, 维度在后, 以一个空格分隔. |          |               |                     |

    `pay_type` 有如下选项:

    | pay_type | 交易类型            |
    |----------|---------------------|
    | 800108   | 支付宝扫码          |
    | 800151   | 支付宝线上PC预下单  |
    | 800152   | 支付宝线上WAP预下单 |
    | 800201   | 微信预下单          |
    | 800207   | 微信 H5 下单        |
    | 800208   | 微信扫码            |

    **注意事项**:
    1. 支付宝撤销订单即使是已完成的订单也会撤销**并退款**.
    2. 微信关闭订单只会关闭未完成的订单.
    3. 取决于支付方案的不同, 某些交易类型可能无法使用.

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "Success",
        "pay_type": "800101",
        "syssn": "201607280901020011216135",  // 流水号
        "sysdtm": "2016-07-28 14:13:10",
        "out_trade_no": "1469686389649",
        "txdtm":"2016-07-28 14:13:09",
        "txamt": "1",
        "txcurrcd": "HKD",
        "qrcode": "https://qr.alipay.com/xxxx"  // 预下单时返回的二维码地址
    }
    ```

### 2.2. /trade/v1/close 关闭订单

订单未支付成功时, 关闭订单; 订单支付成功后无法关闭. **目前仅支持微信支付**.

- POST

    | 参数名       | 描述                          | 是否必填                     | 备注         | 示例                     |
    |--------------|-------------------------------|------------------------------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid       | 必填                         | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn.       | 和 `out_trade_no` 二选一必填 |              | 201607280901020011216135 |
    | out_trade_no | 创建订单时使用的 out_trade_no | 和 `syssn` 二选一必填        |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填        | 必填                         |              | 2016-08-01 11:07:22      |
    | txamt        | 订单金额                      | 必填                         |              | 10                       |
    | udid         | 设备唯一标示                  |                              |              |                          |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "sysdtm": "2016-07-28 11:01:39",
        "orig_syssn": "201607280901020011216108", // 原订单号
        "syssn": "201607280901020011216110"  // 关闭订单操作的订单号
    }
    ```

### 2.3. /trade/v1/refund 退款

- POST

    | 参数名       | 描述                     | 是否必填 | 备注         | 示例                     |
    |--------------|--------------------------|----------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid  | 必填     | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn   | 必填     |              | 201607280901020011216135 |
    | out_trade_no | 外部订单号, 必填         | 必填     |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填   | 必填     |              | 2016-08-01 11:07:22      |
    | txamt        | 交易金额, 必填, 单位为分 | 必填     |              | 10                       |
    | udid         | 设备唯一标示             |          |              |

- Response

    ``` javascript
    {
        "orig_syssn": "201607280901020011216135",
        "respmsg": "",
        "txdtm": "2016-07-28 14:13:50",
        "txamt": "1",
        "out_trade_no": "1469686430937",
        "sysdtm": "2016-07-28 14:13:51",
        "syssn": "201607280901020011216137",
    }
    ```

### 2.4. /trade/v1/reversal 撤销/冲正

支付失败时关闭订单. 支付成功时退款并关闭订单.

- POST

    | 参数名       | 描述                     | 是否必填 | 备注         | 示例                     |
    |--------------|--------------------------|----------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid  | 必填     | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn   | 必填     |              | 201607280901020011216135 |
    | out_trade_no | 外部订单号, 必填         | 必填     |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填   | 必填     |              | 2016-08-01 11:07:22      |
    | txamt        | 交易金额, 必填, 单位为分 | 必填     |              | 10                       |
    | udid         | 设备唯一标示             |          |              |

- Response

    ``` javascript
    {
        "orig_syssn": "201607280901020011216135",
        "respmsg": "",
        "txdtm": "2016-07-28 14:13:50",
        "txamt": "1",
        "out_trade_no": "1469686430937",
        "sysdtm": "2016-07-28 14:13:51",
        "syssn": "201607280901020011216137",
    }
    ```

### 2.5. /trade/v1/query 查询订单信息

- POST

    | 参数名       | 描述                    | 是否必填 | 备注                                              | 示例                                              |
    |--------------|-------------------------|----------|---------------------------------------------------|---------------------------------------------------|
    | mchid        | signup 接口返回的 mchid | 必填     | 子商户 mchid                                      | BvDtmKJA5mx7GpN0                                  |
    | syssn        | 订单号                  |          | 支持批量查询, 使用 "," 分割多个 syssn             | 201607280901020011216135,201607280901020011216136 |
    | out_trade_no | 外部订单号              |          | 支持批量查询, 使用 "," 分割                       | 1470020842103,1470020842104                       |
    | pay_type     | 支付类型                |          | 支持批量查询, 使用 "," 分割                       | 800201,800208                                     |
    | respcd       | 交易状态码              |          |                                                   | 0000                                              |
    | start_time   | 起始时间                |          | 默认会从本月 1 日 0 点开始.                       | 2016-08-01 11:00:00                               |
    | end_time     | 结束时间                |          | 结束时间和起始时间必须在一个月内, 即不能跨月查询. | 2016-08-21 11:00:00                               |
    | page         | 查询页码                |          |                                                   | 1                                                 |
    | page_size    | 每页返回数量            |          | 默认为 10                                         | 20                                                |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "", 
        "page": 1,
        "page_size": 10,
        "data": [{
            "pay_type": "800201",
            "sysdtm": "2016-07-26 17:02:01",
            "order_type": "payment",
            "txcurrcd": "HKD", 
            "txdtm": "2016-07-26 17:02:01",
            "txamt": "1",
            "out_trade_no": "1469523721486",
            "syssn": "201607260901020011216001",
            "cancel": "2",
            "respcd": "1142",
            "errmsg": ""
        }]
    }
    ```

### 2.6. 预下单结果通知.

对预下单订单, 当得到支付结果后, 会将结果以 HTTP POST 的形式通知给开发者预先设置的回调地址, 并在 HTTP Header 中设置签名 `X-QF-SIGN`. 其中包括以下数据:

| 参数名       | 描述                    | 备注 | 示例                                              |
|--------------|-------------------------|------|---------------------------------------------------|
| mchid        | signup 接口返回的 mchid |      | BvDtmKJA5mx7GpN0                                  |
| syssn        | 订单号                  |      | 201607280901020011216135,201607280901020011216136 |
| out_trade_no | 外部订单号              |      | 1470020842103,1470020842104                       |
| pay_type     | 支付类型                |      | 800201,800208                                     |
| txamt        | 交易金额                |      | 1                                                 |
| txdtm        | 交易时间                |      | 2016-08-21 11:00:00                               |
| respcd       | 交易状态码              |      | 0000                                              |
| respmsg      | 交易结果文字描述        |      |                                                   |

### 2.7. 支付宝线上支付重定向参数

对支付宝线上支付, 支付完成后会重定向到商户设定的 return_url, 并以 GET 方式附带上以下参数:

| 参数名       | 描述             | 备注 | 示例                     |
|--------------|------------------|------|--------------------------|
| mchid        | 子商户 mchid     |      | BvDtmKJA5mx7GpN0         |
| syssn        | 订单号           |      | 201607280901020011216135 |
| out_trade_no | 外部订单号       |      | 1470020842103            |
| pay_type     | 支付类型         |      | 800151                   |
| txamt        | 交易金额         |      | 1                        |
| txdtm        | 交易时间         |      | 2016-08-21 11:00:00      |
| respcd       | 交易状态码       |      | 0000                     |
| respmsg      | 交易结果文字描述 |      |                          |

---

## 3. OAuth 授权接口

### 3.1. /oauth/v2/authorize 用户授权

商户引导已登录钱方用户重定向到该页面, 并用 GET 方式附加上以下参数. 用户在页面上点击同意授权后, 会带着 `code` 参数重定向到 `redirect_uri`.

一个完整的调用 url 应该类似这样: `http://127.0.0.1:9527/oauth/v2/authorize?response_type=code&client_id=123456&scope=user_tradelist,user_baseinfo&redirect_uri=http%3A%2F%2Fbaidu.com`

- GET

    | 参数名        | 描述                                                                    | 是否必填 | 备注                 |
    |---------------|-------------------------------------------------------------------------|----------|----------------------|
    | response_type | 固定为 "code"                                                           | 必填     |                      |
    | client_id     | 分配给商户的 appcode                                                    | 必填     |                      |
    | scope         | 需要申请的权限, 可选的有 user_tradelist user_baseinfo, 多选时用逗号分隔 |          | 默认为 user_baseinfo |
    | redirect_uri  | 用户同意授权后跳转到的 url.                                             |          |                      |

### 3.2. /oauth/v2/access_token 获取/刷新 access_token

商户获取到 code 后, 使用 code 首次获取 access_token. 或是使用 refresh_token 获取 access_token.

- POST

    | 参数名        | 描述                                                                                          | 是否必填                               | 备注 |
    |---------------|-----------------------------------------------------------------------------------------------|----------------------------------------|------|
    | grant_type    | 首次使用 code 时为 "authorization_code", 后续使用 refresh_token 重新获取时为 "refresh_token". | 必填                                   |      |
    | client_id     | 分配给商户的 appcode                                                                          | 必填                                   |      |
    | client_secret | 分配给商户的 appkey                                                                           | 必填                                   |      |
    | code          | authorize 接口重定向时附带的 code                                                             | 必填                                   |      |
    | refresh_token | 刷新 access_token 使用的 refresh_token.                                                       | `grant_type` 为 `refresh_token` 时必填 |      |

- Response

    ``` javascript
    {
        "userid": "8JRvY",
        "token_type": "Bearer",
        "access_token": "93a4bb59-5d65-451f-903e-67c01264803e",
        "refresh_token": "7248224a-aa2a-47ca-a056-b9a1b363d4d7"
        "expires_in": 7200,
    }
    ```

---

## 4. OAuth 查询接口

### 4.1. /user/v1/baseinfo 获取用户信息

使用 access_token 获取用户信息

- GET

    | 参数名       | 描述         | 是否必填 | 备注 |
    |--------------|--------------|----------|------|
    | access_token | access_token | 必填     |      |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "data": {
            "nickname": "\u94b1\u53f0\u6d4b\u8bd5",
            "userid": "rVY8nPPzMYLJV"
        },
        "resperr": "",
        "respmsg": ""
    }
    ```

### 4.2. /user/v1/tradelist 获取订单信息

使用 access_token 获取订单信息, 只能获取 3 个月内的交易.

- GET

    | 参数名       | 描述         | 是否必填 | 备注 |
    |--------------|--------------|----------|------|
    | access_token | access_token | 必填     |      |
    | start_time   | 起始时间     | 必填     |      |
    | end_time     | 结束时间     | 必填     |      |
    | page         | 页码         | 必填     |      |
    | page_size    | 每页数量     | 必填     |      |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "data": {
            "page": 1,
            "pagesize": 10,
            "tradelist": [{
                "pay_type": "800208",
                "sysdtm": "2016-07-20 14:47:50",
                "txcurrcd": "HKD",
                "txdtm": "2016-07-20 14:47:49",
                "txamt": "1",
                "syssn": "201607200901020011215925",
                "customer_id": ""
            }]
        },
        "resperr": "",
        "respmsg": ""
    }
    ```

---
## 5. 清算查询接口
### 5.1 /settlement/v1/query
- GET

    | 参数名      | 描述                              | 是否必填   | 备注         | 示例                     |
    |-------------|-----------------------------------|------------|--------------|--------------------------|
    | mchid       | signup 接口返回的 mchid           | 必填       | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | date_start  | 查询清算的起始时间                | 必填       |              | 20160101                 |
    | date_end    | 查询清算的结束时间                | 必填       |              | 20160130                 |
    | paymethod   | 支付方式                          | 非必填     |              | 2(支付宝), 3(微信)       |

- Response

    ``` javascript
    {
        "resperr": "",
        "respcd": "0000",
        "respmsg": "",
        "data": [
            {
                "chnl_mch_id": "1315135101",            //通道分配的商户号
                "refund_amt": 0,                        //交易退款总金额
                "date_settlement": "2016-02-29",        //清算时间
                "paymethod": "微信",                    //支付方式
                "date_end": "2016-02-29",               //交易起始时间
                "date_start": "2016-02-23",             //交易结束时间
                "currency": "HKD",                      //币种
                "pay_amt": 433280000,                   //交易支付总金额
                "chnl_poundage_amt": 4309600,           //通道手续费
                "pay_net_amt": 433280000,               //交易支付净额
                "settlement_amt": 428970400             //清算总金额
            }
        ]
    }

    ```

---
## 6. 附录

### 6.1 respcd 列表

| respcd | 错误原因                         |
|--------|----------------------------------|
| 1100   | 系统维护                         |
| 1101   | 需要主动冲正                     |
| 1102   | 重复请求                         |
| 1103   | 报文格式错误                     |
| 1104   | 报文参数错误                     |
| 1105   | 终端未激活                       |
| 1106   | 终端不匹配                       |
| 1107   | 终端被封禁                       |
| 1108   | MAC 校验失败                     |
| 1109   | 加解密错误                       |
| 1110   | 客户端充值, 流水号错误           |
| 1111   | 外部服务不可用                   |
| 1112   | 内部服务不可用                   |
| 1113   | 用户不存在                       |
| 1114   | 用户被封禁                       |
| 1115   | 用户受限                         |
| 1116   | 用户密码错误                     |
| 1117   | 用户不在线                       |
| 1118   | 风控禁止交易                     |
| 1119   | 交易类型受限                     |
| 1120   | 交易时间受限                     |
| 1121   | 交易卡类型受限                   |
| 1122   | 交易币种受限                     |
| 1123   | 交易额度受限                     |
| 1124   | 无效交易                         |
| 1125   | 已退货                           |
| 1126   | 原交易信息不匹配                 |
| 1127   | 数据库错误                       |
| 1128   | 文件系统错误                     |
| 1129   | 已上传凭证                       |
| 1130   | 交易不在允许日期                 |
| 1131   | 渠道错误                         |
| 1132   | 客户端版本信息错误               |
| 1133   | 用户渠道信息错误                 |
| 1134   | 撤销交易刷卡与消费时不是同一张卡 |
| 1135   | 用户配置错误                     |
| 1136   | 交易不存在                       |
| 1137   | 联系方式不存在                   |
| 1138   | 用户更新密钥错                   |
| 1139   | 卡号或者卡磁错误                 |
| 1140   | 账户未审核通过                   |
| 1141   | 计算通道MAC错误                  |
| 1142   | 订单已关闭                       |
| 1143   | 交易不存在                       |
| 1144   | 请求处理失败(协议)               |
| 1145   | 订单状态等待支付                 |
| 1146   | 订单处理业务错误                 |
| 1141   | 通道加密磁道错误                 |
| 1147   | 微信刷卡失败                     |
| 2001   | 机构不存在                       |
| 2002   | 商户绑定失败                     |
| 2003   | 签到失败                         |
| 2004   | 消费者余额不足                   |
| 2005   | 消费者二维码过期                 |
| 2006   | 消费者二维码非法                 |
| 2007   | 消费者关闭了这次交易             |
| 2008   | 传递给通道的参数错误             |
| 2009   | 连接通道失败                     |
| 2010   | 和通道交互的未知错误             |
| 2011   | 交易流水号重复                   |
| 2012   | 用户的通道证书配置错误           |
| 2999   | 通道处理中                       |
| 1151   | 原预授权信息不匹配               |
| 1152   | 预授权完成不在允许日期           |
| 1153   | 预授权完成金额错误               |
| 1154   | 内部错误                         |
| 1155   | 不允许撤销的交易                 |
| 1161   | 交易结果未知，须查询             |
| 1170   | channeld不能提供服务             |
| 1180   | 路由重置, 需重新路由             |
| 1181   | 订单过期                         |
| 1201   | 余额不足                         |
| 1202   | 付款码错误                       |
| 1203   | 账户错误                         |
| 1204   | 银行错误                         |

---

<!--
vim: foldlevel=2
-->


QFPay API in Japanese Business

Index

●1. Business Process
●1.1 Merchant registration 
●1.2 Payment Process
○1.2.1 Barcode payment
○1.2.2 Query transaction
○1.2.3 Cancel transaction
○1.2.4 Response code description
●1.3 Reconciliation

●2. API
●2.1 Scan barcode interface
●2.2 Dynamic QR code pay interface   
●2.3 Refund interface
●2.4 Query interface
●2.5 Close interface
●2.6 Download reconciliation interface 
●2.7 Parameter Description
●3. Developer Guide
●3.1 Environment Description
●3.2 Signature Algorithm
●3.3 Request Description 
●3.4 Verifying the Signature 
●3.5 Notification Callback 
●3.6 Sample code

1. Business Process
1.1 Merchant registration 

Merchants have to register to the QFPay services.
After registration, merchants will receive their app-code, key for integration, and the unique merchant ID (mchid) for each store.

Example:
AppCode: <32_character_MD5_string_qfpay>
Key: <32_character_MD5_string_qfpay>
1.2 Payment Process

Now, we provide an example about how merchants can use the API to do a barcode payment.

Note: There are limitations to the payment amount (depending on bank). Please refer to kf.qq.com/touch/sappfaq/151210NZzmuY151210ZRj2y2.html?platform=15&ADTAG=veda.weixinpay.wenti
1.2.1 Barcode payment

After receiving the code, key and mchid, merchants are ready to use the QFPay services. By using code scanner, Merchant can scan consumer's Wechat/Alipay QR code to get the auth_code, cash will be deducted from consumer's Wechat/Alipay account. The whole process is shown as the following flow chart

Process:
Step 1, using code scanner to scan consumer's Wechat/Alipay QR code, send the transaction request to QFPay system. (The detail of the barcode payment interface can be found at API 2.1)

1a, if get response code as 0000 from the QFPay system, then the transaction is successful.
1c, if get response code which is not 0000, 1143 or 1145, then the transaction is failed.
1b, if get response code as 1143, 1145 from the QFPay system, then the transaction is still in processing.


1.2.2 Query transaction

Step 2, call the query interface to QFPay system and check the status of the transaction again. 
 (The detail of the query interface can be found at API 2.4)

	2a, if get response code as 0000 from the QFPay system, then the 			transaction is successful.
	2c, if get response code which is not 0000, 1143 or 1145, then the 			transaction is failed.
	2b, if get response code as 1143, 1145 from the QFPay system, then 			the transaction is still in processing. Go back to Step 2.




1.2.3 Cancel transaction
After calling query interface for more than 2 minutes, if the response code is still 1143 or 1145, please cancel the transaction and place the order again.
 (The detail of the cancel interface can be found at API 2.5)




1.2.4 Response code description
Response code	Description
0000	Operation successful
1147	Wechat card failed | brushed Hong Kong wallet, does not support or password input error
2007	The consumer closed the transaction | the consumer needs to enter the password, but the consumer closes the password input box, causing the order to close.
2006	QR code is illegal;
Alipay QR code was scanned by wechat;
QR code is invalid
1142	The order has been closed | the merchant opened an order, then timed out and the order closed.
1144	Wechat pay QR code was scanned by Alipay
Bank card binded is invalid.
2004	Insufficient account balance
2005	QR code is invalid | payment is made within one minute of the consumer opening the payment APP
1142	Order has been closed.
1145	Waiting for payment.
1296	Channel error (configuration error or Wechat is suspended)
1297	Please try other payment types | Configuration error, scaner side and payment side are the same Wechat user.
1143	"The order does not exist" , The refund channel request was not passed, resulting in no refund
















2. API
2.1 Barcode Payment
2.1.1 Interface Description
In this scenario, merchant can scan consumer's Wechat/Alipay QR code via scanning gun to get auth_code, cash will be deducted from consumer's Wechat/Alipay account 
2.1.2 Process Flow





2.1.3 Request Information
Method	HTTP Post
Path	/trade/v1/payment
Description	Merchants scan the QR code in the consumer’s smartphone.

2.1.4 Request Parameters

Parameter	Required	Definition	Description	Example
txamt	Y	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 
(Maximum 10,000 USD)	1
txcurrcd	Y	String	Currency code: JPY	JPY
pay_type	Y	String	Payment type:
Wechat:800208;Alipay:800108;
All-in-one swipe 800008;	800208
out_trade_no	Y	String
No longer than 128 byte	Merchant's order number:
The merchant’s order number has to be unique	1469686389649
txdtm	Y	String 	Transaction time :
format: YYYY-MM-DD HH:MM:SS
	2016-07-28 14:13:10
auth_code	Y	String(128)	Authentication code:
The QR code shown from the user’s Wechat or Alipay	120061098828009406
goods_name	Y	String
No long than 20 byte	Name of the product:
special character is not allowed 	
mchid	Y	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in Agent Management System)	BvDtmKJA5mx7GpN0
udid	N		UUID of device, no longer than 40 characters	
2.1.5 Request Example
{'pay_type': '800201', 
'out_trade_no': 755004603759, 
'goods_name': '\xe9\x92\xb1\xe6\x96\xb9\xe6\xb5\x8b\xe8\xaf\x95',
 'limit_pay': 'no_credit',
 'udid': '1880105',
 'txcurrcd': 'JPY', 
'txdtm': '2018-07-20 04:11:11', 
'mchid': '8w5pdhDJkm', '
txamt': 1}

2.1.6 Response Parameters

Parameter	Definition	Description	Example
pay_type	String	Payment type:
wechat: 800201;alipay: 800101;	800201
sysdtm	String	System time	2016-07-28 14:13:10
txdtm	String	Transaction time :
format: YYYY-MM-DD HH:MM:SS 	2016-07-28 14:13:09
resperr	String	Information Description	success
txcurrcd	String	 Currency code: JPY	JPY
txamt	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000
respmsg	String	Debug information	success
out_trade_no	String
No longer than 128 byte	Merchant's order number	1470020842103
syssn	String	Order number in QFPAY system	201607280901020011216135
respcd	String	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction in processing, merchant need to continue querying transaction. if other codes returned, transaction failed	0000
mchid	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)	BvDtmKJA5mx7GpN0
2.1.7 Response Example
{"pay_type": "800208",
"sysdtm": "2018-01-12 17:10:26", 
"cardcd":"oo3Lss-DzPSygtHtAbfuXeQFCz18",
 "txdtm": "2018-01-12  17:10:32",
 "resperr": "\u4ea4\u6613\u6210\u529f",
"txcurrcd": "CNY", 
"txamt": "1",
"respmsg": "",
 "out_trade_no":"1301459478787530052", 
"syssn":"20180112000100020001659358",
 "respcd": "0000"}
2.2 Merchant QR Code Payment
2.2.1 Interface Description
Merchant enter the price of product and call QFPay’s interface to generate a dynamic QR code, consumer scan the QR code by using Wechat or Alipay app to finish the payment. The dynamic QR code will expire in 30 minutes.
2.2.2 Request Information
Method	HTTP Post
Path	/trade/v1/payment
Description	Consumers scan the QR code from merchants.

2.2.3 Request Parameters
Parameter	Required	Definition	Description	Example
txamt	Y	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 
(Max 10,000 USD)	10000
txcurrcd	Y	String	Currency code: JPY	JPY
CNY
pay_type	Y	String	Payment type:
wechat: 800201;alipay: 800101;	800201
out_trade_no	Y	String
No longer than 128 byte	Merchant's order number
The merchant’s order number has to be unique	1470020842103
txdtm	Y	String	Transaction time:
format: YYYY-MM-DD HH:MM:SS 	2016-08-01 11:07:22
goods_name	Y	String	Name of the product	Cosmetics
mchid	Y	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)	BvDtmKJA5mx7GpN0
limit_pay	N		The value of this parameter can only be set to no_credit, which means credit is not allowed	
udid	N		UUID of device, no longer than 40 characters	

2.2.4 Request Example
{'pay_type': '800208',
 'out_trade_no': 725526999001L, 
'goods_name': '\xe9\x92\xb1\xe6\x96\xb9\xe6\xb5\x8b\xe8\xaf\x95',
 'limit_pay': 'no_credit',
 'udid': '1880105', 
'txcurrcd': 'JPY',
 'auth_code': '134676275354204254', 
'txdtm': '2018-07-20 04:27:18',
'mchid': '8w5pdhDJkm', 
'txamt': 1}

2.2.5 Response Parameters


Parameter	Definition	Description	Example
pay_type	String	Payment type:
wechat: 800201;alipay: 800101;	800201
sysdtm	String	System time	2016-07-28 14:13:10
txdtm	String	Transaction time :
format: YYYY-MM-DD HH:MM:SS 	2016-08-01 11:07:22
resperr	String	Information Description	
txamt	String	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000
respmsg	String	Debug information	
out_trade_no	String
No longer than 128 byte	Merchant's order number	1470020842103
syssn	String	Order number in QFPAY system	201607280901020011216135
qrcode	String	QR code url:
merchant need to convert it to QR code image	weixin://wxpay/bizpayurl?pr=xxxxxxx
respcd	String	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction on going, merchant need to continue querying transaction. if other codes returned, transaction failed	1143
2.2.6 Response Example
{"pay_type": "800201", 
"sysdtm": "2018-01-12 17:38:50", 
"cardcd": "",
 "txdtm": "2018-01-12 17:38:56", 
"resperr": "\u4ea4\u6613\u6210\u529f", 
"txcurrcd": "JPY",
 "txamt": "1", 
"respmsg": "", 
"out_trade_no": "13014597457448787530052", 
"syssn": "20180112000100020001659801", 
"qrcode": "weixin://wxpay/bizpayurl?pr=hBvNxhc", 
"respcd": "0000"}
2.2.7 Notification callback
Merchants receive the notification callback when the payment is successfully made
Note: The notification callback may be delayed due to external factors. Therefore, please use query interface for the scene with real-time processing requirements. For security reasons, notification callback only supports ports 80 and 443, and custom port assignments are not supported. For more detail, please see Developer Guide 3.5.

2.3 Refund interface
2.3.1 Interface Description
The merchant and partner can use this interface to refund an existing payment. A refund can be partial and a transaction can have multiple refunds as long as the total refund amount is no greater than the original transaction amount.



2.3.2 Request Information
Method	HTTP Post
Path	/trade/v1/refund
Description	The Payer will be refunded with the request refund amount.

2.3.3 Request Parameters
Parameter	Required	Definition	Description	Example
txamt	Y	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000
syssn	Y	String	Order number in QFPAY system.
It supports batch query, use "," to delimit multiple syssn numbers	201607280901020011216135
out_trade_no	Y	String
No longer than 128 byte	Merchant's order number	1470020842103
txdtm	Y	String	Transaction time:
format: YYYY-MM-DD HH:MM:SS	2016-08-01 11:07:22
mchid	Y	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)	BvDtmKJA5mx7GpN0
udid	N		UUID of device, no longer than 40 characters	

2.3.4 Request Example
{'txdtm': '2018-07-20 05:10:04', 
'mchid': '8w5pdhDJkm', 
'txamt': 1, 
'out_trade_no': 428870566675L, 
'syssn': '20180112000100020001661542'}
}

2.3.5 Response Parameters
Parameter	Definition	Description	Example
syssn	String	Refund order number in QFPAY system	201607280901020011216135
orig_syssn	String	Original order number in QFPAY system	20180112000100020001659358
sysdtm	String	System time	2016-07-28 14:13:10
txdtm	String	Transaction time :
format: YYYY-MM-DD HH:MM:SS 	2016-08-01 11:07:22
txamt	String	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000

2.3.6 Response Example
{"orig_syssn": "20180112000100020001659358", 
"sysdtm": "2018-01-12 19:03:23", 
"cardcd": "",
 "txdtm": "2018-01-12 19:03:29", 
"resperr": "\u4ea4\u6613\u6210\u529f", 
"txcurrcd": "JPY",
 "txamt": "1",
 "respmsg": "", 
"out_trade_no": "13014597435743348787530052",
 "syssn": "20180112000100020001661542",
 "respcd": "0000"}

2.4 Query interface  

2.4.1 Interface Description

When the transaction is in processed, call the query interface to acquire the status of transaction. Using the start-time and end-time to check the status of transactions in multiple months. Or, using QFPay order number(syssn) to check the status of specific transaction.
2.4.2 Request Information
Method	HTTP Post
Path	/trade/v1/query
Description	Query a transaction

2.4.3 Request Parameters
Parameter	Required	Definition	Description	Example
mchid	Y	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)	BvDtmKJA5mx7GpN0
syssn	N	String	Order number in QFPAY system.
It supports batch query, use "," to delimit multiple syssn numbers	201607280901020011216135
out_trade_no	N	String
No longer than 128 byte	Merchant's order number
	1470020842103
pay_type	N	String	Payment type:
wechat: 800201;alipay: 800101;
	800201
respcd	N	String	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction on going, merchant need to continue querying transaction. if other codes returned, transaction failed	0000
start_time	N	String	Start time:
Default : starting at this month
format:YYYY-MM-DD HH:MM:SS
(in Local Time)	2016-08-01 11:00:00
end_time	N	String	End time:
Default : end at this month
format:YYYY-MM-DD HH:MM:SS
(in Local Time)	2016-08-21 11:00:00
page	N	Int	Number of pages:
default:1	1
page_size	N	Int	Number of orders displayed on each page
Default:10， max pagesize is 100	10

2.4.4 Request Example
{'pay_type': '800201',
 'page_size': 10,
 'mchid': '9GGGDCRjYa',
 'Page': 2}

2.4.5 Response Parameters
Parameters	Definition	Description	Example
page	Int	Number of pages	1
resperr	String	Information Description	
page_size	Int	Number of orders displayed on each page	10
respcd	String	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction on going, merchant need to continue querying transaction. if other codes returned, transaction failed	0000
data	List	Trade data
type:list	

Description of parameter <data>:

Parameters	Definition	Description	Example
syssn	String	Order number in QFPAY system	201607280901020011216135
out_trade_no	String
No longer than 128 byte	Merchant's order number
	1470020842103
pay_type	String	Payment type:
wechat: 800201;alipay: 800101;	800201
order_type	String	Order type:
Payed order:payment;
Refunded order:refund;
Closed order:close;	payment
txdtm	String	Transaction time:
format:YYYY-MM-DD HH:MM:SS	2016-08-01 11:07:22
txamt	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency.	10000
sysdtm	String	System time	2016-07-28 11:01:39
cancel	Int	Cancel/refund:
Normal trade:0
Cancel:2
refund:3	2
respcd	String	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction on going, merchant need to continue querying transaction. if other codes returned, transaction failed	
errmsg	String	Message of payment result	

The parameter respcd=0000 inside the data_list stands for the transaction is complete. 
The respcd=0000 outside of the data_list stands for the order has been placed.
2.4.6 Response Example
{'resperr': u'\u8bf7\u6c42\u6210\u529f',
 'page_size': 1, 
 'respmsg': '', 
'respcd': '0000', 
'data': [{'pay_type': '800201', 'sysdtm': '2018-07-24 12:30:57', 'paydtm': '', 'txcurrcd': 'CNY', 'txdtm': '2018-07-24 13:31:03', 'txamt': '1', 'chnlsn': '', 'out_trade_no': '923500563158', 'syssn': '20180724000200020076586895', 'cancel': '0', 'respcd': '1143', 'errmsg': '\u8ba2
\u5355\u8fd8\u672a\u652f\u4ed8\uff0c\u6216\u8005\u6b63\u5728\u8f93\u5165\u5bc6\u
7801\u4e2d', 'order_type': 'payment'}], 'page': 1}



2.5 Cancel interface 
2.5.1 Interface Description

1. What’s kind of order can be closed? 
If the payment is unsuccessful, the order can always be canceled. If  the cancel interface is called while the payment is successful made, which actually stands for calling the WeChat revoke API, and the payment for this order will be returned.

2. Scenarios of canceling the transaction
Cancel interface is only applicable at the time,which is mainly used to avoid double payments.
In order to avoid causing various unknown problems, please do not use the cancel interface as a refund interface.
2.5.2 Request Information
Method	HTTP Post
Path	/trade/v1/close
Description	Close a transaction
2.5.3 Request Parameters
Parameter	Required	Definition	Description	Example
mchid	Y	String	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)	BvDtmKJA5mx7GpN0
txamt	Y	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000
txdtm	Y	String	Transaction time :
format: YYYY-MM-DD HH:MM:SS	2016-08-01 11:07:22
syssn	N	String	Order number in QFPAY system.	201607280901020011216137
out_trade_no	N	String
No longer than 128 byte	Merchant's order number	1470020842103
udid	N	String	UUID of device, no longer than 40 characters	
2.5.4 Request Example
{'txdtm': '2018-07-24 15:20:09',
 'syssn': '20180724000200020076667504', 
'mchid': '9GGGDCRjYa',
 'txamt': 1}

2.5.5 Response Parameters
Parameters	Definition	Description	Example
syssn	String	Order number of the cancellation 	201607280901020011216137
orig_syssn	String	Original Order number of this transaction in QFPAY system	201607280901020011216135
txamt	Int	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 	10000
txdtm	String	Request Transaction time :
format: YYYY-MM-DD HH:MM:SS 	2016-07-26 17:02:01
sysdtm	String	System Transaction time	2016-07-20 14:47:50
2.5.6 Response Example
{"orig_syssn": "20180112000100020001659801",
 "sysdtm": "2018-01-12 19:07:02",
 "cardcd": "", 
"txdtm": "2018-01-12 19:07:09",
 "resperr": "\u4ea4\u6613\u6210\u529f",
 "txcurrcd": "CNY",
 "txamt": "1",
 "respmsg": "",
 "syssn": "20180112000100020001661611", 
"respcd": "0000"}


Note: Cancel transaction is non-generic supported, there may be cases that do not support canceling the transaction. If the cancel interface response return code is "1297" or the returned content has no syssn field (note: not orig_syssn), the transaction does not support canceling the order. If the user Successfully paid, the refund interface can be called for a refund.


2.6 Parameter Description
2.6.1 out_trade_no
The merchant’s order number is custom generated by the merchant. It only supports the combination of alphanumeric characters, such as alphanumeric, underline _, vertical bar|, and asterisk *. Do not use special characters such as Chinese characters or full-width characters. The merchant order number is required to be unique (it is recommended to be generated based on the current system time plus a random sequence). To re-initiate a payment, use the original order number to avoid double payment; the order number that has been paid or has been called, and the cancelled order number cannot be reused for a new payment.

Notice: If not using, please not input this parameter to request parameters as the form of none. 
2.6.2 goods_name
no longer than 20 characters, special characters are not allowed. Note: special characters include full-width characters and symbols (★☆★$ & ¤ § | °゜ ¨ ± · × ÷ ˇ ˉ ˊ ˋ ˙ Γ Δ Θ Ξ Π Σ Υ Φ Ψ Ω α β γ δ ε ζ η θ ι κ λ μ ν ξ π ρ σ τ υ φ ψ ω Ё Б Г Д Е Ж З ИЙ К Л Ф У Ц Ч Ш Щ Ъ Ы Э Ю Я а б в г д ж з и й к л ф ц ч ш щ ъ ы ю я ａｂｃｄｅｆｇｈｉｊｋｌｍｎｏｐｑｒｓｔｕｖｗｘｙｚ - ― ‖ ‥ … ‰ ′ ″ ※ ℃ ℅ ℉ № ℡)



3. Developer Guide
3.1 Environment Description

QFPay provides two environment for using our services:
3.1.1 Test Environment:
Request Url: https://openapi-test.qfpay.com
1.During test, the money will not be operated. Please use little amount.
2.Mchid can be found in the channel system, which will be provided when formally applying the app-code and key.
3.Merchants under the same channel system can use the same app-code and key.
3.1.2 Formal Environment:
Request Url: https://openapi.qfpay.com
Please use your formal information including the mchid, code and key.
3.2 Signature Algorithm

When there is no special statement, the signature in all request interfaces uses the format below:
How to make the interface parameter signature:
Step 1: Ascending order all parameters by parameter name。
Parameter list :abc=value1 bcd=value2 bad=value3
The ascending result:abc=value1 bad=value3 bcd=value2
Step 2:Connect all parameters with & to get the string to be signed.
abc=value1&bad=value3&bcd=value2
Step 3:Splice the string to be signed with the developer's Key.
abc=value1&bad=value3&bcd=value2Key
Step 4:Apply  MD5 for the spliced string.
MD5(abc=value1&bad=value3&bcd=value2Key)
Step 5:Use signature to request the interface.
Store the signed results into the X-QF-SIGN in the HTTP head.

Python Sample:

3.3 Request Description

When requesting API interface, parameters have to be set up as follows in the HTTP head.
Name	Required	Description
X-QF-APPCODE	Y	The app_code assigned to the developer is the unique identifier of the developer. This parameter requires to be directly assigned from QFPay.
X-QF-SIGN	Y	Signature generated according to the signature algorithm
3.4 Verifying the Signature
Description: after successfully requesting the interface, a JSON format buffer will be received and to be verified.
The response HTTP head is set up as follows’
Name	Description
X-QF-SIGN	Step 1: Get the signature from X-QF-SIGN in HTTP head.
Step 2: Connect the body of the HTTP response with Key to get the string to be verified.
Step 3: Apply MD5 for the spliced string. (SIGN = MD5(body + key))
Step 4: Compare the computing result with the signature from X-QF-SIGN to verify whether they are identical.
3.4.1 Head Signature sample
{'Server': 'nginx', 'Date': 'Fri, 12 Jan 2018 11:31:18 GMT', 'Content-Type': 'application/json; charset=UTF-8', 'Content-Length': '302', 'Connection': 'keep-alive', 'X-QF-SIGN': '1E2CE7C2A7F8F581C354A857182B7A31', 'X-Powered-By': 'QF/1.1'}


3.4.2 Public response parameters:
Parameters	Description
respcd	When the respcd outside of data buffer is 0000, it stands for the order is successfully generated.
Resperr	Response information
pay_type	Payment types
respmsg	Debug information
3.4.3 Response Signature sample
MD5 data:{"pay_type": "800108", "sysdtm": "2018-01-12 19:31:16", "cardcd": "2088802362210279", "txdtm": "2018-01-12 19:31:22", "resperr": "\u4ea4\u6613\u6210\u529f", "txcurrcd": "CNY", "txamt": "1", "respmsg": "", "out_trade_no": "130145934788787530052", "syssn": "20180112000100020001662134", "respcd": "0000"}615ED178BA524459976CE40FAB78000F
MD5 results:1E2CE7C2A7F8F581C354A857182B7A31

3.5 Notification Callback 
3.5.1 Note 
The notification callback address has to be provided to the technical support of QFPay through channel system. Each notification callback  address can only be configured with one set of code and key, which can be modified. 

If the order is successfully paid, the result of the notification callback will be returned. However, the notification callback may be delayed due to external factors. Therefore, please use query interface for the scene with real-time processing requirements. It is recommended that the notification callback can be used together with the query interface. For security reasons, notification callback only supports ports 80 and 443, and custom port assignments are not supported.

3.5.2 Notification Callback Rules

1.The address of the notification callback has to be provided to the technical support of QFPay by setting up in the channel  system.
2.After the transaction succeeds, the QFPay will POST the notification data (JSON format) to the configured notification callback address.
3.After receiving the notification callback and verifying the signature, the developer needs to return the SUCCESS string to the QFPay, indicating that it has been processed.
4.If the developer returns other data to the QFPay, or if there is no return, then the QFpay will send the notification callback again with a certain strategy. The time interval is configured as 1m, 5m, 10m, 60m, 2h, 6h, 15h. Once the SUCCESS string is sent back to QFPay, the follow-up callback notices will not be continued.

3.5.3 Verifying the signature in Callback:
Step 1: Get the signature from X-QF-SIGN in HTTP head.
Step 2: Connect the body of the HTTP response with Key to get the string to be verified.
Step 3: Apply MD5 for the spliced string. (SIGN = MD5(body + key))
Step 4: Compare the computing result with the signature from X-QF-SIGN to verify whether they are identical. If they are identical, the verification is successful and return SUCCESS string to QFPay.

3.5.4 Sample Code
import hashlib
unicode_to_utf8 = lambda s: s.encode('utf-8') if isinstance(s, unicode) else s
def make_resp_sign(data, key):
    unsign_str = unicode_to_utf8(data) + unicode_to_utf8(key)
    s = hashlib.md5(unsign_str).hexdigest()
    return s.upper()

3.5.5 Callback Response Parameter

Parameter	Description
notify_type	Notification types;
payment; refund: close:
syssn	Order number in QFPAY system
out_trade_no

String
No longer than 128 byte	Merchant's order number:
The merchant’s order number has to be unique.
pay_type	Payment type:
Wechat:800208;Alipay:800108;
All-in-one swipe 800008;
txdtm	Transaction time:
format: YYYY-MM-DD HH:MM:SS
txamt	Amount of payment: 
It is measured in the minimum price unit of the transaction currency. 
(Maximum 10,000 USD)
respcd	Response code:
0000 stands for order placed successfully
1143,1145 stand for transaction in processing, merchant 
sysdtm	System time
paydtm	Time when the user made the payment.
cancel	Marks for refunded or closed
Processing:0;Closed:2;Refunded:3;
cardcd	OpenID from Wechat or Alipay users
goods_name	Name of the product:
special character is not allowed 
status	Status of Transaction:1:The transaction is successful;
txcurrcd	Currency code: JPY
mchid	Merchant id:
allocate by QFPAY(You can look up merchant ID in channel system)
3.5.6 Response Callback sample
{"status": "1",
 "sysdtm": "2017-12-28 13:41:47", 
"paydtm": "2017-12-28 13:42:20",
 "goods_name": "", "txcurrcd": "CNY", 
"mchid": "XXxxxx", "cancel": "0", 
"pay_type": "800207",
"cardcd": "oo3Lss1m9-eHSEyY2OGKzxFaRflY",
 "txdtm": "2017-12-28 13:41:47",
 "txamt": "200", 
"out_trade_no": "BO201712280117290001",
"syssn": "20171228000300020044178249", 
"respcd": "0000",
 "goods_info": "", "notify_type": "payment"}

3.6 Python Sample code

#!/usr/bin/env python
#encoding=utf8
import urllib, urllib2, hashlib
import json
import argparse
import random
import qrcode
from datetime import datetime
from PIL import ImageFile
import hashlib

parser = argparse.ArgumentParser(description="Simulate Payment Process for QFPay and CIL")
parser.add_argument('--txamt', default=1, help='Payment Amount')
parser.add_argument('--type', default='wechat', help='Payment Type')
parser.add_argument('--cur', default='CNY', help='Payment Currency')
parser.add_argument('--authcode', help='Auth Code')
parser.add_argument('--syssn', help='Syssn Code')
parser.add_argument('--tradeno', help='Out Trade No')
parser.add_argument('--query', action='store_true', help='Perform Query')
parser.add_argument('--refund', action='store_true', help='Perform Refund')
parser.add_argument('--txdtm', help='Transaction Time')
parser.add_argument('--starttime', default=None)
parser.add_argument('--endtime', default=None)
parser.add_argument('--close', action='store_true')
parser.add_argument('--reconcile', action='store_true')
parser.add_argument('--tradedate', default=None)


unicode_to_utf8 = lambda s: s.encode('utf-8') if isinstance(s, unicode) else s

def make_req_sign(data, key):
    keys = data.keys()
    keys.sort()
    p = []
    for k in keys:
        k = unicode_to_utf8(k)
        v = unicode_to_utf8(data[k])
        p.append('%s=%s'%(k,v))
    unsign_str = '&'.join(p) + unicode_to_utf8(key)
    s = hashlib.md5(unsign_str).hexdigest()
    return s.upper()

def main():
    pay_type = {
        'wechat': '800201', #dynamic QR code generation
        'alipay': '800101',
        'wechat_app': '800208', #scan from user's app
        'alipay_app': '800108', #scan from user's app
        'general': '800008',
    }
    d = datetime.now()

    # mchid = None

    args = parser.parse_args()
    txamt = args.txamt
    txcurrcd = args.cur 
    pay_type = pay_type[args.type]
    out_trade_no = random.randint(10**11, 10**12)
    txdtm = d.strftime("%Y-%m-%d %H:%M:%S")
    # goods_name = '商品名１:缶コーヒー'
    goods_name = 'test'
    limit_pay = 'no_credit'
    udid = 'xxxxxxxx'
    auth_code = args.authcode
    start_time = args.starttime
    end_time = args.endtime

    mchid = 'xxxxxxxxx'

    app_code = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
    key = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'

    is_pay = True
    url = 'https://openapi.qfpay.com/trade/v1/payment'
    data ={'txamt': txamt, 'txcurrcd': txcurrcd, 'pay_type': pay_type, 'out_trade_no': out_trade_no, 'txdtm': txdtm, 'goods_name':goods_name,'limit_pay':limit_pay,'udid':udid}

    if mchid: data['mchid'] = mchid
    if args.type in ['wechat_app', 'alipay_app', 'general']:
        data['auth_code'] = auth_code

    if args.query:
        is_pay = False
        url = 'https://openapi.qfpay.com/trade/v1/query'
        data = { 'pay_type': pay_type, 'mchid': mchid, 'page_size': 1, 'Page': 2}
        if start_time: data['start_time'] = start_time
        if end_time: data['end_time'] = end_time
    elif args.refund:
        is_pay = False
        url = 'https://openapi.qfpay.com/trade/v1/refund'
        data = { 'syssn': args.syssn, 'out_trade_no': args.tradeno, 'pay_type': pay_type,'mchid': mchid, 'txamt': txamt, 'txdtm': args.txdtm}
    elif args.close:
        is_pay = False
        url = 'https://openapi.qfpay.com/trade/v1/close'
        data = {'txamt': txamt, 'txdtm': args.txdtm, 'mchid': mchid }
        if args.syssn is not None: data['syssn'] = syssn
        if args.tradeno is not None: data['out_trade_no'] = args.tradeno
    elif args.reconcile:
        url = 'https://openapi.qfpay.com/download/v1/trade_bill'
        data = { 'trade_date': args.tradedate }
    
    print(data)
    req = urllib2.Request(url, urllib.urlencode(data))
    req.add_header('X-QF-APPCODE', app_code)
    req.add_header('X-QF-SIGN', make_req_sign(data, key))

    resp = urllib2.urlopen(req)
    output = json.loads(resp.read())
    print resp.info()
    print output
    print output['respcd']
    print output['resperr'].encode("utf-8")

    if output['respcd'] == '0000' and args.type in ['wechat', 'alipay'] and is_pay:
        img = qrcode.make(output['qrcode'])
        img.save('qrcode.png')

if __name__ == '__main__':
    main()
