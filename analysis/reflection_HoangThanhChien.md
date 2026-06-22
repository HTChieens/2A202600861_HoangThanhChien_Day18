# Individual Reflection — Lab 18

**Tên:** Hoàng Thanh Chiến
**Hình thức:** Bài tập cá nhân
**Module phụ trách:** M1, M2, M3, M4, M5

## 1. Mapping bài giảng vào code

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Threshold 0.85 tạo 208 chunks, trong khi basic tạo 51 chunks trên tập tài liệu hiện tại. |
| Parent-child chunking | M1 | `chunk_hierarchical()` | Tạo 109 child chunks; pipeline retrieve child và trả parent context để tránh mất thông tin. |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | RRF hợp nhất hai danh sách theo thứ hạng nên không phụ thuộc thang điểm BM25 và cosine. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Benchmark fallback trên test nhỏ khoảng 0.28 ms; production model được cấu hình là BGE reranker v2 M3. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Context recall tăng từ 0.7819 lên 0.8765; các failure còn lại chủ yếu là multi-hop và numerical matching. |
| Contextual embeddings | M5 | `_enrich_single_call()` | Context, summary và HyQA questions được prepend vào chunk trước khi index để giảm vocabulary gap. |

## 2. Khó khăn và cách giải quyết

- **Exact error:** `.venv\Scripts\python.exe` trỏ tới `C:\Users\84337\AppData\Local\Programs\Python\Python310\python.exe`, nhưng interpreter này không còn tồn tại.
- **Cách debug:** Kiểm tra `pyvenv.cfg`, Python launcher, imports và từng dependency; dùng runtime Python còn hoạt động để compile và chạy trực tiếp 37 test functions.
- **Giải pháp:** Bổ sung fallback thuần Python cho tokenization, BM25, local cosine retrieval, lexical reranking, enrichment và evaluation; vẫn giữ đầy đủ code path Qdrant, BGE và RAGAS chính thức.
- **Kiến thức còn thiếu:** Version-aware retrieval, numerical range matching và query decomposition. Tôi bổ sung bằng cách phân tích bottom failures và lập kế hoạch metadata/version + multi-query retrieval.

## 3. Action Plan cho project

### Project: Trợ lý tra cứu chính sách nội bộ doanh nghiệp

### Hiện tại

- Pipeline: Markdown/PDF → hierarchical chunking → enrichment → hybrid search → reranking → grounded answer → evaluation.
- Known issues: PDF scan chưa OCR; câu hỏi nhiều ý thiếu context; tài liệu cũ/mới có thể cùng được retrieve; số tiền và khoảng ngày dễ gây lexical collision.

### Plan áp dụng

1. [ ] **Chunking:** Hierarchical parent-child và structure-aware cho bảng/chính sách.
2. [ ] **Search:** BM25 + BGE-M3, hợp nhất bằng RRF.
3. [ ] **Reranking:** `BAAI/bge-reranker-v2-m3`, top-20 → top-3.
4. [ ] **Evaluation:** RAGAS + custom metrics cho version, số liệu và source citation.
5. [ ] **Enrichment:** Contextual prepend + summary + HyQA + metadata version/effective date.

### Timeline

- Tuần 1: OCR và chuẩn hóa metadata.
- Tuần 2: Triển khai Qdrant/BGE-M3 và benchmark retrieval.
- Tuần 3: Reranker, query decomposition và parent retrieval.
- Tuần 4: RAGAS chính thức, failure analysis và tối ưu.
