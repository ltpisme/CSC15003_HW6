# HW06 — Kiểm Toán Thiết Kế Kiểm Thử (Phase 1: Scope & Test Design Audit)

> **Môn học:** Kiểm thử phần mềm (CSC15003) — Bài tập HW06  
> **Sinh viên thực hiện:** Lê Thanh Phong — MSSV: 23127452  
> **Công cụ trợ lý:** Antigravity — Gemini  
> **Tài liệu căn cứ:** `temp/HW06/01_scope.md`, `02_api_contract.md`, `03_test_blueprint.md`

---

## 1. Phân Tích Yêu Cầu HW06 & Lựa Chọn 3 API Mục Tiêu

<a id="log-1"></a>

- **Timestamp**: 14:15 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `ref/HW06.md`, `temp/Requirement/01_extract.md`, `eshop/api_specification.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Phân tích các ràng buộc kỹ thuật của HW06, rà soát danh sách endpoint trong SUT E-Shop và đề xuất lựa chọn 3 API mục tiêu độc lập đại diện cho 3 phân hệ (Public Catalog, User Cart, Admin Operation) để xây dựng bộ kiểm thử API toàn diện.

==Prompt End==

- **Output**

==Output Start==

Xác định 3 API mục tiêu thỏa mãn yêu cầu HW06:
1. API A — `GET /api/products`: Đại diện nhóm truy xuất dữ liệu công khai (Read-heavy / Search Catalog), kiểm tra query parameters, tìm kiếm từ khóa, phân trang, lọc danh mục.
2. API B — `POST /api/cart`: Đại diện nhóm thao tác người dùng (Transactional / Cart State), yêu cầu Bearer Token người dùng, kiểm tra tính toàn vẹn dữ liệu đơn giá, số lượng và cách ly phiên giỏ hàng.
3. API C — `POST /api/admin/import-products`: Đại diện nhóm quản trị hệ thống (Admin Operation / Batch Data), yêu cầu quyền Admin, xử lý giao dịch import mảng JSON/CSV và ghi nhận lỗi từng dòng.
Xuất tài liệu: `temp/HW06/01_scope.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Lựa chọn 3 API chính xác theo đặc tả của SUT, bao phủ đủ 3 mức phân quyền (Public, User, Admin) và 3 mô hình xử lý dữ liệu khác nhau.
- **Student fix**: Không cần thiết

---

## 2. Thẩm Định Hợp Đồng API & Bóc Tách Mã Nguồn Backend SUT

<a id="log-2"></a>

- **Timestamp**: 14:45 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `eshop/api_specification.md`, `eshop/backend/server.js`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Đối chiếu hợp đồng đặc tả trong `api_specification.md` với mã nguồn hiện thực thực tế trong `server.js` của 3 API đã chọn; bóc tách cấu trúc request body, headers, tham số và quy tắc nghiệp vụ để lập hồ sơ API Contract.

==Prompt End==

- **Output**

==Output Start==

Bóc tách chi tiết hợp đồng và chỉ ra sự khác biệt giữa đặc tả và mã nguồn thực tế:
- API A: Hợp đồng quy định trả về mảng sản phẩm kèm 6 thuộc tính; mã nguồn `server.js:144` nối chuỗi trực tiếp tham số `search` vào câu lệnh SQL (`LIKE '%${searchQuery}%'`), tạo ra lỗ hổng SQL Injection tiềm ẩn.
- API B: Hợp đồng yêu cầu body có `id, name, price, quantity` nguyên dương; mã nguồn `server.js:284-295` nhận trực tiếp body và lưu vào mảng RAM `userCarts` mà không kiểm tra tính hợp lệ của số lượng hay đơn giá.
- API C: Hợp đồng yêu cầu quyền Admin; mã nguồn `server.js:199` chỉ kiểm tra token hợp lệ qua `authenticateToken` mà thiếu middleware phân quyền `req.user.role === 'admin'`.
Xuất tài liệu: `temp/HW06/02_api_contract.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Phát hiện sớm các lỗ hổng và điểm yếu thực tế trong mã nguồn backend của SUT ngay từ khâu phân tích hợp đồng.
- **Student fix**: Không cần thiết

---

## 3. Thiết Kế Ma Trận Bao Phủ 9 Khía Cạnh Kiểm Thử (Test Blueprint)

<a id="log-3"></a>

- **Timestamp**: 15:10 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/01_scope.md`, `temp/HW06/02_api_contract.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Xây dựng bản đồ thiết kế kiểm thử (Test Blueprint) phân bổ chỉ tiêu số lượng ca kiểm thử cho từng API, đảm bảo bao phủ đầy đủ 9 khía cạnh kỹ thuật theo yêu cầu HW06: Phân hoạch tương đương, Giá trị biên, Kiểm thử bảo mật (SEC01–SEC07), Kiểm thử trạng thái, Quy tắc nghiệp vụ, JSON Schema, Phụ thuộc dữ liệu, Chuỗi gọi API và Ca mở rộng.

==Prompt End==

- **Output**

==Output Start==

Thiết lập ma trận phân bổ chỉ tiêu kiểm thử cho 3 API:
- Mỗi API được thiết kế tối thiểu 35 ca kiểm thử cơ sở do AI hỗ trợ + 5 ca kiểm thử mở rộng do sinh viên tự thiết kế ($\ge 40$ ca/API).
- Phân bổ chỉ tiêu chi tiết: Domain/Boundary (15–20 ca), Security (4–6 ca), State Transitions (2–4 ca), Schema Validation (3–4 ca), Business Rules (1–2 ca), Data Dependency (1–2 ca), Extended Behaviors (đúng 5 ca).
Xuất tài liệu: `temp/HW06/03_test_blueprint.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Khung thiết kế kiểm thử có cấu trúc khoa học, đáp ứng đầy đủ và vượt mức chỉ tiêu 35 ca/API của đề bài.
- **Student fix**: Không cần thiết
