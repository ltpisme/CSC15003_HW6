# Newman Headless CLI Execution & CI Integration

## 1. Newman CLI Overview
Newman is the command-line collection runner for Postman. It allows executing API test suites in automated pipelines and local headless environments.

### Required Packages
```bash
npm install -g newman newman-reporter-htmlextra
```

---

## 2. Core Execution Commands

### A. Basic Execution
Run a collection with a specific environment:
```bash
newman run postman/collections/<collection_name>.postman_collection.json \
  -e postman/environments/<environment_name>.postman_environment.json
```

### B. Execution with HTML & CLI Reporting
Generate both human-readable terminal output and rich HTML reports using `htmlextra`:
```bash
newman run postman/collections/<collection_name>.postman_collection.json \
  -e postman/environments/<environment_name>.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/newman-<report_name>.html \
  --reporter-htmlextra-title "API Test Report - <Scenario>"
```

### C. Overriding Variables via CLI
Override environment variables directly from the terminal without altering JSON files:
```bash
newman run postman/collections/<collection_name>.postman_collection.json \
  -e postman/environments/<environment_name>.postman_environment.json \
  --env-var "baseUrl=http://localhost:3000" \
  --env-var "studentId=21120000"
```

---

## 3. Exit Code & Pipeline Failure Handling
* **Zero Exit Code (`0`):** All requests succeeded, and all assertions passed.
* **Non-Zero Exit Code (`1`):** At least one assertion failed or a request timed out/errored.
* **CI/CD Quality Gate:** Newman inherently exits with code `1` when tests fail. This signals CI/CD pipelines (e.g., GitHub Actions) that the build or contract verification failed.
* When executing multiple collections in sequence within a script, propagate exit codes properly (e.g., using `set -e` or accumulating status codes) so errors are not silently masked.

---

## 4. Execution Workflow Patterns

### A. Local Automation Script Pattern
```bash
#!/usr/bin/env bash
set -e

# 1. Verify SUT readiness
echo "Verifying SUT availability..."
curl -s -f http://localhost:3000/api/products > /dev/null || {
  echo "Error: Backend server is not running on http://localhost:3000"
  exit 1
}

# 2. Prepare output directory
mkdir -p reports

# 3. Run collections
newman run postman/collections/API_A_Products.postman_collection.json \
  -e postman/environments/API_A_Environment.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/newman-API-A.html
```

### B. GitHub Actions CI/CD Integration
In GitHub Actions workflow (`.github/workflows/api-testing.yml`), execute Newman after spinning up the backend service:
```yaml
- name: Run Newman Test Suite
  run: |
    newman run postman/collections/<collection>.postman_collection.json \
      -e postman/environments/<env>.postman_environment.json \
      --reporters cli,htmlextra \
      --reporter-htmlextra-export reports/newman-report.html
```
Align with existing workflow steps without modifying the core CI/CD architecture unnecessarily.
