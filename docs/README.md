# HW06 - API Testing với Postman, Newman & GitHub Actions

> Lê Thanh Phong - 23127452

---

## 1. Báo Cáo Tóm Tắt Kiểm Thử (Test Summary Report)

Dự án thực hiện kiểm thử tự động hóa API toàn diện cho hệ thống **E-Shop REST API** (`eshop/`), bao phủ 3 endpoint mục tiêu đại diện cho 3 cấp độ phân quyền và mô hình xử lý:

| API             | Endpoint                       |  Method  | Role               | Chức năng nghiệp vụ chính                                                               |
| :-------------- | :----------------------------- | :------: | :----------------- | :------------------------------------------------------------------------------------------- |
| **API A** | `/api/products`              | `GET` | Public             | Xem danh mục sản phẩm, tìm kiếm từ khóa, phân trang, lọc theo danh mục             |
| **API B** | `/api/cart`                  | `POST` | Authenticated User | Thêm sản phẩm vào giỏ hàng cá nhân, kiểm soát số lượng, tính toàn vẹn phiên |
| **API C** | `/api/admin/import-products` | `POST` | Admin Only         | Nhập danh mục sản phẩm hàng loạt từ mảng JSON hoặc dữ liệu CSV                    |

### Bảng Tổng Hợp Kịch Bản & Kết Quả Thực Nghiệm

| API             | Tệp Postman Collection                                              | Phân Bổ Ca Kiểm Thử (Categories)                                                                              |  Ca AI Sinh  | Ca Mở Rộng |              Tổng Số Ca              | Kết Quả Thực Nghiệm Newman (Result)                                                                                                                                                | Bằng Chứng Báo Cáo (Evidence)                                                                                     |
| :-------------- | :------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------- | :-----------: | :----------: | :-------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| **API A** | `postman/collections/API_A_Products.postman_collection.json`       | Domain (20), Security (5), State (2), Schema (3), Business (2), Dependency (2), Extension (5)                     |      34      |      5      |              **39**              | • Requests:**39** \| Asserts: **79**• Fail: **2** (Lỗi biểu thức SQLi SUT SQLite)                                                                               | •`reports/post-audit/postman/newman-API-A.html`• `reports/post-audit/github-action/newman-API-A.html`           |
| **API B** | `postman/collections/API_B_Cart.postman_collection.json`           | Domain (21), Security (4), State (4), Schema (3), HTTP (2), Business (1), Dependency (1), Extension (5) + 5 Setup |      36      |      5      |        **41***(+ 5 setup)*        | •**All-Pass Mode:** 46 reqs, 62 asserts, 2 syntax fails• **Bug Evidence Mode:** 46 reqs, 62 asserts, **19 contract fails** + 2 syntax                              | •`reports/post-audit/postman/newman-API-B-allpass.html`• `reports/post-audit/postman/newman-API-B-failure.html` |
| **API C** | `postman/collections/API_C_ImportProducts.postman_collection.json` | Domain (21), Security (5), State (4), Schema (4), Business (1), Dependency (1), Extension (5) + 2 Setup           |      36      |      5      |        **41***(+ 2 setup)*        | •**All-Pass Mode:** 43 reqs, 83 asserts, **0 fail (100% Pass)**• **Bug Evidence Mode:** 43 reqs, 83 asserts, **2 contract fails** (Crash 500, Bypass SEC-03) | •`reports/post-audit/postman/newman-API-C-allpass.html`• `reports/post-audit/postman/newman-API-C-failure.html` |
| **TỔNG** | **3 Collections**                                              | **7 nhóm tiêu chuẩn chất lượng**                                                                      | **106** | **15** | **121 chính thức***(+ 7 setup)* | **Phát hiện 8 nhóm khiếm khuyết SUT thực tế**                                                                                                                             | **7 báo cáo HTML Post-audit**                                                                                 |

- **Ràng buộc chống gian lận (`REQ-41`, `REQ-64`):** 100% request được tự động gắn header `X-Student-Id: 23127452` và in nhật ký console qua Collection Pre-request Script.
- **Ca kiểm thử mở rộng (`REQ-38`):** Mỗi API có đúng 5 ca mở rộng do sinh viên tự thiết kế độc lập tại thư mục `07_Extended_Behaviors` (Tổng 15 ca).

