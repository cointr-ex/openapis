---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json
toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:

spot_goods: Spot

spot_goods_url: 'spot.html'

spot_goods_active: active

contract: Futures

contract_url: 'index.html'

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---
[toc]

# General info

Reset endpoint URL:https://api.cointr.vip

Websocket endpoint URL: ws://ws-newfutures.cointr.vip/ws

## Terminology

- Kline/candlestick bars:

  `1M` ,`5M`, `15M`, `30M`, `1H`, `4H`, `12H`, `1D`, `1W`

(M -> minutes; H -> hours; D -> days; W -> weeks)

- Role: `MAKER`, `TAKER`

- Order state: `ACCEPTED`, `SUBMITTING`, `SUBMITTED`, `PARTIAL-FILLED`, `CANCELING`, `CANCELED`, `FILLED`, `STOPPED`, `TRIGGERED`, `EXPIRED`

(In some extreme scenarios, order state of `SUBMITTING` will be returned by order-placing requests. State of `SUBMITTING` indicates that asset involved in the order has been frozen and whereas the order has not been fed into our matching engine yet. Orders of `SUBMITTING` state will never trigger data pushes of our Websocket channels. )

- Order/trade side/direction (side): `BUY`, `SELL`

- Order type (ordType): `MARKET`, `LIMIT`, `STOP`, `STOP_LIMIT`

- Order flags: `POST_ONLY`

- Time in force (timeInForce): `GTC`, `IOC`, `FOK`

- Transfer type: `SPOT_TO_FUTURE`, `FUTURE_TO_SPOT`

## Sign

For authenticated requests, the following headers should be sent with the request:

- X-COINTR-APIKEY: `your_API_access_key`

- X-COINTR-SIGN: `signature`

- Content-Type: `'application/json'`

Our signature algorithm involves two steps of `HMAC SHA256` operations:

1. `HMAC SHA256` of the following string, with your API secret key as the key: `f"{math.floor(currentTime/30000)}"` (`currentTime`，unix timestamp format in milliseconds)
2. With the value obtained from `HMAC SHA256` operation in step 1 as the key, perform `HMAC SHA256` operation on `totalParams` which is the concatenation of `query string` and `request body`. Parameter `timestamp` must be included in the `query string` of all requests requiring authitication.

## Client-supplied ID

Client-supplied IDs should be a combination of up to 36 BASE64 characters.

clOrdId (client-supplied order id) is a mandatory field while submitting an order.

Professional users use this clOrdId to manage their orders. But if it is not required, please set this field with a random UUID.

User should ensure the uniqueness of clOrdIds. clOrdIds of active orders and orders completed within 1 minute must be unique at account level.

If a duplicated clOrdId is received, versus an existing active order, that new order request will be rejected by the Exchange.

User is able to cancel an order based on its clOrdId.

User is also able to query an order based on its clOrdId. But if that order has been fully filled or cancelled for more than 2 hours, it can be only queried by ordId instead.

To avoid duplicated order submission caused by auto retry taken by user's application, the Exchange will reject the 2nd order request if the same clOrdId was received within 1 minute, even though that 1st order already fully filled.

**Suggestion to user**

1. Always use a different clOrdId for a different order.
2. Application auto retry for order submission should not last for more than 1minute.

# Market Data

