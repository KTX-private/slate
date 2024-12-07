---
title: Madex API文档(v4)

language_tabs:
  - javascript
  - python
  - csharp

toc_footers:
  - <a href='https://www.madex.com'>开始交易</a>

includes:
  - errors

search: true

code_clipboard: true

---
# 基本信息

## API

**使用API开发应用程序，您可以准确地获取Madex现货市场的行情数据，快速进行自动化交易。API包含众多接口，按功能大致分为以下几组：**

* Market Data Endpoints 用于获取行情数据的REST接口
* User Data Endpoints 用于获取用户私有数据的REST接口
* Market Data Streams 用于获取行情数据的WebSocket接口
* User Data Streams 用于获取用户私有数据的WebSocket接口

**API使用如下Base URL：**

* Market Data Endpoints: https://ma-tapi.tonetou.com/api
* User Data Endpoints: https://api-user.tonetou.com/api
* Market Data Stream: wss://stream-market.tonetou.com
* User Data Stream: wss://madex-user.tonetou.com

**备用api域名列表**

[https://api.madex.tel/v1/public/queryApiDomain](https://api.madex.tel/v1/public/queryApiDomain)

**API的REST接口使用以下HTTP方法：**

* GET 用于获取行情或私有数据
* POST 用于提交委托等操作

**API的REST接口所需的参数应根据以下规则附加于请求中：**

* GET类型的接口参数应附加于Query String中
* POST类型的接口参数应以JSON格式附加于Request Body中

**Response**

API的响应数据都以JSON格式返回，具体格式请参考各接口的描述。

**Error**

API的错误以如下JSON格式返回：

{<br/>
&nbsp;&nbsp;"state": error code,<br/>
&nbsp;&nbsp;"msg": "error message"<br/>
}

*其中，state表示错误的类型，msg包含错误产生的原因或如何避免错误的提示。具体的错误类型请参考[Error](#errors)章节的内容。*

**Time or Timestamp**

API接口参数和响应数据中所涉及的时间值都是UNIX时间，单位为毫秒。

---

## 流量限制

**Madex对来自于同一IP的请求做以下访问限制:**

1. Access Limits 访问频率限制
2. Usage Limits CPU用量限制

* Access Limits访问频率限制
  1. 同一IP每10秒最多10000次请求，超出限制的请求会收到-20007错误。
  2. 用户可以根据需要在10秒内以任意频率发送最多10000次请求，可以大约每10ms发送一次，也可以在1秒内连续发送10000次，然后等待9秒。
     <br/>
     <br/>
* Usage Limits用量限制
  1. 同一IP每10秒最多消耗10000点CPU 时间，超出限制的请求会收到-20006错误。
  2. 不同API消耗的CPU时间不同，这取决于API如何访问数据。
  3. 在本文中, 每个API接口访问数据的方式会以“缓存”, “数据库”的形式标明。访问缓存的API消耗的CPU时间较少，访问数据库的API消耗的CPU时间较多。根据用户发送的参数，API可能混合访问缓存和数据库，甚至多次访问数据库，这会增加API消耗的CPU时间。
  4. 每次API请求消耗的CPU时间会包含在响应头Madex-Usage中，其格式为t1:t2:t3，其中，t1表示本次API请求消耗的CPU时间，t2表示最近10秒内当前IP消耗的CPU时间，t3表示最近10秒内当前IP剩余的可用CPU时间。

---

## Authentication

> 完整例子

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'asset=BTC';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString(); // POST or DELETE  replace queryStr with bodyStr
const url = `${endpoints}/v1/accounts?${queryStr}`;

request.get(url,{
          headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000 // optional
          },
        },

        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }
          console.log(body) // 7.the result

        });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/accounts'
    query_str = 'asset=BTC'
    # POST or DELETE replace query_str with body_str
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

**身份验证**

* 私有接口用于访问账户、委托等私有信息，在请求时需要附加签名，以满足Madex进行身份验证。本节将描述如何创建签名。

**生成Api Key**

* 要创建签名，首先需要生成Api Key和Secret Key组合。请牢记在此过程中生成的Secret Key，因为该值仅显示一次，如果忘记了Secret Key，请删除该Api Key，并生成新的Api Key和Secret Key组合。

**HTTP 请求头**

**访问私有接口的请求都必需附加以下HTTP请求头：**

* api-key 已生成的Api Key
* api-sign 签名

**如果需要，也可以附加以下HTTP请求头：**

* api-expire-time
  1. 接口过期时间。
  2. 该值是以毫秒为单位的Unix时间，服务器会忽略该时间之后收到的请求，这主要用于避免网络延迟带来的影响。

**创建签名**

在发送请求前，首先确定用于签名的消息体。对于GET类型的请求，Query String是需要签名的消息体，对于POST请求，Body String是需要签名的消息体。签名的具体方法如下：

* 第一步：以Secret Key作为Key对需要签名的消息体执行HmacSHA256算法
* 第二步：将以上结果转化为Hex String
* 第三步：将Hex String作为请求头api-sign的值

## Api Key权限

**私有接口需要特定的权限才能执行。可以为Api Key授予适当的权限。如果Api Key未被授予某个接口需要的权限，那么使用该Api Key提交的请求将被拒绝。**

**可以授予Api Key以下权限：**

* View权限允许Api Key获取私有数据。
* Trade权限允许Api Key提交或撤销委托，并允许Api Key获取交易相关的数据。

*接口需要的权限将在每个接口的描述中给出。*

---

# Market Data Endpoints



## Get Listed Pairs

> Request

```javascript
let request = require("request");
const endPoint = 'https://ma-tapi.tonetou.com/api';
const url = `${endPoint}/v1/products?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://ma-tapi.tonetou.com/api';

