# Link trace project — LangSmith

Bài lab yêu cầu gắn **một trong hai** backend tracing (Braintrust hoặc LangSmith).
Repo này dùng **LangSmith**. `eval/tracing.py` tự nhận backend theo biến môi trường,
nên không phải sửa code — chỉ cần key trong `.env`.

| | |
|---|---|
| Backend | LangSmith (`smith.langchain.com`) |
| Project | `ai-evaluation` |
| Project ID | `a7acf325-b296-456f-a188-d2a0517f84e8` |
| Workspace/Tenant ID | `c1e9fc53-7944-449e-b164-42d48d99dfcc` |
| Link | https://smith.langchain.com/o/c1e9fc53-7944-449e-b164-42d48d99dfcc/projects/p/a7acf325-b296-456f-a188-d2a0517f84e8 |

> Project ở chế độ riêng tư — người chấm cần được mời vào workspace mới xem được.
> Nếu cần nộp bản xem offline: mở project → chọn run → Export, rồi lưu file cạnh đây.

## Những gì được log

`eval/run_eval.py` log mỗi câu hỏi thành một trace tên `tutor-run`; `eval/judge.py`
log mỗi lượt chấm thành một trace. Mỗi trace gồm:

- **inputs** — câu hỏi, `slide` (bối cảnh), tên model
- **outputs** — JSON tutor trả về (`scope`, `answer`, `sources`, `followup_questions`)
- **metrics** — `prompt_tokens`, `completion_tokens`, `total_tokens`, `latency_s`, `cost_usd`
- **metadata** — `steps` (số vòng tool-calling), `scenario_id` để đối chiếu với `results-vN.jsonl`

## Cấu hình đã dùng

| | |
|---|---|
| Tutor | `openrouter/google/gemini-3.7-flash` |
| Judge | `openai/gpt-4o-mini` |

Tutor và judge **khác nhà cung cấp** — để judge không chấm chính output do cùng một
model sinh ra. Đây là lựa chọn có chủ đích, xem mục 5 (Calibration Report) trong
`deliverables/REPORT.md`.

## Cách kiểm chứng

`scenario_id` trong metadata của trace khớp với `scenario_id` trong
`evidence/results-vN.jsonl`, nên mỗi dòng data thô đều truy ngược được về một trace.