## Market Specifications

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
        "instId": "USDTTRY",
        "baseCcy": "USDT",
        "quotecy": "TRY",
        "pxPrecision": 3, 
        "amtPrecision": 2, 
        "minLmtSz": "10", 
        "minMktBuySz": "100", 
        "minMktSellSz": "10", 
        "steps": "[0.001, 0.01, 0.1, 1, 10]"
        }, ......
    ]
}
```

- HTTP request

GET  /v1/spot/public/instruments

- Request parameters

There are 3 possible options:

| **Option**   | **Type** | **Examples**                                                 |
| ------------ | -------- | ------------------------------------------------------------ |
| No parameter |          | curl -X GET " https://api.cointr.vip/v1/spot/public/instruments" |
| instId       | String   | curl -X GET " https://api.cointr.vip/v1/spot/public/instruments?instId=USDTTRY" |
| instIds      | String   | curl -X GET " https://api.cointr.vip/v1/spot/public/instruments?instIds=USDTTRY,BTCUSDT" |

If any instId provided in either instId or instIds does not exist,  or both instId and instIds are provided,  the endpoint will throw an error. If neither is sent, specifications regarding all instruments will be returned.

- Response parameters

| **Parameter** | **Type** | **Description**                              |
| ------------- | -------- | -------------------------------------------- |
| instId        | String   | Instrument ID.                               |
| baseCcy       | String   | Base currency.                               |
| quoteCcy      | String   | Quote currency.                              |
| pxPrecision   | Int      | Price precision.                             |
| amtPrecision  | Int      | Amount precision.                            |
| minLmtSz      | String   | Minimum limit order size in base currency.   |
| minMktBuySz   | String   | Minimum market order size in quote currency. |
| minMktSellSz  | String   | Minimum market order size in base currency.  |
| steps         | String   | Supported step aggregations in order book.   |



## Candlesticks

```json
{
    "code": "0",
    "msg": "",
    "data": [
     [
        1657630800,             //open time
        "3.721",                //open price
        "3.743",                //highest price
        "3.677",                //lowest price
        "3.708",                //close price
        "8422410",              //trading volume in base currency
        "22698348.04828491"     //trading volume in quote currency
    ],
    [
        1657627200,
        "3.731",
        "3.799",
        "3.494",
        "3.72",
        "24912403",
        "67632347.24399722"
    ]
    ]
}
```

- HTTP request

GET  /v1/spot/market/candlesticks

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| bar           | Enum     | Yes          | Bar size.Default: `1M`.                                      |
| endTs         | String   | No           | Filter with a endTs, unix timestamp format in seconds.Default: current timestamp. |
| limit         | Int      | No           | Number of results per request.Default: 100;Max: 2000.        |

Data is limited to the most recent 2000 ones. If limit sent excedes the maximum requirement of 2000, the most recent 2000 ones if available will be returned.

- Response parameters

| **Parameter** | **Type** | **Description**                                     |
| ------------- | -------- | --------------------------------------------------- |
| ts            | Long     | Kline start time, unix timestamp format in seconds. |
| o             | String   | Open price.                                         |
| h             | String   | Highest price.                                      |
| l             | String   | Lowest price.                                       |
| c             | String   | Close price.                                        |
| vol           | String   | Trading volume in base currency.                    |
| quoteVol      | String   | Trading volume in quote currency.                   |

## Orderbook

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "asks": [
                [
                    "41006.8",      //depth price
                    "0.60038921"    //the number of base currency at the price
                ]
            ],
            "bids": [
                [
                    "41006.3",      //depth price
                    "0.30178218"    //the number of base currency at the price
                ]
            ],
            "uTime": 1629966436396
        }
    ]
}
```

- HTTP request

GET  /v1/spot/market/depths

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| limit         | Int      | No           | Order book depth per side. Default 1; <br />valid limits: [1, 5, 10, 20, 50, 100]. |

- Response parameters

| **Parameter** | **Type** | **Description**                                     |
| ------------- | -------- | --------------------------------------------------- |
| asks          | Array    | Order book on sell side                             |
| bids          | Array    | Order book on buy side                              |
| uTime         | Long     | Update time, unix timestamp format in milliseconds. |

## Market Snapshot

```json
{
    "code":"0",
    "msg":"",
    "data":[
     {
        "instId":"BTCUSDT",
        "lastPx":"9999.99",
        "lastSz":"0.1",
        "askPx":"9999.99",
        "askSz":"11",
        "bidPx":"8888.88",
        "bidSz":"5",
        "open24h":"9000",
        "high24h":"10000",
        "low24h":"8888.88",
        "quoteVol24h":"2222",
        "vol24h":"2222",
        "uTime":1597026383085
    }, ......
  ]
}
```

Retrieve the latest price snapshot, best bid/ask price, and trading volume in the last 24 hours.

- HTTP request

GET  /v1/spot/market/tickers

- Request parameters

There are 3 possible options:

| **Option**   | **Type** | **Examples**                                                 |
| ------------ | -------- | ------------------------------------------------------------ |
| No parameter |          | curl -X GET " https://api.cointr.vip/v1/spot/market/tickers" |
| instId       | String   | curl -X GET " https://api.cointr.vip/v1/spot/market/tickers?instId=USDTTRY" |
| instIds      | List     | curl -X GET " https://api.cointr.vip/v1/spot/market/tickers?instIds=USDTTRY,BTCUSDT" |

If any instId provided in either instId or instIds do not exist,  or both instId and instIds are provided,  the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| instId        | String   | Instrument ID                                                |
| lastPx        | String   | Last traded price                                            |
| lastSz        | String   | Last traded size in base currency                            |
| askPx         | String   | Best ask price                                               |
| askSz         | String   | Best ask size in base currency                               |
| bidPx         | String   | Best bid price                                               |
| bidSz         | String   | Best bid size in base currency                               |
| open24h       | String   | Open price in the past 24 hours                              |
| high24h       | String   | Highest price in the past 24 hours                           |
| low24h        | String   | Lowest price in the past 24 hours                            |
| vol24h        | String   | 24h trading volume in base currency                          |
| quoteVol24h   | String   | 24h trading volume in quote currency                         |
| uTime         | Long     | Ticker data generation time, unix timestamp format in miliseconds |

