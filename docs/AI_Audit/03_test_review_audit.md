# HW06 — Kiểm Toán Rà Soát Thiết Kế (Phase 3: Human Review & Revisions Audit)

> **Môn học:** Kiểm thử phần mềm (CSC15003) — Bài tập HW06  
> **Sinh viên thực hiện:** Lê Thanh Phong — MSSV: 23127452  
> **Công cụ trợ lý:** Antigravity — Gemini  
> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/01_overview.md` – `05_test_design_decision.md`

---

## 7. Rà Soát Đối Chiếu Chéo 3 Bộ Thiết Kế & Nhận Diện Khoảng Trống (Review & Gap Analysis)

<a id="log-7"></a>

- **Timestamp**: 16:20 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/generated/API_A.md`, `API_B.md`, `API_C.md`, `ref/HW06.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Tiến hành kiểm tra chéo (Cross-API Review) giữa 3 bộ thiết kế kiểm thử với các yêu cầu của HW06; rà soát tính khả thi khi chuyển đổi sang Postman Collection, kiểm tra việc tuân thủ các quy tắc chống gian lận (X-Student-Id), tính cô lập phiên người dùng và chiến lược xử lý khi SUT vi phạm hợp đồng đặc tả.

==Prompt End==

- **Output**

==Output Start==

Phát hiện 4 khoảng trống kỹ thuật quan trọng:
1. Thiếu cơ chế tự động hóa gắn header bắt buộc `X-Student-Id` kèm vết log console theo REQ-41 và REQ-64.
2. Nguy cơ gãy assertion khi SUT nhận dữ liệu sai phạm (API B nhận số lượng âm trả về 200 thay vì 400; API C cho phép User thường import dữ liệu của Admin trả về 200 thay vì 403).
3. Đề xuất của AI về việc nhân bản collection thành 2 bộ riêng biệt (AllPass và Failure) vi phạm nguyên tắc DRY và làm tăng gấp đôi số lượng tạo tác cần duy trì.
4. Trạng thái CSDL SQLite bị tích lũy dữ liệu qua các lần chạy test liên tiếp làm sai lệch các assertion kiểm tra số lượng tuyệt đối.
Xuất tài liệu: `temp/HW06/analysis-generated/01_overview.md` – `04_final_review.md`, `generation_gap.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Phân tích rà soát chỉ ra chính xác các rủi ro kỹ thuật cốt lõi trước khi bước vào giai đoạn sinh tạo tác Postman thực thi.
- **Student fix**: Không cần thiết

---

## 8. Khóa 8 Quyết Định Thiết Kế Kiểm Thử (Test Design Decisions 1–8)

<a id="log-8"></a>

- **Timestamp**: 16:40 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `temp/HW06/analysis-generated/04_final_review.md`, `temp/Requirement/03_master.md`
- **Prompt**:

==Prompt Start==

[Prompt không được lưu đầy đủ trong workspace]
Mục đích interaction: Tổng hợp các phát hiện từ khâu rà soát để ban hành 8 quyết định thiết kế kiểm thử chính thức (Test Design Decisions 1–8), chốt phương án xử lý cấu trúc collection, chiến lược assertion dual-run cho REQ-49, cơ chế gắn header chống gian lận và quy tắc bảo lưu lỗi phần mềm của SUT.

==Prompt End==

- **Output**

==Output Start==

Khóa 8 quyết định kỹ thuật cốt lõi:
- Quyết định 1: Giữ nguyên số lượng 121 ca kiểm thử chính thức ($39 + 41 + 41$) chia đều 3 API.
- Quyết định 2: Tách biệt hoàn toàn thành 3 tệp Collection độc lập, không gộp chung.
- Quyết định 3: Thiết lập Collection-level Pre-request Script tự động hóa 100% việc chèn header `X-Student-Id` và in console log.
- Quyết định 4: Bổ sung thư mục `00_Setup_Prerequisites` trong Collection API B và C để tự động đăng ký/đăng nhập sinh token động.
- Quyết định 5: Thiết lập tài khoản cô lập `clean_user_st_b01` cho kịch bản giỏ hàng rỗng và `user_b_sec_b03` cho kiểm thử Anti-IDOR.
- Quyết định 6: Triển khai cờ môi trường `strict_contract_assertion` duy nhất cho giải pháp kiểm thử 2 nhánh (All-Pass Baseline vs Bug Evidence) mà không nhân bản collection.
- Quyết định 7: Chuyển đổi toàn bộ assertion số lượng bản ghi CSDL sang kiểm tra delta tương đối.
- Quyết định 8: Nghiêm cấm sửa đổi mã nguồn SUT, cấm làm yếu assertion và cấm đổi Expected thành Actual.
Xuất tài liệu: `temp/HW06/analysis-generated/05_test_design_decision.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Toàn bộ 8 quyết định đều thể hiện tư duy kiểm thử chặt chẽ, bảo toàn tính liêm chính kỹ thuật và giải quyết triệt để các khoảng trống thiết kế.
- **Student fix**: Không cần thiết