def do_request():
    path = '/v1/products?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "id": 5 // id
    "market": "lpc", // spot or lpc 现货或者合约
    "symbol": "BTC_USDT_SWAP", // 交易对
    "takerFee": "0.001", // taker手续费
    "makerFee": "0.001", // maker 手续费
    "minOrderSize": "0.0001", // 最小下单数量
    "maxOrderSize": "10000000",// 最大下单数量
    "quantityScale": 4, // 数量精度
    "priceScale": 4, // 价格精度
    "minOrderValue": "0.0001", // 最小下单价值
    "maxOrderValue": "10000000000", // 最大下单价值
    "fundingRate": "0.0001" // 资金费率
    "nextFundingTime": "1733472000000", // 资金费率结算时间
    "predictedFundingRate": "0.0001", // 预期资金费率
    "markPrice": "98042.3405", // 标记价格
  }
]
```

**获取币种列表**

* 请求方式 GET
* 请求路径 /v1/products
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                      |
|--------| ---------- |------|-----------------------------------------|
| market | string   | 是    | 交易对市场，如 spot, lpc 等，spot为现货,lpc为U本位合约   |
| symbol | string   | 否    | 交易对代码，如 BTC_USDT, ETH_USDT 等，不指定返回全部交易对 |

* Data source

  Cache

## Get Order Book

> Request

```javascript
let request = require("request");
const endPoint = 'https://ma-tapi.tonetou.com/api';
const url = `${endPoint}/v1/order_book?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://ma-tapi.tonetou.com/api';

def do_request():
    path = '/v1/order_book?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": "1027024", // update id
  "t": "1644558642100", // update time
  "b": [ // 买盘
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ]
    ...
  ],
  "a": [ // 卖盘
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ]
    ...
  ]
}
```

**获取深度数据**

* 请求方式 GET
* 请求路径 /v1/order_book
* 请求参数


| 参数名称    | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| market | string   | 是    | 交易对市场，如 spot, lpc 等，spot为现货,lpc为U本位合约   |
| symbol      | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT 等                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| level       | int32    | 否       | 指定最多返回多少级深度<br/>有效值 1, 2, 5, 10, 20, 50, 100, 200, 500, 1000<br/>默认值 100                                                                                                                                                                                                                                                                                                                                                                                                 |
| price_scale | integer  | 否       | 指定按价格合并深度，如指定交易对的价格最多含4位小数<br/>price_scale=0 时返回的价格最多含4位小数,<br/>price_scale=1 时返回的价格最多含3位小数, 委托量是价格区间0.0010中全部委托量的和<br/>price_scale=2 时返回的价格最多含2位小数, 委托量是价格区间0.0100中全部委托量的和<br/>price_scale=3 时返回的价格最多含1位小数, 委托量是价格区间0.1000中全部委托量的和<br/>price_scale=4 时返回的价格最多含0位小数, 委托量是价格区间1.0000中全部委托量的和<br/>有效值 0, 1, 2, 3, 4, 5<br/>默认值 0 |

> 注意: 数据按价格最优排序, 即买侧深度按价格由大到小排序, 卖侧深度按价格由小到大排序

* Data source

  Cache

## Get Candles

> Request

```javascript
let request = require("request");
const endPoint = 'https://ma-tapi.tonetou.com/api';
const url = `${endPoint}/v1/candles?market=spot&symbol=BTC_USDT&time_frame=1m`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://ma-tapi.tonetou.com/api';

def do_request():
    path = '/v1/candles?market=spot&symbol=BTC_USDT&time_frame=1m'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "t": 60000, // 时间周期
  "e": [
    [
      "1644224940000", // 起始时间
      "10190.53", // 开盘价格
      "10192.5", // 最高价格
      "9806.82", // 最低价格
      "10127.37", // 收盘价格
      "0.834", // 成交量
      "8370.40506", // 成交价值
      "1", // 首个成交的id
      278 // 区间内总成交次数
    ],
    [
      "1644224940000",
      "10190.53",
      "10192.5",
      "9806.82",
      "10127.37",
      "0.834",
      "8370.40506",
      "1",
      278
    ],
    ...
  ]
}
```

**获取K线数据**

* 请求方式 GET
* 请求路径 /v1/candles
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                   |
| ------------ | ---------- | ---------- | ---------------------------------------------------------------------------------------- |
| market | string   | 是    | 交易对市场，如 spot, lpc 等，spot为现货,lpc为U本位合约   |
| symbol     | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT 等                                                   |
| time_frame | string   | 是       | K线数据的时间周期<br/>有效值 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d, 3d, 1W或1M |
| before     | int64    | 否       | utc时间<br/>限定返回K线记录的最近时间                                                  |
| after      | int64    | 否       | utc时间<br/>限定返回K线记录的最早时间                                                  |
| limit      | integer  | 否       | 获取K线记录的最大数量<br/>默认值100，最大值1000                                        |

* 该接口支持的参数组合和数据源

  1. symbol + time_frame  --> cache
  2. symbol + time_frame + limit  --> cache
  3. symbol + time_frame + before  --> database
  4. symbol + time_frame + before + limit  --> database
  5. symbol + time_frame + after  --> database
  6. symbol + time_frame + after + limit  --> database

> 返回结果按时间由早及近排序

## Get Trades

> Request

```javascript
let request = require("request");
const endPoint = 'https://ma-tapi.tonetou.com/api';
const url = `${endPoint}/v1/trades?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://ma-tapi.tonetou.com/api';