## Recent Trades

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "instId": "BTCUSDT",
            "side": "sell",
            "sz": "0.00001",
            "px": "29963.2",
            "tradeId": "242720720",
            "mTime": 1654161646974
        },
        {
            "instId": "BTCUSDT",
            "side": "sell",
            "sz": "0.00001",
            "px": "29964.1",
            "tradeId": "242720719",
            "mTime": 1654161641568
        }, ......
    ]
}
```

- HTTP request

GET  /v1/spot/market/trades

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| limit         | Int      | No           | Number of results per request.<br />Default: 100;<br />Max: 500. |

- Response parameters

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| instId        | String   | Instrument ID                                     |
| tradeId       | String   | Trade ID                                          |
| px            | String   | Trade price                                       |
| sz            | String   | Trade quantity in base currency                   |
| side          | String   | Trade side of taker.                              |
| mTime         | Long     | Match time, unix timestamp format in milliseconds |

- Response sample

## Trading

## Place an Order

```json
{
    "code": "0",
    "msg": "",
    "data": 
        {
            "clOrdId": "cointrspot6",
            "ordId": "312269865356374016",
            "state": "submitted"
        }
}

OR

{
    "code": "5050",
    "msg": "",
    "data": 
        {
            "clOrdId": "cointrspot6"  
        }
}
```

- HTTP request

POST  /v1/spot/trade/order

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| clOrdId       | String   | No           | Client-supplied order ID.                                    |
| px            | String   | Optional     | Price                                                        |
| baseQty       | String   | Optional     | Order quantity in base currency, applicable to non-market-buy orders. |
| quoteQty      | String   | Optional     | Order quantity in quote currency, only applicable to market-buy orders. |
| side          | Enum     | Yes          | Order side.                                                  |
| ordType       | Enum     | Yes          | Order type: `MARKET`, `LIMIT`                                |
| flags         | String   | No           | Additional flag or multiple flags separated with comma.      |
| timeInForce   | Enum     | No           | Time in force. Default: `GTC`.                               |
| async         | Boolean  | No           | Allowing asynchronous order placing. If `True`, {} will be returned.  Default: `False`. |

- Response parameters

| **Parameter** | **Type** | **Description**                                          |
| ------------- | -------- | -------------------------------------------------------- |
| ordId         | String   | Order ID. Applicable if order is successfully placed.    |
| clOrdId       | String   | Client-supplied order ID.                                |
| state         | String   | Order state. Applicable if order is successfully placed. |

## Place Orders

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "clOrdId": "cointrspot6",
            "ordId": "312269865356374016",
            "state": "submitted"
        },
        {
            "clOrdId": "cointrspot6",
            "error": 123456
        }, ......
    ] 
}
```

Place orders in batches. Maximum 20 orders can be placed at a time. Request parameters should be passed in the form of an array. Order types are limited to `LIMIT`.

- HTTP request

POST  /v1/spot/trade/batch-orders

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID.                                               |
| async         | Boolean  | No           | Allowing asynchronous order placing. If `True`, {} will be returned. Default: `False`. |
| details       | Array    | Yes          | Detailed order information.                                  |
| >clOrdId      | String   | No           | Client-supplied order ID.                                    |
| >px           | String   | Optional     | Price                                                        |
| >baseQty      | String   | Optional     | Order quantity in base currency, applicable to non-market-buy orders. |
| >quoteQty     | String   | Optional     | Order quantity in quote currency, only applicable to market-buy orders. |
| >side         | Enum     | Yes          | Order side.                                                  |
| >ordType      | Enum     | Yes          | Order type: `LIMIT`.                                         |
| >flags        | String   | No           | Additional flag or multiple flags separated with comma.      |
| >timeInForce  | String   | No           | Time in force. Default: `GTC`.                               |

- Response parameters

| **Parameter** | **Type** | **Description**                                          |
| ------------- | -------- | -------------------------------------------------------- |
| ordId         | String   | Order ID. Applicable if order is successfully placed.    |
| clOrdId       | String   | Client-supplied order ID.                                |
| state         | String   | Order state. Applicable if order is successfully placed. |
| error         | Int      | Erro code. Returned if order is not successfully placed. |

## Cancel an Order

```json
{
    "code": "0",
    "msg": "",
    "data": 
        {
            "clOrdId": "cointrspot6",
            "ordId": "312269865356374016"
        }
}
```

```json
{
    "code": "0",
    "msg": "",
    "data": [ 
        {
            "clOrdId": "cointrspot6",
            "ordId": "312269865356374016"
        },
        {
            "clOrdId": "cointrspot6",
            "error": 12344
        }, ......
    ]
}
```

Cancel an active order.

- HTTP request

