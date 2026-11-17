# Judge audit

Chạy: `python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5` và `python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (tất cả ≥ sàn) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (κ=0.375 < 0.6, R=0.300 < 0.7) | loại |

- `tone_ok` confusion matrix: `{'tp': 24, 'fp': 4, 'fn': 6, 'tn': 66}`, fail rate observed `0.28` → Rogan-Gladen corrected `0.30`.
- `helpful` confusion matrix: `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}`, fail rate observed `0.09` → Rogan-Gladen corrected `0.30`.

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú / sửa rubric |
|---|---|---|---|---|---|
| 1 | `H-008` (`tone_ok`) | PASS | FAIL | Người đúng | `"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."` — xưng hô "em/anh", dùng từ "nha", thiếu chuẩn mực CSKH LogiX. Bổ sung ví dụ cấm xưng hô thân mật quá mức vào rubric `tone_ok`. |
| 2 | `H-017` (`tone_ok`) | PASS | FAIL | Người đúng | `"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."` — dùng "bạn ơi / rồi nha" không đạt chuẩn lịch sự "anh/chị / ạ". Bổ sung quy tắc từ khóa/ngữ điệu vào rubric. |
| 3 | `H-029` (`tone_ok`) | PASS | FAIL | Người đúng | `"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."` — câu "không cần nhắn thêm" mang sắc thái đuổi khách dù có chữ "Dạ". Thêm tiêu chí đánh giá thái độ tôn trọng khách hàng. |
| 4 | `H-042` (`tone_ok`) | FAIL | PASS | Người đúng | `"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."` — câu trả lời hoàn toàn lịch sự, rõ ràng; judge phạt nhầm do câu ngắn. Thêm ví dụ câu ngắn đạt chuẩn vào rubric. |
| 5 | `H-051` (`tone_ok`) | FAIL | PASS | Người đúng | `"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."` — có từ tiếng Anh "refund" nhưng ngữ điệu vẫn lịch sự; làm rõ trong rubric có chấp nhận thuật ngữ vay mượn hay không. |
| 6 | `HL-015` (`helpful`) | PASS | FAIL | Người đúng | `"Dạ, LogiX đã hoàn 55.000đ cho đơn O-112 ạ."` — judge thấy ngắn gọn nên chấm PASS, nhưng thực chất thiếu thông tin giải thích/hướng dẫn bước tiếp theo (timeline nhận tiền). |
| 7 | `HL-016` (`helpful`) | PASS | FAIL | Người đúng | `"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này..."` — chỉ xin lỗi suông, không trả lời kết quả xử lý hoàn tiền nhưng judge bị đánh lừa bởi lời xin lỗi dài. |

## Kết luận
- **Loại `helpful`, không được dùng làm CI gate:** Dù Precision = 1.0, Recall chỉ đạt 0.300 (`fn=21` trên tổng số `30` ca người chấm FAIL) và κ = 0.375 (< 0.6). Judge bỏ sót 70% ca thực sự không hữu ích (chỉ đo được tỉ lệ lỗi quan sát 0.09 trong khi tỉ lệ thật sau hiệu chỉnh Rogan-Gladen là 0.30), nên nếu dùng làm gate sẽ cho qua hàng loạt bản tệ.
- **Giữ `tone_ok` (theo dõi / non-blocking):** Đạt FLOORS (κ=0.757, P=0.857, R=0.800), vẫn cần cập nhật rubric cho các câu ngắn và từ ngữ thân mật.
- **Rủi ro còn lại:** Judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có tập held-out thật.
