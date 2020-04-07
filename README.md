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
- [USER API](#user-api)
  - [Profile](#profile)
  - [My Trades](#mytrades)
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

Best price/amount on the order book for a market.

**Url :**
https://changekon.com/api/v1/market/depth

```
GET /market/depth
```
**Parameters:**

Name | Type | Required | Example
------------ | ------------ | ------------ | ------------
market | STRING | YES | BTC_TMN

**Example :**

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