POST   /v1/spot/trade/cancel-order

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| ordId         | String   | Optional     | Order ID.                                                    |
| clOrdId       | String   | Optional     | Client-supplied order ID.                                    |
| async         | Boolean  | No           | Allowing asynchronous order canceling. If `True`, {} will be returned.<br /> Default: `False`. |

Either `orderId` or `clOrdId` must be sent. If both are sent, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| ordId         | String   | Order ID. Applicable if order is successfully canceled by this request. |
| clOrdId       | String   | Client-supplied order ID.                                    |

## Cancel Orders

Cancel active orders in batches. Maximum 20 orders can be canceled at a time. Request parameters should be passed in the form of an array.

- HTTP request

POST   /v1/spot/trade/cancel-batch-orders

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| ordIds        | String   | Optional     | Order ID or order IDs separated with comma.                  |
| clOrdIds      | String   | Optional     | Client-supplied order ID or client-supplied order IDs separated with comma. |
| async         | Boolean  | No           | Allowing asynchronous order canceling. If `True`, {} will be returned. <br />Default: `False`. |

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| ordId         | String   | Order ID. Applicable if order is successfully canceled by this request. |
| clOrdId       | String   | Client-supplied order ID.                                    |
| error         | Int      | Erro code. Returned if order is not successfully canceled by this request. |

## Cancel All Orders

Cancel all orders at a single request and {} will be returned.

- HTTP request

POST   /v1/spot/trade/cancel-all

- Request parameters

None.

- Response parameters

None.

## Query Active Orders

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
           "instId": "BTCUSDT",
            "ordId": "301835739059335168",
            "clOrdId": "myOrder1111",        
            "px": "59200",      
            "quoteQty": "100",
            "side": "buy",
            "accFillSz": "30",  
            "avgPx": "59199",            
            "state": "canceiling",
            "ordType": "market",
            "timeInForce": "ioc",
            "flags": "",          
            "cTime": 1618235248000,
            "uTime": 1618235248028
        }, ......
    ]
}
```

Retrieve details of all active orders under current account ordered by create time.

- HTTP request

GET  /v1/spot/trade/orders-active

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | No           | Instrument ID.<br />If instId is not sent, orders for all instruments will be returned in an array. |

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| instId        | String   | Instrument ID.                                               |
| ordId         | String   | Order ID.                                                    |
| clOrdId       | String   | Client-supplied order ID.                                    |
| px            | String   | Price.                                                       |
| baseQty       | String   | Order quantity in base currency, applicable to non-market-buy orders. |
| quoteQty      | String   | Order quantity in quote currency, only applicable to market-buy orders. |
| side          | Enum     | Order side.                                                  |
| ordType       | Enum     | Order type.                                                  |
| flags         | String   | Additional flag or multiple flags separated with comma.      |
| timeInForce   | Enum     | Time in force.                                               |
| accFillSz     | String   | Accumulated fill quantity.                                   |
| avgPx         | String   | Average filled price. If none if filled, 0 will be returned. |
| state         | String   | Order state.                                                 |
| cTime         | Long     | Create time, unix timestamp format in milliseconds.          |
| uTime         | Long     | Update time, unix timestamp format in milliseconds.          |

## Query Recent Orders

Retrieve order details. Supported orders include:

- active orders;

- completed orders whose create times are within the past 2 hours.

If illegal orders are sent, the error message "Order does not exist or was created more than 2 hours earlier from now." will be returned.

Max number of results per request: 10.

- HTTP request

GET  /v1/spot/trade/orders-recent

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| ordIds        | String   | Optional     | Order ID or multiple order IDs separated with comma.         |
| clOrdIds      | String   | Optional     | Client-supplied order ID or multiple client-supplied IDs separated with comma. |

Either `orderIds` or `clOrdIds` must be sent. If both are sent, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| instId        | String   | Instrument ID.                                               |
| ordId         | String   | Order ID.                                                    |
| clOrdId       | String   | Client-supplied order ID.                                    |
| px            | String   | Price.                                                       |
| baseQty       | String   | Order quantity in base currency, applicable to non-market-buy orders. |
| quoteQty      | String   | Order quantity in quote currency, only applicable to market-buy orders. |
| side          | Enum     | Order side.                                                  |
| ordType       | Enum     | Order type.                                                  |
| flags         | String   | Additional flag or multiple flags separated with comma.      |
| timeInForce   | Enum     | Time in force.                                               |
| accFillSz     | String   | Accumulated fill quantity.                                   |
| avgPx         | String   | Average filled price. If none if filled, 0 will be returned. |
| state         | String   | Order state.                                                 |
| cTime         | Long     | Create time, unix timestamp format in milliseconds.          |
| uTime         | Long     | Update time, unix timestamp format in milliseconds.          |

## Query Filled Orders

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
           "instId": "BTCUSDT",
            "ordId": "301835739059335168",
            "clOrdId": "myOrder1111",        
            "px": "59200",      
            "quoteQty": "100",
            "side": "buy",
            "accFillSz": "30",  
            "avgPx": "59199",            
            "state": "canceiling",
            "ordType": "market",
            "timeInForce": "ioc",
            "flags": "",          
            "cTime": 1618235248000,
            "uTime": 1618235248028
        }, ......
    ],
    "next": "1618235248028_301835739059335168"
}
```

