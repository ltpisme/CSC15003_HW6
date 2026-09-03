# HW06 — Kiểm Toán Tạo Tác Thực Thi & Hiệu Chỉnh Của Sinh Viên (Phase 4 & 5: Executable Artifacts Audit)

> **Môn học:** Kiểm thử phần mềm (CSC15003) — Bài tập HW06  
> **Sinh viên thực hiện:** Lê Thanh Phong — MSSV: 23127452  
> **Công cụ trợ lý:** Antigravity — Gemini  
> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/06_generation_decision.md`, `07_postman_audit.md`, `07_postman_ai_audit.md`, `postman/`, `.github/workflows/api-testing.yml`

---

## 9. Quyết Định Sinh Tạo Tác Thực Thi Postman & CI/CD (Generation Decisions 1–5)

<a id="log-9"></a>

- **Timestamp**: 17:00 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/analysis-generated/05_test_design_decision.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Chuyển giao các quyết định thiết kế sang các nguyên tắc kỹ thuật cụ thể để sinh mã JSON cho Postman Collections, Environment files, Data files và cấu hình pipeline CI/CD trên GitHub Actions.

==Prompt End==

- **Output**

==Output Start==

Xác lập 5 quyết định sinh tạo tác:
1. Sinh 3 tệp Postman Collection v2.1.0 tương ứng với 3 API.
2. Sinh các tệp Postman Environment chuẩn hóa (`API_A_Environment`, `API_B_Environment`, `API_B_BugEvidence`, `API_C_Environment`, `API_C_BugEvidence`).
3. Tạo tệp CSV mẫu `postman/data/API_C_import_sample.csv` phục vụ kiểm thử data-driven cho tính năng FR-16.
4. Xây dựng workflow `.github/workflows/api-testing.yml` thực thi tuần tự Newman và xuất báo cáo `newman-reporter-htmlextra`.
5. Bảo đảm tất cả các ca phát hiện lỗi SUT sử dụng cấu trúc assertion kiểm tra cờ `strict_contract_assertion`.
Xuất tài liệu: `temp/HW06/analysis-generated/06_generation_decision.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Các quyết định ánh xạ hoàn hảo từ thiết kế trừu tượng sang các tạo tác thực thi chuẩn của Postman và Newman.
- **Student fix**: Không cần thiết

---

## 10. Khởi Tạo Ban Đầu Các Tạo Tác Thực Thi Postman & CI/CD

<a id="log-10"></a>

- **Timestamp**: 17:30 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/analysis-generated/06_generation_decision.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Khởi tạo toàn bộ các tệp JSON Collection trong `postman/collections/`, các tệp Environment trong `postman/environments/`, tệp dữ liệu CSV trong `postman/data/` và tệp GitHub Actions workflow `.github/workflows/api-testing.yml`.

==Prompt End==

- **Output**

==Output Start==

Khởi tạo các tệp tạo tác trong workspace:
- `postman/collections/API_A_Products.postman_collection.json` (39 requests)
- `postman/collections/API_B_Cart.postman_collection.json` (46 requests)
- `postman/collections/API_C_ImportProducts.postman_collection.json` (43 requests)
- `postman/environments/API_A_Environment.postman_environment.json`
- `postman/environments/API_B_Environment.postman_environment.json`, `API_B_BugEvidence.postman_environment.json`
- `postman/environments/API_C_Environment.postman_environment.json`, `API_C_BugEvidence.postman_environment.json`
- `postman/data/API_C_import_sample.csv`
- `.github/workflows/api-testing.yml`
*(Được đưa vào lịch sử commit: `1a9d92d`, `0e4f867`)*.

==Output End==

- **Verdict**: Incomplete
- **Reasoning**: Bản khởi tạo ban đầu có 2 thiếu sót kỹ thuật: (1) API A thiếu tệp môi trường `BugEvidence` và thiếu biến `strict_contract_assertion`, gây mất đồng bộ với API B và C; (2) Workflow CI/CD thiếu cấu hình `continue-on-error`, khiến việc Step 1 (API A) gặp assertion fail sẽ làm ngắt toàn bộ pipeline và bỏ qua API B, C.
- **Student fix**: Cần thực hiện kiểm toán độc lập và hiệu chỉnh tạo tác ở giai đoạn Human Audit tiếp theo.