---

## 2. Tổng Hợp Lỗi Hệ Thống (Bug Summary)

Tổng hợp các khiếm khuyết phần mềm thực tế (**Confirmed Genuine SUT Bugs**) được phát hiện và xác nhận qua thực thi Newman:

|        Bug ID        |       API       | Mô Tả Lỗi Thực Tế (Action + Actual Result)                                                                                                                                     |     Mức Độ     |               Test Case ID               | Bằng Chứng Thực Nghiệm (Evidence)                    |
| :------------------: | :-------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------: | :---------------------------------------: | :------------------------------------------------------- |
| **`BUG-01`** | **API C** | Gửi token User thường gọi endpoint import của Admin; SUT phản hồi`200 OK` và chèn CSDL thành công (vi phạm nghiêm trọng SEC-03).                                    | **Critical** |              `TC-SEC-C01`              | `reports/post-audit/postman/newman-API-C-failure.html` |
| **`BUG-02`** | **API C** | Gửi mảng chứa phần tử`null` (`products: [null]`); SUT sập với mã `500 Internal Server Error` do `TypeError: Cannot read properties of null` tại `server.js:214`. |   **High**   |              `TC-DOM-C21`              | `reports/post-audit/postman/newman-API-C-failure.html` |
| **`BUG-03`** | **API B** | Thao túng đơn giá từ client (`price = 1000` thay vì 30.000.000); backend lưu giá giả mạo vào RAM mà không kiểm tra lại CSDL.                                       | **Critical** |              `TC-SEC-B04`              | `reports/post-audit/postman/newman-API-B-failure.html` |
| **`BUG-04`** | **API B** | Thêm sản phẩm với số lượng âm (`quantity = -1`); backend chấp nhận dẫn đến làm sai lệch tổng tiền giỏ hàng.                                                    |   **High**   |              `TC-DOM-B06`              | `reports/post-audit/postman/newman-API-B-failure.html` |
| **`BUG-05`** | **API B** | Thiếu hoàn toàn tầng validate dữ liệu: chấp nhận số lượng bằng 0, đơn giá âm, body rỗng`{}`, trường mang giá trị `null`.                                   |  **Medium**  | `TC-DOM-B05`, `B11`, `B15`, `B16` | `reports/post-audit/postman/newman-API-B-failure.html` |
| **`BUG-06`** | **API B** | Thêm sản phẩm không tồn tại (`id = 99999`); backend lưu sản phẩm ảo vào giỏ hàng mà không kiểm tra CSDL.                                                          |  **Medium**  |              `TC-BUS-B01`              | `reports/post-audit/postman/newman-API-B-failure.html` |
| **`BUG-07`** | **API A** | Lỗ hổng SQL Injection dạng UNION trên tham số`search` cho phép trích xuất bảng nhạy cảm `users` chứa email và mật khẩu (SEC-05).                                 | **Critical** |              `TC-SEC-A02`              | `reports/post-audit/postman/newman-API-A.html`         |
| **`BUG-08`** | **API A** | Trang lỗi CSDL phản hồi MIME type`text/html` với tiêu đề `<h1>Database Error</h1>` thay vì JSON error object chuẩn (SEC-04).                                           |   **Low**   |              `TC-SCH-A03`              | `reports/post-audit/postman/newman-API-A.html`         |

*Vấn đề kỹ thuật về kịch bản & môi trường (Artifact & Tooling Issues):*

- **Lỗi cú pháp test script:** Ca `TC-DOM-B18` và `TC-DOM-B19` trong API B gặp lỗi `SyntaxError` do nháy đơn lồng nhau trong lệnh `console.warn` của test script, không phải lỗi từ backend API.
- **Ô nhiễm trạng thái CSDL:** Việc thực thi API C (import dữ liệu bẩn `price: "free"`) trước API A làm sai lệch các assertion kiểm tra danh mục sạch ban đầu của API A (`TC-DOM-A09`, `TC-SCH-A01`, `TC-BUS-A01`).