Retrieve (partially) filled orders under current account ordered by update time.  Returned orders are limited to those whose update times are within the past 6 months.

- HTTP request

GET  /v1/spot/trade/orders-filled

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | No           | Instrument ID.<br />If instIds is not sent, orders for all instruments will be returned in an array. |
| ordIds        | String   | No           | Order ID or multiple order IDs separated with comma.         |
| after         | String   | No           | Pagination of data to return records with ranks are lower than the requested one. EXCLUSIVE. |
| startTime     | Long     | No           | Filter with a startTime, unix timestamp format in milliseconds.<br />Default: 3 days from current timestamp. |
| endTime       | Long     | No           | Filter with a endTime, unix timestamp format in milliseconds.<br />Default: current timestamp. |
| limit         | Int      | No           | Number of results per request. <br />Default: 50;<br />Max: 100. |

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| Data          | Array    | Requested data.                                              |
| >instId       | String   | Instrument ID.                                               |
| >ordId        | String   | Order ID.                                                    |
| >px           | String   | Price.                                                       |
| >baseQty      | String   | Order quantity in base currency, applicable to non-market-buy orders. |
| >quoteQty     | String   | Order quantity in quote currency, only applicable to market-buy orders. |
| >side         | Enum     | Order side.                                                  |
| >ordType      | Enum     | Order type.                                                  |
| >flags        | String   | Additional flag or multiple flags separated with comma.      |
| >timeInForce  | Enum     | Time in force.                                               |
| >accFillSz    | String   | Accumulated fill quantity.                                   |
| >avgPx        | String   | Average filled price. If none if filled, 0 will be returned. |
| >state        | String   | Order state.                                                 |
| >cTime        | Long     | Create time, unix timestamp format in milliseconds.          |
| >uTime        | Long     | Update time, unix timestamp format in milliseconds.          |
| next          | String   | Pagination of data.                                          |

## Query Trade History

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
           "tradeId": "301835739059335",
            "ordId": "301835739059335168",
            "px": "59200",      
            "sz": "0.1",
            "side": "BUY",
            "role": "TAKER",
            "fee": "0.0001",
            "mTime": 1618235248028
        }, ......
    ],
    "next": "1618235248028_301835739059335"
}
```

Retrieve trade details in the last 6 months.

- HTTP request

GET  /v1/spot/trade/fills

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID.                                               |
| after         | String   | No           | Pagination of data to return records with ranks are lower than the requested one. EXCLUSIVE. |
| startTime     | String   | No           | Filter with a startTime, unix timestamp format in milliseconds.<br />Default: 6 months from current timestamp. |
| endTime       | String   | No           | Filter with a endTime, unix timestamp format in milliseconds.<br />Default: current timestamp. |
| limit         | String   | No           | Number of results per request. <br />Default: 50;<br />Max: 100. |

- Response parameters

| **Parameter** | **Type** | **Description**                                    |
| ------------- | -------- | -------------------------------------------------- |
| data          | Array    | Requested data.                                    |
| >tradeId      | String   | Trade ID.                                          |
| >ordId        | String   | Order ID.                                          |
| >px           | String   | Price.                                             |
| >sz           | String   | Quantity purchased or sold.                        |
| >side         | String   | Trade side.                                        |
| >fee          | String   | Fee amount.                                        |
| >role         | Enum     | Role.                                              |
| >mTime        | String   | Match time, unix timestamp format in milliseconds. |
| next          | String   | Pagination of data.                                |

# Account

## Query Balance

```json
{
    "code": "0",
    "msg": "",
    "data": {} or []
}
{
    "code": "0",
    "msg": "", 
    "data": [
        {
            "acctId": "123455667",
            "uTime": 1623392334718, 
            "details": [
                {                           
                    "ccy": "BTC", 
                    "bal": "0.1", 
                    "availBal": "0.05", 
                    "frozenBal": "0.05"
                },
                {           
                    "ccy": "USDT", 
                    "bal": "1000", 
                    "availBal": "0", 
                    "frozenBal": "1000"                
                }
            ]
        }
    ]
}
```

Retrieve balance data of non-zero assets in the spot account.

- HTTP request

GET  /v1/spot/asset/balance

- Request parameters

There are 3 possible options:

| **Option**   | **Type** | **Examples**                                                 |
| ------------ | -------- | ------------------------------------------------------------ |
| No parameter |          | curl -X GET " https://api.cointr.vip/v1/spot/account/balance" |
| ccy          | String   | curl -X GET " https://api.cointr.vip/v1/spot/account/balance?ccy=BTC" |
| ccys         | String   | curl -X GET ' https://api.cointr.vip/v1/spot/account/balance?ccys=BTC,USDT"' |

If any ccy provided in either ccy or ccys do not exist,  or both ccy and ccys are provided,  the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| acctId        | String   | Account ID.                                                  |
| uTime         | Long     | Update time of position account, unix timestamp format in milliseconds. |
| details       | Array    | Detailed balance information of requested currency(s).       |
| >ccy          | String   | Currency.                                                    |
| >bal          | String   | Total balance of the currency.                               |
| >availBal     | String   | Available balance of the currency.                           |
| >frozenBal    | String   | Frozen balance of the currency.                              |

## Transfer

```json
{
  "code": "0",
  "msg": "",
  "data": [
    {
      "transId": "754147",
      "clId": "",
      "ccy": "USDT",
      "amt": "1000",
      "type": "SPOT_TO_FUTURE"
    }
  ]
}
```

Transfer funds between your spot account and futures account.

- HTTP request

POST  /v1/asset/transfer

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**           |
| ------------- | -------- | ------------ | ------------------------- |
| ccy           | String   | Yes          | Currency.                 |
| amt           | String   | Yes          | Amount to be transferred. |
| type          | Enum     | Yes          | Transfer type.            |
| clTransId     | String   | Yes          | Client-supplied ID.       |

- Response parameters

| **Parameter** | **Type** | **Description**     |
| ------------- | -------- | ------------------- |
| transId       | String   | Transfer ID.        |
| ccy           | String   | Currency.           |
| amt           | String   | Transfer amount.    |
| type          | Enum     | Transfer type.      |
| clTransId     | String   | Client-supplied ID. |

## Query Trade Fee

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "instId": "BTCUSDT",
            "taker": "-0.001",
            "maker": "0.00005"
        },
        {
            "instId": "USDTTRY",
            "taker": "-0.001",
            "maker": "0.0001"
        },
        {
            "instId": "OTHERS",
            "taker": "0.001",
            "maker": "0.001"
        }
    ]
}
```

