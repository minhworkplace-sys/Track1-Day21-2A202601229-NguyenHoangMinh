# AI Support Log

> Ghi lại bạn đã dùng AI (ChatGPT/Claude/Kimi...) ở những bước nào khi làm deliverables.
> Trung thực là một phần của bài nộp — không ai làm một mình, quan trọng là bạn giữ
> quyền kiểm soát chất lượng.

| # | Bước | AI dùng để làm gì | Bạn kiểm chứng kết quả thế nào |
|---|------|-------------------|-------------------------------|
| 1 | Setup môi trường (trước Phase 1) | Claude Code dò repo xem thiếu gì, tạo `.env`, test 3 API key, vá retry 429 và lỗi UTF-8 trên Windows | Chạy thật: gọi API thấy response, xem trace lên LangSmith, chạy `tests/test_eval_kit.py` 44 pass |
| 2 | Phase 1 — lưới coverage + 29 câu ứng viên | Nhóm dùng AI dựng `phase1-coverage-grid.xlsx` (5 dimension, 13 combination, 2–3 paraphrase mỗi ô). Cả 3 sheet đều mang nhãn "AI DRAFT — HUMAN REVIEW REQUIRED" | Đối chiếu 29 câu với corpus thật bằng `tutor.load_corpus()` — phát hiện 11 câu nhắc thuật ngữ corpus có **0 lần** xuất hiện |
| 3 | Phase 1 — viết mục 1 REPORT.md | Claude Code chuyển lưới từ Excel sang REPORT.md và chạy kiểm tra độ phủ corpus | Số đếm chạy lại được bằng lệnh trong mục 1.4; file Excel gốc nằm trong `evidence/` để đối chiếu |
| 4 | Phase 2 — chạy tutor 2 vòng | Claude Code chẩn đoán 2/26 row `results-v1` vỡ JSON; xác định nguyên nhân là trần `max_tokens=2000` cắt ngang output chứ không phải model viết sai | Mở `raw_content` của `VLT-008`/`VLT-019` thấy cắt giữa chừng; nâng trần lên 3000, chạy lại `results-v2` sạch 26/26 |
| 5 | Phase 3 — ánh xạ id nhãn | Claude Code khớp nội dung 29 câu ứng viên (`S###`) với 26 row dataset v1 (`VLT-###`) để `judge.py` đối chiếu được nhãn | Kiểm bằng script: tập id trong `labels.csv` khớp 100% với `results.jsonl`; bảng ánh xạ từng dòng trong `evidence/label-id-mapping.md` để nhóm rà tay |
| 6 | Phase 3 — rubric R1–R11 | Claude Code rút ngược tiêu chí từ **26 note chấm tay của nhóm**, không tự nghĩ tiêu chí mới; mỗi tiêu chí phải gắn ít nhất một `scenario_id` thật | Rà từng tiêu chí xem có row thật làm ví dụ không; 2 chỗ nhóm chấm không nhất quán được đưa ra họp chốt (D1, D2) chứ AI không tự quyết |
| 7 | Phase 4 — judge prompt v2/v3 | Claude Code viết prompt theo rubric của nhóm, và sửa `eval/judge.py` nạp text nguồn thật vào prompt | Chạy thật 2 vòng, đối chiếu confusion matrix với nhãn vàng: 69% → 77%; đọc từng verdict để tìm nguyên nhân, không tin số tổng |
| 8 | Phase 6–7 — scorecard, verdict | Claude Code tổng hợp số liệu từ `results-v2`/`verdicts-v2`/`labels-golden-v1` và dựng bảng theo slice | Chạy script đối chiếu lại toàn bộ số trong REPORT với data thô trong `evidence/` — mọi con số tái lập được bằng lệnh |

### Phần nào AI gợi ý mà tôi bác bỏ

1. **Vòng rà 29 câu (Phase 1)** — bác 3 đề xuất, chi tiết ở mục dưới.
2. **Sửa nhãn `VLT-018` thành `fail`** (Phase 5) — AI lập luận bằng chứng nghiêng về judge.
   Tôi giữ nhãn `pass`: nhãn người là thước để đo judge, sửa ngược theo judge là mất thước.
3. **Để trống nhãn `VLT-017`, `VLT-023`** (Phase 3) — AI đề xuất bỏ trống vì hai câu này đã
   bị rewrite sau khi nhóm chấm. Nhóm giữ nhãn đã thống nhất trong `labels-group.csv`.

### Phần nào tôi hoàn toàn tự làm

