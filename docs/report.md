# BÁO CÁO

> HW06 - API Testing
> Lê Thanh Phong - 23127452

---

## 1. Phạm Vi & Các API Lựa Chọn (Scope and Selected APIs)

Dự án kiểm thử HW06 tập trung vào 3 API đại diện cho 3 nhóm tính năng và quyền hạn khác nhau của ứng dụng E-Shop:

1. **API A - `GET /api/products` (Catalog & Product Search):**
   - *Phạm vi:* Truy xuất danh mục sản phẩm, tìm kiếm theo từ khóa `search`, lọc theo `category`, phân trang dữ liệu (`page`, `limit`).
   - *Quyền hạn:* Public (Không yêu cầu xác thực).
2. **API B - `POST /api/cart` (Shopping Cart Management):**
   - *Phạm vi:* Thao tác giỏ hàng người dùng, thêm sản phẩm, cập nhật số lượng, kiểm soát tính toán đơn giá và tính toàn vẹn phiên.
   - *Quyền hạn:* Authenticated User (Yêu cầu JWT Bearer Token).
3. **API C - `POST /api/admin/import-products` (Bulk Product Import):**
   - *Phạm vi:* Nhập hàng loạt sản phẩm từ cấu trúc mảng JSON hoặc tệp CSV, xử lý giao dịch CSDL, ghi nhận lỗi từng dòng.
   - *Quyền hạn:* Admin Only (Yêu cầu JWT Bearer Token có role `admin`).

---

## 2. Hợp Đồng Đặc Tả API (API Contracts)

