# Postman Collection & Request Template

Use this reference template when generating Postman Collection v2.1.0 JSON artifacts.

## 1. Postman Collection Skeleton (JSON v2.1.0)

```json
{
  "info": {
    "_postman_id": "00000000-0000-0000-0000-000000000000",
    "name": "API_TestSuite_Name",
    "description": "Comprehensive API Test Suite for EShop",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "01_Domain_Tests",
      "item": []
    },
    {
      "name": "02_State_Business_Tests",
      "item": []
    },
    {
      "name": "03_Security_Tests",
      "item": []
    },
    {
      "name": "04_Schema_Tests",
      "item": []
    }
  ],
  "variable": [
    {
      "key": "baseUrl",
      "value": "http://localhost:3000",
      "type": "string"
    }
  ]
}
```

---

## 2. Standard Request Item Template (with X-Student-Id & Assertions)

```json
{
  "name": "TC-DOM-01 - Create Resource with Valid Data",
  "event": [
    {
      "listen": "prerequest",
      "script": {
        "exec": [
          "// Optional Pre-request logic",
          "pm.variables.set('reqTimestamp', Date.now());"
        ],
        "type": "text/javascript"
      }
    },
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test(\"Status code is 201 Created\", function () {",
          "    pm.response.to.have.status(201);",
          "});",
          "",
          "pm.test(\"Response time is acceptable (< 2000ms)\", function () {",
          "    pm.expect(pm.response.responseTime).to.be.below(2000);",
          "});",
          "",
          "pm.test(\"Response has correct JSON structure and types\", function () {",
          "    const jsonData = pm.response.json();",
          "    pm.expect(jsonData).to.be.an(\"object\");",
          "    pm.expect(jsonData).to.have.property(\"id\");",
          "    pm.expect(jsonData.id).to.be.a(\"number\");",
          "    pm.expect(jsonData).to.have.property(\"message\");",
          "    pm.expect(jsonData.message).to.be.a(\"string\");",
          "});",
          "",
          "// Dynamic variable chaining for downstream requests",
          "if (pm.response.code === 201) {",
          "    const jsonData = pm.response.json();",
          "    pm.environment.set(\"createdResourceId\", jsonData.id);",
          "}"
        ],
        "type": "text/javascript"
      }
    }
  ],
  "request": {
    "method": "POST",
    "header": [
      {
        "key": "Content-Type",
        "value": "application/json",
        "type": "text"
      },
      {
        "key": "X-Student-Id",
        "value": "{{studentId}}",
        "type": "text",
        "description": "Student ID header"
      },
      {
        "key": "Authorization",
        "value": "Bearer {{token}}",
        "type": "text",
        "description": "Bearer token for authenticated endpoint"
      }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"name\": \"Standard Product\",\n  \"price\": 100,\n  \"quantity\": 5\n}",
      "options": {
        "raw": {
          "language": "json"
        }
      }
    },
    "url": {
      "raw": "{{baseUrl}}/api/products",
      "host": [
        "{{baseUrl}}"
      ],
      "path": [
        "api",
        "products"
      ]
    }
  },
  "response": []
}
```

---

## 3. Postman Environment Template

```json
{
  "id": "11111111-1111-1111-1111-111111111111",
  "name": "EShop_Environment",
  "values": [
    {
      "key": "baseUrl",
      "value": "http://localhost:3000",
      "type": "default",
      "enabled": true
    },
    {
      "key": "studentId",
      "value": "21120000",
      "type": "default",
      "enabled": true
    },
    {
      "key": "token",
      "value": "",
      "type": "secret",
      "enabled": true
    }
  ],
  "_postman_variable_scope": "environment"
}
```