- Viết 10/29 câu ứng viên (câu 10–19, các ô C05–C10 — nhóm ô rủi ro cao).
- Chấm tay độc lập toàn bộ scenario trước khi nhóm họp hợp nhất.
- Bốn quyết định D1–D4 (định nghĩa "nói rõ giả định", "nói rõ vế thiếu", quote verbatim
  không phải blocker, giữ nhãn `VLT-018`) — AI trình bày lựa chọn, người chốt.
- Quyết định gate theo slice và verdict SHIP WITH CONDITIONS.

## Ghi chú Phase 1 — phần nào là của AI, phần nào nhóm phải tự quyết

Lưới coverage do AI dựng, **nhóm chưa xác nhận** — chính file Excel tự ghi
"AI DRAFT — HUMAN REVIEW REQUIRED" ở cả 3 sheet, cột `Human decision` trống toàn bộ,
29/29 câu còn ở trạng thái `PENDING HUMAN REVIEW`. Slide s29 nói rõ:
*"Không giao cho LLM tự chọn test coverage — human kiểm soát coverage, LLM chỉ diễn
đạt lại."* Nên những mục dưới đây bắt buộc nhóm phải tự quyết, nếu bê nguyên bản nháp
đi nộp là vi phạm chính nguyên tắc của bài học:

- [x] **Sửa nhãn các câu lệch độ phủ corpus** (mục 1.4 + 2.4 REPORT.md) — đã xong:
      7 câu sửa nhãn nhưng giữ nguyên câu chữ, và 7/13 tổ hợp ở sheet `02_Combinations`
      được đồng bộ nhãn `D2` với `evidence/dataset-v1.jsonl`. Để nguyên nhãn cũ thì
      judge sẽ chấm tutor sai đúng lúc tutor làm đúng.
- [x] 5 dimension đã đúng chưa? Nhóm bỏ persona — **giữ quyết định đó**, ghi vào
      dòng cuối sheet `01_Input_Grid`: tutor không biết người hỏi là ai nên hành vi
      đúng đổi theo nội dung câu hỏi, không theo nhãn persona.
- [x] Điền cột `Human decision` (Keep / Rewrite / Reject) cho 29 câu, đổi `Status` khỏi PENDING.
- [x] 6/13 tổ hợp là high-risk (→ 12/26 câu) — **giữ**, lý do ở mục 2.2 REPORT.md:
      over-sample có chủ ý theo s30, kèm cảnh báo pass rate trên bộ này KHÔNG phải
      production success rate. Xem lại sau khi có trace thật ở P2.
- [ ] Ô "tần suất cao nhất" (C01) hiện là **giả định**, chưa có trace thật chống lưng.
      → Không đóng được ở Phase 1 vì sản phẩm chưa launch. Mở tiếp sang P2/P5:
      khi có traffic thật thì đối chiếu phân bố ô của dataset với phân bố ô của trace.

Phần nào nhóm **bác bỏ** và vì sao — ghi lại ngay đây, vì đó là bằng chứng nhóm
kiểm soát chất lượng chứ không nhận bừa output của AI:

>
> **Vòng rà 29 câu (Phase 1 → Dataset v1).** Bác 3 đề xuất của AI, mỗi chỗ đều vì
> kiểm chứng lại corpus chứ không vì cảm tính:
>
> 1. **q10** — AI đề xuất REWRITE với lý do corpus chỉ phủ một phần chủ đề *"tại sao
>    retrieval quan trọng trong RAG"*. Grep lại thấy `module-06-code-based-evaluation.md`
>    nói thẳng *"If the right documents weren't retrieved, the generation step never had
>    a chance"* + glossary có định nghĩa RAG → corpus phủ đủ, giữ nguyên câu (KEEP).
> 2. **q14** — AI khẳng định corpus *"chưa bao giờ so sánh RAG với fine-tuning"*. Sai:
>    `hamel-evals.md` có câu so sánh trực tiếp về cách mỗi kỹ thuật đưa kiến thức vào
>    hệ thống. Chỉ nhãn `D2` cần sửa, câu chữ giữ nguyên (KEEP).
> 3. **q04** — AI đề xuất KEEP vì "paraphrase giọng học viên thật". Nhóm bác: q04 trùng
>    **cả 5 trục** với q03 nên không thêm coverage, chỉ thêm chi phí chạy → REWRITE
>    thành câu ba ý để lấy thêm trục D3.
>
> Bài học rút ra: bảng đếm từ khoá AI đưa ra ở vòng đầu **không tái lập được** (ví dụ
> `offline eval` ghi 3 lần, đếm lại là 39). Số nào cũng phải chạy lại bằng script trước
> khi dùng làm căn cứ loại câu.


