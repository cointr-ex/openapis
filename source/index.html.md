---
title: CoinTR API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json
toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:
  - errors

active: active


spot_goods: Spot

spot_goods_url: 'spot.html'

contract: Futures

contract_active: active

contract_url: 'index.html'

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the CoinTR Futures API
---

# General Info

## Terminology

- Position mode (posMode): `LONG_SHORT_MODE`, `NET_MODE`

- Margin mode (mgnMode): `CROSS`, `ISOLATED`

- Position side (posSide): `LONG`, `SHORT`

- Order flags: `POST_ONLY`, `REDUCE_ONL`

# Market Data

## Contract Specification

```json
{
  "code": 0,
  "data": [
    {
      "baseCcy": "ETH",
      "ctVal": "0.01",
      "defLever": "125",
      "instId": "ETHUSDT",
      "maxLever": "125",
      "maxPos": "100000",
      "maxSz": "0",
      "mgnMode": "cross",
      "minMmr": "0",
      "minSz": "0",
      "pxPrecision": "2",
      "quoteCcy": "USDT",
      "settleCcy": "USDT",
      "steps": "0.01,0.1,1,10",
      "tickSz": "12"
    }
  ]
}
```

- HTTP request

GET  /v1/futures/public/instruments

- Request parameters

There are 3 possible options:

| **Option**   | **Type** | **Examples**                                                 |
| ------------ | -------- | ------------------------------------------------------------ |
| No parameter |          | curl -X GET " https://api.cointr.vip/v1/futures/public/instruments" |
| instId       | String   | curl -X GET " https://api.cointr.vip/v1/futures/public/instruments?instId=USDTTRY" |
| instIds      | String   | curl -X GET " https://api.cointr.vip/v1/futures/public/instruments?instIds=USDTTRY,BTCUSDT" |

If any instId provided in either instId or instIds does not exist,  or both instId and instIds are provided,  the endpoint will throw an error. If neither is sent, specifications regarding all instruments will be returned.

- Response parameters

| **Parameter** | **Type** | **Description**                 | **Example**               |
| ------------- | -------- | ------------------------------- | ------------------------- |
| instId        | String   | Instrument ID                   | BTCUSDT                   |
| settleCcy     | String   | Settlement and margin currency  | USDT                      |
| ctVal         | String   | Contract value                  | 0.01                      |
| baseCcy       | String   | Contract base currency          | BTC                       |
| quoteCcy      | String   | Contract quote currency         | USDT                      |
| maxLever      | String   | Max leverage                    | 125                       |
| tickSz        | String   | Tick size.                      | 0.1                       |
| pxPrecision   | Int      | Price precision.                | 1                         |
| minSz         | String   | Minimum order size (cont).      | 1                         |
| maxSz         | String   | Maximum order size (cont).      | 1000                      |
| maxPos        | String   | Max position allowed (cont).    | 1000                      |
| steps         | String   | Aggregated steps in order book. | [0.001, 0.01, 0.1, 1, 10] |
| minMmr        | String   | Minimum maintenance margin rate | 2%                        |
| mgnMode       | Enum     | Default margin mode.            | `CROSS`                   |
| defLever      | Int      | Default leverage.               | 20                        |

## Market Risk Limit

```json
{
  "code": 0,
  "data": [
    {
      "maxLever": 125,
      "maxSz": "1000",
      "mmgnRatio": "0.004"
    },......,
    {
      "maxLever": 2,
      "maxSz": "9999999999",
      "mmgnRatio": "0.25"
    }
  ]
}
```

- HTTP request

GET  /v1/futures/public/position-tiers

- Request parameters

| **Parameter** | **Type** | **Required** | **Description** |
| ------------- | -------- | ------------ | --------------- |
| instId        | String   | Yes          | Instrument  ID. |

- Response parameters

| **Parameter** | **Type** | **Description**                                      | **Example** |
| ------------- | -------- | ---------------------------------------------------- | ----------- |
| maxSz         | String   | Bound bound of the position range of a teir (cont.). | 5000        |
| mMgnRatio     | String   | Maintenance margin ratio.                            | 0.03        |
| maxLever      | Int      | Maximum leverage allowed.                            | 20          |

## Candlesticks

```json
{
  "code": 0,
  "data": [
    [
        "1597026383085",        //open time
        "3.721",                //open price
        "3.743",                //highest price
        "3.677",                //lowest price
        "3.708",                //close price
        "8422410",              //the number of contracts traded
        "22698348.04828491"     //trading volume in quote currency
    ],
    [
        "1597026383085",
        "3.731",
        "3.799",
        "3.494",
        "3.72",
        "24912403",
        "67632347.24399722"
    ]
  ],
  "message": "success"
}
```

- HTTP request

GET  /v1/futures/market/candlesticks

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| bar           | Enum     | Yes          | Bar size.Default: `1M`.                                      |
| endTs         | String   | No           | Filter with a endTs, unix timestamp format in seconds.<br />Default: current timestamp. |
| limit         | Int      | No           | Number of results per request.<br />Default: 100;<br />Max: 2000. |

Data is limited to the most recent 2000 ones. If limit sent excedes the maximum requirement of 2000, the most recent 2000 ones if available will be returned.

