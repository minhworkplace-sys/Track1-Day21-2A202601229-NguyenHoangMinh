# Nhãn người — hai vòng, và chuyện id không cùng hệ

## Các file

| File | Là gì |
|---|---|
| `labels-round1-bach.csv` · `labels-round1-thinh.csv` | **Vòng 1** — hai thành viên chấm **độc lập**, chưa có rubric |
| `labels-round1-*-normalized.csv` | Cùng nội dung, id đã đưa về hệ `VLT-###` của dataset v1 để `agreement.py` so được |
| `labels-round2-group.csv` | **Vòng 2** — bản cả 3 người họp và thống nhất |
| `labels-golden-v1.csv` | Nhãn vàng cuối, sau khi chốt rubric R1–R11 và các quyết định D1–D4 |

## Ba hệ id khác nhau — phải chuẩn hoá trước khi so

Đây là cái bẫy lớn nhất của bước này, và nó **không báo lỗi** nếu ghép ẩu:

| Nguồn | Hệ id | Ý nghĩa |
|---|---|---|
| Bách | `S001`–`S029` | 29 câu ứng viên (thứ tự trong `phase1-question-review.md`) |
| Thịnh | `VLT-001`–`VLT-029` | **Cũng là 29 câu ứng viên**, chỉ khác tiền tố — nhận ra vì có `VLT-027/028/029` trong khi dataset chỉ tới `VLT-026` |
| Dataset v1 | `VLT-001`–`VLT-026` | 26 row sau khi chốt Keep/Rewrite/Reject |

Ghép thẳng theo chuỗi id thì từ dòng 15 trở đi nhãn sẽ gán **sai row** mà không có lỗi nào
báo — vì 4 câu REJECT (số 15, 16, 21, 29) bị rút ra làm lệch toàn bộ phần sau. Phép ánh xạ
dùng chung với `label-id-mapping.md`.

## Kết quả đo

```
python3 eval/agreement.py labels-round1-bach-normalized.csv labels-round1-thinh-normalized.csv

Case chung: 25
Đồng thuận hoàn toàn: 7/25 = 28%
18 case bất đồng
```

Khoảng cách tới nhãn vàng: Bách 16/25 = **64%** · Thịnh 11/25 = **44%**. Bản hợp nhất không
phải nhãn của riêng ai — cả hai phía đều dịch chuyển sau thảo luận.

Nguyên nhân bất đồng không phải đọc câu trả lời khác nhau, mà là **hai rubric ngầm khác
nhau**: Thịnh coi citation/quote sai là blocker (17/25 `fail`, note gần như chỉ có
`citation` / `scope`), Bách thì không (18/25 `pass`). Chính câu hỏi đó về sau được chốt
thành quyết định **D3** ở mục 3 của `REPORT.md`.

## Một khiếm khuyết của file Bách — ghi để người chấm biết

`labels-round1-bach.csv` **mất note gốc và 4 dòng** (ứng với 4 câu REJECT: `S015`, `S016`,
`S021`, `S029` — phân bố 3 `pass` · 1 `fail`), do một bước xử lý file ghi đè nhầm: file
chuẩn hoá được đặt tên chữ thường (`labels-bach.csv`) trong khi bản gốc là `labels-Bach.csv`,
mà Windows không phân biệt hoa–thường.

25 nhãn còn lại **nguyên giá trị** và là phần được dùng để tính agreement (4 dòng mất đều
thuộc câu REJECT, vốn không nằm trong dataset v1 nên không ảnh hưởng con số 28%). File của
Thịnh còn nguyên vẹn 29 dòng kèm note.
