# Individual Report — Lab 18: Production RAG

**Sinh viên:** Hoàng Thanh Chiến
**Hình thức:** Bài tập cá nhân
**Ngày:** 22/06/2026

## Module đã hoàn thành

| Sinh viên | Module | Hoàn thành | Tests pass |
|-----------|--------|------------|-----------:|
| Hoàng Thanh Chiến | M1: Advanced Chunking | ✓ | 13/13 |
| Hoàng Thanh Chiến | M2: Hybrid Search | ✓ | 5/5 |
| Hoàng Thanh Chiến | M3: Reranking | ✓ | 5/5 |
| Hoàng Thanh Chiến | M4: Evaluation | ✓ | 4/4 |
| Hoàng Thanh Chiến | M5: Enrichment | ✓ | 10/10 |
| **Tổng** | | **Hoàn thành** | **37/37** |

## Kết quả chạy pipeline

- Documents có text layer: 26.
- PDF scan bị bỏ qua: 2.
- Hierarchical child chunks: 109.
- Enriched chunks: 109.
- Evaluation questions: 20.
- Pipeline runtime: khoảng 0.4 giây với local fallback.

| Metric | Naive | Production | Δ |
|--------|------:|-----------:|--:|
| Faithfulness | 1.0000 | 1.0000 | +0.0000 |
| Answer Relevancy | 0.8251 | 0.8774 | +0.0523 |
| Context Precision | 0.9500 | 0.9750 | +0.0250 |
| Context Recall | 0.7819 | 0.8765 | +0.0946 |

Các metric là lexical proxy vì runtime hiện tại không có `datasets`, RAGAS,
Qdrant client, Sentence Transformers và model weights. Vì vậy chúng phù hợp để
so sánh hai cấu hình trong cùng môi trường, không thay thế điểm RAGAS chính thức.

## Key Findings

1. **Biggest win:** Pipeline chạy end-to-end dù thiếu dịch vụ ML nhờ fallback cho semantic chunking, dense retrieval, reranking, enrichment và evaluation.
2. **Biggest challenge:** Câu hỏi multi-hop và câu hỏi có số tiền/ngưỡng ngày vẫn dễ lexical collision.
3. **Surprise finding:** Parent retrieval cùng enrichment cải thiện context recall thêm 0.0946 so với baseline ngay cả trong local fallback.

## Kết luận

Kiến trúc production đã đầy đủ M1→M5 và 37/37 tests pass. Bước tiếp theo là sửa
Python environment, chạy Qdrant + BGE-M3 + BGE reranker thật, dùng OpenAI cho
generation/RAGAS, sau đó so sánh lại với cùng test set.
