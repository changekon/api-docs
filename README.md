[![alt text][1.1]][1]
[![alt text][2.1]][2]
[![alt text][3.1]][3]

[1.1]: http://i.imgur.com/tXSoThF.png (twitter icon with padding)
[2.1]: http://i.imgur.com/P3YfQoD.png (facebook icon with padding)
[3.1]: http://i.imgur.com/0o48UoR.png (github icon with padding)

[1.2]: http://i.imgur.com/wWzX9uB.png (twitter icon without padding)
[2.2]: http://i.imgur.com/fep1WsG.png (facebook icon without padding)
[3.2]: http://i.imgur.com/9I6NRUm.png (github icon without padding)

[1]: http://www.twitter.com/changekon
[2]: http://www.facebook.com/changekon
[3]: http://www.github.com/changekon
# Changekon API

Table of Contents :

- [General API Information](#general-api-information)
  - [Endpoint security type](#endpoint-security-type)
  - [SIGNED (TRADE and USER_DATA) Endpoint security](#signed-trade-and-user_data-endpoint-security)
    - [Example](#example)
  - [Terminology](#terminology)
  - [HTTP Return Codes](#http-return-codes)
- [Public API Endpoints](#public-api-endpoints)
  - [Check server time](#check-server-time)
  - [Market prices](#market-prices)
- [USER API](#user-api)
  - [Profile](#profile)
  - [My Trades](#my-trades)
  - [ORDERS](#orders)
  - [OPEN ORDERS](#open-orders)
- [Market API](#market-api)
  - [Order Book](#order-book)
- [ORDER API](#order-api)
  - [New Buy order  (TRADE)](#buy-order)
  - [New Sell order  (TRADE)](#sell-order)
  - [Cancel order (TRADE)](#cancel-order)
  - [ORDER STATUS](#order-status)
- [WALLET API](#order-api)
  - [BALANCES](#balances)

## General API Information
* The base endpoint is: **https://changekon.com/api/v1/**

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it. This is stated next to the NAME of the endpoint.
    * If no security type is stated, assume the security type is NONE.
* API-keys are passed into the Rest API via the `PUBKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
MARKET_DATA | Endpoint can be accessed freely.
ORDER | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
WALLET_DATA | Endpoint requires sending a valid API-Key and signature.

* `ORDER` and `USER` and `WALLET` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specify the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

  **Serious trading is about timing.** Networks can be unstable and unreliable,
  which can lead to requests taking varying amounts of time to reach the
  servers. With `recvWindow`, you can specify that the request must be
  processed within a certain number of milliseconds or be rejected by the
  server.
  **It is recommended to use a small recvWindow of 5000 or less! The max cannot go beyond 60,000!**



## SIGNED Endpoint Examples for GET /api/v1/order/buy
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `curl`.

Key | Value
------------ | ------------
apiKey | Xm89ak2z8cA2MjP4ywRlshyeOboAkZI9rDBNWjKu58DCiTUgrmJGfsbxCrmFcYcz
secretKey | QvZYxOALthYZS9rkGbXxTSfKwc1vS1NON2nr4qrurfkxyOeRyprGip1nfLKF5dJY


Parameter | Value
------------ | ------------
price | 1000000
amount | 0.01
market | BTC_TMN
timestamp | 1577165821400

### Example:

* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "PUBKEY: Xm89ak2z8cA2MjP4ywRlshyeOboAkZI9rDBNWjKu58DCiTUgrmJGfsbxCrmFcYcz" -X GET 'https://changekon.com/api/v1/order/buy?signature=1199881f4826ff4ff8e236494480a77ef8fd969cf7981ba98b41053a638435d6&timestamp=1577165476400&price=1000000&amount=0.01&market=BTC_TMN'
    ```

## Terminology

**Order status (status):**

* PENDING
* FILLED
* CANCELLED

**Order side (side):**

* BUY
* SELL
## HTTP Return Codes
* HTTP `400` return codes are used when required parameter are missing from query. for example `market` in a buy order.
* HTTP `401` return codes are used for malformed requests;
  the issue is on the sender's side. usually for missing `PUBKEY` or invalid `signature`
* HTTP `403` return code is used when the WAF Limit (Web Application Firewall) has been violated.
* HTTP `404` return code is used when the requested  order are not found.
* HTTP `406` return code is used when user has not enough balance to do requested query
* HTTP `408` return code is used when request timeout use [Check server time](#check-server-time) to synchronize your time to server
* HTTP `5XX` return codes are used for internal errors; the issue is on
  changekon's side.
  It is important to **NOT** treat this as a failure operation; the execution status is
  **UNKNOWN** and could have been a success.


# Public API Endpoints

## Check server time

Test connectivity to the Rest API and get the current server time.

**Url :**
https://changekon.com/api/v1/server/time

```
GET /api/v1/server/time
```
**Parameters:**
NONE


**Response:**
```javascript
{
  "serverTime": 1577168266421
}
```

## Market prices

Shows the best current order in each market

**Url :**
https://changekon.com/api/v1/market/prices

```
GET /api/v1/market/prices
```
**Parameters:**
NONE


**Response:**
```javascript
  [
   {"market":"BTC_TMN","prices":{"buy":"192,949,328","sell":"195,571,152"},"24h%":"0.4"},
   {"market":"BTC_USDT","prices":{"buy":"9,031.34","sell":"9,147.95"},"24h%":"0.5"},
   {"market":"ETH_TMN","prices":{"buy":"4,788,118","sell":"4,839,234"},"24h%":"-0.6"},
   {"market":"ETH_USDT","prices":{"buy":"224.28","sell":"226.69"},"24h%":"0.6"},
   {"market":"USDT_TMN","prices":{"buy":"21,176","sell":"21,668"},"24h%":"-0.4"}
 ]
```

# USER API
## Profile

To view current user profile.

**Url :**
https://changekon.com/api/v1/user/profile

```
GET /api/v1/user/profile
```

**Example :**
```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/user/profile?signature=72ca26693b0a389782e16d3bd780e707bdc99a3ec2257e3777bb6441ac600e1e&timestamp=1577168720400'
```
**Response:**
```javascript
{
    "status": "OK",
    "IP": "5.39.126.48",
    "email": "taherimajid1362@gmail.com",
    "first_name": "مجید",
    "last_name": "طاهری",
    "birthdate": "1362/03/15",
    "avatarlink": "loadimg?id=avatar",
    "KYC": "VERIFIED"
}
```


## My Trades

Get your recent trades.

**Url :**
https://changekon.com/api/v1/user/myTrades

```
GET /user/myTrades
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 1577164260500
signature | LONG | YES |

**Example :**
```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/user/myTrades?signature=eef99c704a6148fa9d13a6ae0d20bb7c429480695e210903302ef047bde9f60a&timestamp=1577164260500'
```

**Response:**
```javascript
[
    {
        "transaction_id": "1023",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "93836219",
        "amount": "0.00569792",
        "fee": "0.001",
        "timestamp": "2019-12-14 15:56:14"
    },
    {
        "transaction_id": "1020",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "93536219",
        "amount": "0.00994950",
        "fee": "0.001",
        "timestamp": "2019-12-14 15:56:14"
    },
    {
        "transaction_id": "1015",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "98836219",
        "amount": "0.00270200",
        "fee": "0.001",
        "timestamp": "2019-12-13 00:12:56"
    },
    {
        "transaction_id": "1010",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "100000000",
        "amount": "0.00739900",
        "fee": "0.001",
        "timestamp": "2019-12-13 00:08:07"
    },
    {
        "transaction_id": "982",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "100000000",
        "amount": "0.00260100",
        "fee": "0.001",
        "timestamp": "2019-12-13 00:08:07"
    },
    {
        "transaction_id": "963",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "97864543",
        "amount": "0.00603338",
        "fee": "0.001",
        "timestamp": "2019-12-12 16:32:35"
    },
    {
        "transaction_id": "950",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "100000000",
        "amount": "0.00750000",
        "fee": "0.001",
        "timestamp": "2019-12-12 16:31:29"
    },
    {
        "transaction_id": "930",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "100000010",
        "amount": "0.00681719",
        "fee": "0.001",
        "timestamp": "2019-12-11 18:25:42"
    },
    {
        "transaction_id": "914",
        "market": "BTC_TMN",
        "type": "LIMIT",
        "price": "100000000",
        "amount": "0.00121000",
        "fee": "0.001",
        "timestamp": "2019-12-11 18:21:50"
    }
]
```
## ORDERS

To view recent all orders from current user.

**Url :**
https://changekon.com/api/v1/user/orders

```
GET /api/v1/user/orders
```

**Example :**
```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/user/orders?signature=6a30f42ef5508ece54a760ee7d44bedb713124f0ff397602059cbf3e1ae49ba7&timestamp=1577174304400'
```
**Response:**
```javascript
[
    {
        "order_id": "154",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "92836219",
        "amount": "0.03129484",
        "filled": "0.00000000",
        "status": "cancelled",
        "timestamp": "2019-12-14 15:56:05"
    },
    {
        "order_id": "155",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "93836219",
        "amount": "0.01564742",
        "filled": "0.01564742",
        "status": "filled",
        "timestamp": "2019-12-14 15:56:14"
    },
    {
        "order_id": "156",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "91836219",
        "amount": "0.01564742",
        "filled": "0.00000000",
        "status": "cancelled",
        "timestamp": "2019-12-14 15:56:38"
    },
    {
        "order_id": "157",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "90836219",
        "amount": "0.01564742",
        "filled": "0.00000000",
        "status": "cancelled",
        "timestamp": "2019-12-14 15:56:43"
    },
    {
        "order_id": "1077",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "1000000",
        "amount": "0.01000000",
        "filled": "0.00000000",
        "status": "pending",
        "timestamp": "2019-12-24 09:01:16"
    }
]
```
## OPEN ORDERS

To view recent open orders from current user.

**Url :**
https://changekon.com/api/v1/user/openOrders

```
GET /api/v1/user/openOrders
```

**Example :**
```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/user/openOrders?signature=27dae63c1cb071316afa80d5ded48db7fd7931c13f885bc2e9fcb1c4c5f03ff7&timestamp=1577174502400'
```
**Response:**
```javascript
[
    {
        "order_id": "1093",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "1000000",
        "amount": "0.01000000",
        "filled": "0.00000000",
        "status": "pending",
        "timestamp": "2019-12-24 11:31:38"
    }
]
```

# Market API
## Order Book

Best price/amount on the order book.

**Url :**
https://changekon.com/api/v1/market/depth

```
GET /market/depth
```
**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
market | STRING | NO | BTC_TMN

**Example1 :**

GET Order Book on BTC_TMN market:

```
curl -X GET 'https://changekon.com/api/v1/market/depth?market=BTC_TMN'
```

**Response:**
```javascript
{
  "bids": {
    "78000000": 0.022,
    "79000000": 0.01,
    "80000000": 0.02,
    "85000000": 0.032,
    "88521965": 0.02922297,
    "89891596": 0.0057753,
    "90021690": 0.01773499,
    "90027643": 0.05116765
  },
  "asks": {
    "99073066": 0.02269855,
    "99093993": 0.02302748,
    "100175395": 0.02316553,
    "100589908": 0.0462273,
    "102113784": 0.03160666,
    "103574785": 0.01454559,
    "103704960": 0.02532168,
    "112000000": 0.01
  }
}

```

**Example2 :**

GET Order Book on all market:

```
curl -X GET 'https://changekon.com/api/v1/market/depth'
```

**Response:**
```javascript
{
  "BTC_TMN": {
    "bids": {
      "855427400.00000000": "0.01453700",
      "847322200.00000000": "0.01998100",
      "845580500.00000000": "0.00854100",
      "844321200.00000000": "0.02894700",
      "843238600.00000000": "0.01316100",
      "840564900.00000000": "0.00627100",
      "837186700.00000000": "0.03791000",
      "836872200.00000000": "0.01821900",
      "836474000.00000000": "0.03535300",
      "835337000.00000000": "0.03310200",
      "834482300.00000000": "0.03007600",
      "831651200.00000000": "0.01894000",
      "826893000.00000000": "0.03108300",
      "826708200.00000000": "0.02532900",
      "826638200.00000000": "0.03432000",
      "825396700.00000000": "0.03316000",
      "824628800.00000000": "0.02125000",
      "823420000.00000000": "0.03369200",
      "822069700.00000000": "0.00675900",
      "820115900.00000000": "0.01029300"
    },
    "asks": {
      "867143000.00000000": "0.00115400",
      "873831100.00000000": "0.00149300",
      "877175600.00000000": "0.00053300",
      "877983000.00000000": "0.00146200",
      "893762100.00000000": "0.01483800",
      "899205300.00000000": "0.01756700",
      "902845900.00000000": "0.02283000",
      "905509100.00000000": "0.02797700",
      "906409300.00000000": "0.01876000",
      "906768900.00000000": "0.01996900",
      "907298800.00000000": "0.01797800",
      "910678900.00000000": "0.01881500",
      "911291500.00000000": "0.02884600",
      "912604800.00000000": "0.02715200",
      "914849000.00000000": "0.02300700",
      "917188800.00000000": "0.03111500",
      "917817100.00000000": "0.02431700",
      "919753500.00000000": "0.01574300",
      "922480000.00000000": "0.02577800",
      "928595200.00000000": "0.00543100"
    }
  },
  "ETH_TMN": {
    "bids": {
      "58094700.00000000": "0.35398000",
      "58003100.00000000": "0.56079000",
      "57800000.00000000": "0.00550000",
      "57767100.00000000": "0.54539000",
      "57755700.00000000": "0.96428000",
      "57741700.00000000": "0.77539000",
      "57677900.00000000": "0.79183000",
      "57661100.00000000": "0.24453000",
      "57529000.00000000": "0.01000000",
      "57430000.00000000": "0.38076000",
      "55901100.00000000": "0.23446000",
      "44000000.00000000": "0.10640000"
    },
    "asks": {
      "60192900.00000000": "0.05743000",
      "60389200.00000000": "0.05815000",
      "61500500.00000000": "1.69006000",
      "61526800.00000000": "0.59714000",
      "61650600.00000000": "1.19512000",
      "61690800.00000000": "1.38410000",
      "61902600.00000000": "1.33744000",
      "62092800.00000000": "1.47997000",
      "62109800.00000000": "1.33611000",
      "62351000.00000000": "1.52924000",
      "62357600.00000000": "1.35587000",
      "62444900.00000000": "1.07617000",
      "62483700.00000000": "1.09500000",
      "62729700.00000000": "1.32245000",
      "62736800.00000000": "0.43372000",
      "62831800.00000000": "0.16252000",
      "62873500.00000000": "0.97782000",
      "63155800.00000000": "0.22783000",
      "63159600.00000000": "0.94856000",
      "63401600.00000000": "0.25618000"
    }
  },
  "LINK_TMN": {
    "bids": {
      "670600.00000000": "1.79000000",
      "657600.00000000": "14.36000000",
      "656600.00000000": "32.60000000",
      "656500.00000000": "24.97000000",
      "655200.00000000": "47.23000000",
      "654100.00000000": "34.91000000",
      "634100.00000000": "29.60000000",
      "633200.00000000": "23.46000000"
    },
    "asks": {
      "682100.00000000": "2.39000000",
      "718500.00000000": "30.07000000",
      "718900.00000000": "64.61000000",
      "721100.00000000": "50.83000000",
      "721400.00000000": "19.08000000",
      "722300.00000000": "15.52000000",
      "724000.00000000": "23.24000000",
      "724300.00000000": "36.42000000"
    }
  },
  "BTC_USDT": {
    "bids": {
      "34475.61000000": "0.02570900",
      "34275.42000000": "0.01382400",
      "34176.71000000": "0.00549700",
      "34151.39000000": "0.02237600",
      "34093.74000000": "0.00815900",
      "34083.77000000": "0.01582300",
      "33857.01000000": "0.01074300",
      "33834.77000000": "0.02395600",
      "33808.67000000": "0.02217200",
      "33752.53000000": "0.03701500",
      "33678.81000000": "0.03692700",
      "33644.83000000": "0.01771500",
      "33614.19000000": "0.03415900"
    },
    "asks": {
      "35550.97000000": "0.00134600",
      "35944.34000000": "0.03361400",
      "36158.73000000": "0.02201900",
      "36301.88000000": "0.02503700",
      "36532.80000000": "0.02707600",
      "36569.90000000": "0.01049900",
      "36587.17000000": "0.00567000",
      "36696.12000000": "0.02131200",
      "36698.03000000": "0.03008300",
      "37095.76000000": "0.02291800",
      "37150.83000000": "0.00535000",
      "37162.43000000": "0.02419400",
      "37240.18000000": "0.03386300"
    }
  },
  "USDT_TMN": {
    "bids": {
      "24651.00000000": "33.33000000",
      "24650.00000000": "290.46000000",
      "24641.00000000": "247.50000000",
      "23979.00000000": "268.24000000",
      "23930.00000000": "2868.24000000",
      "23900.00000000": "344.03000000",
      "23876.00000000": "1096.94000000",
      "23851.00000000": "1341.06000000",
      "23847.00000000": "180.00000000",
      "23805.00000000": "610.17000000"
    },
    "asks": {
      "24783.00000000": "34.57000000",
      "24789.00000000": "39.57000000",
      "24832.00000000": "18.83000000",
      "24836.00000000": "36.11000000",
      "25670.00000000": "379.69000000",
      "25825.00000000": "1514.44000000",
      "25953.00000000": "5829.55000000",
      "25983.00000000": "1183.64000000",
      "25986.00000000": "313.55000000"
    }
  },
  "DOGE_TMN": {
    "bids": {
      "7506.00000000": "118.00000000",
      "7494.00000000": "158.00000000",
      "7487.00000000": "548.00000000",
      "7308.00000000": "1330.00000000",
      "7300.00000000": "6010.00000000",
      "7294.00000000": "2374.00000000",
      "7293.00000000": "811.00000000",
      "7273.00000000": "1472.00000000",
      "7184.00000000": "3466.00000000",
      "7179.00000000": "3113.00000000",
      "7177.00000000": "5418.00000000",
      "7165.00000000": "1695.00000000",
      "7136.00000000": "3742.00000000",
      "7130.00000000": "5125.00000000"
    },
    "asks": {
      "7494.00000000": "158.00000000",
      "7558.00000000": "269.00000000",
      "7584.00000000": "240.00000000",
      "7805.00000000": "2522.00000000",
      "7833.00000000": "2893.00000000",
      "7845.00000000": "5211.00000000",
      "7877.00000000": "1081.00000000",
      "7878.00000000": "2351.00000000",
      "7883.00000000": "1164.00000000",
      "7919.00000000": "6082.00000000",
      "7938.00000000": "8448.00000000"
    }
  },
  "ETH_USDT": {
    "bids": {
      "2356.49000000": "0.40272000",
      "2352.29000000": "0.61567000",
      "2352.10000000": "0.39328000",
      "2304.60000000": "0.32650000",
      "2301.46000000": "0.34229000",
      "2296.84000000": "0.87721000",
      "2295.61000000": "0.45057000",
      "2294.32000000": "0.09938000",
      "2291.14000000": "0.59771000"
    },
    "asks": {
      "2442.61000000": "0.01299000",
      "2453.81000000": "0.02264000",
      "2454.99000000": "0.03544000",
      "2528.47000000": "0.51584000",
      "2531.25000000": "0.17580000",
      "2531.56000000": "0.25646000",
      "2539.13000000": "0.67526000",
      "2540.01000000": "0.53870000",
      "2541.00000000": "0.38591000",
      "2553.83000000": "0.31277000"
    }
  },
  "TRX_TMN": {
    "bids": {
      "1757.00000000": "661.10000000",
      "1741.00000000": "3795.70000000",
      "1728.00000000": "5166.80000000",
      "1725.00000000": "3647.60000000",
      "1723.00000000": "2617.30000000",
      "1722.00000000": "11819.10000000",
      "1721.00000000": "19201.10000000",
      "1675.00000000": "9533.20000000"
    },
    "asks": {
      "1774.00000000": "743.10000000",
      "1779.00000000": "846.90000000",
      "1780.00000000": "655.10000000",
      "1784.00000000": "473.40000000",
      "1785.00000000": "445.00000000",
      "1808.00000000": "10673.50000000",
      "1809.00000000": "6236.20000000",
      "1811.00000000": "16506.90000000",
      "1812.00000000": "35947.80000000",

    }
  },
  "LINK_USDT": {
    "bids": {
      "27.15000000": "0.82000000",
      "27.14000000": "2.17000000",
      "27.02000000": "1.72000000",
      "26.69000000": "62.51000000",
      "26.57000000": "9.06000000",
      "26.56000000": "10.33000000",
      "26.55000000": "72.16000000",
      "26.52000000": "25.33000000",
      "26.50000000": "46.05000000",
      "26.48000000": "14.47000000",
      "26.39000000": "48.06000000",

    },
    "asks": {
      "27.15000000": "0.82000000",
      "27.26000000": "2.13000000",
      "27.47000000": "1.30000000",
      "27.58000000": "0.95000000",
      "28.13000000": "7.79000000",
      "28.25000000": "47.56000000",
      "28.31000000": "73.88000000",
      "28.32000000": "16.44000000",
      "28.40000000": "35.36000000",
      "28.45000000": "57.01000000",
      "28.47000000": "38.55000000",
      "28.81000000": "34.08000000"
    }
  },
  "DOGE_USDT": {
    "bids": {
      "0.29771910": "865.00000000",
      "0.29671090": "2272.00000000",
      "0.29594830": "895.00000000",
      "0.29593670": "1115.00000000",
      "0.29585300": "974.00000000",
      "0.29552750": "2987.00000000",
      "0.29527590": "1290.00000000",
      "0.29522310": "1483.00000000",
      "0.29491230": "983.00000000",
      "0.29480190": "1972.00000000",
      "0.29477760": "2936.00000000",
    },
    "asks": {
      "0.30676140": "59.00000000",
      "0.30677940": "72.00000000",
      "0.30718880": "74.00000000",
      "0.30735210": "82.00000000",
      "0.30782030": "41.00000000",
      "0.31089850": "852.00000000",
      "0.31106710": "352.00000000",
      "0.31211150": "2069.00000000",
      "0.31280440": "1522.00000000",
      "0.31341850": "1429.00000000",
      "0.31394270": "622.00000000",
      "0.31491940": "1079.00000000",

    }
  },
  "TRX_USDT": {
    "bids": {
      "0.07111000": "768.10000000",
      "0.07103000": "1601.50000000",
      "0.07095000": "392.10000000",
      "0.07088000": "646.10000000",
      "0.07027000": "21568.40000000",
      "0.06989000": "13644.70000000",
      "0.06988000": "11245.90000000",
      "0.06979000": "14785.70000000",
      "0.06975000": "16676.00000000",
      "0.06967000": "10238.70000000",
    },
    "asks": {
      "0.07186000": "536.20000000",
      "0.07199000": "755.20000000",
      "0.07205000": "1141.30000000",
      "0.07208000": "761.70000000",
      "0.07214000": "698.00000000",
      "0.07272000": "9183.80000000",
      "0.07324000": "8907.40000000",
      "0.07330000": "8523.50000000",
      "0.07363000": "6176.40000000",
      "0.07366000": "4941.50000000",
      "0.07370000": "10739.70000000",
      "0.07373000": "8162.10000000",
    }
  }
}

```
# ORDER API

## Buy Order

Send in a new buy order

**Url :**
https://changekon.com/api/v1/order/buy

```
GET /user/buy
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
market | STRING | YES | BTC_TMN
amount | DECIMAL | YES | 0.01
price | DECIMAL | YES | 98500000
timestamp | LONG | YES | 1577165476400
signature | LONG | YES |

**Example :**

```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/order/buy?signature=1199881f4826ff4ff8e236494480a77ef8fd969cf7981ba98b41053a638435d6&timestamp=1577165476400&price=1000000&amount=0.01&market=BTC_TMN'
```

**Response:**
```javascript
{
    "status": "success",
    "order_id": 1077
}
```

## Sell Order

Send in a new sell order

**Url :**
https://changekon.com/api/v1/order/sell

```
GET /user/buy
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
market | STRING | YES | BTC_TMN
amount | DECIMAL | YES | 0.01
price | DECIMAL | YES | 99500000
timestamp | LONG | YES | 1577165476400
signature | LONG | YES |

**Example :**

```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/order/sell?signature=1199881f4826ff4ff8e236494480a77ef8fd969cf7981ba98b41053a638435d6&timestamp=1577165476400&price=1000000&amount=0.01&market=BTC_TMN'
```

**Response:**
```javascript
{
    "status": "success",
    "order_id": 1078
}
```

## CANCEL Order

Cancel an active order.

**Url :**
https://changekon.com/api/v1/order/cancel

```
GET /order/cancel
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | 1077
timestamp | LONG | YES | 1577165476400
signature | LONG | YES |

**Example :**

```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/order/cancel?signature=9aa118086f773137f516faa2c83c4943ccf9cc1de6bb5a243779804a32a097da&timestamp=1577165821400&order_id=1077'
```

**Response:**
```javascript
{
    "status": "success",
    "order_id": "1077"
}
```

## ORDER STATUS

GET current state and more details of an order.

**Url :**
https://changekon.com/api/v1/order/status

```
GET /order/status
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | 1077
timestamp | LONG | YES | 1577165476400
signature | LONG | YES |

**Example :**

```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/order/status?signature=84baa956fad71615ab6524d4632cd394b27bbcde71962867d0a25e93c8d52a3d&timestamp=1577174582400&order_id=1093'
```

**Response:**
```javascript
[
    {
        "order_id": "1093",
        "side": "buy",
        "type": "LIMIT",
        "market": "BTC_TMN",
        "price": "1000000",
        "amount": "0.01000000",
        "filled": "0.00000000",
        "status": "pending",
        "timestamp": "2019-12-24 11:31:38"
    }
]
```
# WALLET API

## BALANCES

Return current balance of user's active wallets

**Url :**
https://changekon.com/api/v1/wallet/balances

```
GET /wallet/balances
```

**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 1577165476400
signature | LONG | YES |

**Example :**

```
curl -H "PUBKEY: PUBLIC_KEY" -X GET 'https://changekon.com/api/v1/wallet/balances?signature=84baa956fad71615ab6524d4632cd394b27bbcde71962867d0a25e93c8d52a3d&timestamp=1577174582400'
```

**Response:**
```javascript
{
   "TMN":"46248973.013508",
   "BTC":"0.002775",
   "DOGE":"0",
   "USDT":"5000"
}
```
