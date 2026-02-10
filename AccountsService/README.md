# Accounts Service API Documentation

The **Accounts Service** handles account-related queries such as balance enquiries, account statements, and other account operations. All requests are routed through the **Service Request API**.


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
