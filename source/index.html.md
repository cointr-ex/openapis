---
title: CoinTR API Reference

language_tabs: # must be one of https://git.io/vQNgJ
- json

toc_footers:
- <a href='https://www.cointr.com/apikey' target='_blank'>Sign Up for a Developer Key</a>

top_nav:
- name: General
  active: active
  url: 'index.html'
- name: Futures
  url: 'futures.html'
- name: Spot
  url: 'spot.html'

active: active

search: true

code_clipboard: true

meta:
- name: description
  content: Documentation for the CoinTR Futures API
---

# Access URLs

Rest endpoint URL: https://api.cointr.pro

Websocket endpoint URL: wss://stream.cointr.pro/ws

# Terminology

- Kline/candlestick bars:

`1m`, `5m`, `15m`, `30m`, `1H`, `4H`, `12H`, `1D`, `1W`

(m -> minutes; H -> hours; D -> days; W -> weeks)

- Role: `MAKER`, `TAKER`

- Order state: `ACCEPTED`, `SUBMITTING`, `SUBMITTED`, `PARTIAL-FILLED`, `CANCELING`, `CANCELED`, `FILLED`, `STOPPED`, `TRIGGERED`, `EXPIRED`

(In some extreme scenarios, order state of `SUBMITTING` will be returned by order-placing requests. State of `SUBMITTING` indicates that asset involved in the order has been frozen and whereas the order has not been fed into our matching engine yet. Orders of `SUBMITTING` state will never trigger data pushes of our Websocket channels. )

- Order/trade side/direction (side): `BUY`, `SELL`

- Order type (ordType): `MARKET`, `LIMIT`, `STOP`, `STOP_LIMIT`

- Order flags: `POST_ONLY`, `REDUCE_ONL`

- Time in force (timeInForce): `GTC`, `IOC`, `FOK`

- Transfer type: `SPOT_TO_FUTURE`, `FUTURE_TO_SPOT`

- Position mode (posMode): `LONG_SHORT_MODE`, `NET_MODE`

- Margin mode (mgnMode): `CROSS`, `ISOLATED`

- Position side (posSide): `LONG`, `SHORT`

# Sign

For authenticated requests, the following headers should be sent with the request:

- X-COINTR-APIKEY: `your_API_access_key`

- X-COINTR-SIGN: `signature`

- Content-Type: `'application/json'`

Our signature algorithm involves two steps of `HMAC SHA256` operations:

1. `HMAC SHA256` of the following string, with your API secret key as the key: `f"{math.floor(currentTime/30000)}"` (`currentTime`，unix timestamp format in milliseconds)

2. With the value obtained from `HMAC SHA256` operation in step 1 as the key, perform `HMAC SHA256` operation on `totalParams` which is the concatenation of `query string` and `request body`. Parameter `timestamp` must be included in the `query string` of all requests requiring authitication.

# Pagination

Pagination is supported on some rest API endpoints. The `next` field in response data indicates that the number of items requested exceeds the specified page limit and thus some items within the query window are not returned. To get these items, API users should set the `after` parameter as the `next` field value and make another request without changing any other request parameter(s).

# Websocket

## Request format

Messages sent to the server should contain the following dictionary items:

- `op`: the operation to be run. Should be one of the following:

  - `subscribe` to subscribe to a channel;

  - `unsubscribe` to unsubscribe from a channel.

- `channel`: the channel to subscribe or unsubscribe.

- `args`: channel parameters in JSON format.

## Response format

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

## Data push format

Data pushed from the server will contina the following dictionary items:

- `channel`: channel name.

- `instId` (optional): instrument ID.

- `action`: type of pushed data. Should be one of the following:

  - `initial`: indicates that data in the accompanying `data` field is a snapshot of current data;

  - `update`: indicates that data in the accompanying `data` field is incremental;

  - `snapshot`: indicates that data sent via this channel is always a snapshot of current data;

  - `resumed`: indicates that our system detects data loss(es) from the client side, and therefore the server will send a snapshot of current data in the accompanying `data` field. <u>[unavailable for now]</u>.

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

|               |          |              |                                                |
| ------------- | -------- | ------------ | ---------------------------------------------- |
| **Parameter** | **Type** | **Required** | **Description**                                |
| op            | Enum     | Yes          | Operation: `auth`                              |
| args          | Array    | Yes          | Request parameters.                            |
| >apiKey       | String   | Yes          | API access key.                                |
| >timestamp    | String   | Yes          | Current timestamp in milliseconds.             |
| >sign         | String   | Yes          | SHA256 HMAC operation details are shown below. |

Our signature algorithm involves two steps of `HMAC SHA256` operations:

1. `HMAC SHA256` of the following string, with your API secret key as the key: `currentTime` (unix timestamp format in milliseconds) //30000

2. With the value obtained from `HMAC SHA256` operation in step 1 as the key, perform `HMAC SHA256` operation on `timestamp={currentTime}`

# Client-supplied ID

Client-supplied IDs should be a combination of up to 36 BASE64 characters.

clOrdId (client-supplied order id) is a mandatory field while submitting an order.

Professional users use this clOrdId to manage their orders. But if it is not required, please set this field with a random UUID.

User should ensure the uniqueness of clOrdIds. clOrdIds of active orders and orders completed within 1 minute must be unique at account level.

If a duplicated clOrdId is received, versus an existing active order, that new order request will be rejected by the Exchange.

User is able to cancel an order based on its clOrdId.

User is also able to query an order based on its clOrdId. But if that order has been fully filled or cancelled for more than 2 hours, it can only be queried by ordId instead.

To avoid duplicated order submission caused by auto retry taken by user's application, the Exchange will reject the 2nd order request if the same clOrdId was received within 1 minute, even though that 1st order is already fully filled.

**Suggestion to user**

1. Always use a different clOrdId for a different order.

2. Application auto retry for order submission should not last for more than 1minute.
