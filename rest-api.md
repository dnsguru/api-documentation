# Public Rest API for Namebase
## General API Information
* Parameters for GET requests must be sent as a `query string`.
* All other request methods require parameters in the request body as `Content-Type: application/json`
* Parameter order does not matter.
* All timestamps are in milliseconds.
* API errors take the following form:
```javascript
{
  "code": "NAMEBASE_API_ERROR_CODE",
  "message": "Error message appears here."
}
```

## Rate limiting
If your account repeatedly gets rate limited, then you are using the API improperly and must back off. Failure to do so will cause your account to lose API access. Namebase retains the right to ban your account with no notice if you are abusing the API.

The rate limits are set generously and are not meant to prohibit any amount of normal usage. In the future, we will standardize our rate limits and provide more explicit feedback about how many remaining requests you can send each minute.

## Timing security
* Some endpoints require you to send a `timestamp` of when you created the request.
* You can optionally specify a `receiveWindow` which determines when your timestamped request expires.
* The `receiveWindow` defaults to 5 seconds (so, `5000`)
* If your timestamp is ahead of the server time by too much, you will receive an error complaining about sending a timestamp from the future.
* You can query the exchange info endpoint to synchronize your computer's clock with the exchange's.

# Public Namebase API
## Terminology
* For a symbol like HNSBTC, HNS is the "base" asset and BTC is the "quote" asset
* All prices are in the units of quote asset per 1 unit of the base asset
* All quantities are in the units of the base asset, and all quote quantities are in the units of the quote asset
* Similarly, volumes are in the units of the base asset, and quote volumes are in the units of the quote asset

## ENUM definitions
### Order status
* `NEW`
* `PARTIALLY_FILLED`
* `FILLED`
* `CANCELED`

### Order types
* `LMT` (limit)
* `MKT` (market)

### Order side
* `BUY`
* `SELL`

**Valid kline intervals:**

* `1m` (one minute)
* `5m` (five minutes)
* `15m` (fifteen minutes)
* `1h` (one hour)
* `4h` (four hours)
* `12h` (twelve hours)
* `1d` (one day)
* `1w` (one week)

## Miscellaneous endpoints
### Exchange information
```
GET /api/v0/info
```
Current exchange trading rules and symbol information. Use to test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1555556529865,
  "symbols": [{
    "symbol": "HNSBTC",
    "status": "TRADING",
    "baseAsset": "HNS",
    "basePrecision": 6,
    "quoteAsset": "BTC",
    "quotePrecision": 8,
    "orderTypes": ["LMT", "MKT"]
  }]
}
```

## Market Data endpoints
### Order book
```
GET /api/v0/depth
```

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
limit  | INT| NO | Default 100; max 1000. Valid limits:[5, 50, 100, 500, 1000]

**Response:**
```javascript
{
  "lastEventId": 6828,         // The last event id this includes
  "bids": [
    ["0.00003000",  "200.000000"] // [Price level, Quantity]
  ],
  "asks": [
    ["0.00003100", "100.000000"]
  ]
}
```

### Trade lookup
```
GET /api/v0/trade
```
Get older trades.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
tradeId | LONG | NO | Trade id to fetch from. Default gets most recent trades.
limit | INT | NO | Default 100; max 1000.
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "tradeId": 28457,
    "price": "0.00003000",
    "quantity": "500.000000",
    "quoteQuantity": "0.01500000",
    "createdAt": 1555556529865,
    "isBuyerMaker": true
  }
]
```

### Kline data
```
GET /api/v0/ticker/klines
```
Kline (candlestick) bars for a symbol.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
interval | ENUM | YES |
startTime | LONG | NO | Inclusive
endTime | LONG | NO | Inclusive
limit | INT | NO | Default 100; max 1000.

* If startTime and endTime are not sent, the most recent klines are returned.

**Response:**
```javascript
[
  {

    "openTime": 1557190800000,
    "closeTime": 1557190859999,
    "openPrice": "0.00002247",
    "highPrice": "0.00002256",
    "lowPrice": "0.00002243",
    "closePrice": "0.00002253",
    "volume": "10.001301",
    "quoteVolume": "0.000224824",
    "numberOfTrades": 42
  }
]
```


### 24hr ticker price change statistics
```
GET /api/v0/ticker/day
```
24 hour rolling window price change statistics.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |

