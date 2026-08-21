# Ánh xạ nhãn người: `labels-group.csv` (S###) → `labels.csv` (VLT-###)

Nhóm chấm tay trên **29 câu ứng viên** (đánh số `S001–S029` theo thứ tự trong
`phase1-question-review.md`). Dataset v1 (`dataset-v1.jsonl`) là bản sau khi chốt
Keep/Rewrite/Reject: bỏ 4 câu REJECT, thêm 1 câu mới → 26 row `VLT-001–VLT-026`.
File này lock lại phép ánh xạ để `eval/judge.py` đối chiếu đúng row.

**Kết quả:** 26/26 row có nhãn · 4 nhãn bỏ (câu REJECT) · 1 row (`VLT-015`) nhóm chấm bổ sung.

| S### | VLT-### | Xử lý | Lý do | Nhãn cũ | Note cũ |
|---|---|---|---|---|---|
| S001 | VLT-001 | GIỮ |  | pass | Dẫn chứng đầy đủ |
| S002 | VLT-002 | GIỮ |  | uncertain | Giải thích vẫn quá rườm rà, dài dòng |
| S003 | VLT-003 | GIỮ |  | pass |  |
| S004 | VLT-004 | GIỮ |  | pass |  |
| S005 | VLT-005 | GIỮ |  | uncertain | Khái niệm thì đúng, nhưng các con số thì chưa chứng minh được rõ ràng |
| S006 | VLT-006 | GIỮ |  | pass | Nội dung đúng, citation khá sát |
| S007 | VLT-007 | GIỮ |  | pass | Trả lời đúng framework và có tính áp dụng |
| S008 | VLT-008 | GIỮ |  | pass | Rất sát câu hỏi, citation tốt nhất trong 3 câu |
| S009 | VLT-009 | GIỮ |  | uncertain | Trả lời sai yêu cầu của người hỏi. |
| S010 | VLT-010 | GIỮ |  | uncertain | Đúng bản chất RAG, một số claim rộng hơn citation |
| S011 | VLT-011 | GIỮ |  | pass | Không trả lời phần quan trọng nhất: RAG nào chính xác nhất trên GPT-5.6 |
| S012 | VLT-012 | GIỮ |  | pass | Từ chối đúng vì tài liệu không có căn cứ chọn database “tốt nhất” |
| S013 | VLT-013 | GIỮ |  | fail | Câu hỏi mơ hồ, câu trả lời tự suy diễn “cái này” là RAG |
| S014 | VLT-014 | GIỮ |  | pass | So sánh đúng, citation hỗ trợ trực tiếp |
| S015 | — | BỎ | REJECT (supervised/unsupervised) | pass | Out-of-scope và từ chối đúng |
| S016 | — | BỎ | REJECT (embedding vs vector db) | pass | Trả lời đủ cả 3 ý, citation khớp rất tốt |
| S017 | VLT-016 | GIỮ |  | pass | Sửa đúng hiểu lầm về RAG và fine-tuning |
| S018 | VLT-017 | GIỮ (nhóm chốt) | câu gốc chunk-size, dataset dùng bản rewrite 'pass rate 100%' | pass | Out-of-scope và không tự bịa quan hệ chunk size–accuracy |
| S019 | VLT-018 | GIỮ |  | pass | Nhiều nội dung cụ thể về chunking/vector DB không được citation hỗ trợ |
| S020 | VLT-019 | GIỮ |  | pass | Trả lời đúng pipeline RAG và citation gần như khớp từng ý |
| S021 | — | BỎ | REJECT (project 10.000 tài liệu) | pass | Out-of-scope và từ chối đúng |
| S022 | VLT-020 | GIỮ |  | pass | Phân biệt RAG/fine-tuning đúng, citation support ý cốt lõi |
| S023 | VLT-021 | GIỮ |  | uncertain | Câu hỏi quá mơ hồ; câu trả lời tự chọn cách hiểu “model” |
| S024 | VLT-022 | GIỮ |  | uncertain | Phần RAG vs fine-tuning đúng, nhưng không trả lời được phần “framework RAG mới phổ biến” |
| S025 | VLT-023 | GIỮ (nhóm chốt) | câu gốc supervised/unsupervised, dataset dùng bản rewrite 'code-based eval vs LLM-judge' | pass | Out-of-scope và xử lý đúng |
| S026 | VLT-024 | GIỮ |  | fail | Từ chối luôn cả phần embedding vs vector DB dù phần này đã xuất hiện trong corpus |
| S027 | VLT-025 | GIỮ |  | pass | Trả lời đúng hai ý, có điều kiện hóa hợp lý |
| S028 | VLT-026 | GIỮ |  | fail | Từ chối dù corpus thực tế đã có thông tin về embedding và retrieval |
| S029 | — | BỎ | REJECT (chunking → retrieval) | fail | Nội dung về chunking gần như không được citation hỗ trợ |
| — | VLT-015 | CHẤM LẠI | câu mới ở dataset v1, không có trong 29 câu ứng viên |  |  |

## Ghi chú về 3 row đặc biệt

- `VLT-015` — câu viết thêm ở dataset v1 (bù ô C08), không nằm trong 29 câu ứng viên nên
  `labels-group.csv` chưa có. Nhóm chấm bổ sung: **pass** — tutor bác đúng tiền đề trộn
  khái niệm và trả lời đúng thứ tự làm; `quote_verbatim` fail ở 1/3 quote (`s34`) được
  tính là điểm trừ, không phải blocker. Nhãn đã ghi ngược lại vào `labels-group.csv`.
- `VLT-017`, `VLT-023` — nhóm chấm trên **câu hỏi gốc**, sau đó Phase 1 rewrite nội dung
  câu hỏi. Nhóm đã họp và **thống nhất giữ nguyên nhãn trong `labels-group.csv`** cho hai
  row này; note gốc được giữ kèm dấu truy vết trong `labels.csv`.