- Response parameters

| **Parameter** | **Type** | **Description**                                     |
| ------------- | -------- | --------------------------------------------------- |
| ts            | Long     | Kline start time, unix timestamp format in seconds. |
| o             | String   | Open price.                                         |
| h             | String   | Highest price.                                      |
| l             | String   | Lowest price.                                       |
| c             | String   | Close price.                                        |
| vol           | String   | The number of contracts traded.                     |
| quoteVol      | String   | Trading volume in quote currency.                   |

## OrderBook

```json
{
  "code": 0,
  "data": {
    "asks": [
      [
        "1600",    //depth price
        "298"      //the number of contracts at the price 
      ]
    ],
    "bids": [
      [
        "1450",
        "2"
      ]
    ],
    "utime": 1658917769615
  },
  "message": "success"
}
```

- HTTP request

GET  /v1/futures/market/depths

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| limit         | Int      | No           | Order book depth per side. <br />Default 1; <br />valid limits: [1, 5, 10, 20, 50, 100]. |

- Response parameters

| **Parameter** | **Type** | **Description**                                     |
| ------------- | -------- | --------------------------------------------------- |
| asks          | Array    | Order book on sell side                             |
| bids          | Array    | Order book on buy side                              |
| uTime         | Long     | Update time, unix timestamp format in milliseconds. |

## Funding Rate

```json
{
  "code": 0,
  "data": {
    "fundingRate": "-0.003",
    "fundingTime": 1659052800000,
    "nextFundingRate": "-0.003"
  },
  "message": "success"
}
```

- HTTP request

GET  /v1/futures/market/funding-rate

- Request parameters

| **Parameter** | **Type** | **Required** | **Description** |
| ------------- | -------- | ------------ | --------------- |
| instId        | String   | Yes          | Instrument ID.  |

- Response parameters

| **Parameter**   | **Type** | **Description**                                         |
| --------------- | -------- | ------------------------------------------------------- |
| fundingRate     | String   | Current funding rate.                                   |
| nextFundingRate | String   | Estimated funding rate for the next period.             |
| fundingTime     | Long     | Settlement time, unix timestamp format in milliseconds. |

## Market Snapshot

```json
{
  "code": 0,
  "data": [
    {
      "askPx": "1600",
      "askSz": "298",
      "bidPx": "1559",
      "bidSz": "2",
      "high24h": "1680",
      "index": "1720.39",
      "instId": "ethusdt",
      "lastPx": "1450",
      "lastSz": "1",
      "low24h": "1202.21",
      "mPx": "1715.61",
      "oi": "200343",
      "open24h": "0",
      "uTime": 1659026160608,
      "vol24h": "215748",
      "volCcy24h": "21574.8"
    }
  ],
  "message": "success"
}
```

- HTTP request

GET  /v1/futures/market/tickers

- Request parameters

There are 3 possible options:

| **Option**   | **Type** | **Examples**                                                 |
| ------------ | -------- | ------------------------------------------------------------ |
| No parameter |          | curl -X GET " https://api.cointr.vip/v1/futures/market/tickers" |
| instId       | String   | curl -X GET " https://api.cointr.vip/v1/futures/market/tickers?instId=USDTTRY" |
| instIds      | List     | curl -X GET " https://api.cointr.vip/v1/futures/market/tickers?instIds=USDTTRY,BTCUSDT" |

If any instId provided in either instId or instIds do not exist,  or both instId and instIds are provided,  the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                     |
| ------------- | -------- | --------------------------------------------------- |
| instId        | String   | Instrument ID                                       |
| oi            | String   | Open interest (cont.).                              |
| mPx           | String   | Mark price.                                         |
| index         | String   | Index.                                              |
| lastPx        | String   | Last traded price.                                  |
| lastSz        | String   | Last traded size (cont.).                           |
| askPx         | String   | Best ask price                                      |
| askSz         | String   | Best ask size (cont.).                              |
| bidPx         | String   | Best bid price                                      |
| bidSz         | String   | Best bid size (cont.).                              |
| open24h       | String   | Open price in the past 24 hours                     |
| high24h       | String   | Highest price in the past 24 hours                  |
| low24h        | String   | Lowest price in the past 24 hours                   |
| vol24h        | String   | The number of contracts traded in the past 24 hours |
| volCcy24h     | String   | 24h trading volume in base currency                 |
| uTime         | Long     | Update time, unix timestamp format in milliseconds  |

## Recent Trades

```json
{
  "code": 0,
  "data": [
    {
      "mTime": 1658906727120,
      "px": "1450",
      "side": "sell",
      "sz": "1",
      "tradeId": "2"
    },
    {
      "mTime": 1658913721271,
      "px": "1450",
      "side": "buy",
      "sz": "1",
      "tradeId": "3"
    }
  ],
  "message": "success"
}
```



Retrieve the recent transactions of an instrument.

- HTTP request

GET  /v1/futures/market/trades

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| limit         | Int      | No           | Number of results per request.<br />Default: 100;<br />Max: 500. |

- Response parameters

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| tradeId       | String   | Trade ID                                          |
| px            | String   | Trade price                                       |
| sz            | String   | Trade quantity in base currency                   |
| side          | String   | Trade side of taker.                              |
| mTime         | Long     | Match time, unix timestamp format in milliseconds |

