# Postman Design Patterns & Best Practices

## 1. Collection Architecture (v2.1.0)
Organize collections to ensure maintainability, clear execution order, and clean reporting:

* **Top-Level Information:** Name, descriptive description, and schema definition (`https://schema.getpostman.com/json/collection/v2.1.0/collection.json`).
* **Folder Hierarchy:**
  - By Endpoint / Resource (e.g., `01_Products`, `02_Cart`, `03_Import`).
  - Or by Test Suite Category (e.g., `Domain Tests`, `State & Business Tests`, `Security Tests`, `Schema Tests`).
* **Environment Configuration:**
  - Base URL: `{{baseUrl}}` (defaulting to `http://localhost:3000`).
  - Student Identifier: `{{studentId}}` (used across all requests).
  - Dynamic Tokens: `{{token}}`, `{{adminToken}}`.
  - Dynamic Entity IDs: `{{productId}}`, `{{cartId}}`.

---

## 2. Standard Request Configuration

### A. Mandatory Headers
Attach the following headers to every request item:
```json
[
  {
    "key": "Content-Type",
    "value": "application/json",
    "type": "text"
  },
  {
    "key": "X-Student-Id",
    "value": "{{studentId}}",
    "type": "text",
    "description": "Student identification header"
  }
]
```
For protected endpoints, include:
```json
{
  "key": "Authorization",
  "value": "Bearer {{token}}",
  "type": "text"
}
```

### B. Pre-Request Script Patterns
Use pre-request scripts to prepare test preconditions or unique dynamic values:
```javascript
// Generate unique email or string when uniqueness is required
const timestamp = Date.now();
pm.variables.set("dynamicEmail", `testuser_${timestamp}@domain.com`);
```

---

## 3. Postman Test Scripts & Assertion Patterns

### A. Status Code Assertions
```javascript
pm.test("Status code is 200 OK", function () {
    pm.response.to.have.status(200);
});

pm.test("Status code is 400 Bad Request", function () {
    pm.response.to.have.status(400);
});
```

### B. Response Structure & JSON Schema Assertions
```javascript
pm.test("Response is valid JSON with required fields", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an("object");
    pm.expect(jsonData).to.have.property("id");
    pm.expect(jsonData.id).to.be.a("number");
    pm.expect(jsonData).to.have.property("name");
    pm.expect(jsonData.name).to.be.a("string");
});
```

### C. Business State & Value Assertions
```javascript
pm.test("Response body matches expected business value", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.quantity).to.eql(2);
    pm.expect(jsonData.status).to.eql("active");
});
```

### D. Negative Test & Error Message Assertions
Never assert HTTP 200 for negative test scenarios:
```javascript
pm.test("Error response contains meaningful message", function () {
    pm.response.to.have.status(400);
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property("message");
    pm.expect(jsonData.message).to.be.a("string").and.not.empty;
});
```

### E. Request Chaining & State Propagation
```javascript
pm.test("Extract and store authentication token", function () {
    pm.response.to.have.status(200);
    const jsonData = pm.response.json();
    pm.expect(jsonData.token).to.be.a("string");
    pm.environment.set("token", jsonData.token);
});
```

---

## 4. Test Integrity Rules
* **No Test Deletion:** Never remove a test case because the target backend fails the assertion. A failing test serves as contract defect evidence.
* **No Assertion Weakening:** Do not change `status(400)` to `status(200)` or delete payload checks to force a test to pass.
* **Preserve Contract Expectations:** Expected results must strictly reflect the specification, not buggy SUT behavior.
* **No Backend Patching:** Do not modify application source code to accommodate failing test assertions.