**Response:**
```javascript
{
  "volumeWeightedAveragePrice": "0.00001959",
  "priceChange": "0.00000019",
  "priceChangePercent": "0.8528",
  "openPrice": "0.00002228",
  "highPrice": "0.00002247",
  "lowPrice": "0.00001414",
  "closePrice": "0.00002247",
  "volume": "11413.935399",
  "quoteVolume": "0.22363732",
  "openTime": 1555467560001,
  "closeTime": 1555553960000,
  "firstTradeId": 19761,
  "lastTradeId": 20926,
  "numberOfTrades": 1166
}
```


### Symbol price ticker
```
GET /api/v0/ticker/price
```
Latest price for a symbol or symbols.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |

**Response:**
```javascript
{
  "price": "0.00002300"
}
```


### Symbol order book ticker
```
GET /api/v0/ticker/book
```
Best price/quantity on the order book for a symbol or symbols.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |

**Response:**
```javascript
{
  "bidPrice": "0.00002000",
  "bidQuantity": "100.000000",
  "askPrice": "0.00002300",
  "askQuantity": "9000.100000"
}
```

## Account endpoints
### New order
```
POST /api/v0/order
```
Send in a new order.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
side | ENUM | YES | Buy or sell etc
type | ENUM | YES | Limit or market etc
quantity | DECIMAL | YES |
price | DECIMAL | NO | Required for limit orders
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "orderId": 174,
  "createdAt": 1555556529865,
  "price": "0.00000000",
  "originalQuantity": "1000.00000000",
  "executedQuantity": "1000.00000000",
  "status": "FILLED",
  "type": "MKT",
  "side": "SELL",
  "fills": [
    {
      "price": "0.00003000",
      "quantity": "500.000000",
      "quoteQuantity": "0.01500000",
      "commission": "0.00000750",
      "commissionAsset": "BTC"
    }
    {
      "price": "0.00002000",
      "quantity": "500.000000",
      "quoteQuantity": "0.01000000",
      "commission": "0.00000500",
      "commissionAsset": "BTC"
    }
  ]
}
```

### Query order
```
GET /api/v0/order
```
Check an order's status.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
orderId | LONG | YES |
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "orderId": 1,
  "price": "0.1",
  "originalQuantity": "1.0",
  "executedQuantity": "0.0",
  "status": "NEW",
  "type": "LMT",
  "side": "BUY",
  "createdAt": 1555556529865,
  "updatedAt": 1555556529865
}
```

### Cancel order
```
DELETE /api/v0/order
```
Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
orderId | LONG | YES |
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "orderId": 28,
  "price": "1.00000000",
  "originalQuantity": "910.00000000",
  "executedQuantity": "19.00000000",
  "status": "CANCELED",
  "type": "LMT",
  "side": "SELL",
  "createdAt": 1555556529865,
}
```

### Current open orders
```
GET /api/v0/order/open
```
Get all open orders on a symbol. **Careful** when accessing this with no symbol.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "orderId": 1,
    "price": "0.1",
    "originalQuantity": "1.0",
    "executedQuantity": "0.0",
    "status": "NEW",
    "type": "LMT",
    "side": "BUY",
    "createdAt": 1555556529865,
    "updatedAt": 1555556529865
  }
]
```

### All orders
```
GET /api/v0/order/all
```
Get all account orders; active, canceled, or filled.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
orderId | LONG | NO | Greater than or equal to 0.
limit | INT | NO | Default 100; max 1000.
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**
* If `orderId` is set, it will get orders >= that `orderId`. Otherwise you will receive the most recent orders.

**Response:**
```javascript
[
  {
    "orderId": 1,
    "price": "0.1",
    "originalQuantity": "1.0",
    "executedQuantity": "0.0",
    "status": "NEW",
    "type": "LMT",
    "side": "BUY",
    "createdAt": 1555556529865,
    "updatedAt": 1555556529865
  }
]
```

