# Newman Execution Scripts & Command Templates

Use these templates when constructing executable Newman commands and shell scripts.

## 1. Automated Local Execution Script Template (`scripts/run_api_tests.sh`)

```bash
#!/usr/bin/env bash
# ==============================================================================
# Script: run_api_tests.sh
# Purpose: Execute Postman collections via Newman CLI with HTML reporting
# ==============================================================================

set -eo pipefail

# Configuration Defaults
BASE_URL="${BASE_URL:-http://localhost:3000}"
STUDENT_ID="${STUDENT_ID:-21120000}"
REPORTS_DIR="reports"
COLLECTIONS_DIR="postman/collections"
ENVIRONMENTS_DIR="postman/environments"

mkdir -p "$REPORTS_DIR"

echo "=========================================="
echo " Starting Postman Test Suite Execution"
echo " Target Base URL: $BASE_URL"
echo " Student ID:      $STUDENT_ID"
echo " Reports Folder:  $REPORTS_DIR"
echo "=========================================="

# 1. Health Check
echo "Checking SUT availability at $BASE_URL..."
if ! curl -s -f "$BASE_URL/api/products" > /dev/null; then
    echo "Error: Backend server is not reachable at $BASE_URL"
    echo "Please start the server (e.g., 'cd eshop/backend && node server.js &') before testing."
    exit 1
fi
echo "SUT is reachable."

# 2. Function to execute a single collection
run_suite() {
    local collection_file="$1"
    local environment_file="$2"
    local report_name="$3"
    local title="$4"

    echo ""
    echo "----------------------------------------------------"
    echo "Running: $title"
    echo "Collection:  $collection_file"
    echo "Environment: $environment_file"
    echo "----------------------------------------------------"

    newman run "$collection_file" \
        -e "$environment_file" \
        --env-var "baseUrl=$BASE_URL" \
        --env-var "studentId=$STUDENT_ID" \
        --reporters cli,htmlextra \
        --reporter-htmlextra-export "$REPORTS_DIR/newman-${report_name}.html" \
        --reporter-htmlextra-title "$title"
}

# 3. Execute Suites
# Example: Run API A
if [ -f "$COLLECTIONS_DIR/API_A_Products.postman_collection.json" ]; then
    run_suite \
        "$COLLECTIONS_DIR/API_A_Products.postman_collection.json" \
        "$ENVIRONMENTS_DIR/API_A_Environment.postman_environment.json" \
        "API-A" \
        "HW06 - API A (GET /api/products) Test Report"
fi

# Example: Run API B
if [ -f "$COLLECTIONS_DIR/API_B_Cart.postman_collection.json" ]; then
    run_suite \
        "$COLLECTIONS_DIR/API_B_Cart.postman_collection.json" \
        "$ENVIRONMENTS_DIR/API_B_Environment.postman_environment.json" \
        "API-B" \
        "HW06 - API B (POST /api/cart) Test Report"
fi

# Example: Run API C
if [ -f "$COLLECTIONS_DIR/API_C_ImportProducts.postman_collection.json" ]; then
    run_suite \
        "$COLLECTIONS_DIR/API_C_ImportProducts.postman_collection.json" \
        "$ENVIRONMENTS_DIR/API_C_Environment.postman_environment.json" \
        "API-C" \
        "HW06 - API C (POST /api/admin/import-products) Test Report"
fi

echo ""
echo "=========================================="
echo " All test suites finished successfully!"
echo " Reports generated under '$REPORTS_DIR/'"
echo "=========================================="
```

---

## 2. GitHub Actions CI Step Template

```yaml
- name: Install Newman & HTML Reporter
  run: npm install -g newman newman-reporter-htmlextra

- name: Run Newman Test Suite
  run: |
    newman run postman/collections/<COLLECTION_FILE>.json \
      -e postman/environments/<ENVIRONMENT_FILE>.json \
      --reporters cli,htmlextra \
      --reporter-htmlextra-export reports/newman-<REPORT_NAME>.html \
      --reporter-htmlextra-title "API Test Report"

- name: Upload HTML Test Report Artifact
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: newman-reports
    path: reports/
    retention-days: 14
```
