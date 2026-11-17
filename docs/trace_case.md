# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

## Báo cáo Trace Case: C-002@11 (Failure Mode FM-03)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11`, run ID `E-seeded_regression-20261118-7be6` (hoặc `E-seeded_regression-C-002-11`) |
| Triệu chứng | Grader `lab8_pass` FAIL (`status resolved != denied`), `RefundAmountCorrect` FAIL (`expected 0, got 20000`), `DbFactKept` FAIL (`1 override event(s)`); reply nói "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." trong khi ledger ghi 20.000đ còn nhãn kỳ vọng từ chối (0đ). |
| Bước sai đầu tiên | Bước 3: `model_override` `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Bước 1 (`handoff`) thiết lập ngữ cảnh chuẩn xác theo `handoff.v1`. Bước 2 (`investigator`) gọi `lookup_order` đọc dữ liệu hệ thống đúng (`order_id: O-202, delay_days: 0`). Tại bước 3, agent tin lời khai trong ticket ("Đơn giao trễ 5 ngày") và phát sinh event `model_override` ghi đè sự thật của DB thành `delay_days: 5`, dẫn tới các bước sai tiếp theo (`quote` ra 20.000đ và hoàn tiền sai). |
| Failure mode | FM-03 (`ticket_trust — tin ticket hơn DB`) |
| Đối chứng | Cùng claim@seed trên shape `fixed`: Không có bước `model_override`. Bước 3 chuyển thẳng sang `quote` với `amount_vnd: 0, reason: no_entitlement`, ra status `denied`, ledger hoàn 0đ, pass tất cả các grader. |
| Regression test | Assertions trong `tests/test_lab9_graders.py` (`DbFactKept` và `RefundAmountCorrect`), cùng item `E-002` trong eval set. Test đã xác nhận FAIL trên `seeded_regression` và PASS trên `fixed`. |
| Chưa chắc chắn | Môi trường kiểm thử dùng MockLLM với parse rule xác định. Trên LLM thực tế, các biến thể câu từ mập mờ hoặc prompt injection tinh vi có thể dẫn tới hành vi sai lệch khác nếu không có chốt chặn policy cứng ở tầng orchestration. |

## Quy tắc áp dụng
- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn tiền sai trái phép trong sổ cái.
- "Bước sai đầu tiên" được chỉ đích danh bằng số thứ tự event (`event 3: model_override`), kèm trích dẫn payload cụ thể.
- Regression test chạy độc lập và tự động hóa trong bộ test CI của repo.
