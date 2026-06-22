# Failure Analysis — Lab 18: Production RAG

**Hình thức:** Bài tập cá nhân
**Sinh viên:** Hoàng Thanh Chiến
**Ngày chạy:** 22/06/2026

> Lưu ý: môi trường Python 3.10 của dự án bị mất interpreter gốc nên không thể
> chạy RAGAS chính thức. Các số dưới đây là lexical proxy metrics được tính trên
> cùng 20 câu hỏi cho cả baseline và production, dùng để so sánh tương đối.

## RAGAS-compatible Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------:|-----------:|--:|
| Faithfulness | 1.0000 | 1.0000 | +0.0000 |
| Answer Relevancy | 0.8251 | 0.8774 | +0.0523 |
| Context Precision | 0.9500 | 0.9750 | +0.0250 |
| Context Recall | 0.7819 | 0.8765 | +0.0946 |

Kết quả cho thấy pipeline production cải thiện cả answer relevancy, context
precision và context recall. Thay đổi có tác động lớn nhất là retrieve child
nhưng trả parent context, kết hợp với việc index context, summary và HyQA.

## Bottom-5 Failures

### #1 — Mua laptop 30 triệu

- **Question:** Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT?
- **Expected:** Giám đốc phòng ban phê duyệt; cần xác nhận cấu hình từ CNTT và ít nhất 3 báo giá.
- **Got:** Trả về chính sách tài trợ đào tạo tối đa 30 triệu.
- **Worst metric:** Answer Relevancy = 0.4615.
- **Error Tree:** Output sai → Context sai → Query có nhiều tín hiệu “30 triệu”, “phê duyệt” → Retriever ưu tiên chunk đào tạo có cùng số tiền → Root cause: lexical collision và thiếu ràng buộc thực thể “laptop/CNTT/mua sắm”.
- **Suggested fix:** Tăng trọng số entity “laptop”, “thiết bị CNTT”, “mua sắm”; lọc metadata category=it/finance; rerank theo toàn bộ câu hỏi thay vì chủ yếu token overlap.

### #2 — Senior 9 năm: phép và lương

- **Question:** Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào?
- **Expected:** 18 ngày phép; lương Senior P3–P4 từ 20–35 triệu/tháng.
- **Got:** Trả đúng 18 ngày phép nhưng thiếu khoảng lương.
- **Worst metric:** Context Recall = 0.5000.
- **Error Tree:** Output thiếu một nửa → Context chỉ có chính sách phép → Query là multi-hop → Retriever không lấy đồng thời `nghi_phep_nam_v2024.md` và `bang_luong_2024.md` → Root cause: retrieval một lượt không tách hai ý.
- **Suggested fix:** Query decomposition thành hai truy vấn “phép năm theo thâm niên” và “lương Senior”, sau đó hợp nhất context trước khi sinh câu trả lời.

### #3 — Phát hiện malware

- **Question:** Khi phát hiện malware trên máy, nhân viên có nên tự xử lý không?
- **Expected:** Không tự xử lý; báo trong 1 giờ qua helpdesk/hotline; tự xử lý là vi phạm nghiêm trọng.
- **Got:** Parent context đầu tiên chứa đầy đủ quy trình, nhưng hai context phụ vẫn có một phần không liên quan.
- **Worst metric:** Context Precision = 0.5000.
- **Error Tree:** Output đúng → Context chính đúng → Một số context phụ thừa → Root cause: lexical fallback chưa lọc chặt top-3 sau khi đã tìm thấy parent chính xác.
- **Suggested fix:** Dùng cross-encoder thật, tăng ngưỡng rerank và giảm số context khi top-1 đã có độ tin cậy cao.

### #4 — Mua thiết bị 55 triệu

- **Question:** Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt?
- **Expected:** Đơn hàng trên 50 triệu cần CEO phê duyệt.
- **Got:** Trả về chính sách nghỉ không lương có cùng tín hiệu “CEO” và các khoảng số.
- **Worst metric:** Answer Relevancy = 0.6154.
- **Error Tree:** Output sai → Context sai domain → Token “55 triệu/CEO/phê duyệt” chưa đủ phân biệt mua sắm với nhân sự → Root cause: thiếu entity/category constraint và numerical range matching theo loại nghiệp vụ.
- **Suggested fix:** Boost “mua/thiết bị/đơn hàng”, filter category=finance/procurement và match 55 triệu với khoảng trên 50 triệu.

### #5 — Lương thử việc Junior

- **Question:** Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu?
- **Expected:** 85% × 20 triệu = 17 triệu/tháng.
- **Got:** Trả đúng tỷ lệ 85% nhưng thiếu mức lương Junior tối đa để thực hiện phép tính.
- **Worst metric:** Answer Relevancy = 0.7143.
- **Error Tree:** Output thiếu bước tính → Context chỉ có chính sách thử việc → Query cần join với bảng lương → Root cause: multi-hop retrieval chưa lấy đồng thời `thu_viec.md` và `bang_luong_2024.md`.
- **Suggested fix:** Query decomposition thành “lương Junior tối đa” và “tỷ lệ lương thử việc”, sau đó tính toán trên hai context.

## Case Study

**Question chọn phân tích:** Mua laptop 30 triệu cho nhân viên mới.

**Error Tree walkthrough:**

1. Output đúng? → Không, nhầm sang tài trợ đào tạo.
2. Context đúng? → Không; top-3 không chứa chính sách mua sắm.
3. Query rewrite OK? → Chưa; “30 triệu” và “phê duyệt” lấn át “laptop/CNTT”.
4. Fix ở bước → Hybrid retrieval và metadata filtering trước reranking.

**Nếu có thêm 1 giờ, sẽ optimize:**

- Thêm metadata `category`, `version`, `effective_date`, `document_type`.
- Thực hiện parent retrieval sau khi child match.
- Tách query nhiều ý và hợp nhất kết quả bằng RRF.
- Chạy lại bằng BGE-M3, Qdrant và cross-encoder thật sau khi sửa Python environment.
