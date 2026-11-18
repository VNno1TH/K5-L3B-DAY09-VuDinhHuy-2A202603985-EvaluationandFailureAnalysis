# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID `E-seeded_regression-20261118-2ac6` |
| Triệu chứng | Grader Lab 8 fail: `status resolved != denied`; grader `refund_amount_correct` fail (ledger ghi 20.000đ thay vì 0đ); grader `db_fact_kept` fail; reply thông báo hoàn 20.000đ cho đơn O-202 trong khi nhãn quy định từ chối (`denied`, 0đ) |
| Bước sai đầu tiên | Event 3: `model_override {"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event 1 (`handoff`) và Event 2 (`investigator`) lấy dữ liệu chuẩn xác từ SQLite DB (`delay_days = 2`). Đến Event 3, agent tự động đè thông số này thành 5 theo lời khai trong ticket của khách hàng, trực tiếp làm sai lệch bước tính phí kế tiếp (`quote` tính ra 20.000đ ở Event 4) |
| Failure mode | Mã `FM-03` (`ticket_trust — tin ticket hơn DB`) trong `docs/failure_taxonomy.md` |
| Đối chứng | Cùng `C-002@11` trên shape `fixed`: hoàn toàn không xuất hiện Event `model_override`. Giữ nguyên `delay_days = 2` từ DB, dẫn tới Event 3 `quote` trả về 0đ (`no_entitlement`) và reply từ chối hợp lệ |
| Regression test | `test_override_event_breaks_db_fact` (grader `DbFactKept`) và `RefundAmountCorrect` trong `tests/test_lab9_graders.py`. Đã xác nhận fail trên `seeded_regression` và pass trên `fixed` |
| Chưa chắc chắn | Trace chỉ ghi nhận hành vi thay đổi giá trị (`model_override`) chứ không chứa chuỗi lập luận nội tâm (reasoning tokens) giải thích vì sao agent lại tin ticket hơn DB |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