---

## 11. Kiểm Toán & Hiệu Chỉnh Tạo Tác Thực Thi (Prompt 1 — Audit & Correction)

<a id="log-11"></a>

- **Timestamp**: 19:10 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `postman/`, `.github/workflows/api-testing.yml`, `reports/pre-audit/`
- **Prompt**:

==Prompt Start==

### PROMPT 1 — AUDIT AND CORRECT EXECUTABLE API TEST ARTIFACTS

Complete the Human Audit → Student Correction stage for HW06.

Do NOT regenerate the test design.
Do NOT execute the tests yet.
Do NOT create commits.

### P0 — Read only
...
### Audit each API independently
For API A, B, C verify:
1. Request/method/URL
2. Environment variables
3. Request chaining
4. Authentication/authorization where applicable
5. X-Student-Id
6. Domain/boundary tests
7. State/business tests
8. SEC01–SEC07 coverage where applicable
9. Schema validation
10. Assertions
11. Negative tests
12. Student-authored extension >=5/API if actually evidenced
13. CSV/data-driven execution where applicable
14. CI compatibility

Compare the executable tests against the FINAL test design, not against the earlier AI draft.

### Allowed correction
Fix only small executable-artifact problems, such as:
- wrong variable name/reference
- broken variable mapping
- missing X-Student-Id
- incorrect request chaining
- incorrect test-script syntax
- missing/incorrect assertion
- incorrect test-data mapping
- collection/environment inconsistency
- small Newman/CI path or configuration issue

Preserve the intended test design.

### Forbidden
Do NOT:
- modify application source code
- fix SUT/business bugs
- weaken assertions
- delete failing tests
- change Expected to Actual
- remove security/state/schema tests
- redesign the test suite only because the SUT fails

Create:
`temp/HW06/analysis-generated/07_postman_audit.md`
`temp/HW06/analysis-generated/07_postman_ai_audit.md`

==Prompt End==

- **Output**

==Output Start==

Hoàn thành kiểm toán 14 tiêu chí kỹ thuật độc lập trên cả 3 API:
- Kiểm tra cú pháp script, ánh xạ biến, đường dẫn URL, logic phân quyền, header `X-Student-Id`.
- Phát hiện và ghi nhận 2 ca lỗi đặc thù SUT SQLite của API A (`TC-SEC-A01` và `TC-NEW-A03`), giữ nguyên vẹn assertion theo quy tắc không sửa Expected thành Actual.
- Hiệu chỉnh tệp `.github/workflows/api-testing.yml`: bổ sung `continue-on-error: true` cho từng bước chạy Newman, hỗ trợ tham số `RUN_MODE`, xuất tên báo cáo phân biệt `-allpass.html` / `-failure.html`, và thêm step đánh giá trạng thái cuối cùng theo REQ-49.
- Bổ sung biến `"strict_contract_assertion": "false"` vào `API_A_Environment.json` và tạo mới `postman/environments/API_A_BugEvidence.postman_environment.json` (`"strict_contract_assertion": "true"`) để bảo đảm tính đối xứng môi trường 100% cho cả 3 API.
Xuất tài liệu: `temp/HW06/analysis-generated/07_postman_audit.md`, `07_postman_ai_audit.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Thực hiện chính xác các hiệu chỉnh kỹ thuật cho phép (sửa lỗi ngắt luồng CI, đồng bộ tệp môi trường) và kiên quyết bảo lưu các lỗi phần mềm của SUT mà không làm yếu assertion.
- **Student fix**: Đã áp dụng các sửa đổi vào `.github/workflows/api-testing.yml` và `postman/environments/API_A_BugEvidence.postman_environment.json`.
