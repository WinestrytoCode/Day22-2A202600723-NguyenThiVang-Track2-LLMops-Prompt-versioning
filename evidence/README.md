# Evidence — Day 22: LangSmith + Prompt Versioning

Thư mục này chứa bằng chứng cho 4 nhiệm vụ của lab. Dưới đây là tóm tắt kết quả và phân tích.

## Danh sách bằng chứng

| Tệp | Nhiệm vụ | Nội dung |
|-----|----------|----------|
| `01_langsmith_traces.png` | 1 | Ảnh chụp LangSmith — ≥ 50 traces |
| `01_langsmith_console.txt` | 1 | Log console RAG pipeline (50 câu hỏi) |
| `02_prompt_hub.png` | 2 | Ảnh chụp Prompt Hub — 2 phiên bản prompt |
| `02_ab_routing_log.txt` | 2 | Log A/B routing (50 câu, nhãn v1/v2) |
| `03_ragas_scores.png` | 3 | Ảnh terminal bảng so sánh RAGAS V1 vs V2 |
| `03_ragas_report.json` | 3 | Báo cáo RAGAS (điểm V1 + V2) |
| `04_pii_demo_log.txt` | 4 | Log demo PII Detector |
| `04_json_demo_log.txt` | 4 | Log demo JSON Formatter |

---

## Kết quả RAGAS — Phân tích V1 vs V2

| Metric | V1 (ngắn gọn) | V2 (có cấu trúc) | Thắng |
|--------|--------------:|-----------------:|:-----:|
| **faithfulness**     | **0.9718** ⭐ | **0.8773** ⭐ | V1 |
| answer_relevancy     | 0.9126        | 0.8866           | V1 |
| context_recall       | 1.0000        | 1.0000           | hòa |
| context_precision    | 0.9369        | 0.9426           | V2 |

**Mục tiêu faithfulness ≥ 0.8 — ĐẠT ở cả hai phiên bản** (V1 = 0.9718, V2 = 0.8773).

### Vì sao V1 (prompt ngắn gọn) có faithfulness cao hơn?

- **V1** yêu cầu trả lời **ngắn gọn 2–4 câu** và **chỉ dùng context**. Câu trả lời ngắn → ít
  mệnh đề (claims) hơn → mỗi mệnh đề dễ được context hỗ trợ trực tiếp → **faithfulness cao
  hơn (0.9718)**. Tính trung thực với nguồn được tối ưu khi mô hình không "nói thêm".

- **V2** đóng vai chuyên gia, trả lời **có cấu trúc 3–5 câu** và giải thích lý do. Câu trả lời
  dài hơn, thêm diễn giải → xuất hiện những mệnh đề **suy luận/khái quát hóa** không nằm
  nguyên văn trong context → faithfulness giảm nhẹ (0.8773) dù vẫn đạt ngưỡng.

- **context_recall = 1.0** ở cả hai: retriever (FAISS, k=3) lấy đủ thông tin cần thiết cho mọi
  câu — chất lượng retrieval không phải là điểm khác biệt giữa hai prompt.

- **context_precision**: V2 nhỉnh hơn rất ít (0.9426 vs 0.9369) — không đáng kể, vì chỉ số này
  phụ thuộc thứ tự document do retriever trả về, gần như độc lập với prompt.

### Kết luận

Với bộ dữ liệu kiến thức ML/NLP/RAG này, **prompt ngắn gọn (V1) cho độ trung thực cao hơn**
mà vẫn giữ answer_relevancy tốt. Nếu ưu tiên độ tin cậy / ít "bịa", chọn V1. Nếu cần câu trả
lời đầy đủ, có giải thích cho người đọc và chấp nhận faithfulness thấp hơn một chút, chọn V2.

---

## Lưu ý kỹ thuật

- **Provider**: OpenAI `gpt-4o-mini` + embeddings `text-embedding-3-small`.
- **RAGAS RunConfig**: dùng `max_workers=4`, `timeout=300`, `max_retries=5` để tránh
  `TimeoutError` hàng loạt khi chạy ~400 lượt chấm metric (mặc định concurrency cao gây
  timeout → mọi điểm thành NaN).
- **Guardrails AI 0.10.x**: validator biến đổi đầu ra phải trả về
  `FailResult(fix_value=...)` kèm `OnFailAction.FIX` — `PassResult(value_override=...)`
  không được phản ánh vào `validated_output` ở phiên bản này.
</content>
