# HW06 — Kiểm Toán Sinh Ca Kiểm Thử (Phase 2: AI Test Generation Audit)

> **Môn học:** Kiểm thử phần mềm (CSC15003) — Bài tập HW06  
> **Sinh viên thực hiện:** Lê Thanh Phong — MSSV: 23127452  
> **Công cụ trợ lý:** Antigravity — Gemini  
> **Tài liệu căn cứ:** `temp/HW06/03_test_blueprint.md`, `temp/HW06/generated/API_A.md`, `API_B.md`, `API_C.md`

---

## 4. Sinh Bản Thảo Ca Kiểm Thử Cho API A (`GET /api/products`)

<a id="log-4"></a>

- **Timestamp**: 15:30 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/03_test_blueprint.md`, `eshop/backend/server.js:141-157`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Sinh bản thảo bộ ca kiểm thử cho API A (`GET /api/products`) bao phủ đầy đủ các phân hoạch tương đương của tham số tìm kiếm `search`, phân trang, lọc danh mục, các kiểu tấn công SQL Injection và xác thực cấu trúc JSON Schema.

==Prompt End==

- **Output**

==Output Start==

Sinh 39 ca kiểm thử chi tiết phân chia thành 7 thư mục cho API A:
- 20 ca Phân hoạch tương đương và giá trị biên (`TC-DOM-A01` đến `A20`).
- 5 ca Bảo mật (`TC-SEC-A01` đến `A05`): Tautology SQLi (`search=test' OR '1'='1`), UNION SQLi (`search=' UNION SELECT id,email,password... FROM users--`), XSS Reflection, Security Headers.
- 2 ca Trạng thái (`TC-ST-A01`, `A02`): Kiểm chứng tính bất biến Safe Read.
- 3 ca JSON Schema (`TC-SCH-A01` đến `A03`).
- 4 ca Quy tắc nghiệp vụ và phụ thuộc dữ liệu (`TC-BUS-A01..A02`, `TC-DEP-A01..A02`).
- 5 ca Mở rộng do sinh viên thiết kế (`TC-NEW-A01` đến `A05`).
Xuất tài liệu: `temp/HW06/generated/API_A.md`.

==Output End==

- **Verdict**: Incomplete
- **Reasoning**: AI sinh bộ test case phong phú nhưng giả định lý thuyết về payload SQLi (`TC-SEC-A01` và `TC-NEW-A03`) bị sai lệch khi chạy trên CSDL SQLite thực tế: SQLite đánh giá `'1' = '1%'` là `false` khiến câu truy vấn trả về 0 dòng thay vì 5 dòng, dẫn đến assertion bị fail trên SUT.
- **Student fix**: Giữ nguyên vẹn 100% test case và kỳ vọng ban đầu theo quy tắc không làm yếu assertion; ghi nhận trung thực đây là phát hiện kiểm toán về giới hạn của payload injection khi đi qua hàm so khớp chuỗi `LIKE` của SUT.

---

## 5. Sinh Bản Thảo Ca Kiểm Thử Cho API B (`POST /api/cart`)

<a id="log-5"></a>

