# Individual Reflection — Lab 18

**Tên:** Hoàng Thanh Chiến
**Hình thức:** Bài tập cá nhân
**Module phụ trách:** M1, M2, M3, M4, M5

## 1. Đóng góp kỹ thuật

- Implement ba chiến lược chunking: semantic, hierarchical và structure-aware.
- Xây dựng BM25, dense retrieval, RRF và local cosine fallback.
- Implement cross-encoder reranking, score normalization và lexical fallback.
- Implement RAGAS evaluation, local proxy metrics và Diagnostic Tree.
- Implement summarization, HyQA, contextual prepend, metadata extraction và combined one-call enrichment.
- Bổ sung xử lý empty input, metadata isolation, validation, offline fallback và compatibility giữa các phiên bản thư viện.
- Kết quả kiểm thử: **37/37 tests pass**.

## 2. Mapping bài giảng vào code

| Concept | Code áp dụng | Ý nghĩa |
|---------|--------------|---------|
| Semantic chunking | `chunk_semantic()` | Nhóm câu theo độ tương đồng, tránh cắt giữa ý. |
| Parent-child retrieval | `chunk_hierarchical()` | Retrieve child để tăng precision, dùng parent để bổ sung context. |
| Structure-aware chunking | `chunk_structure_aware()` | Giữ header, bảng, list và code block theo cấu trúc Markdown. |
| Sparse retrieval | `BM25Search` | Tìm kiếm tốt với từ khóa, số tiền, tên chính sách. |
| Dense retrieval | `DenseSearch` | Tìm theo ngữ nghĩa; có local cosine fallback khi model unavailable. |
| Rank fusion | `reciprocal_rank_fusion()` | Hợp nhất BM25 và dense mà không cần đồng nhất thang điểm. |
| Cross-encoder reranking | `CrossEncoderReranker` | Chấm lại cặp query-document để chọn top context chính xác hơn. |
| RAG evaluation | `evaluate_ragas()` | Đo faithfulness, relevancy, precision và recall. |
| Diagnostic Tree | `failure_analysis()` | Chuyển metric thấp thành root cause và action cụ thể. |
| Contextual enrichment | `_enrich_single_call()` | Thêm summary, câu hỏi giả thuyết, context và metadata trước embedding. |

## 3. Kiến thức học được

- **Khái niệm quan trọng nhất:** Retrieval quality không chỉ phụ thuộc embedding; chunk boundary, metadata, versioning và query decomposition ảnh hưởng trực tiếp đến context recall.
- **Điều bất ngờ nhất:** Sau khi retrieve child nhưng trả parent, context recall tăng từ 0.7819 lên 0.8765. Điều này cho thấy cùng một retriever nhưng cách trả context có thể thay đổi kết quả đáng kể.
- **Kết nối với bài giảng:** Lab thể hiện đầy đủ chuỗi Chunking → Enrichment → Hybrid Search → Reranking → Generation → Evaluation.

## 4. Khó khăn và cách giải quyết

- **Lỗi chính xác:** `.venv\Scripts\python.exe` tham chiếu tới `C:\Users\84337\AppData\Local\Programs\Python\Python310\python.exe`, nhưng interpreter này không còn tồn tại.
- **Ảnh hưởng:** Không thể chạy pytest, Sentence Transformers, Qdrant client và RAGAS từ virtualenv.
- **Cách giải quyết:** Dùng Python runtime khả dụng để kiểm tra syntax; bổ sung fallback thuần Python cho tokenization, BM25, dense cosine, reranking, enrichment và evaluation; chạy trực tiếp 37 test functions.
- **Khó khăn retrieval:** Các câu hỏi multi-hop như phép năm + lương chỉ retrieve được một nguồn.
- **Cách giải quyết đề xuất:** Query decomposition, parent retrieval, metadata versioning và cross-encoder thật.
- **Thời gian debug:** Khoảng 60–90 phút cho việc xác định lỗi runtime, hoàn thiện fallback và chạy end-to-end.

## 5. Nếu làm lại

- Thiết lập và kiểm tra Python/Qdrant/model cache trước khi code.
- Viết integration test cho từng tầng, đặc biệt retrieval theo version và numerical range.
- Lưu toàn bộ per-question results vào report để phân tích dễ hơn.
- Chạy ablation test: baseline → +chunking → +hybrid → +rerank → +enrichment.
- Module muốn thử tiếp: query rewriting/decomposition và parent-document retrieval.

## 6. Action Plan cho project

### Project: Trợ lý tra cứu chính sách nội bộ doanh nghiệp

### Hiện tại

- RAG pipeline hiện tại: tài liệu Markdown/PDF → hierarchical chunking → contextual enrichment → BM25 + dense retrieval → RRF → cross-encoder reranking → grounded answer → RAGAS.
- Known issues: chưa OCR được PDF scan; truy vấn multi-hop đôi khi chỉ lấy một nguồn; chính sách cũ và mới chưa được lọc chắc chắn theo version; numerical range như “20 ngày” hoặc “30 triệu” dễ lexical collision.

### Plan áp dụng

1. [ ] **Chunking:** Dùng hierarchical parent-child; retrieve child để tìm chính xác nhưng trả parent để đủ context. Structure-aware chunking được dùng cho bảng và chính sách Markdown.
2. [ ] **Search:** Dùng hybrid BM25 + BGE-M3 vì BM25 mạnh với số tiền/tên chính sách, còn dense search xử lý paraphrase. Hợp nhất bằng RRF.
3. [ ] **Reranking:** Dùng `BAAI/bge-reranker-v2-m3` cho top-20 → top-3; thêm query decomposition với câu hỏi nhiều ý.
4. [ ] **Evaluation:** Dùng RAGAS cho bốn metrics và thêm custom checks cho version correctness, numerical correctness và citation/source accuracy.
5. [ ] **Enrichment:** Dùng combined mode gồm contextual prepend, summary, HyQA questions và metadata `category`, `version`, `effective_date`.

### Timeline

| Thời gian | Hành động |
|-----------|-----------|
| Tuần 1 | Chuẩn hóa metadata, version/effective date và OCR tài liệu scan. |
| Tuần 2 | Triển khai Qdrant + BGE-M3, benchmark BM25/dense/hybrid. |
| Tuần 3 | Thêm BGE reranker, query decomposition và parent retrieval. |
| Tuần 4 | Xây evaluation set nghiệp vụ, chạy RAGAS và failure analysis định kỳ. |

## 7. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------:|
| Hiểu bài giảng | 4 |
| Code quality | 4 |
| Tính chủ động | 5 |
| Problem solving | 5 |

Tôi hoàn thành toàn bộ năm module và pipeline cá nhân. Điểm cần cải thiện lớn
nhất là chạy lại với đúng production dependencies để xác nhận kết quả bằng
embedding, reranker và RAGAS chính thức.