# Trading

## Place an Order

```json
{
  "code": 0,
  "data": {
    "clOrdId": "12341ADK5578905123",
    "ordId": 1739615825821697,
    "state": "SUBMITTED"
  }
}
```

- HTTP request

POST  /v1/futures/trade/order

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| mgnMode       | Enum     | No           | Margin mode.                                                 |
| lever         | Int      | No           | Leverage.                                                    |
| clOrdId       | String   | Yes          | Client-supplied order ID.                                    |
| px            | String   | No           | Price.                                                       |
| sz            | String   | Yes          | Order size (count).                                          |
| side          | Enum     | Yes          | Order side.                                                  |
| ordType       | Enum     | Yes          | Order type: `MARKET`, `LIMIT`                                |
| flags         | String   | Yes          | Additional flag or multiple flags separated with comma.      |
| timeInForce   | Enum     | No           | Time in force. Default: `GTC`.                               |
| async         | Boolean  | No           | Allowing asynchronous order placing. If `True`, {} will be returned.  Default: `False`. |

`mgnMode` and `lever` are for verification purpose only. If `mgnMode` and/or `lever` sent  are not consistent with trading configurations of the requested `instId`, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                          |
| ------------- | -------- | -------------------------------------------------------- |
| ordId         | Int      | Order ID. Applicable if order is successfully placed.    |
| clOrdId       | String   | Client-supplied order ID.                                |
| state         | Enum     | Order state. Applicable if order is successfully placed. |

## Place Orders

```json
{
  "code": 0,
  "data": [
    {
      "clOrdId": "094732",
      "ordId": 1739616027148289,
      "state": "SUBMITTED"
    },
    {
      "clOrdId": "094733",
      "ordId": 1739616027148290,
      "state": "SUBMITTED"
    }
  ]
}
```

- HTTP request

POST  /v1/futures/trade/batch-orders

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| mgnMode       | Enum     | No           | Margin mode.                                                 |
| lever         | Int      | No           | Leverage.                                                    |
| async         | Boolean  | No           | Allowing asynchronous order placing. If `True`, {} will be returned.  Default: `False`. |
| details       | Array    | Yes          | Detailed order information.                                  |
| >clOrdId      | String   | Yes          | Client-supplied order ID.                                    |
| >px           | String   | No           | Price.                                                       |
| >sz           | String   | Yes          | Order size (count).                                          |
| >side         | Enum     | Yes          | Order side.                                                  |
| >ordType      | Enum     | Yes          | Order type: `LIMIT`                                          |
| >flags        | String   | Yes          | Additional flag or multiple flags separated with comma.      |
| >timeInForce  | Enum     | No           | Time in force. Default: `GTC`.                               |

- Response parameters

| **Parameter** | **Type** | **Description**                                          |
| ------------- | -------- | -------------------------------------------------------- |
| ordId         | Int      | Order ID. Applicable if order is successfully placed.    |
| clOrdId       | String   | Client-supplied order ID.                                |
| state         | Enum     | Order state. Applicable if order is successfully placed. |
| error         | Int      | Erro code. Returned if order is not successfully placed. |

## Cancel and Order

```json
{
  "code": 0,
  "data": {
    "clOrdId": "094733",
    "ordId": 1739616027148290,
    "state": "CANCELED"
  }
}
```

- HTTP request

POST   /v1/futures/trade/cancel-order

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| ordId         | String   | Optional     | Order ID.                                                    |
| clOrdId       | String   | Optional     | Client-supplied order ID.                                    |
| async         | Boolean  | No           | Allowing asynchronous order canceling. If `True`, {} will be returned. Default: `False`. |

Either `orderId` or `clOrdId` must be sent. If both are sent, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| ordId         | String   | Order ID. Applicable if order is successfully canceled by this request. |
| clOrdId       | String   | Client-supplied order ID.                                    |

## Cancel Orders

```json
{
  "code": 0,
  "data": [
    {
      "clOrdId": "12341ADK5578905123",
      "ordId": 1739615825821697
    },
    {
      "clOrdId": "0947akfsw33",
      "ordId": 1739616638468097
    }
  ]
}
```

Cancel active orders in batches. Maximum 20 orders can be canceled at a time. Request parameters should be passed in the form of an array.

- HTTP request

POST   /v1/futures/trade/cancel-batch-orders

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID                                                |
| ordIds        | String   | Optional     | Order ID or order IDs separated with comma.                  |
| clOrdIds      | String   | Optional     | Client-supplied order ID or client-supplied order IDs separated with comma. |
| async         | Boolean  | No           | Allowing asynchronous order canceling. If `True`, {} will be returned. Default: `False`. |

Either `orderId` or `clOrdId` must be sent. If both are sent, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| ordId         | String   | Order ID. Applicable if order is successfully canceled by this request. |
| clOrdId       | String   | Client-supplied order ID.                                    |
| error         | Int      | Erro code. Returned if order is not successfully canceled by this request. |

## Cancel All Orders

```json
{
  "code": 0
}
```

Cancel all orders at a single request and {} will be returned.

- HTTP request