### Account information
```
GET /api/v0/account
```
Get current account information.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "makerFee": 15, // in basis points, 0.15%
  "takerFee": 15, // in basis points, 0.15%
  "canTrade": true,
  "balances": [
    {
      "asset": "HNS",
      "unlocked": "779.900092",
      "lockedInOrders": "100.000000",
      "canDeposit": true,
      "canWithdraw": true
    },
    {
      "asset": "BTC",
      "unlocked": "5.10000012",
      "lockedInOrders": "1.000000",
      "canDeposit": true,
      "canWithdraw": true
    }
  ]
}
```

### Account trade list
```
GET /api/v0/trade/account
```
Get trades for a specific account and symbol.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
tradeId | LONG | NO | TradeId to fetch from. Default gets most recent trades.
limit | INT | NO | Default 100; max 1000.
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Notes:**
* If `tradeId` is set, it will get trades >= that `tradeId`.  Otherwise you will get your most recent trades.

**Response:**
```javascript
[
  {
    "tradeId": 10921,
    "orderId": 61313,
    "price": "8.00000000",
    "quantity": "200.000000",
    "quoteQuantity": "1600.00000000",
    "commission": "4.500000",
    "commissionAsset": "HNS",
    "createdAt": 1555556529865,
    "isBuyer": true,
    "isMaker": false,
  }
]
```

### Order trade list
```
GET /api/v0/trade/order
```
Get trades for a specific order and symbol.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
symbol | STRING | YES |
orderId | LONG | YES | OrderId to get the trades for.
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "tradeId": 10921,
    "orderId": 61313,
    "price": "8.00000000",
    "quantity": "200.000000",
    "quoteQuantity": "1600.00000000",
    "commission": "4.500000",
    "commissionAsset": "HNS",
    "createdAt": 1555556529865,
    "isBuyer": true,
    "isMaker": false,
  }
]
```

### Deposit Address
```
POST /api/v0/deposit/address
```
Generate a deposit address.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
asset | STRING | YES |
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
    "address": "ts1qjg8chhk2t4zff4ltdaug3g9f7sxgne98jyv6ar",
    "success": true,
    "asset": "HNS"
}

```

### Withdraw
```
POST /api/v0/withdraw
```
Submit a withdraw request.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
asset |  STRING | YES |
address  | STRING | YES |
amount | DECIMAL | YES |
receiveWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "message": "success",
  "success": true,
  "id": "df7282ad-df8c-44f7-b747-5b09079ee852"
}
```

### Deposit History
```
GET /api/v0/deposit/history
```
Fetch deposit history.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
asset | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
receiveWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**
```javascript
[
  {
    "asset": "HNS",
    "amount": "31.853300",
    "address": "ts1qtq6ymgcep8mz2ag32ftrktwws0hr4uygprjurf",
    "txHash": "e7714680a4d93e3b29348eab38c22bb99949ed4d8aea7006091ff5f9712d1ec6",
    "createdAt": 1555556529865,
  },
  {
    "asset": "HNS",
    "amount": "210.000333",
    "address": "n1M5Rw3r7WkujB2dG1L84M3a4pzr2NKvfp",
    "txHash": "1d0827c642bd67781f80fe15c0fbb349aa4e35117adba06a52add4b207d334dd",
    "createdAt": 1555556529865,
  }
],
```

### Withdraw History
```
GET /api/v0/withdraw/history
```
Fetch withdraw history.

**Parameters:**

Name | Type | Mandatory | Description
-----|------|-----------|------------
asset | STRING | YES |
startTime | LONG | NO |
endTime | LONG | NO |
receiveWindow | LONG | NO |
timestamp | LONG | YES |


**Response:**

```javascript
[
  {
    "id": "3333edc6-e5c6-4d23-bf84-7b1072a90e37",
    "asset": "HNS",
    "amount": "1.000000",
    "minerFee": "0.100000",
    "address": "ts1qtq6ymgcep8mz2ag32ftrktwws0hr4uygprjurf",
    "txHash": "e7714680a4d93e3b29348eab38c22bb99949ed4d8aea7006091ff5f9712d1ec6",
    "createdAt": 1555556529865,
  },
  {
    "id": "180ceb4d-d303-4fed-9af6-213b5137255a",
    "asset": "HNS",
    "amount": "1200.000000",
    "minerFee": "0.200000",
    "address": "ts1qygv5nh38e9sl8npm4pcx8mqqqfp9sjaq4jrsn5",
    "txHash": "c5c398802554b861bef2ec7c4805846ff400a90f71059619974685848bbc4fd3",
    "createdAt": 1555556529865,
  }
]
```