- HTTP request

GET  /v1/spot/account/trade-fee

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instIds       | String   | No           | Istrument ID or multiple instrument IDs separated with comma.<br />If `instIds` is not sent, trade fee records of all instIds will be returned in an array. |

- Response parameters

| **Parameter** | **Type** | **Description**           |
| ------------- | -------- | ------------------------- |
| instId        | String   | Instrument ID or `OTHERS` |
| taker         | String   | Taker fee rate.           |
| maker         | String   | Maker fee rate.           |

# Websocket

## General info

Requests and responses use JSON.

- Request parameters

- Push data parameters

### Request format

Messages sent to the server should contain the following dictionary items:

- `op`: the operation to be runned. Should be one of the following:
  - `subscribe` to subscribe to a channel;
  - `unsubscribe` to unsubscribe from a channel.

- `channel`: the channel to subscribe or unsubscribe.

- `args`: channel parameters in JSON format.

### Response format

After subscription, unsubscription, or authentication request, response from the server will contain the following dictionary items:

- `event`: event happened. Should be one of the following:
  - `subscribe`: Indicates a successful subscription;
  - `unsubscribe`: Indicates a successful unsubscription;
  - `auth`: Indicates a successful authentication.
  - `error`: Occurs when there is an error. When `event` is `error`, there will also be `code` and `msg` fields instead of `channel` and `args`.

- `channel`

- `args`

- `code`

- `msg`

### Data push format

Data pushed from the server will contina the following dictionary items:

- `channel`: channel name.

- `instId` (optional): instrument ID.

- `action`: type of pushed data. Should be one of the following:
  - `initial`: indicates that data in the accompanying `data` field is a snapshot of current data;
  - `update`: indicates that data in the accompanying `data` field is incremental;
  - `snapshot`: indicates that data sent via this channel is always a snapshot of current data;
  - `resumed`: indicates that our system detects data loss(es) from the client side, and therefore the server will send a snapshot of current data in the accompanying `data` field. [<u>unavailable for now</u>].

- `data`

- `pTime`: data generated time, unix timestamp format in milliseconds.

## Authentication

