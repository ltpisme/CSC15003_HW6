# HW06 — Kiểm Toán Đánh Giá Rà Soát Của AI (AI Review Audit)

> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/01_overview.md` – `04_final_review.md`  
> **Phạm vi:** Đánh giá các khuyến nghị rà soát kiểm thử do AI Agent đưa ra trong quá trình đối chiếu thiết kế.

---

## 1. Bảng Kiểm Toán Các Khuyến Nghị Rà Soát Của AI

| STT | Khuyến Nghị Rà Soát Của AI (AI Review Finding) | Phân Loại | Lý Do Kỹ Thuật (Reason) | Hành Động Hiệu Chỉnh Của Sinh Viên (Student Fix) |
| :---: | :--- | :---: | :--- | :--- |
| **1** | Phát hiện thiếu ca kiểm thử HTTP Method không hỗ trợ (`POST /api/products` và `GET /api/cart` cho nghiệp vụ thêm). | **VALID** | Việc gửi method sai chuẩn REST là lỗi phổ biến của client cần được kiểm tra để đảm bảo SUT phản hồi 404 hoặc 405 thích hợp. | Bổ sung các ca kiểm thử Method Not Allowed vào bộ thiết kế. |
| **2** | Đề xuất xóa bỏ các ca kiểm thử phát hiện lỗi SUT (như số lượng âm trong giỏ hàng) để báo cáo CI không bị đánh dấu fail. | **INVALID** | Xóa bỏ test case phát hiện lỗi là hành vi ngụy tạo kết quả, đi ngược lại bản chất của công tác kiểm thử phần mềm và rubric HW06. | Giữ nguyên 100% các test case phát hiện lỗi; triển khai cơ chế kiểm thử 2 nhánh qua cờ môi trường `strict_contract_assertion`. |
| **3** | Khuyến nghị kiểm tra tính bất biến (Safe Read) của endpoint danh mục sản phẩm qua 2 lần gọi liên tiếp. | **VALID** | Đảm bảo phương thức HTTP GET là phương thức an toàn và bất biến (Idempotent / Safe Method), không làm thay đổi trạng thái CSDL. | Bổ sung và giữ nguyên ca kiểm thử `TC-ST-A01` trong suite API A. |
| **4** | Đề xuất dùng assertion số lượng bản ghi CSDL tuyệt đối cố định cho API C (như mong đợi CSDL luôn có đúng 7 dòng). | **INVALID** | CSDL SQLite tích lũy dữ liệu sau mỗi lần import lặp lại; assert số tuyệt đối sẽ bị gãy ngay trong lần chạy thứ hai. | Hiệu chỉnh assert kiểm tra độ tăng tương đối do request tạo ra (`res.inserted === 2`). |
| **5** | Đề xuất xác thực JSON schema phản hồi bằng thư viện AJV hoặc kiểm tra thuộc tính Chai. | **VALID** | Phương pháp kiểm tra cấu trúc đối tượng, sự tồn tại của trường và kiểu dữ liệu đảm bảo tuân thủ chặt chẽ đặc tả API Contract. | Triển khai thống nhất cho toàn bộ các thư mục `04_Schema_Validation` của 3 API. |
