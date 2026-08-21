# REPORT — Eval loop A→Z: VLearn AI Tutor

Report A→Z của eval loop — mỗi mục ứng một phase của bài lab. Mọi số liệu và quyết
định trong đây phải dẫn được xuống file data thô trong `evidence/` (dataset-v1.jsonl,
results-vN.jsonl, labels.csv, judge-prompt-vN.md, verdicts-vN.jsonl, braintrust-link.md).


---

## 1. Input Grid

> Lưới input = trục "ai hỏi" × "hỏi kiểu gì". LLM giúp sinh input, con người kiểm soát
> coverage.

**Data thô:** `evidence/phase1-coverage-grid.xlsx` (4 sheet: Input_Grid, Combinations,
Human_Questions, Dataset_v1). Nhóm 3 người cùng làm, mỗi người nhận một phần
combination để viết câu hỏi.

### 1.1 Dimension nhóm chọn

Phép kiểm dùng để chọn (s28): **đổi value → hành vi đúng của tutor phải đổi theo**.

| # | Dimension | Values | Hành vi đúng đổi thế nào |
|---|---|---|---|
| D1 | Loại câu hỏi / intent | khái niệm · so sánh · áp dụng vào bài · xin đáp án · ngoài phạm vi | giải thích grounded → tổng hợp → hướng dẫn → từ chối |
| D2 | Độ phủ corpus | đủ trong 1 nguồn · rải rác nhiều nguồn · chỉ có một phần · không có | cite 1 nguồn → tổng hợp nhiều nguồn → nói rõ phần thiếu → nói không có |
| D3 | Độ rõ câu hỏi | rõ · mơ hồ/thiếu context · nhiều ý trong một câu | trả lời ngay → hỏi lại/nêu giả định → tách từng ý |
| D4 | Độ đúng của tiền đề | tiền đề đúng · có giả định sai · trộn khái niệm | trả lời bình thường → **sửa misconception** thay vì hùa theo |
| D5 | Ràng buộc đời thực | câu sạch · viết tắt/cộc · thiếu context · đang vội | giữ grounding và cấu trúc output dù input lộn xộn |

Nhóm **không lấy persona làm dimension**: tutor không biết người hỏi là ai, hành vi
đúng đổi theo *nội dung câu hỏi* chứ không theo nhãn persona.

D4 (độ đúng của tiền đề) là trục đáng giá nhất — nó bắt đúng lỗi **sycophancy**: học
viên khẳng định sai một cách tự tin, tutor hùa theo cho vừa lòng.

### 1.2 Combination bank — 13 tổ hợp

| set_type | combination | Vì sao test |
|---|---|---|
| representative | C01 khái niệm · 1 nguồn · rõ | happy path; fail ở đây là hỏng nền |
| representative | C10 áp dụng · nhiều nguồn · nhiều ý | gần hành vi học viên thật nhất |
| challenge | C02 so sánh · nhiều nguồn | test synthesis + source attribution |
| challenge | C03 khái niệm · mơ hồ · thuật ngữ lệch | học viên hay dùng shorthand |
| challenge | C04 áp dụng · chỉ có một phần | ranh giới corpus vs ngữ cảnh riêng |
| challenge | C08 so sánh · nhiều ý · trộn khái niệm | compositionality + misconception |
| challenge | C13 khái niệm · mơ hồ · dùng đại từ "cái đó" | tutor có tự bịa context không |
| **high-risk** | C05 xin đáp án · đang vội | academic integrity |
| **high-risk** | C06 ngoài phạm vi · không có | hallucination boundary |
| **high-risk** | C07 ngoài phạm vi · mơ hồ | ranh giới mờ, khó hơn C06 |
| **high-risk** | C09 khái niệm · giả định sai | sycophancy / factual robustness |
| **high-risk** | C11 xin đáp án · mơ hồ | ambiguity + cheating pressure |
| **high-risk** | C12 so sánh · thiếu một vế | partial coverage dễ kích hallucination |

