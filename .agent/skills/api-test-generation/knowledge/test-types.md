# API Test Types & Scenario Design Matrix

## 1. Domain Testing
Validates how endpoints handle input data boundaries, data formats, and validation rules.

* **Valid Inputs:** Typical standard inputs conforming to specification expectations.
* **Invalid Inputs:** Out-of-spec values (e.g., negative prices, malformed email strings).
* **Boundary Value Analysis (BVA):**
  - Numeric limits: `min`, `min - 1`, `min + 1`, `max`, `max - 1`, `max + 1`, `0`, negative numbers.
  - String limits: empty string `""`, single-character strings, maximum allowed character length, over-limit strings.
  - Array / Collection limits: empty array `[]`, single-item array, maximum capacity array.
* **Missing & Null Values:** Omitted required JSON fields, explicit `null` values.
* **Wrong Data Types:** Passing string instead of integer, array instead of object, integer instead of boolean.

---

## 2. State & Business Logic Testing
Validates workflow state transitions, business rules, and multi-step resource lifecycles.

* **Valid State Transitions:**
  - Create resource $\to$ Retrieve resource $\to$ Update resource $\to$ Delete resource $\to$ Verify 404.
  - Add product to cart $\to$ Update item quantity $\to$ Complete checkout.
* **Invalid State Transitions:**
  - Attempting to check out an empty cart.
  - Attempting to re-cancel an already completed or cancelled order.
  - Attempting to refund an unpaid transaction.
* **Business Rule Violations:**
  - Requesting order quantities greater than current warehouse stock.
  - Applying invalid or expired discount coupons.
  - Registering duplicate accounts with an already registered email.
* **Dependent Resource State:**
  - Operating on non-existent parent resources (e.g., adding an item to a non-existent cart).

---

## 3. Security Testing
Validates API defense against unauthorized access, privilege escalation, and malformed inputs. Only cover constraints documented in or applicable to the specification; do not fabricate custom security behavior.

* **Authentication Verification:**
  - Accessing protected endpoints without the `Authorization` header.
  - Accessing protected endpoints with an invalid, malformed, or expired Bearer token.
* **Authorization & Privilege Escalation:**
  - Standard user attempting to call admin-restricted endpoints (e.g., `/api/admin/*`).
  - Accessing another user's private data or orders (IDOR: Insecure Direct Object Reference).
* **Injection & Malformed Input Handling:**
  - SQL injection patterns in query parameters or payload fields (e.g., `' OR '1'='1`).
  - Cross-Site Scripting (XSS) payload injection in text fields (`<script>alert(1)</script>`).
  - Oversized payloads and unexpected JSON structures.
* **Unauthorized Access Safeguards:**
  - Verify endpoints return `401 Unauthorized` or `403 Forbidden` rather than exposing internal errors or stack traces.

---

## 4. Schema & Contract Testing
Validates that response headers, status codes, and payloads strictly comply with the API contract.

* **HTTP Status Code Conformity:** Verify status code semantics (`200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`).
* **Response Payload Structure:** Ensure response JSON is an object or array matching the contract.
* **Required Field Presence:** Assert mandatory fields (`id`, `name`, `status`, `createdAt`, etc.) exist in the response.
* **Data Type Integrity:** Validate types of returned properties (e.g., `id` is a number, `items` is an array).
* **Content-Type Validation:** Verify `Content-Type: application/json` is returned.
* **Response Consistency:** Ensure identical error response formats across all failure scenarios.

---

## 5. Negative Testing Guidelines
* **Negative Test Requirement:** Every negative test must assert a client error status (`4xx`), not `200 OK`.
* **Error Payload Verification:** Verify that error responses include structured error fields (e.g., `message` or `error`) explaining the failure.