def do_request():
    path = '/v1/trades?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "i": "17122255", // 交易 id
    "p": "46125.7", // 成交价格
    "q": "0.079045", // 成交量
    "s": "1", // Taker 的成交方向 1代表买 -1代表卖
    "t": "1628738748319" // 成交时间
  },
  {
    "i": "17122255", // 交易 id
    "p": "46125.7", // 成交价格
    "q": "0.079045", // 成交量
    "s": "-1", // Taker 的成交方向 1代表买 -1代表卖
    "t": "1628738748319" // 成交时间
  },
  {
    "i": "17122255", // 交易 id
    "p": "46125.7", // 成交价格
    "q": "0.079045", // 成交量
    "s": "1", // Taker 的成交方向 1代表买 -1代表卖
    "t": "1628738748319" // 成交时间
  }
  ...
]
```

**获取交易记录**

* 请求方式 GET
* 请求路径 /v1/trades
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                         |
| ------------ | ---------- | ---------- | ---------------------------------------------- |
| market | string   | 是    | 交易对市场，如 spot, lpc 等，spot为现货,lpc为U本位合约   |
| symbol     | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT 等         |
| start_time | int64    | 否       | 限定返回交易记录的最早时间                   |
| end_time   | int64    | 否       | 限定返回交易记录的最近时间                   |
| before     | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最大id     |
| after      | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最大id     |
| limit      | integer  | 否       | 获取记录的最大数量<br/>默认值100，最大值1000 |

* 该接口支持的参数组合和数据源

  1. symbol  --> cache
  2. symbol + limit  --> cache
  3. symbol + start_time  --> database
  4. symbol + start_time + limit  --> database
  5. symbol + end_time  --> database
  6. symbol + end_time + limit  --> database
  7. symbol + start_time + end_time  --> database
  8. symbol + start_time + end_time + limit  --> database
  9. symbol + before  --> database
  10. symbol + before + limit  --> database
  11. symbol + after  --> database
  12. symbol + after + limit  --> database

  *数据源为cache的参数组合用于获取最近1000条交易记录*

  *数据源为database的参数组合用于获取较早的交易记录*

  *如果用数据源为database的参数组合获取最新交易记录，其结果要比cache数据源稍有延迟*
* Usage
  **用法举例：获取三个月内某交易对的全部交易记录**

  1. 首先使用symbol + limit 参数组合获取最新的交易记录
  2. 将取到的首条记录的tradeId作为before参数的值，反复使用symbol + before + limit参数组合获取更多记录，直至获取三个月内的全部交易记录后停止

> 返回结果按交易记录id由小到大排序

## Get Tickers

> Request

```javascript
let request = require("request");
const endPoint = 'https://ma-tapi.tonetou.com/api';
const url = `${endPoint}/v1/ticker?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://ma-tapi.tonetou.com/api';

def do_request():
    path = '/v1/ticker?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "askPrice": "98100", // 卖一价
    "product": "BTC_USDT", // 交易对
    "amount": "922635", // 24成交价值
    "last": "98000", // 最新成交价
    "firstTradeId": 1, // 第一笔交易id
    "change": "0", // 价格变化
    "bidQty": "1.7", // 卖一数量
    "bidPrice": "98000", // 买一价
    "volume": "9.41", // 24h成交数量
    "lastQty": "0.3", // 24h最新成交
    "askQty": "0.5", // 卖一数量
    "high": "98100", // 24最高价
    "tradeCount": 30, // 成交次数
    "low": "98000", // 24h最低价
    "time": "1733474204000", // 时间
    "open": "98000" // 开盘价格
  }
]
```

**获取报价数据**

* 请求方式 GET
* 请求路径 /v1/ticker
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                                 |
| ---------- | ---------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| market | string   | 是    | 交易对市场，如 spot, lpc 等，spot为现货,lpc为U本位合约   |
| symbol   | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT 等，<br/>可按如下两种形式指定多个交易对代码<br/> 1. /pairs?symbol=BTC_USDT,ETH_USDT<br/> 2. /pairs?symbol=BTC_USDT&symbol=ETH_USDT |

* Data Source

Cache

# Market Data Streams

## Overview

> 例子

```javascript
const WebSocket = require('ws');
const madexws = 'wss://stream-market.tonetou.com';

let wsClass = function () {
};

wsClass.prototype._initWs = async function () {
    let that = this;
    console.log(madexws)
    let ws = new WebSocket(madexws);
    that.ws = ws;

    ws.on('open', function open() {
        console.log(new Date(), 'open')
        ws.send(JSON.stringify({"method":"SUBSCRIBE","params":["spot.BTC_USDT.order_book.5"]}));
        setInterval(function () {
          ws.ping(Date.now())
        },30000)
    });

    ws.on('close', data => {
        console.log('close, ', data);
    });

    ws.on('error', data => {
        console.log('error', data);
    });

    ws.on('ping', data => {
        console.log('ping ', data.toString('utf8'));
    });

    ws.on('pong', data => {
        console.log('pong ', data.toString('utf8'));
    });

    ws.on('message', data => {
        console.log("rece message")
        console.log(data)
    });
};

let instance = new wsClass();

instance._initWs().catch(err => {
    console.log(err);
});

```

```python
import websocket
import json

ws_url = 'wss://stream-market.tonetou.com'

def stringify(obj):
    return json.dumps(obj, sort_keys=True).replace("\'", "\"").replace(" ", "")


def get_sub_str():
    subdata = {"method":"SUBSCRIBE","params":["spot.BTC_USDT.order_book.5"]} 
    return stringify(subdata)


def on_message(ws, message):
    print(message)


def on_error(ws, error):
    print(error)


def on_close(ws):
    print("### closed ###")


def on_open(ws):
    ws.send(get_sub_str())


def connect():
    # websocket.enableTrace(True)
    ws = websocket.WebSocketApp(ws_url,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever(ping_interval=30, ping_timeout=5)


if __name__ == "__main__":
    connect()

```

**使用 Websocket 推送服务可以及时获取行情信息。**

* 连接 Websocket 服务器
  请使用以下 URL 连接 Websocket 服务器：
  <br/>
  wss://stream-market.tonetou.com

> 在连接后，客户端可以发送以下JSON格式的请求给服务器

```json
{
  "id": 123, // 由客户端给定的请求id
  "method":"SUBSCRIBE", // 请求类型
  "params":["spot.BTC_USDT.order_book.5"]
}
```

> 在收到请求后，服务器会发送以下JSON格式的响应给客户端

```json
{
  "result":"success", // 返回结果
  "op":"SUBSCRIBE", 
  "id": 123, // 由客户端给定的请求id
  "events":["spot.BTC_USDT.order_book.5"]
}
```