### Phase 3 — ánh xạ id nhãn người (`labels-group.csv` → `labels.csv`)

Nhóm chấm tay trên **29 câu ứng viên** (`S001–S029`), trong khi dataset v1 chỉ còn 26
row `VLT-001–VLT-026` sau khi chốt Keep/Rewrite/Reject. `eval/judge.py` đối chiếu nhãn
theo đúng `scenario_id`, id lệch thì confusion matrix ra rỗng → không có số calibration.

AI dựng bản ánh xạ bằng cách khớp **nội dung câu hỏi** giữa hai file (bảng đầy đủ:
`evidence/label-id-mapping.md`). Kết quả: 25 nhãn mang sang được, 4 nhãn bỏ (4 câu
REJECT), 1 row (`VLT-015` — câu mới thêm ở dataset v1) chưa có nhãn.

**Nhóm bác một đề xuất của AI ở bước này.** AI đề xuất để trống nhãn của `VLT-017` và
`VLT-023` vì nhóm chấm trên câu hỏi gốc, sau đó Phase 1 mới rewrite nội dung câu hỏi.
Nhóm đã họp và thống nhất **giữ nguyên nhãn trong `labels-group.csv`** cho hai row này —
`labels-group.csv` là nhãn vàng đã chốt. Dấu truy vết ghi trong cột note của `labels.csv`
để người đọc report tự đánh giá được.

### Phase 3 — Rubric v1 + Routing Map

AI **không tự nghĩ ra tiêu chí**: rubric R1–R11 được rút ngược từ 26 note chấm tay của
nhóm trong `evidence/labels-golden-v1.csv`, mỗi tiêu chí gắn ít nhất một row thật làm ví
dụ. Việc AI làm là đối chiếu chéo và phát hiện chỗ nhóm chấm **không nhất quán** — hai cặp
row cùng failure mode nhưng khác nhãn (`VLT-013`/`VLT-021`, `VLT-022`/`VLT-011`).

Nhóm chốt hai quyết định D1 ("fail nếu không nói rõ giả định") và D2 ("phải nói rõ vế
thiếu"), rồi đọc lại `results.jsonl` theo đúng tiêu chí đó → sửa 2 nhãn (`VLT-021`,
`VLT-022` từ `uncertain` lên `pass`). Nhãn vàng cuối: 19 pass · 4 uncertain · 3 fail.

Bài học: nhãn tay vòng đầu **chưa phải rubric**. Chỉ khi ép các note vào một bảng tiêu chí
mới lộ ra chỗ cùng lỗi mà chấm hai kiểu — và đó chính là chỗ judge sẽ học sai nếu bỏ qua.

### Phase 4 — Judge prompt v2 → v3

AI viết lại `judge_prompt.md` theo rubric của nhóm và sửa `eval/judge.py` để nạp **text
thật** của section tutor đã cite vào prompt (`{{cited_sections}}`) — bản gốc của kit chỉ
đưa danh sách `doc_id#section_id`, tức judge phải chấm groundedness mà không được đọc nguồn.

Vòng 1 (v2): **69%**. Đọc verdict thấy judge chấm fail cả 3 câu **từ chối chính đáng**
(`VLT-012`, `VLT-018`, `VLT-021`) với lý do "không có nguồn nào được trích dẫn". Sửa đúng
một thứ — thêm Bước 0 phân loại answer (A) có nội dung / (B) từ chối–hỏi lại → vòng 2 (v3):
**77%**.

**Nhóm dừng tune ở v3 dù còn 6 case bất đồng.** Lý do: 3 case là tiêu chí judge không được
giao (R6/R7/R10 — đúng như Routing Map dự tính), 2 case là khác biệt độ khắt khe trên đúng
2 row (siết tiếp là overfit judge vào nhãn người), và 1 case (`VLT-018`) hoá ra **judge
đúng, nhãn người sai** — tutor từ chối rồi vẫn đưa 3 khẳng định với `sources: []`.

AI đề xuất sửa nhãn `VLT-018` thành `fail` (agreement sẽ lên 91%). **Nhóm bác**: nhãn người
là thước để đo judge, lấy verdict judge chỉnh ngược nhãn vàng là mất luôn thước đo. Giữ 87%.

Bài học: agreement thô 77% trộn hai thứ khác hẳn nhau. Tách ra thì trong đúng làn judge là
20/23 = **87%**, còn phần chênh lại nằm ở tiêu chí mà nhóm **cố ý** không giao cho judge.
Con số tổng một mình không nói lên judge tốt hay dở.