Căn cứ vào tài liệu đặc tả chuẩn [`eshop/api_specification.md`](file:///home/ltp/Code/Course/Testing/HW/CSC15003_HW6/eshop/api_specification.md):

- **Hợp đồng API A:** Phản hồi HTTP 200 kèm JSON Array chứa các đối tượng có tối thiểu 6 thuộc tính: `id` (integer), `name` (string), `price` (number), `description` (string), `imageUrl` (string), `category_id` (integer). Nếu lỗi CSDL, phản hồi HTTP 500 kèm JSON error.
- **Hợp đồng API B:** Request body yêu cầu JSON Object chứa `id`, `name`, `price`, `quantity` (phải là số nguyên dương $\ge 1$). Phản hồi chuẩn là HTTP 200 `{"message": "Added to cart"}` hoặc HTTP 400 Bad Request nếu dữ liệu không hợp lệ; HTTP 401 nếu thiếu token; HTTP 403 nếu token không hợp lệ.
- **Hợp đồng API C:** Request body yêu cầu `{"products": [ ... ]}`. Phản hồi HTTP 200 `{"message": string, "inserted": number, "errors": array}`. Phải từ chối với HTTP 403 Forbidden nếu người gọi không phải tài khoản quản trị viên.

---

## 3. Quy Trình Sinh Kiểm Thử Khởi Đầu Bằng AI (AI-First Test Generation Process)

Quá trình sinh kiểm thử tự động ban đầu được thực hiện bằng cách nạp đặc tả API và 9 tiêu chí thiết kế kiểm thử vào AI Agent (sử dụng mô hình DeepSeek R1 và Claude 3.7 Sonnet).AI Agent phân tích mã nguồn backend `server.js` và sinh ra bản thảo kịch bản kiểm thử theo cấu trúc:

- Phân hoạch tương đương và giá trị biên (Domain & Boundary Partitioning).
- Các kịch bản tấn công bảo mật căn bản (SQL Injection, XSS, Bypass Auth).
- Xác thực định dạng phản hồi (JSON Schema Draft-07).

---

## 4. Quá Trình Đánh Giá & Kiểm Toán Của Con Người (Human Review)

Sinh viên tiến hành rà soát độc lập toàn bộ các đề xuất của AI để loại bỏ các điểm phi thực tế và vi phạm quy định bài tập:

- **Phát hiện 1 (Mất tính độc lập):** AI đề xuất gộp chung 121 ca kiểm thử vào 1 tệp Collection duy nhất $\rightarrow$ Sinh viên bác bỏ vì vi phạm rubric chấm điểm riêng cho từng API.
- **Phát hiện 2 (Bảo mật giả tạo):** AI đề xuất dán cứng chuỗi JWT Token tĩnh vào tệp Environment $\rightarrow$ Sinh viên bác bỏ vì token tĩnh sẽ hết hạn, làm gãy kịch bản tự động sau này.
- **Phát hiện 3 (Che giấu lỗi phần mềm):** Khi SUT chấp nhận số lượng âm trong giỏ hàng (trả về 200 OK thay vì 400), AI đề xuất sửa kỳ vọng thành 200 hoặc xóa test case để pipeline xanh $\rightarrow$ Sinh viên kiên quyết bác bỏ để bảo vệ tính trung thực của kiểm thử.

---

## 5. Các Hiệu Chỉnh Của Sinh Viên (Student Corrections)

Sinh viên trực tiếp thực hiện các hiệu chỉnh kỹ thuật:

1. **Tách và tái cấu trúc:** Chia thành 3 tệp Postman Collection độc lập tương ứng với 3 API trong thư mục `postman/collections/`.
2. **Cơ chế tiền điều kiện tự động (00_Setup_Prerequisites):** Xây dựng các request đăng nhập tự động để lấy token động cho User A, User B (cách ly giỏ hàng) và Admin.
3. **Chiến lược kiểm thử 2 nhánh (Dual-Run Strategy cho REQ-49):** Sử dụng cờ môi trường `strict_contract_assertion` duy nhất để vừa minh chứng được lỗi phần mềm (Bug Evidence Mode) vừa hỗ trợ chạy ổn định (All-Pass Baseline).
4. **Tự động gắn Header bắt buộc:** Cài đặt Collection Pre-request Script tự động chèn `X-Student-Id: 23127452` và in console log cho 100% request.

![console_student_id](https://hackmd.io/_uploads/SkT1nxPOMg.png)
![github_action_failed](https://hackmd.io/_uploads/HJBk2ewuzx.png)

---

## 6. Các Ca Kiểm Thử Mở Rộng Do Sinh Viên Tự Thiết Kế (Student-Authored Extensions $\ge 5$ / API)

Sinh viên độc lập nghiên cứu mã nguồn backend và thiết kế 15 ca kiểm thử mở rộng chuyên sâu (5 ca/API) tại thư mục `07_Extended_Behaviors`:

- **API A:**
  1. `TC-NEW-A01`: Giải mã dấu `+` thành khoảng trắng trong query string (`search=iPhone+15`).
  2. `TC-NEW-A02`: Tìm kiếm từ khóa chữ hoa/thường hỗn hợp ASCII (`search=iPhOnE`).
  3. `TC-NEW-A03`: Boolean-based SQL Injection vi phân (`search=iPhone' AND 1=1--`).
  4. `TC-NEW-A04`: Phòng thủ giải mã URL kép Double URL Encoding (`search=%2550ro`).
  5. `TC-NEW-A05`: Kiểm tra độ nhạy chữ hoa/thường Unicode tiếng Việt trong toán tử `LIKE` của SQLite.
- **API B:**
  1. `TC-NEW-B01`: Phòng thủ JSON hỏng cấu trúc (Malformed JSON Body).
  2. `TC-NEW-B02`: Gửi body dạng mảng JSON `[{...}]` thay vì object đơn.
  3. `TC-NEW-B03`: Từ chối Token đã hết hạn (`exp` trong quá khứ) với HTTP 403.
  4. `TC-NEW-B04`: Header thiếu tiền tố `Bearer` (`Authorization: eyJ...`) phản hồi 401.
  5. `TC-NEW-B05`: Lưu trữ an toàn thẻ `<script>` trong trường `name` giỏ hàng (chống Stored XSS).
- **API C:**
  1. `TC-NEW-C01`: Phòng thủ JSON body lỗi cú pháp đóng mở ngoặc.
  2. `TC-NEW-C02`: Xử lý trùng lặp tên sản phẩm nội bộ ngay trong cùng một lô import.
  3. `TC-NEW-C03`: Trường giá mang giá trị tường minh `null` (`price: null`).
  4. `TC-NEW-C04`: Giá trị `category_id` âm (`-1`) và hành vi ép kiểu logic.
  5. `TC-NEW-C05`: Từ chối Token Admin đã hết hạn với HTTP 403 Forbidden.

---

## 7. Thiết Kế Bộ Kiểm Thử Postman (Postman Test Design)

Bộ kiểm thử được phân nhóm khoa học thành 7 thư mục chuẩn hóa trên cả 3 Collection:

- `01_Domain_Partitioning`: Kiểm tra giá trị hợp lệ, không hợp lệ, biên trên/dưới.
- `02_Security_Testing`: Kiểm thử các lỗ hổng OWASP API Security (SEC01–SEC07).
- `03_State_Transitions`: Kiểm tra tính bất biến, tích lũy dữ liệu và chuyển trạng thái giỏ hàng/CSDL.
- `04_Schema_Validation`: Xác thực JSON Schema cấu trúc phản hồi.
- `05_Business_Rules`: Kiểm chứng quy tắc ràng buộc giá, danh mục và tính sẵn sàng của sản phẩm.
- `06_Data_Dependency`: Kiểm tra tính toàn vẹn khóa ngoại và phụ thuộc bảng.
- `07_Extended_Behaviors`: 5 ca mở rộng chuyên sâu của sinh viên.

---

## 8. Các Tính Năng Postman Thực Tế Được Sử Dụng (Postman Features Actually Used)

1. **Collection Pre-request Script:** Chèn tự động header `X-Student-Id` và ghi nhật ký console.
2. **Environment Variables:** Quản lý linh hoạt `baseUrl`, `studentId`, `userToken`, `cleanUserToken`, `userBToken`, `adminToken`, `strict_contract_assertion`.
3. **Dynamic Token Extraction & Request Chaining:** Đăng ký và đăng nhập người dùng tự động trong thư mục `00_Setup_Prerequisites` để trích xuất JWT Token lưu vào môi trường runtime.
4. **Chai Assertion Library:** Viết các điều kiện kiểm tra đa dạng (`to.have.status`, `to.be.an`, `to.include`, `to.be.at.least`).
5. **Data-Driven File Support:** Tệp CSV mẫu `postman/data/API_C_import_sample.csv` sẵn sàng cho tính năng Runner theo lô dữ liệu.

---

## 9. Thực Thi Kiểm Thử Bằng Newman (Newman Execution)

Bộ kiểm thử được thực thi tự động qua Newman CLI kết hợp plugin `newman-reporter-htmlextra` để xuất báo cáo HTML trực quan:

- Cấu hình theme tối (dark theme), ghi nhận chi tiết số lượng request, assertion, response body và console log.
- Toàn bộ kết quả chạy thực nghiệm được phân loại và lưu trữ tại `reports/pre-audit/` và `reports/post-audit/`.

---

## 10. Phân Tích Chi Tiết Kết Quả Thực Thi Từng API (API A/B/C Results)

### Vòng đời kỹ thuật hoàn chỉnh cho từng API:

```
[AI Generation] --> [Human Audit] --> [Student Correction] --> [Execution] --> [Analysis]
```

### 10.1 API A - `GET /api/products` (39 Test Cases)

- **AI Generation:** Sinh 39 ca kiểm thử bao phủ từ khóa hợp lệ, chuỗi dài, ký tự đặc biệt và SQL Injection.
- **Human Audit:** Phát hiện AI đề xuất chèn header thủ công từng request; phát hiện giả định SQLi Tautology trả về 5 dòng là có thể bị sai lệch trên SQLite.
- **Student Correction:** Viết Collection Pre-request Script tự động hóa gắn header; đồng bộ hóa môi trường `API_A_Environment` và `API_A_BugEvidence`.
- **Execution:** Thực thi 39 request, 79 assertions. Ghi nhận 2 ca thất bại: `TC-SEC-A01` và `TC-NEW-A03`.
- **Analysis:**
  - `TC-SEC-A01`: SUT ghép chuỗi thành `WHERE name LIKE '%test' OR '1'='1%'`. SQLite đánh giá `'1' = '1%'` là `false` nên trả về `[]` thay vì 5 sản phẩm.
  - `TC-NEW-A03`: Comment `--` cắt bỏ wildcard sau, làm câu lệnh thành `LIKE '%iPhone'`, đòi hỏi tên sản phẩm phải kết thúc bằng "iPhone" nên không khớp "iPhone 15 Pro Max".

### 10.2 API B - `POST /api/cart` (41 Test Cases + 5 Setup Requests)

- **AI Generation:** Sinh 41 ca kiểm thử giỏ hàng; phát hiện SUT nhận giá và số lượng âm.
- **Human Audit:** AI đề xuất restart server bằng tay để giỏ hàng rỗng cho ca `TC-ST-B01`; AI đề xuất nhân bản collection để làm xanh CI.
- **Student Correction:** Bác bỏ nhân bản collection; thiết lập setup tự động cấp phát `clean_user_st_b01` cho giỏ hàng rỗng và `user_b_sec_b03` cho Anti-IDOR; cài đặt cờ `strict_contract_assertion`.
- **Execution:** Chạy 46 request, 62 assertions. Chế độ All-Pass đạt 100% HTTP status pass; Chế độ Bug-Evidence ghi nhận chính xác 19 lỗi vi phạm hợp đồng.
- **Analysis:** Backend `server.js:284-295` thiếu hoàn toàn khâu validation dữ liệu đầu vào.

### 10.3 API C - `POST /api/admin/import-products` (41 Test Cases + 2 Setup Requests)

- **AI Generation:** Sinh 41 ca kiểm thử import lô, xử lý lỗi từng dòng và phân quyền.
- **Human Audit:** Phát hiện assert số lượng CSDL tuyệt đối sẽ bị gãy do SQLite tích lũy dữ liệu qua các lần chạy.
- **Student Correction:** Đổi sang assert kiểm tra độ tăng delta (`res.inserted === 2`); tạo tệp CSV mẫu; tích hợp dual-run cho ca bypass quyền Admin và crash server.
- **Execution:** Chạy 43 request, 83 assertions. Chế độ All-Pass đạt 100% PASS (0 failures); Chế độ Bug-Evidence phát hiện chính xác 2 lỗi nghiêm trọng (`TC-SEC-C01` và `TC-DOM-C21`).
- **Analysis:** Lỗi nghiêm trọng do thiếu middleware kiểm tra role admin tại router `server.js:199` và unhandled exception khi truy cập phần tử `null`.

---

## 11. Danh Sách Lỗi Phần Mềm Thực Tế Được Xác Nhận (Genuine SUT Bugs)

## Critical

### Bug C-01 - Broken Function Level Authorization

**Test Case:** `TC-SEC-C01`

Người dùng thường gọi được API import của Admin (*Critical*).

[GitHub Issue - C-01](https://github.com/ltpisme/CSC15003_HW6/issues/1)

---

### Bug B-01 - Client Price Manipulation

**Test Case:** `TC-SEC-B04`

Thao túng giá từ client (`price = 1000`) mà backend không xác thực lại từ CSDL (*Critical*).

[GitHub Issue - B-01](https://github.com/ltpisme/CSC15003_HW6/issues/3)

---

### Bug A-01 - SQL Injection UNION

**Test Case:** `TC-SEC-A02`

SQL Injection UNION trích xuất bảng nhạy cảm `users` (*Critical*).

[GitHub Issue - A-01](https://github.com/ltpisme/CSC15003_HW6/issues/7)

---

## High

### Bug C-02 - Unhandled Exception

**Test Case:** `TC-DOM-C21`

Unhandled Exception 500 khi mảng chứa phần tử `null` tại `POST /api/admin/import-products` (*High*).

[GitHub Issue - C-02](https://github.com/ltpisme/CSC15003_HW6/issues/2)

---

### Bug B-02 - Negative Quantity

**Test Case:** `TC-DOM-B06`

Nhận số lượng âm (`quantity = -1`) làm sai lệch tổng tiền giỏ hàng (*High*).

[GitHub Issue - B-02](https://github.com/ltpisme/CSC15003_HW6/issues/4)

---

## Medium

### Bug B-03 - Missing Backend Validation

**Test Cases:** `TC-DOM-B05`, `TC-DOM-B07`, `TC-DOM-B11`, `TC-DOM-B15`, `TC-DOM-B16`

Thiếu toàn bộ validation cơ bản phía backend (nhận số lượng 0, giá âm, body rỗng) (*Medium*).

[GitHub Issue - B-03](https://github.com/ltpisme/CSC15003_HW6/issues/5)

---

### Bug B-04 - Non-existent Product Accepted

**Test Case:** `TC-BUS-B01`

Thêm thành công sản phẩm không tồn tại (`id = 99999`) vào giỏ hàng (*Medium*).

[GitHub Issue - B-04](https://github.com/ltpisme/CSC15003_HW6/issues/6)

---

## Low

### Bug A-02 - Incorrect Error MIME Type

**Test Case:** `TC-SCH-A03`

Trang lỗi CSDL trả về MIME type `text/html` thay vì JSON error object (*Low*).

[GitHub Issue - A-02](https://github.com/ltpisme/CSC15003_HW6/issues/8)

---

# Evidence

Ảnh chụp toàn bộ danh sách GitHub Issues:

![github-issue](https://hackmd.io/_uploads/rydAjlvuzx.png)

## GitHub Issues

| Bug  | Severity | Test Case                                          | Issue                                                   |
| ---- | -------- | -------------------------------------------------- | ------------------------------------------------------- |
| C-01 | Critical | `TC-SEC-C01`                                     | [C-01](https://github.com/ltpisme/CSC15003_HW6/issues/1) |
| C-02 | High     | `TC-DOM-C21`                                     | [C-02](https://github.com/ltpisme/CSC15003_HW6/issues/2) |
| B-01 | Critical | `TC-SEC-B04`                                     | [B-01](https://github.com/ltpisme/CSC15003_HW6/issues/3) |
| B-02 | High     | `TC-DOM-B06`                                     | [B-02](https://github.com/ltpisme/CSC15003_HW6/issues/4) |
| B-03 | Medium   | `TC-DOM-B05`, `B07`, `B11`, `B15`, `B16` | [B-03](https://github.com/ltpisme/CSC15003_HW6/issues/5) |
| B-04 | Medium   | `TC-BUS-B01`                                     | [B-04](https://github.com/ltpisme/CSC15003_HW6/issues/6) |
| A-01 | Critical | `TC-SEC-A02`                                     | [A-01](https://github.com/ltpisme/CSC15003_HW6/issues/7) |
| A-02 | Low      | `TC-SCH-A03`                                     | [A-02](https://github.com/ltpisme/CSC15003_HW6/issues/8) |

---

## 12. Tích Hợp CI/CD Trên GitHub Actions (CI/CD Integration)

Workflow `.github/workflows/api-testing.yml` được cấu hình tự động:

- Khởi động backend và chờ endpoint sẵn sàng qua công cụ `wait-on`.
- Thực thi lần lượt Newman cho cả 3 API.
- Thu thập và lưu trữ báo cáo HTML vào Artifacts với thời hạn 14 ngày.
- Bổ sung `continue-on-error: true` để đảm bảo toàn bộ các suite đều được thực thi và sinh báo cáo ngay cả khi phát hiện lỗi SUT.

---

## 13. Minh Chứng Lượt Chạy CI/CD Thành Công & Thất Bại (CI Pass/Fail Evidence)

- **Lượt chạy Thất bại (Failure Run) - ĐÃ CÓ BẰNG CHỨNG:**
  - Commit `0e4f867` kích hoạt workflow GitHub Actions #4.
  - Minh chứng hình ảnh tại [`assets/github_action_failed.png`](file:///home/ltp/Code/Course/Testing/HW/CSC15003_HW6/assets/github_action_failed.png) thể hiện rõ trạng thái thất bại: `Process completed with exit code 1`.
  - Tệp HTML tải về từ artifact: [`reports/post-audit/github-action/newman-API-A.html`](file:///home/ltp/Code/Course/Testing/HW/CSC15003_HW6/reports/post-audit/github-action/newman-API-A.html).
- **Lượt chạy Thành công (All-Pass Run) - KHOẢNG TRỐNG BẰNG CHỨNG GHI NHẬN:**
  - Kết quả All-Pass hiện tại đã được chứng minh đầy đủ tại máy trạm cục bộ qua [`reports/post-audit/postman/newman-API-C-allpass.html`](file:///home/ltp/Code/Course/Testing/HW/CSC15003_HW6/reports/post-audit/postman/newman-API-C-allpass.html).
  - Tuy nhiên, trên GitHub Actions thực tế chưa có lượt chạy All-Pass tương ứng do pipeline ban đầu bị ngắt sớm. Khoảng trống này được ghi nhận trung thực theo quy định đề bài.

---

## 14. Kiểm Thử Bảo Mật (Security Testing - SEC01–SEC07)

Bộ kiểm thử bao phủ toàn diện 7 khía cạnh bảo mật:

- `SEC-01`: Kiểm tra các HTTP Security Headers (`TC-SEC-A05`).
- `SEC-02`: Bắt buộc xác thực Bearer token và từ chối token giả mạo, token hết hạn (`TC-SEC-B01`, `B02`, `TC-NEW-B03`).
- `SEC-03`: Kiểm tra phân quyền truy cập chức năng và cách ly giỏ hàng Anti-IDOR (`TC-SEC-B03`, `TC-SEC-C01`).
- `SEC-04`: Phòng thủ XSS và CSV Formula Injection (`TC-SEC-A04`, `TC-SEC-C04`).
- `SEC-05`: Kiểm thử Parameterized Query và khai thác SQL Injection (`TC-SEC-A01`, `A02`, `TC-SEC-C03`).
- `SEC-06`: Phòng thủ thao túng đơn giá phía client (`TC-SEC-B04`).
- `SEC-07`: Kiểm tra rò rỉ thông tin trong thông báo lỗi CSDL (`TC-SEC-A03`).

---

## 15. Xác Thực Cấu Trúc JSON Schema (Schema Validation)

Thực hiện kiểm chứng cấu trúc JSON Schema Draft-07 cho toàn bộ các phản hồi chuẩn và phản hồi lỗi:

- `TC-SCH-A01..A03`: Kiểm tra schema danh mục sản phẩm, mảng rỗng và MIME type.
- `TC-SCH-B01..B03`: Xác thực schema phản hồi thêm giỏ hàng, phản hồi 401 thiếu token và 403 sai token.
- `TC-SCH-C01..C04`: Xác thực schema phản hồi import 3 trường (`message`, `inserted`, `errors`), cấu trúc mảng lỗi lồng nhau và mã lỗi 400, 401.

---

## 16. Kiểm Thử Trạng Thái & Quy Tắc Nghiệp Vụ (State & Business Testing)

- **Kiểm thử chuyển trạng thái:** Kiểm chứng chu kỳ sống 3 bước của giỏ hàng từ trạng thái rỗng (`length === 0`) đến tích lũy sản phẩm (`TC-ST-B01`).
- **Tính bất biến (Safe Read):** Xác nhận `GET /api/products` không làm thay đổi trạng thái hệ thống (`TC-ST-A01`).
- **Tính phi nguyên tử (Non-atomic import):** Chứng minh tính năng import CSDL của API C ghi nhận một phần bản ghi hợp lệ ngay cả khi có bản ghi lỗi trong cùng một lô (`TC-ST-C02`).
- **Toàn vẹn số liệu:** Kiểm tra đơn giá là số không âm và quan hệ khóa ngoại danh mục (`TC-BUS-A01`, `TC-DEP-C01`).

---

## 17. Tóm Tắt Kết Quả Kiểm Toán AI (AI Audit Summary)

Hồ sơ kiểm toán AI tại [`docs/AI_Audit.md`](file:///home/ltp/Code/Course/Testing/HW/CSC15003_HW6/docs/AI_Audit.md) ghi nhận:

- Sinh viên đã kiểm toán 23 quyết định/suy luận chính của AI qua 4 giai đoạn vòng đời.
- Kết quả phân loại: **8 đề xuất VALID (34.8%), 15 đề xuất INVALID hoặc INCOMPLETE (65.2%)**.
- Toàn bộ các đề xuất sai lệch (như nhân bản collection, dán cứng token, xóa test case lỗi, suy luận sai lệch về CI/CD) đều bị sinh viên phát hiện và hiệu chỉnh triệt để.

---

## 18. Bài Đánh Giá Phê Bình Năng Lực AI

Mô hình AI thể hiện thế mạnh vượt trội trong việc phân tích cú pháp đặc tả API, tự động sinh khung kịch bản kiểm thử Postman với số lượng lớn (hơn 35 ca/API) và bao phủ nhanh chóng các phân hoạch tương đương, giá trị biên cơ bản. Việc gợi ý cấu trúc phân tầng JSON schema và các biến kiểm thử giúp giảm đáng kể thời gian thiết lập ban đầu cho người kiểm thử.

Tuy nhiên, AI bộc lộ những hạn chế nghiêm trọng về tư duy ngữ cảnh và đạo đức kiểm thử:

1. **Thiếu nhận thức về trạng thái thực thi thực tế:** AI có xu hướng dán cứng JWT token tĩnh vào tệp môi trường mà không nhận thức được chu kỳ hết hạn của token, hoặc giả định rằng cơ sở dữ liệu luôn ở trạng thái sạch ban đầu giữa các lần chạy liên tiếp.
2. **Xu hướng che giấu lỗi để đạt tỷ lệ Pass giả tạo:** Khi phát hiện SUT không kiểm tra dữ liệu đầu vào và trả về mã 200 OK thay vì 400 Bad Request, AI thường đề xuất sửa assertion theo hành vi sai trái của SUT hoặc xóa bỏ test case nhằm làm "xanh" kết quả chạy kiểm thử.
3. **Suy diễn sai lệch về hiện trạng CI/CD:** AI dễ nhầm lẫn giữa kết quả chạy Newman cục bộ trên máy cá nhân với kết quả thực tế trên runner của GitHub Actions, dẫn đến việc vội vã tuyên bố hoàn thành yêu cầu khi bằng chứng thực tế chưa đầy đủ.

Tóm lại, AI là công cụ hỗ trợ sinh mã thô xuất sắc, nhưng sự can thiệp, giám sát và hiệu chỉnh độc lập của con người (Human-in-the-loop) là yếu tố mang tính quyết định để bảo đảm tính liêm chính, độ tin cậy và giá trị thực tế của bộ kiểm thử.

---

## 19. Hạn Chế Kỹ Thuật & Khoảng Trống Bằng Chứng (Limitations & Evidence Gaps)

1. **Khoảng trống lượt chạy All-Pass trên GitHub Actions:** Đã có bằng chứng chạy Fail trên GitHub Actions (Run #4), nhưng chưa có bằng chứng chạy All-Pass thực tế trên GitHub Actions (mới có báo cáo All-Pass trên máy trạm Newman). Cần thực hiện kích hoạt pipeline mới sau khi nộp cấu hình workflow.
2. **Lỗi cú pháp JavaScript trong test script API B:** Ca `TC-DOM-B18` và `TC-DOM-B19` trong file `newman-API-B-allpass.html` ghi nhận lỗi cú pháp `SyntaxError` do nháy đơn lồng nhau trong lệnh `console.warn` của sinh viên, không phải lỗi từ backend API.
3. **Sự phụ thuộc dữ liệu chéo CSDL:** Việc chạy API C (import dữ liệu có `price: "free"`) trước API A sẽ làm nhiễm dữ liệu bẩn vào CSDL SQLite, gây ra lỗi assertion phụ thuộc trên API A.