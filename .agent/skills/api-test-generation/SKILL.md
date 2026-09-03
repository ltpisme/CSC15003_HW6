---
name: api-test-generation
description: End-to-end workflow to read API specifications, analyze REST API contracts, design comprehensive test suites (domain, state, security, schema), generate runnable Postman collections and environments with X-Student-Id support, and construct Newman CLI execution scripts and CI commands.
---

# API Test Generation Skill

## Overview
This skill guides an AI agent through analyzing REST API contracts, designing robust API test scenarios, generating executable Postman collections and environments, and constructing automated Newman CLI execution scripts.

The skill strictly focuses on **API test generation and execution setup**. It does not perform human review, AI audit, execution analysis, bug report authoring, `report.md` editing, backend source code modification, or Git commits.

---

## Purpose
The skill reads:
```text
eshop/api_specification.md
```
and uses the specified API contract to create executable API testing artifacts without fabricating endpoints, parameters, or behaviors not documented in the specification.

---

## Input Requirements
* **Primary (Mandatory):** `eshop/api_specification.md` — The authoritative source for endpoints, HTTP methods, headers, schemas, business logic, and security constraints.
* **Secondary (Optional):** Existing repository structure, environment files (`postman/environments/`), or CI workflows (`.github/workflows/`) for alignment with existing conventions.

---

## Standard 6-Step Workflow

```text
┌────────────────────────────────────────────────────────┐
│ Step 1: Read API Specification                         │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Step 2: Analyze Endpoint & Contract                    │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Step 3: Identify Test Scenarios                        │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Step 4: Generate Postman Collection & Environment      │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Step 5: Generate Newman Execution Script & Commands    │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Step 6: Validate Generated Artifacts                   │
└────────────────────────────────────────────────────────┘
```

---

### Step 1: Read API Specification
1. Parse `eshop/api_specification.md` to identify all documented resources and endpoints.
2. Note base URL conventions (e.g., `http://localhost:3000`), port, and authentication scheme (e.g., Bearer JWT, session).
3. Do not invent endpoints, fields, or behaviors not present in the specification.

---

### Step 2: Analyze Endpoint & Contract
1. For each endpoint, extract and document:
   - HTTP Method (`GET`, `POST`, `PUT`, `DELETE`, etc.) and exact URL path.
   - Required headers (e.g., `Content-Type: application/json`, `Authorization: Bearer <token>`).
   - Request parameters (path parameters, query parameters) and body payload schema.
   - Success status codes (`200 OK`, `201 Created`, etc.) and response JSON schema.
   - Error status codes (`400`, `401`, `403`, `404`, `409`, `422`, `500`) and expected error payloads.
   - Validation constraints, business rules, and security access levels.
2. Consult `knowledge/api-analysis.md` for detailed contract analysis patterns.

---

### Step 3: Identify Test Scenarios
Design comprehensive test suites across four required dimensions:
1. **Domain Testing:**
   - Valid inputs, invalid inputs, boundary values (min, max, empty, null, zero, negative).
   - Missing required fields, extra unexpected fields, incorrect data types.
2. **State & Business Logic Testing:**
   - Valid state transitions and workflows (e.g., Cart $\to$ Checkout).
   - Invalid state transitions, business rule violations (e.g., insufficient stock, duplicate resources).
   - Resource dependency verification.
3. **Security Testing:**
   - Authentication enforcement (missing/invalid/expired token).
   - Authorization & Role-Based Access Control (RBAC, privilege escalation).
   - IDOR (Insecure Direct Object Reference), injection attempts, and malformed inputs.
   - Only cover security requirements defined in or applicable to the specification; do not fabricate behaviors.
4. **Schema Testing:**
   - Status code accuracy, response JSON structure, field data types, required fields, and response headers (`Content-Type`).
5. Consult `knowledge/test-types.md` for test design guidelines.

---

### Step 4: Generate Postman Collection & Environment
1. Generate valid Postman Collection v2.1.0 JSON format files under `postman/collections/`.
2. Generate or update corresponding Postman Environment files under `postman/environments/`.
3. Adhere to mandatory Postman generation rules:
   - **Student Identification Header:** Every request must include:
     ```http
     X-Student-Id: {{studentId}}
     ```
     Never hard-code the Student ID; always bind it to the `studentId` variable.
   - **Variable Structure:** Parameterize `baseUrl`, `studentId`, `token`, and entity IDs.
   - **Request Chaining:** When requests depend on earlier responses (e.g., login token, created item ID), extract and set variables in the `test` script (`pm.environment.set(...)`).
   - **Comprehensive Assertions:** Check HTTP status codes, response time, response body schema, field types, and business payload values. Never write assertions that only check for status 200 without payload verification.
   - **Test Integrity Rules:**
     - Do NOT delete a test because it fails against the backend SUT.
     - Do NOT weaken assertions to make tests artificially pass.
     - Do NOT alter expected behavior to match buggy actual behavior.
     - Do NOT modify backend code to satisfy tests.
4. Consult `knowledge/postman-patterns.md` and `templates/postman-collection-template.md`.

---

### Step 5: Generate Newman Execution Script & Commands
1. Construct headless CLI execution commands using Newman:
   ```bash
   newman run postman/collections/<collection_name>.postman_collection.json \
     -e postman/environments/<environment_name>.postman_environment.json \
     --reporters cli,htmlextra \
     --reporter-htmlextra-export reports/newman-<report_name>.html \
     --reporter-htmlextra-title "API Test Report"
   ```
2. If the repository lacks a suitable execution script directory, create `scripts/` (e.g., `scripts/run_api_tests.sh`). If the project already has an established execution mechanism (e.g., `.github/workflows/api-testing.yml`), align commands with it without redesigning the CI/CD architecture.
3. Ensure execution commands:
   - Target the correct collection and environment.
   - Output structured CLI logs and an HTML report via `htmlextra`.
   - Propagate non-zero exit codes upon test failure for CI/CD gating.
   - Support both local CLI execution and headless GitHub Actions runners.
4. Consult `knowledge/newman-execution.md` and `templates/execution-script-template.md`.

---

### Step 6: Validate Generated Artifacts
1. Ensure all JSON files (`postman/collections/*.json`, `postman/environments/*.json`) are syntactically valid JSON.
2. Verify that every request contains the `X-Student-Id: {{studentId}}` header.
3. Verify that environment files declare initial and current values for required variables (`baseUrl`, `studentId`, etc.).
4. Verify that execution commands reference existing collection and environment file paths.

---

## Output Artifacts
* Postman Collections: `postman/collections/<collection_name>.postman_collection.json`
* Postman Environments: `postman/environments/<environment_name>.postman_environment.json`
* Newman Execution Scripts/Commands: `scripts/run_api_tests.sh` (or CLI commands aligned with `.github/workflows/api-testing.yml`)