```json
{
"op": "auth", 
"args": {
    "apiKey": "78fd833df18857aacddbcc5ff3c6c34a", 
    "timestamp": "1657770908268", 
    "sign": "f9eb6f9ad6a49c231d23c2a239f5a71912093a0afdc9a1dc1ef9d723636194a2"
    }
}
```

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                |
| ------------- | -------- | ------------ | ---------------------------------------------- |
| op            | Enum     | Yes          | Operation: `auth`                              |
| args          | Array    | Yes          | Request parameters.                            |
| >apiKey       | String   | Yes          | API access key.                                |
| >timestamp    | String   | Yes          | Current timestamp in milliseconds.             |
| >sign         | String   | Yes          | SHA256 HMAC operation details are shown below. |

Our signature algorithm involves two steps of `HMAC SHA256` operations:

1. `HMAC SHA256` of the following string, with your API secret key as the key: `currentTime` (unix timestamp format in milliseconds) //30000

1. With the value obtained from `HMAC SHA256` operation in step 1 as the key, perform `HMAC SHA256` operation on ~~`accessKey={your_api_access_key}&`~~`timestamp={currentTime}`

## Candlesticks

```json
{
"channel": "kline",
"instId": "BTCUSDT",
"action": "update",
"data": 
    [
        [
            1597026383085,
            "8533.02",
            "8553.74",
            "8527.17",
            "8548.26",
            "529.5858061",
            "45247"
        ]
    ],
"pTime": 1597026388000
}
```

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `kline_spot`                                   |
| args          | Array    | Yes          | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID                                                |
| >bar          | Enum     | Yes          | Kline bar.                                                   |
| >initialNum   | Int      | No           | The number of recent klines sent at `inital` push.<br />Default: 0<br />Max: 200 |

- Push data parameters

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| channel       | String   | Channel name.                                     |
| instId        | String   | Instrument ID.                                    |
| action        | String   | Push data action.                                 |
| data          | Array    | Subscribed data.                                  |
| >ts           | Int      | Generated time,  unix timestamp format in seconds |
| >o            | String   | Open price                                        |
| >h            | String   | Highest price                                     |
| >l            | String   | Lowest price                                      |
| >c            | String   | Close price                                       |
| >vol          | String   | Trading volume in base currency                   |
| >quoteVol     | String   | Trading volume in quote currency                  |
| pTime         | String   | Push time, unix timestamp format in milliseconds  |

## Mini Order Books

```json
{
    "channel": "mini_books",
    "instId": "BTCUSDT", 
    "action": "snapshot", 
    "data": 
        {
            "asks": ["8476.98", "415"],
            "bids": ["8476.97", "256"]              
        }, 
    "pTime": "1597026383085"
}
```

Retrieve best ask and bid.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `mini_books_spot`                              |
| args          | Array    | Yes          | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID                                                |
| >updateSpeed  | Int      | No           | Time intervals of pushes in milliseconds.<br />Default: 500<br />Min: 300. |

- Push data parameters

| **Parameter** | **Type** | **Description**                                  |
| ------------- | -------- | ------------------------------------------------ |
| channel       | String   | Channel name.                                    |
| instId        | String   | Instrument ID.                                   |
| action        | Enum     | `snapshot`                                       |
| data          | Array    | Subscribed data.                                 |
| >ask          | Array    | Best ask.                                        |
| >bid          | Array    | Best bid.                                        |
| pTime         | Long     | Push time, Unix timestamp format in milliseconds |

## Order Books

```json
{
    "channel": "books",
    "instId": "BTCUSDT", 
    "data": 
        {
            "asks": [
                ["8476.98", "415"],
                ["8477", "7"],
                ["8477.34", "85"],
                ["8477.56", "1"],
                ["8505.84", "8"],
                ["8506.37", "85"],
                ["8506.49", "2"],
                ["8506.96", "100"]
                ],
            "bids": [
                ["8476.97", "256"],
                ["8475.55", "101"],
                ["8475.54", "100"],
                ["8475.3", "1"],
                ["8447.32", "6"],
                ["8447.02", "246"],
                ["8446.83", "24"],
                ["8446", "95"]
                ]
        }, 
    "pTime": 1597026383085
}
```

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| :------------ | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `books_spot`                                   |
| args          | Array    | Yes          | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID                                                |
| >step         | String   | Yes          | Arregated step.                                              |
| >limit        | Int      | No           | Order book depth per side.<br />Default: 5;<br />Max: 20.    |
| >updateSpeed  | Int      | No           | Time intervals of pushes in milliseconds.<br />Default: 500;<br />Min: 300. |

- Push data parameters

| **Parameter** | **Type** | **Description**                                  |
| ------------- | -------- | ------------------------------------------------ |
| channel       | String   | Channel name.                                    |
| instId        | String   | Instrument ID.                                   |
| action        | Enum     | `snapshot`                                       |
| data          | Array    | Subscribed data.                                 |
| >asks         | Array    | Order book on sell side.                         |
| >bids         | Array    | Order book on buy side.                          |
| pTime         | Long     | Push time, Unix timestamp format in milliseconds |