---

## 3. AI-First Testing Workflow

Quy trình phát triển bộ kiểm thử tuân thủ chặt chẽ chu trình khép kín:

```text
AI Generation ──► Human Audit ──► Student Correction ──► Newman / CI Execution ──► Evidence Analysis ──► Bug Reporting
```

- **AI Generation:** Sinh bản thảo ca kiểm thử ban đầu, gợi ý phân hoạch tương đương, giá trị biên và schema JSON.
- **Human Audit:** Rà soát tính khả thi, phát hiện và loại bỏ các đề xuất sai lệch của AI (dán cứng token, nhân bản collection, làm yếu assertion).
- **Student Correction:** Sinh viên tự thiết kế 15 ca mở rộng, viết script tự động hóa gắn header `X-Student-Id`, cài đặt setup token động, triển khai cơ chế 2 nhánh chạy qua `strict_contract_assertion`.
- **Newman / CI Execution:** Thực thi thực tế qua Newman CLI và GitHub Actions runner (AI không trực tiếp thực hiện testing).
- **Evidence-Based Analysis:** Bóc tách dữ liệu từ các báo cáo HTML thực tế để xác nhận lỗi SUT và ghi nhận trung thực khoảng trống CI/CD.

*Chi tiết toàn bộ 13 tương tác kỹ thuật được lưu trữ tại [`docs/AI_Audit.md`](docs/AI_Audit.md) và sơ đồ mã giả tại [`docs/api-test-generator.md`](docs/api-test-generator.md).*

---

## 4. Hướng Dẫn Thực Thi (How to Run)

### 4.1 Khởi Động Máy Chủ Backend SUT

```bash
cd eshop/backend
npm install
node server.js
```

*Máy chủ lắng nghe tại: `http://localhost:3000`.*

### 4.2 Lệnh Chạy Newman CLI

Đứng tại thư mục gốc repository, thực thi các lệnh Newman chính xác:Phong

```bash
# API A (GET /api/products) — 39 Ca kiểm thử
newman run postman/collections/API_A_Products.postman_collection.json \
  --environment postman/environments/API_A_Environment.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/post-audit/postman/newman-API-A.html

# API B (POST /api/cart) — Chế độ All-Pass Baseline
newman run postman/collections/API_B_Cart.postman_collection.json \
  --environment postman/environments/API_B_Environment.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/post-audit/postman/newman-API-B-allpass.html

# API B (POST /api/cart) — Chế độ Bug Evidence (Bắt 19 lỗi SUT)
newman run postman/collections/API_B_Cart.postman_collection.json \
  --environment postman/environments/API_B_BugEvidence.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/post-audit/postman/newman-API-B-failure.html

# API C (POST /api/admin/import-products) — Chế độ All-Pass (100% Pass)
newman run postman/collections/API_C_ImportProducts.postman_collection.json \
  --environment postman/environments/API_C_Environment.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/post-audit/postman/newman-API-C-allpass.html

# API C (POST /api/admin/import-products) — Chế độ Bug Evidence (Bắt 2 lỗi SUT)
newman run postman/collections/API_C_ImportProducts.postman_collection.json \
  --environment postman/environments/API_C_BugEvidence.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/post-audit/postman/newman-API-C-failure.html
```

---

## 5. Tích Hợp CI/CD Trên GitHub Actions

