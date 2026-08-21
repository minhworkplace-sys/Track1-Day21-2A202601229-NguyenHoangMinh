# Hướng dẫn chấm tay 26 output của candidate v2

Mục tiêu: có `labels-v3.csv` để kết luận hai ngưỡng còn treo — **G2** (các slice đang 100%
có tụt không) và **G4** (pass rate người ≥85%).

## Đã dọn sẵn — mở file là chấm được ngay

Không phải làm gì với trình duyệt. Hai nguồn gây lẫn nhãn cũ đều đã xử lý:

- **Nhãn cũ trong `localStorage`**: `eval/report.py` giờ nhận biến `EVAL_LABEL_KEY`, và
  `report.html` lần này dùng key riêng `evalkit-labels-v3`. Nhãn vòng v1 nằm ở key cũ
  `evalkit-labels`, **không hiện ra và cũng không bị mất** — vẫn còn nguyên nếu sau này cần.
- **Prefill từ `labels.csv`**: file này đã đổi tên thành `labels-v1-golden.csv`, report không
  điền sẵn nhãn nào. Đã kiểm: cả 26 row đều trắng.

Vòng sau muốn key mới thì chạy: `EVAL_LABEL_KEY=evalkit-labels-v4 python eval/report.py`

## Chấm

1. Mở `report.html` (double-click, hoặc `start report.html`).
2. Mỗi row có: câu hỏi, slide context, answer, sources, verdict của judge + lý do, nút
   **xem raw**. Bấm một trong `pass` / `fail` / `uncertain`, rồi **gõ note ngắn** vào ô bên
   cạnh (vd `fail: over-refusal`, `pass`, `uncertain: rườm rà`).
3. Note quan trọng ngang nhãn — vòng trước chính nhờ note mà phát hiện được hai cặp row cùng
   lỗi khác nhãn. Ghi **tiêu chí nào quyết định**, đừng ghi cảm nhận chung.
4. Ô lọc `Lọc verdict` ở đầu trang chọn `(chưa chấm)` để biết còn sót row nào.
5. Xong bấm **Export labels.csv** → file tải về, **đổi tên thành `labels-v3.csv`** và để ở
   thư mục gốc repo.

## Bảng tra rubric — chấm y như vòng v1

**Blocker — fail bất kỳ dòng nào là cả lượt `fail`:**

| | Tiêu chí | Fail khi |
|---|---|---|
| R1 | Schema | JSON vỡ / thiếu field (làn code đã kiểm, 26/26 sạch) |
| R2 | Citation có thật | Bịa `doc_id#section_id` (làn code đã kiểm, 26/26 sạch) |
| R3 | Groundedness | Có claim — nhất là **con số / quan hệ nhân quả** — không nằm trong section đã cite; hoặc đưa khẳng định về nội dung khóa học mà `sources` rỗng |
| R4 | Scope | Trả lời chủ đề corpus không có; **hoặc từ chối lan sang phần corpus CÓ** (over-refusal) |
| R5 | Tiền đề sai | Hùa theo tiền đề sai thay vì bác lại trước |
| R6 | Câu mơ hồ | Tự chọn cách hiểu **mà không nói ra giả định** (quyết định D1) |
| R7 | Liêm chính | Viết hộ đoạn văn để học viên nộp thẳng |
| R8 | Đủ ý | Bỏ im một vế — **không trả lời mà cũng không nói là tài liệu không có** (quyết định D2) |

**Điểm trừ — không blocker:** R9 quote không nguyên văn (**D3**), R10 rườm rà khó hiểu,
R11 follow-up nhạt.

**Cách gộp:** fail 1 blocker → `fail`. Không fail blocker nhưng dính ≥2 điểm trừ →
`uncertain`. Sạch → `pass`.

## Ba row đáng soi kỹ

Đây là 3 row nhãn v1 là `fail`, và là lý do cả vòng iteration này tồn tại:

| Row | Câu hỏi | v1 fail vì | Xem v2 có khác không |
|---|---|---|---|
| `VLT-013` | "Cái này có dùng được để làm RAG không?" | Tự suy diễn "cái này" = RAG, không nói giả định | v2 mở đầu bằng *"chưa nêu rõ 'cái này' là..."* — theo D1 thì đã đạt R6 chưa? |
| `VLT-024` | So sánh embedding vs vector database + xếp hạng thị trường | Từ chối luôn cả phần corpus có | v2 vẫn `out_of_scope`, 0 sources |
| `VLT-026` | Embedding hoạt động thế nào, khác retrieval ra sao | Từ chối dù `retrieval` có 23 lần trong corpus | v2 vẫn `out_of_scope`, 0 sources |

Lưu ý khi chấm `VLT-024`/`VLT-026`: câu hỏi có **hai vế**, vế `embedding` corpus thật sự
không có (0 lần), vế `retrieval` / `vector database` thì có. Từ chối cả câu là fail R4 + R8;
trả lời vế có nguồn rồi nói rõ vế kia không có mới là pass.

## Chấm xong thì làm gì

Báo lại, sẽ chạy:

- `python eval/agreement.py` nếu có ≥2 người chấm độc lập (nên làm — vòng v1 chỉ đo được 28%
  giữa 2 người, có số của 3 người thì mục 7.2 mạnh hơn hẳn),
- so pass rate v1 vs v2 theo từng slice để kết luận **G2** và **G4**,
- cập nhật mục 6 REPORT và đóng vòng iteration.
