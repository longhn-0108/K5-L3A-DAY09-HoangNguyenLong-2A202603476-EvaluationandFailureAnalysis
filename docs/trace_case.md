# Trace case: từ trace tới quyết định sai đầu tiên

Lệnh đối chứng đã chạy:
```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11
```

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` (report `eval_seeded_regression.json`, eval set `a852077c8681`) |
| Triệu chứng | `lab8 grader` (`pass_rate`), `refund_amount_correct`, `db_fact_kept` đều FAIL (`status resolved != denied`). Ledger ghi `refund_vnd=20000`, `refund_count=1` và reply nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."`, trong khi nhãn chuẩn là `refund_vnd=0`, `refund_count=0`, `status=denied`. |
| Bước sai đầu tiên | Event #3 `model_override`: `{"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event #1 (`handoff`) chuyển đúng `claim_id=C-002` và danh sách `allowed_tools`; Event #2 (`investigator`) truy vấn DB trả về đúng `order O-202` với `delay_days=2` (không đủ điều kiện hoàn phí trễ). Đến Event #3, agent tin thông tin "giao trễ 5 ngày" từ ticket khách hàng và ghi đè `delay_days=5`, dẫn tới Event #4 tính sai `quote=20000` và ghi ledger sai. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) trong `docs/failure_taxonomy.md` |
| Đối chứng | Trên shape `fixed` cùng `C-002@11`: không có event `model_override`; sau Event #2 (`investigator`), Event #3 là `quote {"amount_vnd": 0, "reason": "no_entitlement"}`, Event #4 là `reply` từ chối, ledger giữ `refund_vnd=0`, `refund_count=0`, `status=denied`. |
| Regression test | `E-002` (`C-002`) trong `data/eval/eval_set_v1.jsonl` (gắn `FM-03`) kết hợp grader `DbFactKept` + `RefundAmountCorrect` (`tests/test_lab9_graders.py::test_override_event_breaks_db_fact`, `test_wrong_amount_in_ledger`). Đã xác nhận fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace trên MockLLM không phản ánh xác suất model thật bị prompt injection khi câu chữ ticket biến thể phức tạp hơn. |
