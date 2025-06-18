
# 📘 5paisa Xtream Open APIs Documentation Part1

---

## Table of Contents

### 1. User Authentication  
- [OAuth Login (Web Based)](#oauth-login-web-based)  
- [Access Token Generation](#access-token-generation)  

### 2. 🛒 Order Lifecycle  
- [PlaceOrder](#placeorder)  
- [ModifyOrder](#modifyorder)  
- [CancelOrder](#cancelorder)  

---


# OAuth Login (Web Based)

---

## 🔐 Part 1: OAuth Login (Web-Based)

Enables partners to redirect clients to the 5paisa login page to authenticate using client code, PIN, and OTP as per SEBI 2FA regulations. After successful login, a **Request Token** is returned which is used to generate an **Access Token**.

### ➤ Endpoint

```
GET https://dev-openapi.5paisa.com/WebVendorLogin/VLogin/Index
```

### 📥 Parameters (Query)

| Parameter     | Required | Description                                                                 |
| ------------- | -------- | --------------------------------------------------------------------------- |
| `VendorKey`   | Yes      | API key provided by 5paisa (Partner/User App Key).                          |
| `ResponseURL` | Yes      | Callback URL to which the user will be redirected after login.              |
| `State`       | Optional | Any reference value the partner wants to pass and receive back on redirect. |

### 🔁 Flow

1. Redirect the user to the login URL with required query parameters.
2. User logs in via 5paisa Web Login page.
3. After successful authentication, the user is redirected to the `ResponseURL`.

### ✅ Example Redirect URL After Login

```bash
https://www.yourapp.com/oauth/callback?
RequestToken=eyJhbGciOi...&state=XYZ
```

### 🔑 Response Parameters (Redirected to `ResponseURL`)

| Parameter      | Description                                                      |
| -------------- | ---------------------------------------------------------------- |
| `RequestToken` | JWT token (valid for 60 minutes) to be used in Access Token API. |
| `state`        | Optional reference value returned from original request.         |

Note: We will not Support TOTP.

---
# Access Token Generation

---


## 🪪 Part 2: Generate Access Token (Using Request Token)

Once the **RequestToken** is obtained from OAuth login, use this API to generate a session-wide **AccessToken**.

### ➤ Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/GetAccessToken
```

### 🧾 Headers

| Key          | Value            |
| ------------ | ---------------- |
| Content-Type | application/json |

---

### 📤 Request Body (JSON)

```json
{
  "head": {
    "Key": "{{Partner/User App key}}"
  },
  "body": {
    "RequestToken": "{{request_token}}",
    "EncryKey": "{{Parter/User_encrykey}}",
    "UserId": "{{Parter/User_userid}}"
  }
}
```

| Field               | Mandatory | Description                                     |
| ------------------- | --------- | ----------------------------------------------- |
| `head.Key`          | Yes       | AppKey (API key provided by 5paisa).            |
| `body.RequestToken` | Yes       | Token from OAuth. Valid for 60 mins. |
| `body.EncryKey`     | Yes       | Encryption key shared in API credentials.       |
| `body.UserId`       | Yes       | User ID shared in API credentials.              |

---

### 🕐 Token Validity

* **Request Token:** Valid for **60 minutes**.
* **Access Token:** Valid until **11:59 PM** on the same day.

---

### 📥 Successful Response (Sample)

```json
{
  "body": {
    "AccessToken": "JWT_TOKEN",
    "ClientCode": "123456",
    "ClientName": "Mahendra",
    "POAStatus": "Y",
    "ClientType": "1",
    "CustomerType": "BASIC",
    "AllowBseCash": "Y",
    "AllowNseCash": "Y",
    ...
    "Status": 0,
    "Message": "Success"
  },
  "head": {
    "Status": 0,
    "StatusDescription": "Success"
  }
}
```

| Field                | Description                         |
| -------------------- | ----------------------------------- |
| `AccessToken`        | JWT to be used for all API calls.   |
| `ClientCode`         | 5paisa client code.                 |
| `ClientName`         | Name of logged-in user.             |
| `POAStatus`          | Power of Attorney status (`Y`/`N`). |
| `AllowBseCash`, etc. | Segment activation flags.           |
| `Status`             | `0`: Success, `2`: Invalid Inputs.  |

---

### ❌ Failure Response (Token Expired)

```json
{
  "body": {
    "AccessToken": "",
    "ClientCode": "",
    "ClientName": "",
    ...
    "Message": "Token Expired.",
    "Status": 2
  },
  "head": {
    "Status": 2,
    "StatusDescription": "Invalid Inputs"
  }
}
```

---

### 🔑 Segment Activation Flags

Flags indicating which trading segments are enabled for the user:

* `AllowBseCash`
* `AllowBseDeriv`
* `AllowBseMF`
* `AllowMCXComm`
* `AllowMcxSx`
* `AllowNSECurrency`
* `AllowNSEL`
* `AllowNseCash`
* `AllowNseComm`
* `AllowNseDeriv`
* `AllowNseMF`
* `CommodityEnabled`

---

## 📌 Summary of Authentication Flow

```text
[1] Redirect to OAuth URL (web/app)
      ↓
[2] User logs in with 2FA (client code + PIN + OTP)
      ↓
[3] Receive RequestToken on your callback URL
      ↓
[4] Call Access Token API using RequestToken
      ↓
[5] Get AccessToken to use for other API calls
```

---


# PlaceOrder

---

## 🔍 Overview of V1/PlaceOrderRequest API

The `V1/PlaceOrderRequest` API is a RESTful endpoint designed to **place stock, derivative, currency, or commodity orders** programmatically. It supports multiple order types, and is ideal for **high-frequency and algorithmic trading systems**.

---

## 🧠 Use Case

This API is optimized for:
- **Algo trading bots**
- **Quantitative trading systems**
- **Live trading dashboards**
- **Automated risk management tools**

---

## 📬 Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/PlaceOrderRequest

````

---

## 🔐 Authentication

Authorization is **mandatory** via Bearer Token in the `Authorization` header.

| Header Key       | Value Example              |
|------------------|----------------------------|
| `Content-Type`   | `application/json`         |
| `Authorization`  | `Bearer {your_token_here}` |

---

## 📝 Request Body

### 📦 JSON Schema

```json
{
  "head": {
    "key": "YOUR_APP_KEY"
  },
  "body": {
    "Exchange": "N",
    "ExchangeType": "C",
    "ScripCode": "1660",
    "ScripData": "",
    "Price": 445,
    "StopLossPrice": 0,
    "OrderType": "Buy",
    "Qty": 1,
    "DisQty": 0,
    "IsIntraday": true,
    "iOrderValidity": 0,
    "AHPlaced": "N",
    "RemoteOrderID": "CustomOrder123",
    "ValidTillDate": "/Date(1725516056000)/",
    "AlgoID": 0,
    "DeviceID": "MyAppDevice001"
  }
}
````

---

## 📋 Field Reference

| Field            | Type    | Required | Description                                   |
| ---------------- | ------- | -------- | --------------------------------------------- |
| `head.key`       | string  | ✅        | Your registered app key                       |
| `Exchange`       | string  | ✅        | `N` = NSE, `B` = BSE, `M` = MCX               |
| `ExchangeType`   | string  | ✅        | `C` = Cash, `D` = Derivatives, `U` = Currency |
| `ScripCode`      | string  | ✅        | Unique code of the instrument (recommended)   |
| `ScripData`      | string  | ❌        | Optional (if ScripCode unavailable )          |
| `Price`          | double  | ❌        | Order price. Set to `0` for market order      |
| `StopLossPrice`  | double  | ❌        | Trigger price for SL orders                   |
| `OrderType`      | string  | ✅        | `Buy` or `Sell`                               |
| `Qty`            | integer | ✅        | Total quantity                                |
| `DisQty`         | integer | ❌        | Disclosed quantity (≤ Qty)                    |
| `IsIntraday`     | boolean | ❌        | `true` = intraday, `false` = delivery         |
| `iOrderValidity` | integer | ❌        | 0 = Day, 5 = GTD, 3 = IOC, etc.               |
| `AHPlaced`       | string  | ❌        | After Market Order flag: `Y` or `N`           |
| `RemoteOrderID`  | string  | ✅        | user defined Custom ID for user tracking      |
| `ValidTillDate`  | string  | ❌        | For GTD/VTD orders in `/Date(Unix)` format    |
| `AlgoID`         | integer | ❌        | Strategy-specific algorithm ID                |
| `DeviceID`       | string  | ❌       | Unique device/app instance ID                 |

---

## 📘 iOrderValidity Enum

| Code | Meaning                   |
| ---- | ------------------------- |
| 0    | Day                       |
| 3    | IOC (Immediate or Cancel) |
| 5    | VTD  (Valid till Date)                     |
---

### Supported Order Types

* **Limit Order**: `Price > 0`
* **Market Order**: `Price = 0`
* **Stop Loss Order**: Uses `StopLossPrice` <place Limit Order with StopLoss>
* **Stop Loss Market Order** : `Price = 0` & `StopLossPrice`<place Market Order with StopLoss>
* **After Market Order (AMO)**: `AHPlaced = "Y"`
* **Immediate or Cancel (IOC)**: `iOrderValidity = 3`
* **Valid till Datel (VTD)**: `iOrderValidity = 5`

### Order Nature

* **Intraday**: `IsIntraday = true`
* **Delivery**: `IsIntraday = false`

---

## 🔢 ScripData Format Reference

| Instrument Type | Format                         | Example                       |
| --------------- | ------------------------------ | ----------------------------- |
| Cash Equity     | `RELIANCE_EQ`                  | `RELIANCE_EQ`                 |
| Futures         | `SYMBOL_YYYYMMDD`              | `NIFTY_20240930`              |
| Options         | `SYMBOL_YYYYMMDD_CE/PE_STRIKE` | `BANKNIFTY_20240329_CE_41600` |
| Currency        | `SYMBOL_EXPIRY_CE/PE_STRIKE`   | `GBPINR_1325255400_CE_107.25` |
| Commodity       | Same as above                  | `SILVER_1314489600_CE_63750`  |

---

## 📘 Exchange & Segment Codes

### Exchange (`Exch`)

| Code | Exchange |
| ---- | -------- |
| N    | NSE      |
| B    | BSE      |
| M    | MCX      |

---

### Segment (`ExchType`)

| Code | Segment Description             |
| ---- | ------------------------------- |
| C    | Cash Segment (Equity)           |
| D    | Derivatives (Futures & Options) |
| U    | Currency Derivatives            |



---

## ✅ Sample Success Response

```json
{
  "body": {
    "BrokerOrderID": 672112769,
    "Exch": "N",
    "ExchType": "C",
    "ExchOrderID": "0",
    "LocalOrderID": 0,
    "Message": "Success",
    "RMSResponseCode": 1,
    "RemoteOrderID": "CustomOrder123",
    "ScripCode": 11915,
    "Status": 0,
    "Time": "/Date(1658255400000+0530)/"
  },
  "head": {
    "responseCode": "5PPlaceOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## ❌ Error Responses

### Invalid App Key

```json
{
  "head": {
    "responseCode": "5PPlaceOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid Head Parameters"
  }
}
```

### Invalid JWT Token

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Message": "Authentication Fails",
    "Status": 9
  }
}
```

### Invalid Session / Missing Client Code

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Message": "Invalid Session",
    "Status": 9
  }
}
```

### Invalid Input Parameters

```json
{
  "body": {
    "Message": "Invalid Input Parameters.",
    "Status": 2
  }
}
```

---

## 🔄 Order Tracking

You can track placed orders using the following APIs:

* **Order Book API**
* **Order Status API**
* **WebSocket Confirmations** (recommended for live trading)

-Use `RemoteOrderID` for consistent traceability across systems.
We have introduced the RemoteOrderId field as a user-defined identifier. Partners or clients can create their own RemoteOrderId and include it when placing an order. This identifier serves several purposes:

Track Order Status: Use the RemoteOrderId to track the order status via the Order Stand API.
Obtain ExchangeOrderId: Retrieve the ExchangeOrderId using the RemoteOrderId, which can then be used to modify the order.

Usage and Benefits

In certain scenarios, specifically with Stop-Loss (SL) orders, we have observed issues where the BrokerOrderId changes, causing clients to be unable to map the ExchangeOrderId from the broker ID. To mitigate this issue, we recommend using the RemoteOrderId.

-note:If the price is not passed in the request body, its value will be considered as 0. The 0 value of price indicates order to be of at-market type. It takes market price by default.

### Q.Which characters are allowed in a `RemoteOrderID`?

ans. The `RemoteOrderID` field only supports a limited set of characters to ensure system compatibility and prevent errors.

---

### ✅ Allowed Characters

You may use the following:

| Character Type    | Examples    |
| ----------------- | ----------- |
| Uppercase letters | `A–Z`       |
| Lowercase letters | `a–z`       |
| Digits            | `0–9`       |
| Whitespace        | space (` `) |
| Hyphen            | `-`         |
| Underscore        | `_`         |
| Question mark     | `?`         |

---

### ❌ Disallowed Characters

Any characters **not listed above are not permitted**. This includes:

* Special characters like: `@`, `#`, `$`, `%`, `&`, `*`, `(`, `)`, `!`, `=`, `+`, `/`, `\`, `|`, `<`, `>`, `:`, `;`, `,`, `.`, `"`, `'`, `~`, `^`, `[`, `]`, `{`, `}`
* Control characters: newlines (`\n`, `\r`) and tabs (`\t`)

---

### 🧪 Valid Examples

* `Order123`
* `Order_001`
* `Order-Test 2025`
* `Custom?ID`

---

### 🚫 Invalid Examples

* `Order@123` → contains `@`
* `Order#001` → contains `#`
* `Order/ID` → contains `/`
* `Order(ID)` → contains `(` and `)`

---

**Tip:** Stick to letters, numbers, basic separators like hyphens or underscores, and the question mark to ensure your `RemoteOrderID` is accepted.

---

## 🧠 Best Practices

* Always log both `BrokerOrderID` and `RemoteOrderID`
* Use WebSockets for low-latency confirmation
* Validate request data to avoid rejections
* Assign unique `DeviceID` for each strategy runner
* Keep `AlgoID` dynamic per strategy or signal
* `AlgoID` is mandatory if order frequency is more than 10 orders per second 

---

## 🔗 Reference APIs

* \[🔍 Scrip Master API] — To retrieve valid ScripCode/ScripData
* \[📊 Order Status API] — To verify order status
* \[⚙️ Modify/Cancel Order API] — To update or cancel orders

---

## 🧠 Integration Tips

When integrating this API into your **algo trading system**, index the following for automation:

* `ScripData` formatting rules
* Common error response patterns
* Order status mapping
* Enum values (`OrderType`, `OrderValidity`)

This allows for **intelligent suggestions**, **error handling**, and **dynamic UI inputs**.

---

## 🛡️ Disclaimer

Placing an order through the API does **not guarantee** that it will be accepted by the exchange. Execution depends on:

* Market hours and liquidity
* Margin/funds availability
* Risk management parameters
* Exchange acceptance

---


# ModifyOrder

---

## 🔁 Overview

The `ModifyOrderRequestV1` API allows clients to **modify an existing pending stock order** on the 5paisa trading platform. You can update selected fields such as:

- **Price**
- **Quantity**
- **Stop-loss price**
- **Disclosed quantity**
- **Order type** (by setting `Price = 0` for market orders)

> ✅ Only the **fields being modified** along with the **mandatory Exchange Order ID** need to be passed.

---

## 🔗 Endpoint

```http
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/ModifyOrderRequest
````

---

## 🔐 Headers

| Header        | Value Format            |
| ------------- | ----------------------- |
| Content-Type  | `application/json`      |
| Authorization | `Bearer {access_token}` |

---

## 📥 Request Format

### JSON Body

```json
{
  "head": {
    "key": "{{Your App Key}}"
  },
  "body": {
    "ExchOrderID": "1100000018012644",
    "Price": 0,
    "Qty": 10,
    "StopLossPrice": 449,
    "DisQty": 5
  }
}
```

### 🧾 Field Description

| Field                | Type   | Required | Description                              |
| -------------------- | ------ | -------- | ---------------------------------------- |
| `head.key`           | string | ✅        | Application key assigned to vendor       |
| `body.ExchOrderID`   | string | ✅        | Exchange-assigned order ID to modify     |
| `body.Price`         | float  | ❌        | New price; set `0` for market order      |
| `body.Qty`           | int    | ❌        | New total quantity                       |
| `body.StopLossPrice` | float  | ❌        | New stop loss trigger price              |
| `body.DisQty`        | int    | ❌        | New disclosed quantity (must be ≤ `Qty`) |

---

## ✅ Sample Success Response

```json
{
  "body": {
    "BrokerOrderID": 292699,
    "Exch": "B",
    "ExchOrderID": "116718512092173",
    "ExchType": "D",
    "LocalOrderID": 4,
    "Message": "Exchange is closed. Cannot Modify your order.",
    "RMSResponseCode": -15,
    "RemoteOrderID": "1716729926",
    "ScripCode": 86752,
    "Status": 1,
    "Time": "/Date(171674800000+0530)/"
  },
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## ❌ Sample Error Responses

### 🔸 Missing or Invalid Token

```
Authorization header missing or invalid
Response: Invalid Token
```

### 🔸 Invalid Head Parameters

```json
{
  "body": null,
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid head parameters."
  }
}
```

### 🔸 Missing Exchange Order ID

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Exch": "N",
    "ExchOrderID": "0",
    "ExchType": "C",
    "LocalOrderID": 0,
    "Message": "Order does not exist",
    "RMSResponseCode": 0,
    "RemoteOrderID": "",
    "ScripCode": 0,
    "Status": 1,
    "Time": "/Date(1716811251790+0530)/"
  },
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## 📘 Status Codes (`head.status`)

| Code | Meaning                  |
| ---- | ------------------------ |
| 0    | Success                  |
| 1    | Invalid input parameters |
| 2    | Invalid head parameters  |
| -1   | Internal server error    |

---

## 🔤 Message Codes (`body.Message` or `RMSResponseCode`)

| Code / Text | Description                                        |
| ----------- | -------------------------------------------------- |
| `0`         | Success                                            |
| `1`         | RMS/system response                                |
| `2`         | Invalid request or parameters                      |
| `9`         | Authentication/session failed                      |
| *Text*      | Informational message (e.g., "Exchange is closed") |

---

## 💡 Developer Notes

* Use **OrderBook** or **OrderStatus** API to map `BrokerOrderID` or `RemoteOrderID` to the required `ExchOrderID`.
* **Set `Price = 0` to convert a Limit Order to a Market Order.**
* Validate that `DisQty` ≤ `Qty` before calling the API.
* Include retry logic for transient errors like network timeouts or server busy (`status = -1`).
* This API is critical for **algo-trading bots**, especially those adjusting stop-loss, target, or quantity dynamically.

---


## 🤖 Use Case: Algo Trading 

This API is designed to be easily integrated into:

* 🔁 **Automated trade modification bots**
* 📊 **Dynamic risk management systems**
* 🔄 **AI/ML model-based trade optimizers**

Use it to update open orders in real-time, respond to signals, or adjust trades in sync with your strategy logic.

---

## 📌 Summary

| Feature                 | Supported |
| ----------------------- | --------- |
| Partial Field Updates   | ✅         |
| Limit → Market Switch   | ✅         |
| Live Order Modification | ✅         |
| Algo-trade Ready        | ✅         |

---



# CancelOrder


## 🧠 API Purpose  **CancelOrderRequestV1**

`CancelOrderRequestV1` enables programmatic cancellation of orders that haven't been successfully executed yet on stock exchanges. Ideal for high-frequency and algo trading environments.

---

## 🔗 Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/CancelOrderRequest
```

---

## 📄 Headers

| Key           | Value                      |
| ------------- | -------------------------- |
| Content-Type  | application/json           |
| Authorization | bearer {Your Access Token} |

---

## 📥 Request Body (JSON)

### Top-level structure:

```json
{
  "head": {
    "key": "string"
  },
  "body": {
    "ExchangeOrderID": "string",
    "DeviceId": "string (max 100 characters)",
    "AlgoId": "integer (required for >10 orders/sec)"
  }
}
```

### Field Breakdown

| Field                  | Type    | Mandatory   | Description                                          |
| ---------------------- | ------- | ----------- | ---------------------------------------------------- |
| `head.key`             | string  | Yes         | Application key for the user or partner              |
| `body.ExchangeOrderID` | string  | Yes         | Order ID assigned by the exchange                    |
| `body.DeviceId`        | string  | Optional    | Device identifier (max length 100)                   |
| `body.AlgoId`          | integer | Conditional | Required when placing more than 10 orders per second |

---

## 📤 Response Body (JSON)

### Success Response

```json
{
  "body": {
    "BrokerOrderID": 555919893,
    "ClientCode": "string",
    "Exch": "N",
    "ExchOrderID": "0",
    "ExchType": "C",
    "LocalOrderID": 0,
    "Message": "Success",
    "RMSResponseCode": 0,
    "ScripCode": 2885,
    "Status": 0,
    "Time": "/Date(1637433000000+0530)/"
  },
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

### Failure Response Examples

#### Example 1 – Invalid `head.key`

```json
{
  "body": null,
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid Head Parameters"
  }
}
```

#### Example 2 – Missing Mandatory Fields

```json
{
  "body": {
    "BrokerOrderID": 0,
    "ClientCode": "string",
    "Exch": "?",
    "ExchOrderID": "0",
    "ExchType": "?",
    "LocalOrderID": 0,
    "Message": "Invalid Input Parameters.",
    "RMSResponseCode": 0,
    "ScripCode": 0,
    "Status": 2,
    "Time": "/Date(1637494757568+0530)/"
  },
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## 🔍 Status Codes

### `head.status`

| Code | Meaning                              |
| ---- | ------------------------------------ |
| -1   | Server unable to process the request |
| 0    | Success                              |
| 1    | Invalid input parameters             |
| 2    | Invalid head parameters              |

### `body.Status` & `body.Message`

| Code | Message                      |
| ---- | ---------------------------- |
| 0    | Success                      |
| 1    | 5paisa System (RMS) Response |
| 2    | Invalid Input Parameters     |
| 9    | Authentication Fails         |

---

## 💡 Notes

* ✅ `ExchangeOrderID` is **mandatory** for cancelling orders.
* ⚙️ `AlgoId` becomes **mandatory** when placing orders at a high frequency (>10 per second).
* 🔐 Ensure correct bearer token and app key are passed in the request header for authentication.
* 🧠 Responses may contain RMS-level data, useful for risk-based strategies or real-time order validation.

---

## 📌 Quick Summary

| Aspect         | Details                                                        |
| -------------- | -------------------------------------------------------------- |
| API Name       | `CancelOrderRequestV1`                                         |
| Type           | RESTful JSON POST API                                          |
| Use Case       | Cancel a live or pending order before execution                |
| Authentication | Bearer token + Application key                                 |
| Ideal For      | Algo trading assistants,  high-frequency systems               |
| Extras         | `DeviceId` & `AlgoId` parameters for extended control          |

---

## 📚 For Developers

* Add retry mechanisms for transient failures.
* Always log `ExchangeOrderID`, `Exch`, `ScripCode`, and timestamps for audit purposes.
* Use `Time` field in response to record actual cancellation time in logs.

---

---