> 如果发生错误，服务器会发送以下错误信息给客户端

```json
{
  "id": 123, // 请求id
  "error": -1003, // 错误代码
  "message": "..." // 错误描述
}
```

> 同时，服务器还会发送以下JSON格式的数据流给客户端, 数据流包含市场行情的变化信息

```json
{
  "stream": "spot.BTC_USDT.order_book.5", // 数据流名称
  "data": ..., // 数据
}
```

> 请求：订阅数据流

```json
{
  "id": 1,
  "method": "SUBSCRIBE",
  "params": [
    "stream name",
    "stream name",
    ...
  ]
}
```

> 在连接后，请首先发送该请求给服务器，随后，服务器会在行情变化时发送相应的数据流给客户端。

> "data stream name" 是数据流名称，数据流名称是以下格式的字符串。
> market.symbol.data_type.param1.param2...

> 其中，market是交易对市场，如spot和lpc
> symbol是交易对名称，如 BTC_USDT、ETH_USDT 等。
> data_type是数据类型，目前仅支持以下数据类型
> order_book: 深度
> trades: 交易列表
> candles: K线
> ticker: 最新成交信息

> 在data_type之后是参数列表，不同的数据类型有不同的参数列表，这些将在后续介绍

> 请求: 取消订阅数据流

```javascript
{
  "id": 1,
  "method": "UNSUBSCRIBE",
  "params": [
    "data stream name",
    "data stream name",
    ...
  ]
}
```

> 如果请求被服务器正确处理，客户端会收到以下响应:

```javascript
  {
    "result":"success", // 返回结果
    "op":"SUBSCRIBE",
    ... 
    }
```

> 如果请求出错，客户端会收到以下错误响应:

```javascript
{
  "error": -1003, // 错误代码
  "message": "..." // 错误描述
}
```

## Request Methods

> 请求类型及参数

> 客户端可以发送以下请求给服务器

```javascript
{
  "id": 123, // 由客户端给定的请求id
  "method": "..." // 请求类型
  "params": [ // 请求参数列表
    "...",
    "...",
  ]
}
```

* 其中method字段的值是以下请求类型之一:


| 可选值      | 说明                                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------- |
| SUBSCRIBE   | 1.订阅数据流<br/> 2.参数是数据流名称列表 <br/> 3.在成功订阅后，服务器会在行情发生变化时发送数据流给客户端 |
| UNSUBSCRIBE | 1.取消订阅数据流<br/> 2.参数是数据流名称列表 <br/> 3.在成功取消订阅后，客户端不会再收到相应的数据流       |

## Subscribe Order Book

**订阅深度信息**

> 发送以下请求可订阅深度信息

```javascript
{
  "id": 123,
  "method": "SUBSCRIBE",
  "params": [
    "spot.BTC_USDT.order_book.20",
    "spot.ETH_USDT.order_book.20",
    ...
  ]
}
```

* 参数

  1. 该请求的参数是深度流名称，格式如下：

  * \<market><symbol>.order_book.\<max depth>
    1. \<market> 是交易对市场，如spot，lpc
    2. \<symbol> 是交易对名称，如BTC_USDT，ETH_USDT等
    3. \<max depth> 是最大深度，有效值是5, 10, 20, 50, 100, 200, 500, 1000

> 数据流

```javascript
  {
    "stream": "spot.BTC_USDT.order_book.20",
    "data": {
      "i": "1027024", // update id
      "t": "1644558642100", // update time
      "b": [ // 买盘
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
        ...
      ],
      "a": [ // 卖盘
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
        [
            "46125.7", // 委托价格
            "0.079045" // 委托量
          ],
       ...
      ]
    }
  }
```

  > 在成功订阅后，客户端会首先收到一个完整深度的数据流，之后会收到增量变化数据流，请按以下方法合成完整的深度，或使用SDK.
  >

  ```javascript
  ...
  ```

## Subscribe Trades

**订阅交易列表**

> 发送以下请求可订阅交易列表

```javascript
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.trades",
    "spot.ETH_USDT.trades",
    ...
  ]
}
```

* 参数

  1. 该请求的参数是交易流名称，格式如下：

  * \<symbol>.trades
    1. \<market> 是交易对市场，如spot，lpc
    2. \<symbol> 是交易对名称，如BTC_USDT，ETH_USDT等

> 数据流

```javascript
{
  "stream": "spot.BTC_USDT.trades",
  "data": [
    {
      "i": "17122255", // 交易 id
      "p": "46125.7", // 成交价格
      "q": "0.079045", // 成交量
      "s": "buy", // Taker 的成交方向
      "t": "1628738748319" // 成交时间
    },
    {
      "i": "17122255", // 交易 id
      "p": "46125.7", // 成交价格
      "q": "0.079045", // 成交量
      "s": "buy", // Taker 的成交方向
      "t": "1628738748319" // 成交时间
    },
    {
      "i": "17122255", // 交易 id
      "p": "46125.7", // 成交价格
      "q": "0.079045", // 成交量
      "s": "buy", // Taker 的成交方向
      "t": "1628738748319" // 成交时间
    },
    ...
  ]
}
```

## Subscribe Candles

**订阅K线**

> 发送以下请求可订阅K线

```javascript
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.candles.1m",
    "spot.ETH_USDT.candles.1h",
    ...
  ]
}
```

* 参数

  1. K线流名称格式如下：

  * \<symbol>.candles.\<time_frame>
    1. \<market> 是交易对市场，如spot，lpc
    2. \<symbol> 是交易对名称，如BTC_USDT，ETH_USDT等
    3. \<time_frame> 是K线周期，有效值是1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d, 1w, 1M
> 数据流

