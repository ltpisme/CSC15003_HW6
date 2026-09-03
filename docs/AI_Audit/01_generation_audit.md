# HW06 — Kiểm Toán Đề Xuất Sinh Kịch Bản Của AI (AI Generation Audit)

> **Tài liệu căn cứ:** `temp/HW06/03_test_blueprint.md`, `temp/HW06/generated/API_A.md` – `API_C.md`, `temp/HW06/analysis-generated/05_test_design_decision.md`  
> **Phạm vi:** Đánh giá các quyết định và bản thảo thiết kế ca kiểm thử do AI Agent sinh ra.

---

## 1. Bảng Kiểm Toán Các Đề Xuất Thiết Kế Ca Kiểm Thử

| STT | Đề Xuất Do AI Sinh Ra (AI Proposal) | Phân Loại | Lý Do Kỹ Thuật (Reason) | Hành Động Hiệu Chỉnh Của Sinh Viên (Student Fix) |
| :---: | :--- | :---: | :--- | :--- |
| **1** | Bỏ qua phân nhóm thư mục, đặt toàn bộ 35+ request vào một danh sách phẳng trong Postman. | **INVALID** | Khó bảo trì, không thể hiện được 9 tiêu chí thiết kế kiểm thử bắt buộc theo cấu trúc của HW06. | Tái cấu trúc thành 7 thư mục logic rõ ràng: `01_Domain_Partitioning`, `02_Security_Testing`, `03_State_Transitions`, `04_Schema_Validation`, `05_Business_Rules`, `06_Data_Dependency`, `07_Extended_Behaviors`. |
| **2** | Tự động sinh tối thiểu 35 ca kiểm thử cho mỗi API đáp ứng các tiêu chí biên, phân hoạch, bảo mật và schema. | **VALID** | Đảm bảo bao phủ toàn diện các luồng nghiệp vụ chính, trường hợp hợp lệ, trường hợp ngoại lệ và các kiểu tấn công phổ biến theo đặc tả. | `Không cần thiết` (Sinh viên chấp nhận và phát triển tiếp). |
| **3** | Đề xuất gộp chung kiểm thử quyền Admin của API C bằng tài khoản User thông thường và assert mã 200 OK. | **INVALID** | Vi phạm nghiêm trọng nguyên tắc kiểm thử bảo mật SEC-03. Endpoint Admin bắt buộc phải kiểm tra quyền quản trị và chặn User thường. | Tách riêng ca `TC-SEC-C01` (User thường gọi endpoint Admin) và assert nghiêm ngặt mã `403 Forbidden` để bắt lỗi bảo mật. |
| **4** | Đề xuất kiểm tra giỏ hàng API B bằng cách gọi tuần tự thêm sản phẩm mà không kiểm tra trạng thái khởi đầu rỗng. | **INCOMPLETE** | Không kiểm soát được trạng thái ban đầu của RAM `userCarts`, dễ gây sai lệch assertion kiểm tra độ dài giỏ hàng ban đầu. | Bổ sung ca `TC-ST-B01` kiểm tra chuyển trạng thái 3 bước từ giỏ hàng rỗng sạch hoàn toàn. |
| **5** | Đề xuất sinh 5 ca kiểm thử mở rộng (Student-authored Extension) tự động bằng AI. | **INVALID** | Đề bài yêu cầu ca kiểm thử mở rộng phải do chính sinh viên độc lập thiết kế và hiện thực dựa trên nghiên cứu chuyên sâu về SUT. | Sinh viên tự thiết kế độc lập 5 ca mở rộng chuyên biệt cho từng API (`TC-NEW-A01..A05`, `TC-NEW-B01..B05`, `TC-NEW-C01..C05`). |