## Trades

```json
{
  "channel": "trades",
  "instId": "BTCUSDT",
  "action": "update",
  "data": [
    {
      "tradeId": "130639474",
      "px": "42219.9",
      "sz": "0.12060306",
      "side": "buy",
      "mTime": 1630048897897
    }
  ]
}
```

Retrieve the recent trades data. Data will be pushed whenever there is a trade.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`.                       |
| channel       | Enum     | Yes          | Channel name: `trades_spot`.                                 |
| args          | Array    | No           | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID.                                               |
| >initialNum   | Int      | No           | The number of recent trades pushed at the `initial` push.<br />Default: 50.<br/>Max: 200 |

- Push data parameters

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| channel       | String   | Channel name.                                     |
| instId        | String   | Instrument ID.                                    |
| action        | String   | Push data action: `initial`, `update`             |
| data          | Array    | Subscribed data.                                  |
| >mTime        | Long     | Match time, unix timestamp format in milliseconds |
| >tradeId      | String   | Trade ID.                                         |
| >px           | String   | Trade price.                                      |
| >sz           | String   | Trade size in base currency                       |
| >side         | Enum     | Trade direction of taker.                         |

## User Data

Balance, order, and fill updates will be sent separately via this channel.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                       |
| ------------- | -------- | ------------ | ------------------------------------- |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe` |
| channel       | Enum     | Yes          | Channel name: `user_data_spot`.       |

- Push data parameters

-balance

Balance data will be pushed when triggered by events such as filled order, funding transfer. At `initial` push, Only currencies with non-zero balance will be sent. Events include `order`,  `transfer`, `deposit`, `withdraw`, `commission_rebate`, and `others` to which things like welcome_bonus, referral_kickback, airdrop belong.

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| channel       | String   | `user_data_spot@balance_spot`                     |
| acctId        | String   | Account ID.                                       |
| action        | String   | Push data action: `initial`, `update`             |
| data          | Array    | Subscribed data.                                  |
| >balData      | Array    | Balance data.                                     |
| >>ccy         | String   | Currency.                                         |
| >>availBal    | String   | Available balance of the currency.                |
| >>frozenBal   | String   | Frozen balance of the currency.                   |
| pTime         | String   | Push time, Unix timestamp format in milliseconds. |

1. -orders

Active orders will be pushed at `initial` push. `update` data will be pushed when triggered by the following event types:

`new`: An order has been accepted into the engine.

`canceled` : An order has been canceled by the user.

`filled`:  Part of the order or all of the order's quantity has filled.

`expired`:  An order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance).

`stopped`: An order was stopped for some reason.

`triggered`: A conditional order is triggered.

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| channel       | String   | `user_data_spot@orders_spot`                                 |
| acctId        | String   | Account ID.                                                  |
| action        | String   | Push data action: `initial`, `update`                        |
| data          | Array    | Subscribed data.                                             |
| >instId       | String   | Instrument ID                                                |
| >ordId        | String   | Order ID.                                                    |
| >clOrdId      | String   | Client-supplied order ID.                                    |
| >px           | String   | Price.                                                       |
| >baseQty      | String   | Order quantity in base currency, applicable to non-market-buy orders. |
| >quoteQty     | String   | Order quantity in quote currency, only applicable to market-buy orders. |
| >side         | Enum     | Order side.                                                  |
| >ordType      | Enum     | Order type.                                                  |
| >flags        | String   | Additional flag or multiple flags separated with comma.      |
| >timeInForce  | Enum     | Time in force.                                               |
| >accFillSz    | String   | Accumulated fill quantity.                                   |
| >avgPx        | String   | Average filled price. If none if filled, 0 will be returned. |
| >state        | String   | Order state.                                                 |
| >cTime        | Long     | Create time, unix timestamp format in milliseconds.          |
| pTime         | Long     | Update time, unix timestamp format in milliseconds.          |

-fills

Data will not be pushed at `initial` push. Data will only be pushed when order(s) is filled.

| **Parameter** | **Type** | **Description**                                    |
| ------------- | -------- | -------------------------------------------------- |
| channel       | String   | `user_data_spot@fills_spot`                        |
| acctId        | String   | Account ID.                                        |
| action        | String   | Push data action: `initial`, `update`              |
| data          | Array    | Subscribed data.                                   |
| >instId       | String   | Instrument ID                                      |
| >tradeId      | String   | Trade ID.                                          |
| >ordId        | String   | Order ID.                                          |
| >px           | String   | Price.                                             |
| >sz           | String   | Quantity to buy or sell.                           |
| >side         | String   | Trade side.                                        |
| >fee          | String   | Fee.                                               |
| >role         | Enum     | Role.                                              |
| pTime         | String   | Match time, unix timestamp format in milliseconds. |