POST   /v1/futures/trade/cancel-all

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | No           | Instrument ID.If instId is not sent, active orders for all instruments will be canceled. |

- Response parameters

None.

## Query Active Orders

```json
{
  "code": 0,
  "data": [
    {
      "accFillSz": "0",
      "avgPx": "0",
      "clOrdId": "094732",
      "ctime": 1659027125482,
      "flags": "POST_ONLY",
      "instId": "BTCUSDT",
      "ordId": "1739616027148289",
      "ordType": "LIMIT",
      "px": "20000",
      "side": "BUY",
      "state": "SUBMITTED",
      "sz": "1",
      "timeInForce": "GTC",
      "utime": 1659027125482
    }, ......
  ]
}
```

Retrieve details of all active orders under current account ordered by create time.

- HTTP request

GET  /v1/futures/trade/orders-active

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | No           | Instrument ID.If instId is not sent, orders for all instruments will be returned in an array. |

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| instId        | String   | Instrument ID                                                |
| ordId         | String   | Order ID.                                                    |
| clOrdId       | String   | Client-supplied order ID.                                    |
| px            | String   | Price.                                                       |
| sz            | String   | Order size (count).                                          |
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

```json
{
  "code": "0",
  "data": [
    {
      "accFillSz": "0",
      "avgPx": "0",
      "clOrdId": "094732",
      "ctime": "1659027125000",
      "flags": "POST_ONLY",
      "instId": "BTCIUSD",
      "ordId": "1739616027148289",
      "ordType": "market",
      "px": "20000",
      "side": "buy",
      "state": "submitted",
      "sz": "1",
      "timeInForce": "gtc",
      "utime": "1659027125000"
    }
  ],
  "msg": ""
}
```

Retrieve order details. Supported orders include:

- active orders;

- completed orders whose create times are within the past 2 hours.

If illegal orders are sent, the error message "Order does not exist or was created more than 2 hours earlier from now." will be returned.

Max number of results per request: 10.

- HTTP request

GET  /v1/futures/trade/orders-recent

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| ordIds        | String   | Optional     | Order ID or multiple order IDs separated with comma.         |
| clOrdIds      | String   | Optional     | Client-supplied order ID or multiple client-supplied IDs separated with comma. |

Either `orderIds` or `clOrdIds` must be sent. If both are sent, the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| instId        | String   | Instrument ID                                                |
| ordId         | String   | Order ID.                                                    |
| clOrdId       | String   | Client-supplied order ID.                                    |
| px            | String   | Price.                                                       |
| sz            | String   | Order size (count).                                          |
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
  "data": [
    {
      "accFillSz": "1",
      "avgPx": "21800",
      "clOrdId": "df4a70f4-4531-4b4b-9828-79de6a39c2fb",
      "ctime": "1658731459000",
      "flags": "",
      "instId": "BTCUSDT",
      "ordId": "1739305998876673",
      "ordType": "limit",
      "px": "21800",
      "side": "buy",
      "state": "canceled",
      "sz": "2",
      "timeInForce": "gtc",
      "utime": "1659028705902"
    }, ......
  ],
  "msg": "",
  "next": "1739305998876673"
}
```

Retrieve (partially) filled orders under current account ordered by update time.  Returned orders are limited to those whose update times are within the past 6 months.

- HTTP request

GET  /v1/futures/trade/orders-filled

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | No           | Instrument ID.If instIds is not sent, orders for all instruments will be returned in an array. |
| ordIds        | String   | No           | Order ID or multiple order IDs separated with comma.         |
| after         | String   | No           | Pagination of data to return records with ranks lower than the requested one. EXCLUSIVE. |
| startTime     | Long     | No           | Filter with a startTime, unix timestamp format in milliseconds.Default: 3 days from current timestamp. |
| endTime       | Long     | No           | Filter with a endTime, unix timestamp format in milliseconds.Default: current timestamp. |
| limit         | Int      | No           | Number of results per request. Default: 50;Max: 100.         |

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| data          | Array    | Requested data.                                              |
| >instId       | String   | Instrument ID                                                |
| >ordId        | String   | Order ID.                                                    |
| >px           | String   | Price.                                                       |
| >sz           | String   | Order size (count).                                          |
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
  "code": 0,
  "data": [
    {
      "fee": "0.51935",
      "instId": "BTCIUSD",
      "mtime": 1657873019048,
      "ordId": "1738405854052353",
      "px": "20774",
      "role": "maker",
      "selfTradeQty": "1",
      "side": "buy",
      "sz": "1",
      "tradeId": "1"
    },
    {
      "fee": "1.24644",
      "instId": "BTCIUSD",
      "mtime": 1657873019048,
      "ordId": "1738405859295233",
      "px": "20774",
      "role": "taker",
      "selfTradeQty": "1",
      "side": "sell",
      "sz": "-1",
      "tradeId": "1"
    }, ......
  ]
}
```

Retrieve trade details in the last 6 months.

- HTTP request