```javascript
{
  "stream": "spot.BTC_USDT.candles.1m",
  "data": {
      "t":60000, // 时间周期
      "e":[
      [
        "1644224940000", // 起始时间
        "10190.53", // 开盘价格
        "10192.5", // 最高价格
        "9806.82", // 最低价格
        "10127.37", // 收盘价格
        "0.834", // 成交量
        "8370.40506", // 成交价值
        "1", // 首个成交的id
        278 // 区间内总成交次数
      ],
      [
        "1644224940000", // 起始时间
        "10190.53", // 开盘价格
        "10192.5", // 最高价格
        "9806.82", // 最低价格
        "10127.37", // 收盘价格
        "0.834", // 成交量
        "8370.40506", // 成交价值
        "1", // 首个成交的id
        278 // 区间内总成交次数
      ],
      [
        "1644224940000", // 起始时间
        "10190.53", // 开盘价格
        "10192.5", // 最高价格
        "9806.82", // 最低价格
        "10127.37", // 收盘价格
        "0.834", // 成交量
        "8370.40506", // 成交价值
        "1", // 首个成交的id
        278 // 区间内总成交次数
      ],
      ...
    ]}
  }
}
```

## Subscribe Tickers

**订阅K线**

> 发送以下请求可订阅Ticker

```javascript
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.ticker",
    "spot.ETH_USDT.ticker",
    ...
  ]
}
```

* 参数

  1. Ticker流名称格式如下：

  * \<symbol>.ticker
    1. \<market> 是交易对市场，如spot，lpc
    2. \<symbol> 是交易对名称，如BTC_USDT，ETH_USDT等
> 数据流

```javascript
{
  "stream": "spot.BTC_USDT.ticker",
  "data": {   
    "askPrice": "98100", // 卖一价
    "product": "BTC_USDT", // 交易对
    "amount": "922635", // 24成交价值
    "last": "98000", // 最新成交价
    "firstTradeId": 1, // 第一笔交易id
    "change": "0", // 价格变化
    "bidQty": "1.7", // 卖一数量
    "bidPrice": "98000", // 买一价
    "volume": "9.41", // 24h成交数量
    "lastQty": "0.3", // 24h最新成交
    "askQty": "0.5", // 卖一数量
    "high": "98100", // 24最高价
    "tradeCount": 30, // 成交次数
    "low": "98000", // 24h最低价
    "time": "1733474204000", // 时间
    "open": "98000" // 开盘价格
  }
}
```

---

# User Data Endpoints



## Get Accounts

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'asset=BTC';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString();
const url = `${endpoints}/v1/accounts?${queryStr}`;

request.get(url,{
          headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000 // optional
          },
        },

        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }
          console.log(body) // 7.the result

        });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/accounts'
    query_str = 'asset=BTC'
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "asset":"USDT",  // 资产代码
    "balance":10000,  // 总额
    "holds":0  // 冻结额
  },
  {
    "asset":"USDT",  // 资产代码
    "balance":10000,  // 总额
    "holds":0  // 冻结额
  },
  ...
]
```

**获取 API Key 对应账户中各种资产的余额, 冻结等信息**

* 请求方式 GET
* 请求路径 /v1/accounts
* 权限: View, Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                 |
| ---------- | ---------- | ---------- |----------------------------------------------------------------------------------------------------------------------------------------------------|
| asset    | string   | 否       | 资产代码，如 BTC, ETH 等<br/>可按以下两种形式指定多个资产代码<br/>1. /v1/accounts?asset=BTC,ETH<br/> 2. /v1/accounts?asset=BTC&asset=ETH <br/> 如果不指定 asset 参数, 则返回全部资产的信息 |

* Data Source

  Cache

## Get an account's ledger

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'asset=BTC&end_time=1651895799668&limit=10';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString();
const url = `${endpoints}/v1/ledger?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000 // optional
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/ledger'
    query_str = 'asset=BTC&end_time=1651895799668&limit=10'
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "amount":"10000",   // 变化数量
    "balance":"10000",  // 余额
    "id":"1125899906842624029", // id
    "time":"1733468814795", // 时间
    "asset":"USDT",   // 资产代码
    "type":"transfer" // 账单类型
  }
  ...
]
```

**获取 API Key 对应账户的账单，包含一切改变账户余额的记录，如资金划转、交易、手续费收取等**

* 请求方式 GET
* 请求路径 /v1/ledger
* 权限: View, Trade
* 请求参数(需要排序)


| 参数名称       | 参数类型 | 是否必传 | 说明                                                                                                                                               |
|------------| ---------- | ---------- |--------------------------------------------------------------------------------------------------------------------------------------------------|
| asset      | string   | 否       | 资产代码，如 BTC, ETH 等<br/>可按以下两种形式指定多个资产代码<br/>1. /v1/ledger?asset=BTC,ETH<br/> 2. /v1/ledger?asset=BTC&asset=ETH <br/> 如果不指定 asset 参数, 则返回全部资产的账单记录 |
| start_time | int64    | 否       | 限定返回账单记录的最早时间                                                                                                                                    |
| end_time   | int64    | 否       | 限定返回账单记录的最近时间                                                                                                                                    |
| before     | int64    | 否       | 账单记录id<br/>限定返回账单记录的最大id值                                                                                                                        |
| after      | int64    | 否       | 账单记录id<br/>限定返回账单记录的最小id值                                                                                                                        |
| limit      | int32    | 否       | 限定返回账单记录的最大条数<br/>默认值 100                                                                                                                        |
| type       | string   | 否       | 账单类型 transfer划转，trade交易，fee手续费，rebate系统收取，funding资金费用                                                                                                                                            |

* Data Source

  DB

## Create an order

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    symbol:'BTC_USDT',
    side:'sell',
    type:'limit',
    quantity:'0.0001',
    price:'50000'
}

let bodyStr = JSON.stringify(param);
const sign = CryptoJS.HmacSHA256(bodyStr, secret).toString();
const url = `${endpoints}/v1/order`;

request.post({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000  // optional
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });

```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'
t = time.time()


