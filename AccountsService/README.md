# Accounts Service API Documentation

The **Accounts Service** handles account-related queries such as balance enquiries, account statements, and other account operations. All requests are routed through the **Service Request API**.

## Entry Point (API Gateway)

The **API Gateway** serves as the single entry point for all requests to the Accounts Service. It handles routing, authentication, authorization, caching, and request logging before forwarding the request to the internal Accounts Service.

### Responsibilities

- **Routing:** Directs requests to the correct service (`accounts`) based on the `X-Destination-Service` header.  
- **Authentication & Authorization:** Validates `X-Integration-Id` and `X-Integration-Key` headers to ensure only authorized clients can access the service.