GET  /v1/futures/trade/fills

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instId        | String   | Yes          | Instrument ID.                                               |
| after         | String   | No           | Pagination of data to return records with ranks lower than the requested one. EXCLUSIVE. |
| startTime     | String   | No           | Filter with a startTime, unix timestamp format in milliseconds.Default: 6 months from current timestamp. |
| endTime       | String   | No           | Filter with a endTime, unix timestamp format in milliseconds.Default: current timestamp. |
| limit         | String   | No           | Number of results per request. Default: 50;Max: 100.         |

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
| >selfTradeQty | String   | Self-traded quantity.                              |
| >mTime        | Long     | Match time, unix timestamp format in milliseconds. |
| next          | String   | Pagination of data.                                |

# Account

## Query Account Configuration

```json
{
  "code": 0,
  "data": {
    "acctId": "100002",
    "posMode": "NET_MODE"
  }
}
```

Retrieve current account configurations.

- HTTP request

GET  /v1/futures/account/config

- Request parameters

None

- Response parameters

| **Parameter** | **Type** | **Description** |
| ------------- | -------- | --------------- |
| acctId        | String   | Account ID.     |
| posMode       | Enum     | Position mode   |

## Set Position Mode

```json
{
  "code": 0
}
```

Change the position mode of current account, applicable to every instrument.

- HTTP request

POST  /v1/futures/account/set-position-mode

- Request parameters

| **Parameter** | **Type** | **Required** | **Description** |
| ------------- | -------- | ------------ | --------------- |
| posMode       | Enum     | Yes          | Position mode.  |

- Response parameters

> NONE