def do_request():

    param = {
        'symbol': 'BTC_USDT',
        'side': 'sell',
        'type': 'limit',
        'quantity': '0.0001',
        'price': '50000'
    }
    body_str = json.dumps(param)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), body_str.encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/order'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':str(round(t * 1000 +5000)) # optional
    }
    resp = requests.post(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": 4611688217450643477,  // 交易所分配的委托id
  "I": "",  // 用户指定的委托id
  "m": "BTC_USDT",  // 交易对代码
  "T": "limit",  // 委托类型
  "s": "sell",  // 委托方向
  "Q": -0.0100,  // 委托量
  "P": 10043.8500,  // 委托价格
  "t": "gtc",  // Time In Force
  "o": false,  // Post Only
  "S": "filled",  // 委托状态
  "E": -0.0100,  // 已成交量
  "e": -100.43850000,  // 已成交价值
  "C": 1643193746043,  // 创建时间
  "U": 1643193746464,  // 更新时间
  "n": 2,  // 成交笔数
  "F": [{
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  }, {
    "i": 12,
    "t": 1643193746266,
    "p": 10043.85,
    "q": -0.001,
    "l": "maker",
    "f": {
      "a": "USDT",
      "m": 0.01004385000
    }
  }],
  "f": [{
    "a": "USDT",  // 用于支付手续费的资产
    "m": 0.10043850000  // 手续费总额
  }]
}
```

**提交委托**

* 请求方式 POST
* 请求路径 /v1/order
* 权限: Trade
* 请求参数


| 参数名称        | 参数类型 | 是否必传 | 说明                                                                                                                                                                                     |
| ----------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| symbol          | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT 等                                                                                                                                                     |
| side            | string   | 是       | 委托方向，有效值 buy, sell，不区分大小写                                                                                                                                                 |
| type            | string   | 是       | 委托类型，有效值 limit                                                                                                                                                                   |
| client_order_id | string   | 否       | 委托id，有效值为int64整数的字符串，建议使用提交委托时的Unix时间戳                                                                                                                        |
| quantity        | decimal  | 否       | 委托量                                                                                                                                                                                   |
| price           | decimal  | 否       | 委托限价                                                                                                                                                                                 |
| time_in_force   | string   | 否       | 委托时效性<br/>有效值 gtc, ioc<br/>gtc 表示未完全成交的委托将一直有效, 直到用户撤销该委托<br/>ioc 表示撮合将立即撤销在下单时刻不能完全成交的委托,<br/> 任何成交都将被保留<br/>默认值 gtc |
| post_only       | bool     | 否       | ...                                                                                                                                                                                      |

> 委托对象
> 最多包含该委托的20笔成交
> 如果委托有多于20笔成交，那么该对象仅包含最后20笔，其他成交请通过 fills 接口获取

## Get an order

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'id=1118621391231';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString();
const url = `${endpoints}/v1/order?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/order'
    query_str = 'id=14118828812271651'
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": 4611688217450643477,  // 交易所分配的委托id
  "I": "",  // 用户指定的委托id
  "m": "BTC_USDT",  // 交易对代码
  "T": "limit",  // 委托类型
  "s": "sell",  // 委托方向
  "Q": -0.0100,  // 委托量
  "P": 10043.8500,  // 委托价格
  "t": "gtc",  // Time In Force
  "o": false,  // Post Only
  "S": "filled",  // 委托状态
  "E": -0.0100,  // 已成交量
  "e": -100.43850000,  // 已成交价值
  "C": 1643193746043,  // 创建时间
  "U": 1643193746464,  // 更新时间
  "n": 2,  // 成交笔数
  "F": [{
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  }, {
    "i": 12,
    "t": 1643193746266,
    "p": 10043.85,
    "q": -0.001,
    "l": "maker",
    "f": {
      "a": "USDT",
      "m": 0.01004385000
    }
  }],
  "f": [{
    "a": "USDT",  // 用于支付手续费的资产
    "m": 0.10043850000  // 手续费总额
  }]
}
```

**获取指定 id 的委托**

* 请求方式 GET
* 请求路径 /v1/order
* 权限: View, Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                |
| ---------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id       | string   | 是       | 委托id<br/>委托id可以是交易所分配的, <br/>也可以是用户自定义的 (在提交委托时使用client_order_id参数).<br/>当使用自定义id时, 需要在id前添加 “c:” 前缀.<br/>例如: 提交委托时使用了自定义id “123”, 在获取委托时, 需使用 “c:123”. |

## Get Orders

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'limit=2&status=settled&symbol=BTC_USDT';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString();
const url = `${endpoints}/v1/orders?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/orders'
    query_str = 'limit=2&status=settled&symbol=BTC_USDT'
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```javascript
[
  {
    "i": 4611688217450643477,  // 交易所分配的委托id
    "I": "",  // 用户指定的委托id
    "m": "BTC_USDT",  // 交易对代码
    "T": "limit",  // 委托类型
    "s": "sell",  // 委托方向
    "Q": -0.0100,  // 委托量
    "P": 10043.8500,  // 委托价格
    "t": "gtc",  // Time In Force
    "o": false,  // Post Only
    "S": "filled",  // 委托状态
    "E": -0.0100,  // 已成交量
    "e": -100.43850000,  // 已成交价值
    "C": 1643193746043,  // 创建时间
    "U": 1643193746464,  // 更新时间
    "n": 2,  // 成交笔数
    "F": [{
      "i": 13,  // 成交id
      "t": 1643193746464,  // 成交时间
      "p": 10043.85,  // 成交价格
      "q": -0.009,  // 成交量
      "l": "maker",  // Maker / Taker 成交
      "f": {
        "a": "USDT",  // 该笔成交用于支付手续费的资产
        "m": 0.09039465000  // 该笔成交的手续费额
      }
    }, {
      "i": 12,
      "t": 1643193746266,
      "p": 10043.85,
      "q": -0.001,
      "l": "maker",
      "f": {
        "a": "USDT",
        "m": 0.01004385000
      }
    }],
    "f": [{
      "a": "USDT",  // 用于支付手续费的资产
      "m": 0.10043850000  // 手续费总额
    }]
  }
  ...
]
```

