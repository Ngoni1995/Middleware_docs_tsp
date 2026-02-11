# 📄 Statements Service API Documentation

The **Statements Service** provides access to account transaction statements, including historical debit and credit entries. All requests are routed through the **Service Request API**. The service supports statement retrieval for a **maximum period of three (3) months per request**. Requests exceeding this date range are not permitted. 

## 🚪 Entry Point - API Gateway

The **API Gateway** serves as the single entry point for all requests to the Statements Service. It handles routing, authentication, authorization, caching, and request logging before forwarding the request to the internal Statements Service.

### Responsibilities

- **Routing:** Directs requests to the correct service (`accounts`) based on the `X-Destination-Service` header.  
- **Authentication & Authorization:** Validates `X-Integration-Id` and `X-Integration-Key` headers to ensure only authorized clients can access the service.  
- **Caching:** Supports optional caching via the `X-Cache-Key` header to reduce load on internal services.  
- **Request Logging:** Generates traceable logs using `X-Request-Id` for auditing and debugging.  
- **Timeout Enforcement:** Enforces a TTL of **20 seconds** for all requests.  

### Required Headers

| Header Name             | Description                                          |
|-------------------------|------------------------------------------------------|
| `X-Request-Id`          | Unique request identifier for tracing                |
| `X-Destination-Service` | Target service (must be `accounts`)                 |
| `X-Integration-Id`      | Client integration identifier                        |
| `X-Integration-Key`     | Client integration key                               |
| `X-Cache-Key`           | Optional cache key                                   |
| `Cache-Control`         | Controls caching behavior                             |
| `Content-Type`          | Must be `application/json`                           |

### Example Request via API Gateway

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 9ec3c127-3b79-4154-86f3-8f75573e3853' \
--header 'X-Destination-Service: statements' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: 57839954EF7293FD3F0F462233A77CA6' \
--header 'Cache-Control: no-cache' \
--header 'Content-Type: application/json' \
--data '{
   "action" : "get-deposit-account-statement",
   "account_number": "500007903387",
   "from_date" : "2025-12-02",
   "to_date" : "2026-02-10"
}'
```

## ⚡Actions

The Statements Service supports multiple actions that can be invoked through the API Gateway. Each action requires the caller to have the corresponding **permission**. All actions return JSON responses with HTTP headers that include request tracing and correlation information.

### Permissions

| Action                   | Required Permission                |
|--------------------------|-----------------------------------|
| `get-deposit-account-statement`         | `statements.get-deposit-account-statement`         |

---

### 1. Get Deposit Account Statement

This action retrieves the transaction statement for a specified deposit account within a given date range. The response includes all debit and credit entries posted to the account between the supplied `from_date` and `to_date`.The endpoint consistently returns an HTTP `200 OK` response. For invalid accounts, closed accounts, or periods with no activity, an empty statement will be returned.

The maximum supported date range per request is **three (3) months**. Requests exceeding this limit will be rejected by the Service Request API.

#### Request

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 9ff0eeb2-fb2c-4e88-a9ed-a01a979ffed7' \
--header 'X-Destination-Service: statements' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: 57839954EF7293FD3F0F462233A77CA6' \
--header 'Cache-Control: no-cache' \
--header 'Content-Type: application/json' \
--data '{
   "action" : "get-deposit-account-statement",
   "account_number": "500007903387",
   "from_date" : "2025-12-02",
   "to_date" : "2025-12-05"
}'
```

#### ✅ Success Response

```json
{
    "request_id": "bcc85b47-2884-4fec-92c0-556ba79345f2",
    "action": "get-deposit-account-statement",
    "statement": [
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "ZIMSW ATM Cash Withdrawal-511201513344002-USD430.00 POSB CAUSEWAY ATM      Harare       04ZW   000374756276",
            "starting_balance": 969.6,
            "ending_balance": 539.6,
            "amount": -430.0,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "ZIMSW ATM Cash Withdrawal Fee   000374756276",
            "starting_balance": 539.6,
            "ending_balance": 525.62,
            "amount": -13.98,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "ZIMSW ATM Tax   000374756276",
            "starting_balance": 525.62,
            "ending_balance": 525.57,
            "amount": -0.05,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "ZIMSW Payment-Econet Airtime Payee 100450100130021-USD2.00 POSB CELLBANK ECONET                  ZW   000374778574",
            "starting_balance": 525.57,
            "ending_balance": 523.57,
            "amount": -2.0,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "Master POS Purchase-660450055001195-USD0.21 AWS EMEA               LUXEMBOURG     LU   533651534189",
            "starting_balance": 523.57,
            "ending_balance": 523.36,
            "amount": -0.21,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-02T00:00:00",
            "description": "Master MCD POS Payment Fee   533651534189",
            "starting_balance": 523.36,
            "ending_balance": 521.86,
            "amount": -1.5,
            "available_balance": 1765.07
        },
        {
            "effective_date": "2025-12-03T00:00:00",
            "description": "ZIMSW Payment-Econet Airtime Payee 100450100130021-USD1.00 POSB CELLBANK ECONET                  ZW   000374855107",
            "starting_balance": 521.86,
            "ending_balance": 520.86,
            "amount": -1.0,
            "available_balance": 1765.07
        }
    ],
    "success": true,
    "message": "SUCCESS"
}
```

#### ❌ Bad Response 

```json
{
    "error": "INVALID_SOURCE_ACCOUNT_NUMBER",
    "objects": [
        "500007903389"
    ],
    "message": "Account not found.",
    "code": -1,
    "success": false,
    "timestamp": "2026-02-10T10:45:17.2652555Z"
}
```

