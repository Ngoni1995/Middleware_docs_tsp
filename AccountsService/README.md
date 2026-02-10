# Accounts Service API Documentation

The **Accounts Service** handles account-related queries such as balance enquiries, account lookup, and other account operations. All requests are routed through the **Service Request API**.

## Entry Point - API Gateway

The **API Gateway** serves as the single entry point for all requests to the Accounts Service. It handles routing, authentication, authorization, caching, and request logging before forwarding the request to the internal Accounts Service.

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
--header 'X-Request-Id: 2215032c-9b50-4168-8183-0ff3bb53d52e' \
--header 'X-Destination-Service: accounts' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: no-cache' \
--header 'Content-Type: application/json' \
--data '{
    "action" : "balance-enquiry",
    "account_number" : "500007903387"
}'
```

## Actions

The Accounts Service supports multiple actions that can be invoked through the API Gateway. Each action requires the caller to have the corresponding **permission**. All actions return JSON responses with HTTP headers that include request tracing and correlation information.

### Permissions

| Action                   | Required Permission                |
|--------------------------|-----------------------------------|
| `balance-enquiry`         | `accounts.balance-enquiry`         |
| `get-account-details`     | `accounts.get-account-details`    |

---

### 1. Balance Enquiry

This action retrieves the current available balance for a specified account. The endpoint consistently returns an HTTP 200 OK response. For invalid accounts, the balance will be returned as 0.

#### Request

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 4dc8e184-5eb3-437b-a099-e74ffc75b807' \
--header 'X-Destination-Service: accounts' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: no-cache' \
--header 'Content-Type: application/json' \
--data '{
   "action" : "balance-enquiry",
   "account_number" : "500007903387"
}'
```

#### Success Response

```json
{
    "request_id": "8456542f-e715-4777-b545-e6805a36e31e",
    "action": "balance-enquiry",
    "available_balance": 1865.07,
    "success": true,
    "message": "SUCCESS"
}
```

### 2. Account Details

This action retrieves detailed information about a specified account. The endpoint returns an HTTP 200 OK response for valid accounts. If the account is invalid or not found, it returns an HTTP 400 Bad Request with an error message.

#### Request

```bash
curl --location 'http://10.50.30.217:8070/api/v1/service-request' \
--header 'X-Request-Id: 4dc8e184-5eb3-437b-a099-e74ffc75b807' \
--header 'X-Destination-Service: accounts' \
--header 'X-Integration-Id: 1234' \
--header 'X-Integration-Key: 5678' \
--header 'X-Cache-Key: ad1' \
--header 'Cache-Control: no-cache' \
--header 'Content-Type: application/json' \
--data '{
   "action" : "balance-enquiry",
   "account_number" : "500007903387"
}'
```

#### Success Response

```json
{
    "request_id": "a46c452a-27a3-4f9d-bf42-1082fe88940c",
    "action": "get-account-details",
    "account": {
        "account_number": "500007903387",
        "status": "Active             ",
        "account_type": "FCA",
        "currency_iso_code": "USD",
        "account_name": "RYAN BEN",
        "branch_code": 100
    },
    "message": "SUCCESS",
    "success": true,
    "timestamp": "2026-02-10T10:47:13.0504732Z"
}
```

#### Bad Response 

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