**获取ApiKey对应账户中符合下列条件的委托**

1. 全部未结算委托
2. 三个月内的已结算委托, 含已拒绝, 已撤销和已成交委托
3. 全部已成交委托
4. 全部已撤销的部分成交委托

* 请求方式 GET
* 请求路径 /v1/orders
* 权限: View, Trade
* 请求参数(需要排序)


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                                                                                             |
| ------------ | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| status     | string   | 否       | 有效值 unsettled, settled<br/>unsettled 表示获取未结算委托，返回结果按委托创建时间倒序排序<br/>settled 表示获取已结算委托，返回结果按委托结算时间倒序排序<br/>默认值 unsettled                                                                                                                                   |
| symbol     | string   | 否       | 交易对代码，如 BTC_USDT, ETH_USDT 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |
| start_time | long     | 否       | 限定返回委托的最近创建时间                                                                                                                                                                                                                                                                                       |
| end_time   | long     | 否       | 限定返回委托的最近创建时间                                                                                                                                                                                                                                                                                       |
| before     | int64    | 否       | 委托更新 id<br/>限定返回委托的最大更新id                                                                                                                                                                                                                                                                         |
| after      | int64    | 否       | 委托更新 id<br/>限定返回委托的最小更新id                                                                                                                                                                                                                                                                         |
| limit      | long     | 否       | 指定最多返回多少个委托                                                                                                                                                                                                                                                                                           |

* 该接口支持的参数组合和数据源

  * status=unsettled + symbol
  * status=settled + symbol + start_time
  * status=settled + symbol + start_time + limit
  * status=settled + symbol + end_time
  * status=settled + symbol + end_time + limit
  * status=settled + symbol + start_time + end_time
  * status=settled + symbol + start_time + end_time + limit
  * status=settled + symbol + before
  * status=settled + symbol + before + limit
  * status=settled + symbol + after
  * status=settled + symbol + after + limit

> 返回的unsettled委托按创建时间由早及近排序
> 返回的settled委托按结算时间由早及近排序

## Cancel an Order

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    id:'14244173146202090'
}

let bodyStr = JSON.stringify(param);
const sign = CryptoJS.HmacSHA256(bodyStr, secret).toString();
const url = `${endpoints}/v1/order`;

request.delete({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000 // optional
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'
t = time.time()

def do_request():

    param = {
        'id': '14245272657638034'
    }
    body_str = json.dumps(param)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), body_str.encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/order'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':str(round(t * 1000 +5000)) #optional
    }
    resp = requests.delete(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": 4611688217450643477,  // 交易所分配的委托id
  "I": "",  // 用户指定的委托id
  "m": "BTC_USDT",  // 交易对代码
  "T": "limit",  // 委托类型
  "s": "sell",  // 委托方向
  "Q": -0.0100,  // 委托量
  "P": 10043.8500,  // 委托价格
  "t": "gtc",  // Time In Force
  "o": false,  // Post Only
  "S": "filled",  // 委托状态
  "E": -0.0100,  // 已成交量
  "e": -100.43850000,  // 已成交价值
  "C": 1643193746043,  // 创建时间
  "U": 1643193746464,  // 更新时间
  "n": 2,  // 成交笔数
  "F": [{
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  }, {
    "i": 12,
    "t": 1643193746266,
    "p": 10043.85,
    "q": -0.001,
    "l": "maker",
    "f": {
      "a": "USDT",
      "m": 0.01004385000
    }
  }],
  "f": [{
    "a": "USDT",  // 用于支付手续费的资产
    "m": 0.10043850000  // 手续费总额
  }]
}
```

**撤销指定 id 的委托**

* 请求方式 DELETE
* 请求路径 /v1/order
* 权限: Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                  |
| ---------- | ---------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id       | string   | 是       | 委托id<br>委托id可以是交易所分配的，<br/>也可以是用户自定义的（在提交委托时使用client_order_id参数）。<br>当使用自定义id时，需要在id前添加 “c:” 前缀。<br/>例如：提交委托时使用了自定义id “123”, 在撤销委托时，需使用 “c:123”。 |

> 如果指定id的委托已结算，或者不存在指定id的委托，会收到-3004错误。

## Cancel all Orders

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    symbol:'BTC_USDT'
}

let bodyStr = JSON.stringify(param);
const sign = CryptoJS.HmacSHA256(bodyStr, secret).toString();
const url = `${endpoints}/v1/orders`;

request.delete({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':Date.now()+5000 // optional
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'
t = time.time()

def do_request():

    param = {
        'symbol': 'BTC_USDT'
    }
    body_str = json.dumps(param)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), body_str.encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/orders'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':str(round(t * 1000 +5000)) # optional

    }
    resp = requests.delete(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[]
```

**撤销全部委托**

* 请求方式 DELETE
* 请求路径 /v1/orders
* 权限: Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                    |
| ---------- | ---------- | ---------- | ----------------------------------------- |
| symbol   | string   | 是       | 交易对代码<br/>如 BTC_USDT, ETH_USDT 等 |

> 如果请求被正确执行，返回空数组，否则返回错误信息

## Get fills

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api-user.tonetou.com/api'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'limit=2&symbol=BTC_USDT';
const sign = CryptoJS.HmacSHA256(queryStr, secret).toString();
const url = `${endpoints}/v1/fills?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests

END_POINT = 'https://api-user.tonetou.com/api'
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/fills'
    query_str = 'limit=2&symbol=BTC_USDT'
    # POST or DELETE replace query_str with body_str
    sign = hmac.new(SECRET_KEY.encode("utf-8"), query_str.encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  },
    {
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  },
    {
    "i": 13,  // 成交id
    "t": 1643193746464,  // 成交时间
    "p": 10043.85,  // 成交价格
    "q": -0.009,  // 成交量
    "l": "maker",  // Maker / Taker 成交
    "f": {
      "a": "USDT",  // 该笔成交用于支付手续费的资产
      "m": 0.09039465000  // 该笔成交的手续费额
    }
  },
  ...
]
```

