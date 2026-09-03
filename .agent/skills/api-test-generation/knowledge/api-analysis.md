# REST API Specification Analysis & Contract Modeling

## 1. Principles of Contract Analysis
When reviewing `eshop/api_specification.md`, extract and analyze the following dimensions for each endpoint:

### A. Endpoint Metadata
* **HTTP Method:** Determine the correct verb (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
* **Path & Parameters:** Identify URL path segments, path variables (e.g., `/api/products/:id`), and query parameters (e.g., `?page=1&limit=10&category=electronics`).
* **Content Negotiation:** Expected `Content-Type` (`application/json`, `multipart/form-data`) and `Accept` headers.

### B. Authentication & Authorization Boundaries
* **Public Endpoints:** Accessible without credentials (e.g., `POST /api/register`, `POST /api/login`, public product listings).
* **Protected Endpoints:** Require valid bearer token via `Authorization: Bearer {{token}}`.
* **Role-Based Access Control (RBAC):** Identify endpoints restricted to specific roles (e.g., `admin` vs standard `user`).
* **Student Identification:** Every request must transmit the header `X-Student-Id: {{studentId}}`.

### C. Request Payload Contract
* **Field Definitions:** Name, expected data type (string, integer, float, boolean, array, object).
* **Mandatory vs Optional:** Which fields are strictly required for the request to be accepted.
* **Constraints:** Minimum/maximum lengths, numerical bounds, regex formats (e.g., email format), allowed enumerations.

### D. Response Contract & Status Codes
* **Success Responses:** Status codes (`200 OK`, `201 Created`, `204 No Content`) and corresponding payload structure.
* **Client Error Responses:**
  - `400 Bad Request`: Validation failure, malformed payload syntax, missing required fields.
  - `401 Unauthorized`: Missing or invalid authentication token.
  - `403 Forbidden`: Authenticated user lacks permission / insufficient role.
  - `404 Not Found`: Target resource does not exist.
  - `409 Conflict`: Duplicate entry or conflicting resource state.
  - `422 Unprocessable Entity`: Semantic validation failures.
* **Server Error Responses:** `500 Internal Server Error` indicates unhandled exception or backend crash.

---

## 2. Dependency & Workflow Modeling

### A. Constructing E2E Request Chains
Real-world API workflows require chaining interdependent requests:
```text
[Auth: Login/Register] ──► [Resource Discovery: GET Items] ──► [Action: POST to Cart/Order] ──► [Verification: GET Status]
```

### B. Dynamic Variable Correlation
* **Authentication Token:**
  - Login Request: `POST /api/login`
  - Extraction: Extract `token` from response JSON.
  - Downstream Injection: Store in environment variable `{{token}}` and inject into `Authorization: Bearer {{token}}`.
* **Resource Identifier:**
  - Creation/Query Request: `POST /api/products` or `GET /api/products`
  - Extraction: Extract `id` from response JSON.
  - Downstream Injection: Pass into target endpoint `GET /api/products/{{productId}}` or `DELETE /api/products/{{productId}}`.

---

## 3. Ground Truth & Contract Integrity Rules
1. **Authoritative Specification:** Treat `eshop/api_specification.md` as the definitive ground truth.
2. **No Fabricated Behaviors:** If the specification does not state that an endpoint supports a parameter or behavior, do not assume or invent it.
3. **Behavior Verification:** Always compare actual backend response status and schema against the specification contract.