- **Timestamp**: 15:45 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/03_test_blueprint.md`, `eshop/backend/server.js:284-295`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Sinh bản thảo bộ ca kiểm thử cho API B (`POST /api/cart`) bao phủ các miền giá trị của `quantity` và `price`, thiếu trường, sai kiểu dữ liệu, các kịch bản xác thực Bearer token, cách ly giỏ hàng và chuyển trạng thái giỏ hàng.

==Prompt End==

- **Output**

==Output Start==

Sinh 41 ca kiểm thử phân chia thành 7 nhóm cho API B:
- 21 ca Phân hoạch dữ liệu và giá trị biên (`TC-DOM-B01` đến `B21`): Số lượng âm, bằng 0, số thực, chuỗi rỗng, tràn số 999999, đơn giá âm, giá 0, thiếu trường, body rỗng `{}`, trường null.
- 4 ca Bảo mật (`TC-SEC-B01` đến `B04`): Thiếu token (401), token sai chữ ký (403), cách ly phiên Anti-IDOR, thao túng đơn giá client-side (`price = 1000`).
- 4 ca Trạng thái (`TC-ST-B01` đến `B04`): Kiểm tra giỏ hàng rỗng ban đầu, tích lũy sản phẩm, xử lý sản phẩm trùng lặp, tiêm số lượng âm.
- 5 ca Schema & HTTP (`TC-SCH-B01..B03`, `TC-HTTP-B01..B02`).
- 2 ca Nghiệp vụ và phụ thuộc (`TC-BUS-B01`, `TC-DEP-B01`).
- 5 ca Mở rộng do sinh viên thiết kế (`TC-NEW-B01` đến `B05`).
Xuất tài liệu: `temp/HW06/generated/API_B.md`.

==Output End==

- **Verdict**: Incomplete
- **Reasoning**: AI đề xuất dùng chung một token người dùng cho toàn bộ suite và yêu cầu khởi động lại máy chủ thủ công để giỏ hàng rỗng cho `TC-ST-B01`. Điều này phá vỡ tính tự động hóa và không kiểm chứng được tính cách ly phiên Anti-IDOR.
- **Student fix**: Xây dựng kịch bản thiết lập tiền điều kiện tự động `00_Setup_Prerequisites` cấp phát người dùng sạch `clean_user_st_b01` cho kịch bản giỏ rỗng và người dùng thứ hai `user_b_sec_b03` cho ca cách ly Anti-IDOR `TC-SEC-B03`.

---

## 6. Sinh Bản Thảo Ca Kiểm Thử Cho API C (`POST /api/admin/import-products`)

<a id="log-6"></a>

- **Timestamp**: 16:00 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/03_test_blueprint.md`, `eshop/backend/server.js:199-242`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Sinh bản thảo bộ ca kiểm thử cho API C (`POST /api/admin/import-products`) bao phủ việc import mảng sản phẩm hợp lệ, import một phần, mảng rỗng, phần tử null, định dạng CSV, kiểm tra phân quyền Admin và tính phi nguyên tử của CSDL.

==Prompt End==

- **Output**

==Output Start==

Sinh 41 ca kiểm thử phân chia thành 7 nhóm cho API C:
- 21 ca Phân hoạch dữ liệu (`TC-DOM-C01` đến `C21`): Import đơn lẻ, import lô 50 sản phẩm, thành công một phần kèm mảng lỗi dòng, giá 0, giá âm, category không tồn tại, thiếu key `products`, mảng rỗng, phần tử null trong mảng (`products: [null]`).
- 5 ca Bảo mật (`TC-SEC-C01` đến `C05`): User thường gọi endpoint Admin (SEC-03), thiếu token, Parameterized Query chống SQLi, CSV Formula Injection (`=CMD|' /C calc'!A0`).
- 4 ca Trạng thái (`TC-ST-C01` đến `C04`): Tích lũy CSDL, tính phi nguyên tử (non-atomic import), hiển thị ngay trên catalog.
- 4 ca Schema (`TC-SCH-C01` đến `C04`): Kiểm tra 3 trường `message`, `inserted`, `errors`.
- 2 ca Nghiệp vụ và phụ thuộc (`TC-BUS-C01`, `TC-DEP-C01`).
- 5 ca Mở rộng do sinh viên thiết kế (`TC-NEW-C01` đến `C05`).
Xuất tài liệu: `temp/HW06/generated/API_C.md`.

==Output End==

- **Verdict**: Incomplete
- **Reasoning**: AI thiết kế assertion kiểm tra số lượng bản ghi CSDL bằng con số tuyệt đối cố định. Trên CSDL SQLite thực tế, việc chạy lại test lặp đi lặp lại sẽ làm số lượng bản ghi tăng liên tục, khiến assertion bị fail ngay ở lần chạy thứ hai.
- **Student fix**: Chuyển đổi toàn bộ assertion số lượng bản ghi CSDL sang kiểm tra độ tăng delta tương đối do chính request tạo ra (`res.inserted === 2` hoặc `res.inserted === 1`).