- **Tệp cấu hình:** [`.github/workflows/api-testing.yml`](.github/workflows/api-testing.yml)
- **Quy trình pipeline:** Tự động kích hoạt khi `push` vào nhánh `main` hoặc kích hoạt thủ công qua `workflow_dispatch` với tham số `run_mode` (`all-pass` hoặc `bug-evidence`). Pipeline tự khởi động backend Node.js, sử dụng `wait-on` đợi cổng 3000 sẵn sàng, chạy tuần tự Newman cho 3 API và nén toàn bộ báo cáo HTML vào Artifacts `newman-html-reports`.
- **Phân định hiện trạng bằng chứng CI/CD theo REQ-49:**
  - **Lượt chạy CI có lỗi (Failure Run) — ĐÃ CÓ BẰNG CHỨNG:** Ghi nhận qua commit `0e4f867` (Run #4) tại [`assets/github_action_failed.png`](assets/github_action_failed.png) và artifact tải về [`reports/post-audit/github-action/newman-API-A.html`](reports/post-audit/github-action/newman-API-A.html) hiển thị pipeline thất bại với mã lỗi exit code 1.
  - **Lượt chạy CI All-Pass — KHOẢNG TRỐNG BẰNG CHỨNG:** Đã đạt kết quả All-Pass ở cấp độ Newman cục bộ ([`reports/post-audit/postman/newman-API-C-allpass.html`](reports/post-audit/postman/newman-API-C-allpass.html) với 0 failures), nhưng trên runner GitHub Actions chưa có lượt chạy thành công do pipeline ban đầu bị ngắt sớm. Khoảng trống này sẽ được hoàn thiện khi kích hoạt workflow mới trên GitHub.

---

## 6. Hồ Sơ Bằng Chứng & Sản Phẩm Bàn Giao (Evidence & Deliverables)

- **Ảnh chụp màn hình:**
  - [`assets/console_student_id.png`](assets/console_student_id.png): Minh chứng Postman Desktop Runner in vết log console chứa header `X-Student-Id: 23127452`.
  - [`assets/github_action_failed.png`](assets/github_action_failed.png): Minh chứng GitHub Actions Run #4 ghi nhận trạng thái Failure.
- **Tạo tác Postman thực thi:**
  - `postman/collections/`: 3 Collection JSON (`API_A_Products`, `API_B_Cart`, `API_C_ImportProducts`).
  - `postman/environments/`: 6 tệp Environment cho 3 API (All-Pass & BugEvidence).
  - `postman/data/API_C_import_sample.csv`: Tệp CSV mẫu phục vụ data-driven import.
- **Báo cáo thực thi Newman HTML:**
  - `reports/pre-audit/`: Báo cáo HTML trước kiểm toán.
  - `reports/post-audit/`: Báo cáo HTML sau kiểm toán trên Postman local và GitHub Actions runner.
- **Tài liệu báo cáo & Kiểm toán AI:**
  - [`docs/report.md`](docs/report.md): Báo cáo tổng kết đồ án toàn diện bao phủ 20 mục nội dung chuẩn.
  - [`docs/AI_Audit.md`](docs/AI_Audit.md) & [`docs/AI_Audit/`](docs/AI_Audit/): Hồ sơ kiểm toán AI 4 giai đoạn vòng đời với 13 tương tác được truy vết chi tiết.
  - [`docs/api-test-generator.md`](docs/api-test-generator.md): Sơ đồ quy trình Mermaid tự vẽ và mã giả điều phối vòng đời kiểm thử.
  - `temp/HW06/analysis-generated/08_final_report_review.md`: Ma trận đối chiếu 20 yêu cầu bài tập HW06.
  - `temp/HW06/analysis-generated/08_ai_audit_final.md`: Báo cáo kiểm toán tính trung thực và không ngụy tạo của báo cáo.

---

## 7. Bảng Tự Đánh Giá (Self-Assessment)

|       STT (No.)       | Tiêu Chí Đánh Giá (Criteria)                                             | Điểm Tối Đa (Max Grade) | Điểm Tự Đánh Giá (Self-Assessed Grade) |
| :-------------------: | :---------------------------------------------------------------------------- | :-------------------------: | :------------------------------------------: |
|      **1**      | **API 1 — Full pipeline** (generate + audit + extend + execute + bugs) |        **30**        |                 **30**                 |
|      **2**      | **API 2 — Full pipeline** (generate + audit + extend + execute + bugs) |        **30**        |                 **30**                 |
|      **3**      | **API 3 — Full pipeline** (generate + audit + extend + execute + bugs) |        **30**        |                 **30**                 |
|      **4**      | **Agent Skills — AI-driven test generator**                            |        **10**        |                 **10**                 |
| **TỔNG CỘNG** | **TOTAL**                                                               |        **100**        |                **100**                |

---

## 8. Agent Skill — AI-Driven API Test Generator

### 8.1. Mục Đích

Agent Skill được xây dựng để hỗ trợ tự động hóa quá trình tạo bộ kiểm thử API cho HW06.

Skill đọc API contract từ:

```text
eshop/api_specification.md
```

sau đó phân tích endpoint, request/response, validation rules và các yêu cầu kiểm thử để sinh ra các artifact phục vụ **Postman** và **Newman**.

Skill tập trung vào giai đoạn:

```text
API Specification
        ↓
API Analysis
        ↓
Test Scenario Generation
        ↓
Postman Collection Generation
        ↓
Newman Execution Setup
```

Skill **không thay thế Human Review** và không tự quyết định kết quả kiểm thử.

### 8.2. Input

**Input chính:**

```text
eshop/api_specification.md
```

API specification cung cấp contract để Skill xác định:

* Endpoint và HTTP method
* Parameters và request body
* Headers
* Authentication / Authorization
* Response structure
* Status codes
* Validation rules
* Business constraints

Skill có thể đọc thêm các file cần thiết để hiểu cấu trúc project, nhưng `api_specification.md` là nguồn contract chính.

### 8.3. Output

Skill sinh các artifact phục vụ thực thi API testing:

```text
postman/
├── collections/
├── environments/
└── data/

scripts/
```

Trong đó:

* **Postman Collections:** chứa request, variables, headers, test scripts và assertions.
* **Postman Environments:** chứa base URL, Student ID, token và các biến môi trường cần thiết.
* **Data:** chứa dữ liệu đầu vào cho các test data-driven khi cần.
* **Newman Scripts/Commands:** cung cấp cách chạy Collection bằng Newman và xuất HTML report.

Mọi request được sinh phải hỗ trợ header:

```http
X-Student-Id: {{StudentID}}
```

### 8.4. Test Coverage

Skill hỗ trợ sinh test scenario theo các nhóm:

* **Domain:** valid, invalid, boundary, missing, empty, wrong type.
* **State / Business:** state transition và business rule.
* **Security:** authentication, authorization, IDOR, privilege escalation, injection và unauthorized access khi phù hợp với API contract.
* **Schema:** status code, response structure, required fields, data types và content type.
* **Negative Testing:** các input không hợp lệ và hành vi lỗi.

Skill không tự tạo behavior không có cơ sở từ API specification.

### 8.5. Cấu Trúc Skill

```text
.agent/
└── skills/
    └── api-test-generation/
        ├── SKILL.md
        ├── knowledge/
        │   ├── api-analysis.md
        │   ├── postman-patterns.md
        │   ├── newman-execution.md
        │   └── test-types.md
        └── templates/
            ├── postman-collection-template.md
            └── execution-script-template.md
```

* `SKILL.md`: định nghĩa mục tiêu, workflow, input, output và cách Skill hoạt động.
* `knowledge/api-analysis.md`: hướng dẫn phân tích API specification.
* `knowledge/postman-patterns.md`: các pattern tạo Postman Collection và test scripts.
* `knowledge/newman-execution.md`: hướng dẫn chạy Newman và tạo HTML report.
* `knowledge/test-types.md`: các loại test cần xem xét khi sinh test case.
* `templates/postman-collection-template.md`: template cho Postman artifact.
* `templates/execution-script-template.md`: template cho Newman execution.

### 8.6. Cách Sử Dụng

Skill được sử dụng khi cần tạo hoặc cập nhật bộ API test từ API specification.

Workflow sử dụng:

```text
1. Đọc eshop/api_specification.md
              ↓
2. Phân tích API contract
              ↓
3. Xác định test scenarios
              ↓
4. Sinh Postman Collections / Environments
              ↓
5. Sinh Newman execution commands/scripts
              ↓
6. Human Review
              ↓
7. Execute bằng Postman / Newman
```

Ví dụ yêu cầu Skill:

```text
Read eshop/api_specification.md and generate the Postman
API test collections and Newman execution setup according
to the api-test-generation skill.
```

Skill chỉ **sinh và chuẩn bị artifact**. Việc Human Audit, Student Correction, Execution và Evidence Analysis được thực hiện ở các bước tiếp theo của quy trình HW06.