## Query Trade Fee

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "instId": "BTCUSDT",
            "taker": "-0.001",
            "maker": "0.00000"
        },
        {
            "instId": "ETHUSDT",
            "taker": "-0.001",
            "maker": "0.0000"
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

GET  /v1/futures/account/trade-fee

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instIds       | String   | No           | Istrument ID or multiple instrument IDs separated with comma.If `instIds` is not sent, trade fee records of all instIds will be returned in an array. |

- Response parameters

| **Parameter** | **Type** | **Description**           |
| ------------- | -------- | ------------------------- |
| instId        | String   | Instrument ID or `OTHERS` |
| taker         | String   | Taker fee rate.           |
| maker         | String   | Maker fee rate.           |

## Query Trade Configuration

```json
{
    "code": "0",
    "msg": "",
    "data": [
        {
            "mgnMode": "cross", 
            "lever": "10"
        }
    ]
}
```

Retrieve current instrument-specific trade settings.

- HTTP request

GET  /v1/futures/trade/config

- Request parameters

| **Parameter** | **Type** | **Required** | **Description** |
| ------------- | -------- | ------------ | --------------- |
| instId        | String   | Yes          | Instrument ID.  |

- Response parameters

| **Parameter** | **Type** | **Description** |
| ------------- | -------- | --------------- |
| mgnMode       | String   | Margin mode.    |
| lever         | String   | Leverage.       |

If any instId provided in either instId or instIds is not configured yet, default settings will be returned.

## Set Trade Configuration

```json
{
  "code": 0
}
```

Set leverage and margin mode for at instrument level.

- HTTP request

POST  /v1/futures/trade/set-config

- Request parameters

| **Parameter** | **Type** | **Required** | **Description** |
| ------------- | -------- | ------------ | --------------- |
| instId        | String   | Yes          | Instrument ID.  |
| lever         | String   | Yes          | Leverage.       |
| mgnMode       | String   | Yes          | Margin mode.    |

- Response parameters

NONE.

## Query Balance

```json
{
  "code": 0,
  "data": [
    {
      "acctid": 100002,
      "cashBal": "19.6360404748",
      "ccy": "USDT",
      "details": [
        {
          "cashBal": 0.18748,
          "eq": 1.80931,
          "mmr": 0.0872,
          "posIMR": 0.18748
        }
      ],
      "freeBal": "0",
      "ordIMR": "0.18748",
      "totalEq": 21.2578704748
    }, ......
  ]
}
```

- HTTP request

GET  /v1/futures/account/balance

- Request parameters

NONE.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| uTime         | String   | Update time of account information, unix timestamp format in milliseconds. |
| acctId        | String   | Account ID.                                                  |
| ccy           | String   | Currency.                                                    |
| totalEq       | String   | Total equity.                                                |
| cashBal       | String   | Cash Balance.                                                |
| freeBal       | String   | Cash balance that could be transfered.                       |
| ordIMR        | String   | Equity frozen for open orders.                               |
| details       | Array    | Detailed account information across existing position accounts. |
| >posId        | String   | Position ID. Balance details of position IDs with `CROSS` margin mode will be returned as a whole, in which case "" will be returned. |
| >eq           | String   | Equity.                                                      |
| >cashBal      | String   | Cash balance.                                                |
| >posIMR       | String   | Initial margin required.                                     |
| >mmr          | String   | Maitance margin requirement.                                 |

## Adjust Margin

```json
{
  "code": 0
}
```

Transfer margin from or to cross-margined position account.

- HTTP request

POST /v1/futures/account/margin

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| posId         | String   | Yes          | Position ID. posId of cross-margined position account is ilegal. |
| amt           | String   | Yes          | Amount to be transferred. Positive amount indicates margin increase of posId, and negative one indicates decrease. |

- Response parameters

NONE.

## Query Positions

```json
{
  "code": 0,
  "data": [
    {
      "avgPx": 24000,
      "cashBal": "0.18748",
      "instId": "BTCUSDT",
      "lever": 100,
      "liqPx": "2251.16",
      "mgnMode": "CROSS",
      "mmr": "0.0872",
      "posId": 18,
      "posSide": "LONG",
      "posSz": 1,
      "upl": "1.65341"
    }, ......
  ]
}
```

- HTTP request

GET  /v1/futures/account/positions

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| instIds       | String   | Optional     | Instrument ID or multiple instrument IDs separated with comma. |
| posIds        | String   | Optional     | Position ID or multiple position IDs separated with comma.   |

Either `instIds` or `posIds` must be sent; if both are sent , the endpoint will throw an error.

- Response parameters

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| posId         | String   | Position ID.                                                 |
| mgnMode       | Enum     | Margin mode.                                                 |
| instId        | String   | Instrument ID.                                               |
| posSide       | Enum     | Position side.                                               |
| posSz         | String   | Quantity of positions (cont).                                |
| avgPx         | String   | Average open price.                                          |
| liqPx         | String   | Estimated liquidation price.                                 |
| lever         | Int      | Leverage.                                                    |
| mmr           | String   | Maintenance margin requirement.                              |
| upl           | String   | Unrealized PnL.                                              |
| cashBal       | String   | Cash balance. If mgnMode is `CROSS`, cash balance allocated to all `CROSS` positions will be returned. If mgnMode is `ISOLATED`, cash balance allocated to a posId will be returned. |

# Websocket

## Market

```json
{
  "channel": "market_perp",
  "instId": "BTCUSDT",
  "action": "snapshot",
  "data": {
    "indexPx": "23319.8",
    "markPx": "23321.2",
    "lPx": "23315.4",
    "oi": 1248201,
    "fundingRate": "0.0001"
  },
  "pTime": 1659324622787
}
```

Mraket data will be pushed every second.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                        |
| ------------- | -------- | ------------ | -------------------------------------- |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`. |
| channel       | Enum     | Yes          | Channel name: `market_perp`.           |
| args          | Object   | Yes          | Request parameters                     |
| >instId       | String   | Yes          | Instrument ID.                         |

- Push data parameters

| **Parameter** | **Type** | **Description**                                  |
| ------------- | -------- | ------------------------------------------------ |
| channel       | String   | Channel name.                                    |
| instId        | String   | Instrument ID.                                   |
| action        | Enum     | Push data action.                                |
| data          | Array    | Subscribed data.                                 |
| >indexPx      | String   | Index price.                                     |
| >markPx       | String   | Mark price.                                      |
| >lPx          | String   | Last price.                                      |
| >oi           | String   | Open interest.                                   |
| >fundingRate  | String   | Current funding rate.                            |
| pTime         | Long     | Push time, unix timestamp format in milliseconds |

## Mark Price

```json
{
  "channel": "mark_price",
  "action": "snapshot",
  "data": [
    {
      "instId": "ETHUSDT",
      "markPx": "1686.32"
    },
    {
      "instId": "BTCUSDT",
      "markPx": "23322.66"
    }, ......
  ],
  "pTime": 1659324796144
}
```

Mark prices of all instruments will be pushed every second.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                        |
| ------------- | -------- | ------------ | -------------------------------------- |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`. |
| channel       | Enum     | Yes          | Channel name: `mark_price`.            |

- Push data parameters

| **Parameter** | **Type** | **Description**                                  |
| ------------- | -------- | ------------------------------------------------ |
| channel       | String   | Channel name.                                    |
| action        | Enum     | Push data action.                                |
| data          | Array    | Subscribed data.                                 |
| >instId       | String   | Instrument ID.                                   |
| >markPx       | String   | Mark price.                                      |
| pTime         | Long     | Push time, unix timestamp format in milliseconds |

## Candlesticks

```json
{
  "channel": "kline_perp",
  "instId": "BTCUSDT",
  "action": "update",
  "data": [
    [
      1659324960,
      "20709.6",
      "20715.7",
      "20705.4",
      "20715.4",
      "99",
      "20508"
    ]
  ],
  "pTime": 1659324960742
}
```

The Kline/Candlestick Stream push updates to the current klines/candlestick every second.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `kline_perp`                                   |
| args          | Array    | Yes          | Request parameters.                                          |
| >instId       | String   | Yes          | Instrument ID.                                               |
| >bar          | Enum     | Yes          | Kline bar.                                                   |
| >initialNum   | Int      | No           | The number of recent klines sent at `inital` push.Default: 0Max: 200 |

- Push data parameters

| **Parameter** | **Type** | **Description**                                    |
| ------------- | -------- | -------------------------------------------------- |
| channel       | String   | Channel name.                                      |
| instId        | String   | Instrument ID.                                     |
| action        | String   | Push data action.                                  |
| data          | Array    | Subscribed data.                                   |
| >ts           | Int      | Generated time,  unix timestamp format in seconds. |
| >o            | String   | Open price.                                        |
| >h            | String   | Highest price.                                     |
| >l            | String   | Lowest price.                                      |
| >c            | String   | Close price.                                       |
| >vol          | String   | The number of contracts traded.                    |
| >quoteVol     | String   | Trading volume in quote currency.                  |
| pTime         | Long     | Push time, unix timestamp format in milliseconds.  |

## Mini Order Books

```json
{
    "channel": "mini_books_perp",
    "instId": "BTCUSDT", 
    "action": "snapshot", 
    "data": 
        {
            "ask": ["8476.98", "43"],
            "bid": ["8476.97", "92"]              
        }, 
    "pTime": "1597026383085
}
```

Retrieve best ask and bid.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `mini_books_perp`                              |
| args          | Array    | Yes          | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID                                                |
| >updateSpeed  | Int      | No           | Time intervals of pushes in milliseconds.Default: 500Min: 300. |

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
    "channel": "books_perp",
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
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`                        |
| channel       | Enum     | Yes          | Channel name: `books_perp`                                   |
| args          | Array    | Yes          | Request parameters                                           |
| >instId       | String   | Yes          | Instrument ID                                                |
| >step         | String   | Yes          | Arregated step.                                              |
| >limit        | Int      | No           | Order book depth per side. Default: 5;Max: 20.               |
| >updateSpeed  | Int      | No           | Time intervals of pushes in milliseconds.Default: 500;Min: 300. |

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
  "channel": "trades_perp",
  "instId": "ETHUSDT",
  "action": "initial",
  "data": [
    {
      "mTime": 1658906383508,
      "px": "1450",
      "sz": "1",
      "side": "SELL",
      "tradeId": 1
    },
    {
      "mTime": 1658906727120,
      "px": "1450",
      "sz": "1",
      "side": "SELL",
      "tradeId": 2
    },
    {
      "mTime": 1658913721271,
      "px": "1450",
      "sz": "1",
      "side": "BUY",
      "tradeId": 3
    },
    {
      "mTime": 1659155419582,
      "px": "1600",
      "sz": "10",
      "side": "BUY",
      "tradeId": 4
    }
  ],
  "pTime": 1659327443144
}
```



Retrieve the recent trades data. Data will be pushed whenever there is a trade.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe`.                       |
| channel       | Enum     | Yes          | Channel name: `trades_perp`.                                 |
| args          | Array    | No           | Request parameters                                           |
| >initialNum   | Int      | No           | The number of recent trades pushed at the `initial` push.Default: 50Max: 200 |
| >instId       | String   | Yes          | Instrument ID.                                               |

- Push data parameters

| **Parameter** | **Type** | **Description**                                   |
| ------------- | -------- | ------------------------------------------------- |
| channel       | String   | Channel name.                                     |
| instId        | String   | Instrument ID.                                    |
| action        | String   | Push data action.                                 |
| data          | Array    | Subscribed data.                                  |
| >mTime        | Long     | Match time, unix timestamp format in milliseconds |
| >tradeId      | String   | Trade ID.                                         |
| >px           | String   | Trade price.                                      |
| >sz           | String   | Trade size (cont.)                                |
| >side         | Enum     | Trade direction of taker.                         |

## User Data

```json
{
  "channel": "user_data_perp@balance_perp",
  "action": "initial",
  "acctId": 100002,
  "pTime": 1659410774600,
  "data": [
    {
      "totalEq": "21.7369294658",
      "cashBal": "19.6290494658",
      "freeBal": "19.4415694658",
      "ordFrozen": "0.18748",
      "ccy": "USDT"
    }
  ]
}
```

```json
{
  "channel": "user_data_perp@orders_perp",
  "action": "update",
  "acctId": 100002,
  "pTime": 1659422689551,
  "data": [
    {
      "ordId": "1740030806065153",
      "clOrdId": "0947akfsw33",
      "px": "20000",
      "ordType": "LIMIT",
      "accFillSz": "0",
      "cTime": 1659422689551,
      "avgPx": "0",
      "instId": "BTCUSDT",
      "flags": "POST_ONLY",
      "timeInForce": "GTC",
      "side": "BUY",
      "state": "SUBMITTED",
      "sz": "2"
    }
  ]
}
```

```json
{
  "channel": "user_data_perp@positions_perp",
  "action": "initial",
  "acctId": 100002,
  "pTime": 1659410774600,
  "data": [
    {
      "posId": 12,
      "mgnMode": "cross",
      "posSide": "short",
      "posSz": "0",
      "avgPx": "0",
      "liqPx": "0",
      "lever": "60",
      "mmr": "0",
      "upl": "0",
      "cashBal": "0",
      "instId": "BTCIUSD"
    },
    {
      "posId": 1,
      "mgnMode": "cross",
      "posSide": "short",
      "posSz": "0",
      "avgPx": "0",
      "liqPx": "0",
      "lever": "60",
      "mmr": "0",
      "upl": "0",
      "cashBal": "0",
      "instId": "BTCIUSD"
    },
    {
      "posId": 18,
      "mgnMode": "cross",
      "posSide": "long",
      "posSz": "1",
      "avgPx": "21800.00",
      "liqPx": "2258.15",
      "lever": "125",
      "mmr": "0.0872",
      "upl": "2.10788",
      "cashBal": "0.18748",
      "instId": "BTCUSDT"
    }
  ]
}
```



Balance, position, order, and fill updates will be sent separately via this channel.

- Request parameters

| **Parameter** | **Type** | **Required** | **Description**                       |
| ------------- | -------- | ------------ | ------------------------------------- |
| op            | Enum     | Yes          | Operation: `subscribe`, `unsubscribe` |
| channel       | Enum     | Yes          | Channel name: `user_data_perp`.       |

- Push data parameters

-balance

Balance will be pushed when `totalEq`, `cashBal`, `freeBal`, or `ordFrozen` is updated.

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| channel       | String   | `user_data_perp@balance_perp`                                |
| acctId        | String   | Account ID.                                                  |
| action        | String   | Push data action: `initial`, `update`                        |
| data          | Array    | Subscribed data.                                             |
| >acctId       | String   | Account ID.                                                  |
| >ccy          | String   | Currency.                                                    |
| >totalEq      | String   | Total equity in USDT.                                        |
| >cashBal      | String   | Cash Balance of USDT.                                        |
| >freeBal      | String   | Cash balance that could be transfered.                       |
| >ordFrozen    | String   | Equity frozen for open orders.                               |
| pTime         | String   | Update time of account information, unix timestamp format in milliseconds. |

-orders

Active orders will be pushed at `initial` push. `update` data will be pushed when triggered by the following event types:

`new`: An order has been accepted into the engine.

`canceled` : An order has been canceled by the user.

`filled`:  Part of the order or all of the order's quantity has filled.

`expired`:  An order was canceled according to the order type's rules (e.g. LIMIT FOK orders with no fill, LIMIT IOC or MARKET orders that partially fill) or by the exchange, (e.g. orders canceled during liquidation, orders canceled during maintenance).

`stopped`: An order was stopped for some reason.

`triggered`: A conditional order is triggered.

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| channel       | String   | `user_data_perp@orders_perp`                                 |
| acctId        | String   | Account ID.                                                  |
| action        | String   | Push data action: `initial`, `update`                        |
| data          | Array    | Subscribed data.                                             |
| >instId       | String   | Instrument ID                                                |
| >ordId        | String   | Order ID.                                                    |
| >clOrdId      | String   | Client-supplied order ID.                                    |
| >px           | String   | Price.                                                       |
| >sz           | String   | Order size (count).                                          |
| >side         | Enum     | Order side.                                                  |
| >ordType      | Enum     | Order type.                                                  |
| >flags        | String   | Additional flag or multiple flags separated with comma.      |
| >timeInForce  | Enum     | Time in force: `gtc`, `ioc`, `fok`.                          |
| >accFillSz    | String   | Accumulated fill quantity.                                   |
| >avgPx        | String   | Average filled price. If none if filled, 0 will be returned. |
| >state        | String   | Order state.                                                 |
| >cTime        | Long     | Create time, unix timestamp format in milliseconds.          |
| pTime         | Long     | Push time, unix timestamp format in milliseconds.            |

-fills

Data will not be pushed at `initial` push. Data will only be pushed when order(s) is filled.

| **Parameter** | **Type** | **Description**                                    |
| ------------- | -------- | -------------------------------------------------- |
| channel       | String   | `user_data_perp@fills_perp`                        |
| acctId        | String   | Account ID.                                        |
| action        | Enum     | Push data action: `initial`, `update`              |
| data          | Array    | Subscribed data.                                   |
| >instId       | String   | Instrument ID.                                     |
| >posId        | String   | Position ID.                                       |
| >tradeId      | String   | Trade ID.                                          |
| >ordId        | String   | Order ID.                                          |
| >px           | String   | Price.                                             |
| >sz           | String   | Quantity traded.                                   |
| >side         | String   | Trade side.                                        |
| >fee          | String   | Fee.                                               |
| >role         | Enum     | Role.                                              |
| >selfTradeQty | String   | Self-traded quantity.                              |
| pTime         | String   | Match time, unix timestamp format in milliseconds. |

-positions

Current positions will be pushed at `initial` push. `update` data will be pushed whenever positions is changed.

| **Parameter** | **Type** | **Description**                                              |
| ------------- | -------- | ------------------------------------------------------------ |
| channel       | String   | `user_data_perp@positions_perp`                              |
| acctId        | String   | Account ID.                                                  |
| action        | String   | Push data action: `initial`, `update`                        |
| data          | Array    | Subscribed data.                                             |
| >posId        | String   | Position ID.                                                 |
| >mgnMode      | Enum     | Margin mode.                                                 |
| >instId       | String   | Instrument ID.                                               |
| >posSide      | Enum     | Position side.                                               |
| >posSz        | String   | Quantity of positions (cont).                                |
| >avgPx        | String   | Average open price.                                          |
| >liqPx        | String   | Estimated liquidation price.                                 |
| >lever        | Int      | Leverage.                                                    |
| >mmr          | String   | Maintenance margin requirement.                              |
| >upl          | String   | Unrealized PnL.                                              |
| >cashBal      | String   | Cash balance. If mgnMode is `CROSS`, cash balance allocated to all `CROSS` positions will be returned. If mgnMode is `ISOLATED`, cash balance allocated to a posId will be returned. |
| pTime         | Long     | Push time, unix timestamp format in milliseconds.            |
