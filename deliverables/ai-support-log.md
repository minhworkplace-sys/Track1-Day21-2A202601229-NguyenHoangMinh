# AI Support Log

> Ghi lại bạn đã dùng AI (ChatGPT/Claude/Kimi...) ở những bước nào khi làm deliverables.
> Trung thực là một phần của bài nộp — không ai làm một mình, quan trọng là bạn giữ
> quyền kiểm soát chất lượng.

| # | Bước | AI dùng để làm gì | Bạn kiểm chứng kết quả thế nào |
|---|------|-------------------|-------------------------------|
| 1 | Setup môi trường (trước Phase 1) | Claude Code dò repo xem thiếu gì, tạo `.env`, test 3 API key, vá retry 429 và lỗi UTF-8 trên Windows | Chạy thật: gọi API thấy response, xem trace lên LangSmith, chạy `tests/test_eval_kit.py` 44 pass |
| 2 | Phase 1 — Input Grid (mục 1 REPORT.md) | Claude Code đọc slide s27–s30 rồi dựng bản nháp lưới: 4 dimension, lưới persona × intent, đánh dấu ô rủi ro cao/tần suất cao, phễu tổ hợp | **Chưa kiểm chứng — nhóm phải rà lại trong buổi họp.** Xem ghi chú bên dưới |
| 3 | | | |

- Phần nào AI gợi ý mà bạn **bác bỏ**? Vì sao?
- Phần nào bạn **hoàn toàn tự làm**?

## Ghi chú Phase 1 — phần nào là của AI, phần nào nhóm phải tự quyết

Bản nháp Input Grid do AI dựng, **nhóm chưa xác nhận**. Slide s29 nói rõ:
*"Không giao cho LLM tự chọn test coverage — human kiểm soát coverage, LLM chỉ diễn
đạt lại."* Nên những mục dưới đây bắt buộc nhóm phải tự quyết, nếu bê nguyên bản nháp
đi nộp là vi phạm chính nguyên tắc của bài học:

- [ ] 4 dimension chọn đã đúng với sản phẩm chưa, hay nhóm thấy trục khác quan trọng hơn?
- [ ] 12 ô đánh dấu ■ — giữ hết hay cắt bớt cho vừa thời lượng?
- [ ] 3 ô bị loại là phi lý thật, hay nhóm thấy vẫn đáng test?
- [ ] Ô "rủi ro cao nhất" (xin đáp án capstone) có đúng là rủi ro lớn nhất theo nhóm không?
- [ ] Ô "tần suất cao nhất" — hiện là **giả định**, chưa có trace thật để chống lưng.

Phần nào nhóm **bác bỏ** và vì sao — ghi lại ngay đây, vì đó là bằng chứng nhóm
kiểm soát chất lượng chứ không nhận bừa output của AI:

>
