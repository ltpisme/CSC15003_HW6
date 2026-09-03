# HW06 — Kiểm Toán Phân Tích Thực Thi & Hồ Sơ Nộp Bài (Phase 6: Execution Analysis & Final Deliverables Audit)

> **Môn học:** Kiểm thử phần mềm (CSC15003) — Bài tập HW06  
> **Sinh viên thực hiện:** Lê Thanh Phong — MSSV: 23127452  
> **Công cụ trợ lý:** Antigravity — Gemini  
> **Tài liệu căn cứ:** `temp/HW06/analysis-generated/07_execution_analysis.md`, `07_execution_ai_audit.md`, `reports/post-audit/`, `docs/`

---

## 12. Phân Tích Kết Quả Thực Thi Sau Kiểm Toán & Bằng Chứng CI/CD (Prompt 3)

<a id="log-12"></a>

- **Timestamp**: 19:37 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: `reports/post-audit/`, `reports/pre-audit/`, `assets/github_action_failed.png`
- **Prompt**:

==Prompt Start==

### PROMPT 3 — ANALYZE FINAL EXECUTION AND BUILD AI AUDIT EVIDENCE

Analyze the FINAL post-audit execution results for HW06.

Do NOT modify application code.
Do NOT modify Postman tests unless a factual inconsistency is discovered.
Do NOT create commits.
Do NOT rerun tests.

### Read P0
Requirements: `ref/HW06.md`, `temp/Requirement/01_extract.md` – `03_master.md`
API contract/design: `eshop/api_specification.md`, `temp/HW06/01_scope.md` – `03_test_blueprint.md`
Execution evidence: `reports/pre-audit/`, `reports/post-audit/postman/`, `reports/pre-audit/github-action/`, `reports/post-audit/github-action/`

### Analyze A/B/C independently
Compare: Requirement -> Contract -> Final test design -> Final Postman test -> Newman result -> GitHub Actions result.
Classify every meaningful failure as:
1. API/SUT Bug
2. Test/Assertion Issue
3. CI/CD or Environment Failure
4. Insufficient Evidence

### Bug evidence
For genuine API/SUT bugs, record: API, test case, expected behavior, actual behavior, evidence file, severity/impact.

### CI/CD requirement
Check whether the repository has genuine evidence for:
1. one CI run where all API tests pass
2. one CI run where at least one test fails
If one is missing, explicitly mark the HW06 gap.
A local All Pass report is NOT equivalent to a GitHub Actions All Pass run.

Create:
`temp/HW06/analysis-generated/07_execution_analysis.md`
`temp/HW06/analysis-generated/07_execution_ai_audit.md`

==Prompt End==

- **Output**

==Output Start==

Phân tích độc lập toàn bộ các tệp báo cáo Newman thực tế:
- Bóc tách chi tiết 7 báo cáo HTML của Newman trên máy trạm cục bộ và runner GitHub Actions.
- Phân loại chuẩn xác từng trường hợp thất bại:
  1. API/SUT Bug: Xác nhận 8 lỗi phần mềm SUT có thật (Lỗi phân quyền Admin Critical `TC-SEC-C01`, Sập server 500 `TC-DOM-C21`, Thao túng giá `TC-SEC-B04`, Nhận số lượng âm `TC-DOM-B06`, 15 ca vi phạm validation khác).
  2. Test/Assertion Issue: Nhận diện chính xác 2 ca fail trong `newman-API-B-allpass.html` (`TC-DOM-B18` và `TC-DOM-B19`) là do lỗi cú pháp JavaScript `SyntaxError` (dấu nháy đơn lồng nhau trong `console.warn`), không phải do lỗi của SUT.
  3. Environment Failure: 3 lỗi trong `newman-API-A-failure.html` (`TC-DOM-A09`, `TC-SCH-A01`, `TC-BUS-A01`) phát sinh do CSDL SQLite bị nhiễm dữ liệu bẩn sau khi chạy API C (`price: "free"`).
  4. CI/CD Gap: Xác nhận kho lưu trữ đã có bằng chứng cho lượt chạy Fail (Run #4 tại `assets/github_action_failed.png`), nhưng chưa có lượt chạy All-Pass trên GitHub Actions runner thực tế. Ghi nhận trung thực đây là Khoảng trống Bằng chứng (Unresolved CI/CD Gap).
Xuất tài liệu: `temp/HW06/analysis-generated/07_execution_analysis.md`, `07_execution_ai_audit.md`.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Phân tích khoa học, khách quan, bóc tách nguyên nhân sâu xa từ mã nguồn V8 và SQLite, và minh bạch ghi nhận khoảng trống bằng chứng CI/CD theo đúng quy định.
- **Student fix**: Không cần thiết

---

## 13. Hoàn Thiện Hồ Sơ Nộp Bài & Báo Cáo Tổng Kết (Prompt 4)

<a id="log-13"></a>

- **Timestamp**: 19:42 03/09/2026
- **Tools**: Antigravity - Gemini
- **Input**: Toàn bộ dữ liệu kiểm thử và phân tích trong workspace
- **Prompt**:

==Prompt Start==

### PROMPT 4 — COMPLETE HW06 FINAL DELIVERABLES

Complete the remaining HW06 documentation and submission artifacts using the existing evidence.

Do NOT create commits.
Do NOT invent missing evidence.
Do NOT rerun tests.
Do NOT modify application source code.
Do NOT redesign the API tests.

### 1. Final report
Update: `docs/report.md` (bao phủ 20 mục nội dung chuẩn)
For each API explicitly show: AI Generation -> Human Audit -> Student Correction -> Execution -> Analysis.

### 2. AI Audit
Complete: `docs/AI_Audit.md` and `docs/AI_Audit/`

### 3. README
Update: `docs/README.md` (đầy đủ các lệnh chạy thực tế)

### 4. Agent Skill evidence
Create: `docs/api-test-generator.md` (sơ đồ tự vẽ, mã giả, luồng dữ liệu)

### 5. Final evidence matrix
Create: `temp/HW06/analysis-generated/08_final_report_review.md` (DONE, PARTIAL, MISSING)

### 6. Final AI audit
Create: `temp/HW06/analysis-generated/08_ai_audit_final.md`

==Prompt End==

- **Output**

==Output Start==

Hoàn thiện toàn bộ các tạo tác tài liệu nộp bài:
1. `docs/report.md`: Báo cáo tổng kết đồ án 20 mục nội dung, bao gồm vòng đời hoàn chỉnh của 3 API, phân tích bảo mật SEC01–SEC07, JSON Schema, 8 lỗi SUT, đoạn phê bình AI Critique 284 từ và bảng tự đánh giá.
2. `docs/README.md`: Hướng dẫn toàn diện dự án với các câu lệnh Newman thực thi chính xác.
3. `docs/api-test-generator.md`: Sơ đồ quy trình Mermaid tự vẽ và mã giả điều phối toàn diện.
4. `temp/HW06/analysis-generated/08_final_report_review.md`: Ma trận đối chiếu 20 yêu cầu (19 DONE, 1 PARTIAL).
5. `temp/HW06/analysis-generated/08_ai_audit_final.md`: Báo cáo kiểm toán bảo đảm không có bằng chứng bịa đặt và khớp nối 100% với dữ liệu thực tế.

==Output End==

- **Verdict**: Valid
- **Reasoning**: Toàn bộ tài liệu được hoàn thiện đồng bộ, bám sát các số liệu thực tế từ báo cáo HTML và nhật ký Git, không ngụy tạo kết quả.
- **Student fix**: Không cần thiết
