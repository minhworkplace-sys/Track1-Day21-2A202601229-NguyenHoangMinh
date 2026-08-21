# evidence/ — data thô của từng bước eval loop

Thư mục này chứa **data thô** minh chứng cho mọi quyết định trong các file
`deliverables/REPORT.md`. File làm việc sinh ra ở **root repo**
(`dataset.jsonl`, `results.jsonl`, `verdicts.jsonl`, `labels.csv`) — chốt một vòng
là copy vào đây ngay, đặt tên theo version, KHÔNG ghi đè vòng cũ.

Cần có đủ:

| File | Lấy từ đâu | Là gì |
|---|---|---|
| `dataset-v1.jsonl` | `dataset.jsonl` (root) | Dataset nhóm chốt — đầu vào mọi lần chạy |
| `results-v1.jsonl` (v2, v3...) | `results.jsonl` (root) | Output tutor thật: input, output JSON, `tool_calls`, tokens, cost từng câu |
| `labels.csv` | Export từ `report.html` | Nhãn người của các thành viên (vòng chấm độc lập) |
| `judge-prompt-v1.md` (v2...) | `eval/judge_prompt.md` | Prompt judge TỪNG VÒNG — copy trước mỗi lần sửa |
| `verdicts-v1.jsonl` (v2...) | `verdicts.jsonl` (root) | Output judge từng vòng calibration |
| `braintrust-link.md` | tự tạo | Link project Braintrust/LangSmith — trace mọi run |

Số liệu trong mục 5 (Calibration Report) của `deliverables/REPORT.md` phải đối chiếu được với các
file ở đây (confusion matrix, % agreement in ra từ `eval/judge.py`).

Nhớ: chạy xong một vòng là copy ngay — cuối buổi mới gom là mất dấu các vòng trước.

---

## Inventory thực tế của nhóm

| File | Là gì |
|---|---|
| `dataset-v1.jsonl` | 26 scenario nhóm chốt, có `dimension_values` / `expected_behavior` / `risk_if_fail` / `human_decision` từng row |
| `results-v1.jsonl` | Vòng chạy tutor thứ nhất — **2/26 row vỡ JSON** do trần `max_tokens=2000` cắt ngang. Giữ lại làm bằng chứng lỗi format |
| `results-v2.jsonl` | Vòng chạy sau khi nâng trần lên 3000 — 26/26 sạch. **Đây là bản dùng cho mọi số liệu trong REPORT** |
| `labels.csv` = `labels-golden-v1.csv` | Nhãn vàng cuối cùng, 26/26 row, sau khi chốt rubric R1–R11 và quyết định D1–D4 |
| `labels-round1-bach.csv` · `labels-round1-thinh.csv` | Nhãn **vòng chấm độc lập** (+ bản `-normalized` đã đưa về id dataset) |
| `labels-round2-group.csv` | Bản 3 người họp và thống nhất |
| `labels-README.md` | Ba hệ id khác nhau và cách chuẩn hoá — **đọc file này trước khi so nhãn** |
| `label-id-mapping.md` | Ánh xạ từng dòng `S###` → `VLT-###`, kèm lý do bỏ/giữ |
| `judge-prompt-v0-kit-original.md` | Prompt gốc của kit — chỉ chấm groundedness. **Không dùng để chấm vòng nào**, giữ để so |
| `judge-prompt-v1.md` | Prompt vòng 1 — tách R3/R4/R5/R8, nạp text nguồn thật → agreement 69% |
| `judge-prompt-v2.md` | Prompt vòng 2 — thêm Bước 0 phân loại answer từ chối → agreement 77% |
| `judge-prompt-v1-to-v2.diff` | Diff giữa hai vòng (+31/−3 dòng) |
| `verdicts-v1.jsonl` · `verdicts-v2.jsonl` | Output judge vòng 1 và vòng 2, có field `criteria` chấm riêng từng tiêu chí |
| `phase1-coverage-grid.xlsx` · `phase1-question-review.md` | Lưới coverage và quyết định Keep/Rewrite/Reject cho 29 câu ứng viên |
| `braintrust-link.md` | Link project LangSmith `ai-evaluation` |

**Lưu ý đánh số:** `judge-prompt-v1/v2` là hai vòng **thực sự dùng để chấm**; prompt gốc của
kit để riêng với tên `-v0-kit-original`. `verdicts-vN.jsonl` khớp số với `judge-prompt-vN.md`.