**获取成交记录**

* 请求方式 GET
* 请求路径 /v1/fills
* 权限: View, Trade
* 请求参数(需要排序)


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                             |
| ------------ | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------ |
| order_id   | string   | 否       | 交易所分配的委托id<br/>限定仅返回指定委托的成交记录<br/>如果不指定该参数，请指定 symbol                          |
| symbol     | string   | 否       | 交易对代码<br/>如 BTC_USDT, ETH_USDT 等<br/>限定仅返回指定交易对的成交记录<br/>如果不指定该参数，请指定 order_id |
| start_time | int64    | 否       | 限定返回成交记录的最早时间                                                                                       |
| end_time   | int64    | 否       | 限定返回成交记录的最近时间                                                                                       |
| before     | int64    | 否       | 成交记录 id<br/>限定返回成交记录的最大id                                                                         |
| after      | int64    | 否       | 成交记录 id<br/>限定返回成交记录的最小id                                                                         |
| limit      | int32    | 否       | 限定返回结果的最大条数<br/>默认值 100                                                                            |

* 该接口支持的参数组合和数据源

  * symbol  --> database
  * symbol + limit  --> database
  * symbol + start_time  --> database
  * symbol + start_time + limit  --> database
  * symbol + end_time  --> database
  * symbol + end_time + limit  --> database
  * symbol + start_time + end_time  --> database
  * symbol + start_time + end_time + limit  --> database
  * symbol + before  --> database
  * symbol + before + limit  --> database
  * symbol + after  --> database
  * symbol + after + limit  --> database
  * order_id  --> database
  * order_id + limit  --> database
  * order_id + before  --> database
  * order_id + before + limit  --> database

> 返回结果按成交记录id由小到大排序

# User Data Streams

## Overview

> 例子

```javascript
const CryptoJS = require("crypto-js");
const WebSocket = require('ws');
const madexws = 'wss://user-wss.madex360.com';
const apikey = "9e2bd17ff73e8531c0f3c26f93e48bfa402a3b13"; // your apikey
const secret = "ca55beb9e45d4f30b3959b464402319b9e12bac7"; // your secret
const sign = CryptoJS.HmacSHA256("", secret).toString();

let wsClass = function () {
};


wsClass.prototype._initWs = async function () {
  let that = this;
  console.log(madexws);

  let ws = new WebSocket(madexws,"",{headers:{
      'BIBOX-API-KEY': apikey,
      'BIBOX-API-SIGN': sign,
    }});
  that.ws = ws;

  ws.on('open', function open() {
    console.log(new Date(), 'open')
    setInterval(function () {
      ws.ping(Date.now())
    },30000)
  });

  ws.on('close', data => {
    console.log('close, ', data);
  });

  ws.on('error',  data => {
    console.log('error ',data);
  });

  ws.on('ping', data => {
    console.log('ping ', data.toString('utf8'));
  });

  ws.on('pong', data => {
    console.log('pong ', data.toString('utf8'));
  });

  ws.on('message', data => {
    console.log(data.toString()) // the data may be is error message,check the data's stream is order or account
  });
};

let instance = new wsClass();

instance._initWs().catch(err => {
  console.log(err);
});

```

```python
import websocket
import hashlib
import hmac

ws_url = 'wss://user-wss.madex360.com'
API_KEY = '9e2bd17ff73e8531c0f3c26f93e48bfa402a3b13'
SECRET_KEY = 'ca55beb9e45d4f30b3959b464402319b9e12bac7'

def on_message(ws, message):
    print(message)


def on_error(ws, error):
    print(error)


def on_close(ws):
    print("### closed ###")


def on_open(ws):
    print("### opened ###")


def connect():
    # websocket.enableTrace(True)

    sign = hmac.new(SECRET_KEY.encode("utf-8"), "".encode('utf-8'), hashlib.sha256).hexdigest()
    header = {
        'BIBOX-API-KEY': API_KEY,
        'BIBOX-API-SIGN': sign,
    }
    ws = websocket.WebSocketApp(ws_url,
                                header=header,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever(ping_interval=30, ping_timeout=5)


if __name__ == "__main__":
    connect()

```

使用 Websocket 推送服务可以及时获取账户的余额及委托变动信息。

**连接 Websocket 服务器**

请使用以下 URL 连接 Websocket 服务器：

wss://user-wss.madex360.com

**在连接时，请附加以下HTTP请求头**

* BIBOX-API-KEY
* BIBOX-API-SIGN
* BIBOX-TIMESTAMP

*具体方法请参考[Authentication](#authentication)章节*

> 数据流
> 在成功建立连接后，客户端将收到ApiKey对应账户的余额变动信息及委托变动信息。格式如下:

```javascript
{
  "stream": "account",
  "data": { Account }
}

{
  "stream": "order",
  "data": { Order }
}
```

## Account

**当账户余额发生变更时，会收到account事件**

```javascript
{
  "stream": "account",
  "data": {
    "s":"USDT",  // 资产代码
    "a":10000,  // 可用额
    "h":0  // 冻结额
  }
}
```

## Orders

**当委托发生变更时，会收到order事件**

```javascript
{
  "stream": "order",
  "data":{
    "i": 4611688217450643477,  // 交易所分配的委托id
    "I": "",  // 用户指定的委托id
    "m": "BTC_USDT",  // 交易对代码
    "T": "limit",  // 委托类型
    "s": "sell",  // 委托方向
    "Q": -0.0100,  // 委托量
    "P": 10043.8500,  // 委托价格
    "t": "gtc",  // Time In Force
    "o": false,  // Post Only
    "S": "filled",  // 委托状态
    "E": -0.0100,  // 已成交量
    "e": -100.43850000,  // 已成交价值
    "C": 1643193746043,  // 创建时间
    "U": 1643193746464,  // 更新时间
    "n": 2,  // 成交笔数
  }
}
```