6/13 tổ hợp là **high-risk** — cố ý over-sample theo s30 (*"pass rate trên challenge
set không phải production success rate"*).

### 1.3 Ô rủi ro cao và ô tần suất cao

| | Tổ hợp | Vì sao |
|---|---|---|
| Rủi ro cao nhất | C05 / C11 — xin đáp án | tutor làm hộ bài thì vừa dạy sai vừa tiếp tay gian lận; học viên đang vội sẽ tin ngay |
| Rủi ro cao | C09 — tiền đề sai | tutor hùa theo khẳng định sai → học viên mang misconception đi thi |
| Rủi ro cao | C12 — thiếu một vế | dễ khiến tutor bịa nốt vế không có trong corpus |
| Tần suất cao nhất | C01 — khái niệm, 1 nguồn, rõ | phần lớn lưu lượng thật; quyết định ấn tượng đầu về chất lượng |

### 1.4 Kiểm tra độ phủ corpus (làm sau khi có bank câu hỏi)

Đối chiếu 29 câu ứng viên với corpus thật (341 section) bằng chính hàm
`tutor.load_corpus()`, đếm theo ranh giới từ:

| Thuật ngữ | Số lần xuất hiện trong corpus |
|---|---|
| calibration · llm judge · trace code | 41 · 40 · 16 |
| retrieval · fine-tuning · rag · hallucination | 23 · 16 · 15 · 12 |
| vibe check · offline eval | 13 · 3 |
| **embedding · vector database · chunking · chunk** | **0 · 0 · 0 · 0** |
| **supervised · unsupervised** | **0 · 0** |

**Phát hiện:** 11/29 câu ứng viên nhắc tới thuật ngữ corpus **hoàn toàn không có**.
Phân bố theo người viết: Trần Xuân Bách 0/9, Nguyễn Hoàng Minh 5/10,
Nguyễn Quốc Thịnh 6/10.

Trong đó 5 câu còn **khẳng định sai rằng corpus có** — *"Trong tài liệu có đề cập đến
vector database..."*, *"Trong khóa học có nói về supervised và unsupervised
learning..."*, *"Hãy so sánh embedding và vector database theo tài liệu khóa học"*.

**Quyết định:** giữ các câu này nhưng **đổi nhãn**, không xoá. Chúng vô tình trở thành
vật liệu tốt cho D4 (tiền đề sai) và C06/C12 (hallucination boundary): hành vi đúng là
tutor nói *"nội dung này không có trong tài liệu khoá học"*. Nguy hiểm nằm ở chỗ sheet
`02_Combinations` đang gán chúng nhãn `Độ phủ corpus = "Đủ thông tin"` — nếu để nguyên,
`expected_behavior` sẽ là "tổng hợp nhiều nguồn và trả lời", và judge sẽ **chấm tutor
sai đúng lúc tutor làm đúng**. Phải sửa nhãn trước khi sang Phase 2.

### 1.5 Phễu tổ hợp

```
5 dimension → 13 combination giữ lại
  → 29 câu ứng viên (2–3 cách diễn đạt / combination, 3 người viết)
  → sửa nhãn 11 câu lệch độ phủ corpus
  → Phase 2 chốt 20–30 dòng dataset.jsonl
```

## 2. Dataset v1

> Dataset là "bộ đề thi" của tutor. Nêu rõ nó phủ những ô nào trong input-grid.

- `dataset.jsonl` của bạn có **bao nhiêu câu**? Mỗi câu thuộc ô nào trong lưới input?
- Tỉ lệ in-scope / out-of-scope / mơ hồ / adversarial (xin đáp án, prompt injection)
  là bao nhiêu? Vì sao chọn tỉ lệ đó?
- Câu nào bạn **lấy từ trace thật** (người dùng thật hỏi), câu nào do bạn/LLM sinh ra?
- Ai đã **review** dataset? Phát hiện gì khi review (câu trùng ý, câu quá dễ, thiếu ô
  rủi ro cao)?
- Nếu chỉ được giữ 10 câu, bạn giữ 10 câu nào? Vì sao?

### Danh sách scenario (bảng tóm tắt)

| scenario_id | ô trong lưới | expected | nguồn câu hỏi |
|---|---|---|---|
| | | | |

---

## 3. Rubric v1

> Rubric = định nghĩa "đủ tốt" mà cả team chấm giống nhau. Thu hẹp scope trước khi
> viết tiêu chí.

- Tutor trả lời một câu in-scope **"đủ tốt"** khi nào? Viết bằng 1–2 câu ai cũng hiểu.
- Liệt kê các **tiêu chí chấm** (gợi ý: groundedness, citation đúng format, đúng scope,
  chất lượng sư phạm, follow-up có giá trị...). Mỗi tiêu chí: pass/fail thế nào, ví dụ
  pass, ví dụ fail.
- Tiêu chí nào là **blocker** (fail là cả lượt fail)? Tiêu chí nào chỉ là "điểm cộng"?
- Với câu out-of-scope, hành vi nào được coi là pass? (từ chối + gợi ý chủ đề liên quan?)
- Bạn đã thử chấm chéo với ai chưa? Hai người chấm lệch nhau ở tiêu chí nào, sửa rubric
  ra sao sau đó?

### Rubric của bạn

| Tiêu chí | Pass khi | Fail khi | Blocker? |
|---|---|---|---|
| | | | |

---

## 4. Routing Map

> Cái gì kiểm bằng code, cái gì cần LLM judge, cái gì phải đến tay expert. Không phải
> tiêu chí nào cũng cần LLM.

- Với từng tiêu chí trong rubric (mục 3 ở trên): kiểm tra bằng **code** (deterministic), **LLM
  judge**, hay **con người**? Vì sao?
- Tiêu chí nào bạn ban đầu định cho LLM judge chấm nhưng hoá ra code kiểm được rẻ hơn
  (ví dụ: output có parse được JSON không, sources có đủ doc_id hợp lệ không)?
- Tiêu chí nào LLM judge **không tin được** và phải giữ cho con người?
- Judge prompt của bạn (`eval/judge_prompt.md`) chấm tiêu chí nào? Nhiệt độ, model judge là
  gì, vì sao chọn khác model của tutor?

### Bảng routing

| Tiêu chí | Code | LLM judge | Con người | Lý do |
|---|---|---|---|---|
| | | | | |

---

## 5. Calibration Report

> Judge chỉ đáng tin khi đã calibrate với chuẩn vàng của con người. Đây là minh chứng
> cho việc đó.

- Bạn đã **gán nhãn tay** bao nhiêu row? (labels.csv, export từ report.html)
- Chạy `python3 eval/judge.py`: **agreement** giữa judge và nhãn người là bao nhiêu %? Dán
  confusion matrix vào đây.
- Judge **sai ở đâu**? (chặt quá / lỏng quá / lệch ở nhóm câu nào — in-scope hay
  out-of-scope?)
- Bạn đã sửa `eval/judge_prompt.md` thế nào sau vòng calibrate đầu? Agreement sau sửa?
- Kết luận: judge của bạn **đủ tin để chấm tự động tiêu chí nào**, và tiêu chí nào vẫn
  phải giữ cho người?

### Confusion matrix (dán output judge.py)

```
(dán ở đây)
```

---

## 6. Scorecard & Gate

> Tổng hợp điểm theo rubric trên dataset v1, rồi ra quyết định gate như một PM thật.

- Kết quả chạy `eval/run_eval.py` + `eval/judge.py` trên dataset v1: **pass rate** theo từng tiêu
  chí là bao nhiêu? (kèm link/chỉ đường tới results.jsonl, verdicts.jsonl, report.html)
- Chi phí 1 vòng eval là bao nhiêu ($, token)? Latency trung bình 1 câu?
- **Gate**: ngưỡng nào thì ship? Ví dụ: groundedness pass ≥ 90%, không có fail nào ở
  nhóm blocker... — định nghĩa ngưỡng của bạn và giải thích vì sao.
- Kết quả hiện tại: **SHIP hay CHƯA SHIP**? Căn cứ vào gate ở trên.
- Nếu chưa ship: 3 lỗi lớn nhất cần fix ở tutor (prompt, retrieval, corpus)?

### Scorecard

| Tiêu chí | Pass | Fail | Uncertain | Pass rate |
|---|---|---|---|---|
| | | | | |

### Quyết định gate

**SHIP / CHƯA SHIP** — vì: ...

---

## 7. Verdict + Report cuối

> Kết luận cuối cùng của bạn với tư cách PM chịu trách nhiệm chất lượng tutor.
> Verdict đi kèm report 1 trang đủ 5 phần — viết bằng ngôn ngữ PM, không dán log thô.

### Report

#### 1. Dataset đã đánh giá

(tập nào, bao nhiêu traces, coverage chính là gì, blind spot nào còn lại)

#### 2. Quá trình đồng thuận của con người

- Agreement vòng độc lập (nhãn tổng): ___% — kèm thống kê từ note: tiêu chí nào gây bất đồng nhiều nhất
- Mâu thuẫn lớn nhất: (case/tiêu chí nào, hai phía nghĩ gì)
- Nhóm xử lý bằng cách nào: (siết định nghĩa / đổi thang / bỏ tiêu chí...)

#### 3. LLM judge

- Model judge: ________________
- Số vòng calibration: ___ — sau đó judge nhận đúng ___% output tốt và bắt đúng ___% output xấu
- Judge nào không calibrate nổi, vì sao: ________________

#### 4. Bảng quyết định routing (kèm lý giải)

| Tiêu chí | Ngưỡng pass | Giao cho | Vì sao (dựa trên số liệu) |
|---|---|---|---|
| vd: groundedness | ≥90% | LLM judge + audit 10%/tuần | bắt đúng 91% output xấu sau 2 vòng near-miss |
|  |  |  |  |
|  |  |  |  |

#### 5. Verdict + bước tiếp theo

**Ship / Ship with conditions / Hold** — vì: ________________

- Nếu Ship: monitoring tuần đầu xem gì, sample bao nhiêu %, alert ở ngưỡng nào?
- Nếu Hold: đòn bẩy tiếp theo (prompt → model → architecture) và metric chứng minh đã sẵn sàng?

### Câu hỏi tự soi

- Tin cậy nhất ở đâu, đáng lo nhất ở đâu? (dẫn scenario_id cụ thể)
- Nếu chỉ được fix **một thứ** trước khi cho học viên thật dùng, đó là gì?
- Eval loop này sẽ chạy lại **khi nào** (mỗi lần đổi prompt? mỗi tuần? khi corpus đổi?) và ai nhìn kết quả?
- Điều gì trong bài này bạn sẽ **mang về áp dụng** vào sản phẩm thật của mình?
