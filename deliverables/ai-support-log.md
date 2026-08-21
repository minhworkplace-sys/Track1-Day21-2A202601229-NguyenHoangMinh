# AI Support Log

> Ghi lại bạn đã dùng AI (ChatGPT/Claude/Kimi...) ở những bước nào khi làm deliverables.
> Trung thực là một phần của bài nộp — không ai làm một mình, quan trọng là bạn giữ
> quyền kiểm soát chất lượng.

| # | Bước | AI dùng để làm gì | Bạn kiểm chứng kết quả thế nào |
|---|------|-------------------|-------------------------------|
| 1 | Setup môi trường (trước Phase 1) | Claude Code dò repo xem thiếu gì, tạo `.env`, test 3 API key, vá retry 429 và lỗi UTF-8 trên Windows | Chạy thật: gọi API thấy response, xem trace lên LangSmith, chạy `tests/test_eval_kit.py` 44 pass |
| 2 | Phase 1 — lưới coverage + 29 câu ứng viên | Nhóm dùng AI dựng `phase1-coverage-grid.xlsx` (5 dimension, 13 combination, 2–3 paraphrase mỗi ô). Cả 3 sheet đều mang nhãn "AI DRAFT — HUMAN REVIEW REQUIRED" | Đối chiếu 29 câu với corpus thật bằng `tutor.load_corpus()` — phát hiện 11 câu nhắc thuật ngữ corpus có **0 lần** xuất hiện |
| 3 | Phase 1 — viết mục 1 REPORT.md | Claude Code chuyển lưới từ Excel sang REPORT.md và chạy kiểm tra độ phủ corpus | Số đếm chạy lại được bằng lệnh trong mục 1.4; file Excel gốc nằm trong `evidence/` để đối chiếu |
| 4 | | | |
| 3 | | | |

- Phần nào AI gợi ý mà bạn **bác bỏ**? Vì sao?
- Phần nào bạn **hoàn toàn tự làm**?

## Ghi chú Phase 1 — phần nào là của AI, phần nào nhóm phải tự quyết

Lưới coverage do AI dựng, **nhóm chưa xác nhận** — chính file Excel tự ghi
"AI DRAFT — HUMAN REVIEW REQUIRED" ở cả 3 sheet, cột `Human decision` trống toàn bộ,
29/29 câu còn ở trạng thái `PENDING HUMAN REVIEW`. Slide s29 nói rõ:
*"Không giao cho LLM tự chọn test coverage — human kiểm soát coverage, LLM chỉ diễn
đạt lại."* Nên những mục dưới đây bắt buộc nhóm phải tự quyết, nếu bê nguyên bản nháp
đi nộp là vi phạm chính nguyên tắc của bài học:

- [ ] **Sửa nhãn 11 câu lệch độ phủ corpus** (mục 1.4 REPORT.md) — việc gấp nhất.
      Để nguyên nhãn "Đủ thông tin" thì judge sẽ chấm tutor sai đúng lúc tutor làm đúng.
- [ ] 5 dimension đã đúng chưa? Nhóm bỏ persona — giữ quyết định đó không?
- [ ] Điền cột `Human decision` (Keep / Rewrite / Drop) cho 29 câu, đổi `Status` khỏi PENDING.
- [ ] 6/13 tổ hợp là high-risk — tỉ lệ này có quá nặng so với lưu lượng thật không?
- [ ] Ô "tần suất cao nhất" (C01) hiện là **giả định**, chưa có trace thật chống lưng.

Phần nào nhóm **bác bỏ** và vì sao — ghi lại ngay đây, vì đó là bằng chứng nhóm
kiểm soát chất lượng chứ không nhận bừa output của AI:

>
