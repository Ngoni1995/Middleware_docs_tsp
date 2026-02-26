# 💸 Transfers Service API Documentation

TThe **Transfers Service** enables movement of funds between accounts within the bank and to external banks via RTGS.

All requests are routed through the **Service Request API (API Gateway)** and require proper authentication and authorization headers.

## 🚪 Entry Point - API Gateway

The **API Gateway** serves as the single entry point for all requests to the Transfers Service. It handles routing, authentication, authorization, caching, and request logging before forwarding the request to the internal Transfers Service.

### Responsibilities

- **Routing:** Directs requests to the correct service (`transfers`) based on the `X-Destination-Service` header.  
- **Authentication & Authorization:** Validates `X-Integration-Id` and `X-Integration-Key` headers to ensure only authorized clients can access the service.  
- **Caching:** Supports optional caching via the `X-Cache-Key` header to reduce load on internal services.  
- **Request Logging:** Generates traceable logs using `X-Request-Id` for auditing and debugging.  
- **Timeout Enforcement:** Enforces a TTL of **20 seconds** for all requests.  

### Required Headers

| Header Name             | Description                                          |
|-------------------------|------------------------------------------------------|
| `X-Request-Id`          | Unique request identifier for tracing                |
| `X-Destination-Service` | Target service (must be `transfers`)                 |
| `X-Integration-Id`      | Client integration identifier                        |
| `X-Integration-Key`     | Client integration key                               |
| `X-Cache-Key`           | Optional cache key                                   |
| `Cache-Control`         | Controls caching behavior                             |
| `Content-Type`          | Must be `application/json`                           |

### Example Request via API Gateway

```bash
curl --location '10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 8f9fe0a3-03d4-4178-a0f9-202c014446ae' \
--header 'X-Destination-Service: transfers' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: max-age=0' \
--header 'Content-Type: application/json' \
--data '{
    "action": "rtgs-transfer",
    "source_account" : "500007903380",
    "amount": 10,
    "beneficiary_bank" : "ZB Bank",
    "beneficiary_bank_swift_code" : "ZBBNPKPP",
    "beneficiary_name" : "Malcom Chakoma",
    "beneficiary_bank_account_number" : "100006735528",
    "beneficiary_bank_reference" : "Econet Recharge",
    "channel" : "mobile"
}'
```

## ⚡ Actions

The Transfers Service supports multiple actions that can be invoked through the API Gateway. Each action requires the caller to have the corresponding **permission**. All actions return JSON responses with HTTP headers that include request tracing and correlation information.

### Permissions

| Action                     | Required Permission                      |
|----------------------------|------------------------------------------|
| `internal-funds-transfer`  | `transfers.internal-funds-transfer`      |
| `rtgs-transfer`            | `transfers.rtgs-transfer`                |

---

### 1. Internal Funds Transfer (IFT)

This action transfers funds between internal accounts within the bank.

The service supports the following account combinations:

- DP → DP  
- GL → GL  
- GL → DP  
- DP → GL  

If either the `source_account` or `destination_account` is a **GL account**, any provided `charge_codes` will be ignored.

The endpoint consistently returns an HTTP `200 OK` response. For invalid accounts, insufficient funds, currency mismatches, or validation failures, an error response will be returned in the response body.

#### Request

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: dce9487a-b50d-4417-abb0-c8ad784c514f' \
--header 'X-Destination-Service: transfers' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: max-age=0' \
--header 'Content-Type: application/json' \
--data '{
    "action": "internal-funds-transfer",
    "description": "Test transfer DP to DP",
    "source_account": "500007903379",
    "destination_account": "500009432963",
    "currency_iso_code": "ZWG",
    "charge_codes": [],
    "amount": 18
}'
```

#### ✅ Success Response

```json
{
    "request_id": "36c8d57e-e9e8-489c-b9f8-edebbc387982",
    "action": "rtgs-transfer",
    "amount": 10,
    "success": true,
    "message": "SUCCESS",
    "transaction_reference": "9294251a-3d65",
    "accounts": {
        "source_account": "500007903387"
    }
}
```

#### ❌ Bad Response 
```json
{
    "error": "BAD_PARAMETERS",
    "objects": [
        "500007903380"
    ],
    "message": "Failed to deduct funds: Invalid source account number.",
    "timestamp": "2026-02-26T14:08:13.5680627Z"
}
```

### 3. RTGS Transfer

This action transfers funds from an internal deposit account to an external bank using the RTGS network.

The transfer debits the specified `source_account` and sends funds to the beneficiary account at the destination bank.

The endpoint consistently returns an HTTP `200 OK` response. For invalid accounts, insufficient funds, invalid beneficiary details, or validation failures, an error response will be returned in the response body.

#### Request

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 486c022b-8336-4fb6-9f1d-8ced9029d7bf' \
--header 'X-Destination-Service: transfers' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: max-age=0' \
--header 'Content-Type: application/json' \
--data '{
    "action": "rtgs-transfer",
    "source_account": "500007903387",
    "amount": 10,
    "beneficiary_bank": "ZB Bank",
    "beneficiary_bank_swift_code": "ZBBNPKPP",
    "beneficiary_name": "Malcom Chakoma",
    "beneficiary_bank_account_number": "100006735528",
    "beneficiary_bank_reference": "Econet Recharge",
    "channel": "mobile"
}'
```

#### ✅ Success Response

```json
{
    "request_id": "c199cbbf-2ace-48a1-a2e8-3956fa7a4f8b",
    "action": "rtgs-transfer",
    "amount": 10,
    "success": true,
    "message": "SUCCESS",
    "transaction_reference": "84ac1803-31b8",
    "accounts": {
        "source_account": "500007903387"
    }
}
```

#### ❌ Bad Response 
```json
{
    "error": "BAD_PARAMETERS",
    "objects": [
        "500007903380"
    ],
    "message": "Failed to deduct funds: Invalid source account number.",
    "timestamp": "2026-02-26T14:04:17.1678404Z"
}
```

