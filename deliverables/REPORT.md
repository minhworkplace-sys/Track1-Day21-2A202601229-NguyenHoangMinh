# REPORT — Eval loop A→Z: VLearn AI Tutor

Report A→Z của eval loop — mỗi mục ứng một phase của bài lab. Mọi số liệu và quyết
định trong đây phải dẫn được xuống file data thô trong `evidence/` (dataset-v1.jsonl,
results-vN.jsonl, labels.csv, judge-prompt-vN.md, verdicts-vN.jsonl, braintrust-link.md).


---

## 1. Input Grid

> Lưới input = trục "ai hỏi" × "hỏi kiểu gì". LLM giúp sinh input, con người kiểm soát
> coverage. Trả lời các câu hỏi sau rồi vẽ lưới của bạn.

### 1.1 Chọn dimension (bước 1–2 của s29)

Kiểm tra dùng để chọn (s28): **đổi value của biến → hành vi đúng của tutor phải đổi
theo**. Không đổi thì đó chỉ là paraphrase, không phải dimension.

| Dimension | Values | Đổi value thì hành vi đúng đổi thế nào |
|---|---|---|
| **D1. Persona** (giai đoạn học) | học viên mới · đang làm capstone · ôn lại trước khi nộp · PM ngoài team | Học viên mới cần giải thích từ nền; người đang làm capstone cần câu trả lời gọn + trỏ đúng slide; PM ngoài team hỏi câu không thuộc corpus khoá |
| **D2. Intent** | hỏi khái niệm · tra cứu slide cụ thể · xin ví dụ/áp dụng · xin đáp án bài tập · hỏi ngoài phạm vi | Từ "trả lời + trích nguồn" sang "từ chối khéo + trỏ về nội dung học" — hành vi đúng đảo ngược hoàn toàn |
| **D3. Độ phủ corpus** | có slide trả lời trực tiếp · phải tổng hợp nhiều tài liệu · corpus chưa đề cập | Trực tiếp thì cite 1 nguồn; tổng hợp thì phải cite nhiều `doc_id#section_id`; chưa đề cập thì **phải nói không biết** thay vì suy từ trí nhớ model |
| **D4. Độ rõ câu hỏi** | rõ, đủ ngữ cảnh · thiếu ngữ cảnh · nhiều ý trong một câu | Thiếu ngữ cảnh → hành vi đúng là **hỏi lại**, không phải đoán rồi trả lời |

**Đã loại khỏi dimension:** độ dài câu hỏi, formal/informal, tiếng Việt/tiếng Anh —
đổi các biến này thì nội dung câu trả lời đúng **không đổi**, nên chúng là biến thể
diễn đạt (dùng ở bước 5, khi LLM paraphrase), không phải trục coverage.

### 1.2 Lưới của bạn (D1 × D2)

■ chọn test · ▨ loại (tổ hợp phi lý) · □ cân nhắc sau

| Persona \ Intent | Hỏi khái niệm | Tra cứu slide | Xin ví dụ/áp dụng | Xin đáp án bài tập | Hỏi ngoài phạm vi |
|---|---|---|---|---|---|
| **Học viên mới** | ■ tần suất cao nhất | ■ | ■ | □ | ■ |
| **Đang làm capstone** | ■ | ■ **rủi ro cao** | ■ | ■ **rủi ro cao nhất** | □ |
| **Ôn lại trước khi nộp** | ■ | ■ | □ | □ | ▨ |
| **PM ngoài team** | ■ | ▨ không biết slide nào | □ | ▨ không làm bài tập | ■ |

**Ô loại và vì sao:**

- *Ôn lại × hỏi ngoài phạm vi* — người đang ôn bám sát nội dung thi, hỏi lạc đề là
  hành vi hiếm đến mức không đáng chiếm slot trong dataset.
- *PM ngoài team × tra cứu slide* — họ không dự lớp nên không có khái niệm "slide s47".
- *PM ngoài team × xin đáp án* — họ không nộp bài capstone.

### 1.3 Ô rủi ro cao và ô tần suất cao

| | Ô | Vì sao |
|---|---|---|
| **Rủi ro cao nhất** | Đang làm capstone × xin đáp án bài tập | Tutor bịa đáp án thì vừa dạy sai vừa tiếp tay gian lận. Học viên đang gấp sẽ tin ngay mà không kiểm chứng — failure cost cao nhất trong toàn lưới |
| **Rủi ro cao** | Đang làm capstone × tra cứu slide (kết hợp D3 = phải tổng hợp) | Đây là ô dễ sinh lỗi **citation không support claim**: tutor trả lời đúng ý nhưng gắn sai `doc_id#section_id`. Người học mở nguồn ra không thấy nội dung đó → mất niềm tin vào toàn hệ thống |
| **Rủi ro cao** | Mọi persona × D3 = corpus chưa đề cập | Tutor phải nói "không có trong tài liệu". Nếu nó suy từ trí nhớ model thì vi phạm đúng cam kết lõi của sản phẩm |
| **Tần suất cao nhất** | Học viên mới × hỏi khái niệm | Chiếm phần lớn lưu lượng thật; là ô quyết định ấn tượng đầu tiên về chất lượng tutor |

### 1.4 Phễu tổ hợp (bước 3–4 của s29)

```
D1 (4) × D2 (5) = 20 ô
  → loại 3 ô phi lý                      = 17 ô
  → chốt 12 ô đánh dấu ■                 = 12 ô
  → nhân D3/D4 ở các ô rủi ro cao        ≈ 16–18 scenario
  → Phase 2 viết thành ~20 dòng dataset.jsonl
```

**Constraint đời thật đã thêm** (bước 4 — làm scenario bớt "sạch"):

- Câu hỏi không nêu rõ đang hỏi bài nào ("eval này ổn chưa anh?") — buộc tutor dựa
  vào `metadata.slide` hoặc hỏi lại.
- Học viên dùng thuật ngữ khác slide (nói "chấm điểm tự động" thay vì "LLM judge").
- Gộp nhiều ý trong một câu — kiểm tra tutor có trả lời thiếu vế không.
- Xin đáp án kèm lý do gây áp lực ("sắp hết hạn nộp rồi") — thử xem tutor có mềm lòng
  mà phá rào không.

Ba loại scenario theo s30: phần lớn là **representative**, cố ý **over-sample
challenge** ở ô rủi ro cao (xin đáp án, tổng hợp nhiều nguồn, câu mơ hồ), và giữ vài
**critical regression** (citation không support claim, nói kiến thức ngoài corpus
thành nội dung đã dạy).

---

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
