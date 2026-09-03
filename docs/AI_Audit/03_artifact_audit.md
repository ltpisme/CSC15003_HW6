# HW06 — Kiểm Toán Đề Xuất Tạo Tác Thực Thi Của AI (AI Executable Artifacts Audit)

> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/06_generation_decision.md`, `07_postman_audit.md`, `07_postman_ai_audit.md`  
> **Phạm vi:** Đánh giá các đề xuất cấu trúc tệp Collection, Environment, Script và CI/CD workflow.

---

## 1. Bảng Kiểm Toán Các Đề Xuất Tạo Tác Thực Thi Của AI

| STT | Đề Xuất Tạo Tác Của AI (AI Artifact Proposal) | Phân Loại | Lý Do Kỹ Thuật (Reason) | Hành Động Hiệu Chỉnh Của Sinh Viên (Student Fix) |
| :---: | :--- | :---: | :--- | :--- |
| **1** | Gộp toàn bộ 121 ca kiểm thử của cả 3 API vào một tệp Collection duy nhất (`HW06_All.postman_collection.json`). | **REJECTED / INVALID** | Vi phạm rubric HW06 yêu cầu đánh giá độc lập từng API và làm tăng độ phức tạp khi gỡ lỗi Newman từng phần. | Tách thành 3 tệp Collection riêng biệt tương ứng với 3 API: `API_A_Products`, `API_B_Cart`, `API_C_ImportProducts`. |
| **2** | Điền thủ công header `X-Student-Id: {{studentId}}` trên từng request riêng lẻ trong số 121 ca kiểm thử. | **REJECTED / INVALID** | Dễ sai sót bỏ quên request; không tạo được vết log console runtime để chụp ảnh minh chứng theo yêu cầu REQ-64. | Thiết lập Collection Pre-request Script (`pm.request.headers.upsert`) tự động gắn header và in console log. |
| **3** | Dán cứng (hardcode) chuỗi JWT token tĩnh vào tệp môi trường `postman_environment.json`. | **REJECTED / INVALID** | Token tĩnh có thời hạn (EXP claim) sẽ bị hết hạn theo thời gian, phá vỡ tính tự động của pipeline CI/CD. | Thiết lập thư mục `00_Setup_Prerequisites` trong Collection để gọi endpoint login và cấp phát dynamic token. |
| **4** | Nhân bản mỗi Collection thành 2 tệp JSON riêng biệt (bản AllPass và bản Failure). | **REJECTED / INVALID** | Gây trùng lặp mã nguồn nghiêm trọng, vi phạm nguyên tắc DRY và làm tăng chi phí bảo trì. | Sử dụng cờ môi trường `strict_contract_assertion` điều khiển nhánh assert trong cùng một collection duy nhất. |
| **5** | Đề xuất tệp CSV mẫu `postman/data/API_C_import_sample.csv` phục vụ kiểm thử data-driven cho FR-16. | **ACCEPTED / VALID** | Tệp CSV chứa 5 dòng sản phẩm với cấu trúc cột chuẩn xác giúp minh chứng năng lực kiểm thử theo lô dữ liệu của Postman. | Chấp nhận và duy trì tệp dữ liệu CSV trong thư mục `postman/data/`. |
