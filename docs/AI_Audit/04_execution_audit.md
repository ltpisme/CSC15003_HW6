# HW06 — Kiểm Toán Kết Luận Phân Tích Thực Thi Của AI (AI Execution Analysis Audit)

> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/07_execution_analysis.md`, `07_execution_ai_audit.md`, `reports/post-audit/`  
> **Phạm vi:** Đánh giá các kết luận phân tích kết quả chạy test và hiện trạng CI/CD do AI suy luận.

---

## 1. Bảng Kiểm Toán Các Kết Luận Phân Tích Thực Thi Của AI

| STT | Kết Luận Phân Tích Của AI (AI Execution Claim) | Phân Loại | Lý Do Kỹ Thuật (Reason) | Hành Động Hiệu Chỉnh Của Sinh Viên (Student Fix) |
| :---: | :--- | :---: | :--- | :--- |
| **1** | Tuyên bố tệp `newman-API-B-allpass.html` đạt 100% Pass dựa trên tên tệp. | **INVALID** | AI chỉ nhìn vào tên tệp mà không đọc dữ liệu lỗi bên trong. Thực tế báo cáo ghi nhận 2 ca bị fail do lỗi cú pháp JavaScript `SyntaxError` (`TC-DOM-B18` và `TC-DOM-B19`). | Đọc trực tiếp thẻ lỗi V8 runtime trong HTML; chỉ ra lỗi cú pháp dấu nháy đơn lồng nhau chưa escape trong lệnh `console.warn` của test script. |
| **2** | Nhận định 5 ca fail trong `newman-API-A-failure.html` đều là các lỗi nghiệp vụ nội tại của API A. | **INVALID** | AI phân tích rời rạc mà không nhận thức được sự phụ thuộc trạng thái CSDL. 3 trong số 5 lỗi (`TC-DOM-A09`, `TC-SCH-A01`, `TC-BUS-A01`) thực chất là do API C chạy trước đó chèn dữ liệu bẩn (`price: "free"`). | Phân loại chính xác 3 ca này thuộc nhóm lỗi ô nhiễm môi trường / CSDL (Environment State Accumulation Drift). |
| **3** | Khẳng định repository đã hoàn thành đầy đủ yêu cầu REQ-49 về 2 lượt chạy trên GitHub Actions (1 Pass, 1 Fail). | **INVALID** | AI nhầm lẫn giữa báo cáo chạy cục bộ All-Pass trên máy trạm Newman với lượt chạy trên GitHub Actions. Thực tế thư mục `post-audit/github-action/` chỉ có tệp `newman-API-A.html` từ lượt chạy fail (#4). | Xác định rõ đây là Khoảng trống Bằng chứng bắt buộc (Unresolved CI/CD Gap) của HW06 cần thực hiện kích hoạt pipeline mới. |
| **4** | Cho rằng ca `TC-SEC-A01` fail là do SUT an toàn trước tấn công SQLi Tautology. | **INVALID** | AI kết luận vội vã rằng không có dữ liệu trả về là an toàn. Mã nguồn `server.js:144` vẫn nối chuỗi trực tiếp; nguyên nhân không trả về bản ghi là do đặc thù biểu thức so khớp chuỗi `'1' = '1%'` trong SQLite trả về false. | Phân tích chi tiết cú pháp nội suy chuỗi SQL và khẳng định hệ thống vẫn tồn tại lỗ hổng SQLi nghiêm trọng (minh chứng qua `TC-SEC-A02`). |
| **5** | Nhận diện chính xác 19 ca fail trong `newman-API-B-failure.html` là các lỗi SUT thiếu validation đầu vào. | **VALID** | AI đối chiếu chính xác giữa hợp đồng đặc tả API B và mã nguồn backend nhận trực tiếp số lượng âm, giá âm, body rỗng. | `Không cần thiết` (Sinh viên xác nhận). |
| **6** | Đánh giá ca `TC-DOM-C21` sập server 500 là lỗi unhandled exception nghiêm trọng của SUT. | **VALID** | AI chỉ ra chuẩn xác dòng mã `server.js:214` đọc thuộc tính `.name` từ phần tử `null`, gây ngoại lệ `TypeError` làm sập server. | `Không cần thiết` (Sinh viên xác nhận). |
| **7** | Đánh giá ca `TC-SEC-C01` vi phạm nghiêm trọng yêu cầu bảo mật phân quyền SEC-03. | **VALID** | AI đối chiếu chuẩn xác đặc tả `api_specification.md:173` yêu cầu quyền Admin với router `server.js:199` cho phép User thường import dữ liệu. | `Không cần thiết` (Sinh viên xác nhận). |
| **8** | Đề xuất sửa đổi các assertion thành kỳ vọng mã 200/500 thực tế để báo cáo Newman "xanh" 100%. | **INVALID** | Vi phạm nghiêm trọng nguyên tắc liêm chính học thuật: cấm làm yếu assertion và cấm đổi Expected thành Actual để che giấu lỗi SUT. | Bác bỏ hoàn toàn đề xuất; kiên quyết bảo lưu nguyên vẹn các test case và assertion gốc. |
