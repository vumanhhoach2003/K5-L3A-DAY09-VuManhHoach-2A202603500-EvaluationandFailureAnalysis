# Judge audit

Chạy kiểm tra: `python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5` và `data/eval/judge_helpful_public.csv`.

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | Dùng làm judge tham khảo |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (Recall=0.300 < 0.7, κ=0.375 < 0.6) | LOẠI (Không dùng làm gate) |

## ≥5 case audit thủ công

| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* Văn phong xưng hô ngôi thứ nhất ("em/anh"), dùng từ đệm suồng sã ("nha", "check") vi phạm tiêu chuẩn thương hiệu. **Sửa rubric:** Quy chuẩn hóa xưng hô bắt buộc là *"Dạ, LogiX..."*, cấm sử dụng từ lóng/khẩu ngữ hội thoại. |
| 2 | H-017 | PASS | FAIL | Người đúng | Reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* Từ xưng hô "bạn ơi" và đuôi câu "nha" thiếu tính chuyên nghiệp trong CSKH B2C. **Sửa rubric:** Bổ sung negative few-shot examples về các từ xưng hô thân mật quá mức vào prompt của judge. |
| 3 | H-029 | PASS | FAIL | Người đúng | Reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* Cụm từ "không cần nhắn thêm" mang sắc thái cộc lốc, xua đuổi khách hàng. **Sửa rubric:** Đưa tiêu chí "thấu cảm & tôn trọng" thành hard constraint, phạt nặng các câu từ mang tính mệnh lệnh tiêu cực. |
| 4 | H-042 | FAIL | PASS | Người đúng | Reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* Judge phạt vì cho rằng cung cấp mốc thời gian ngoài policy. Thực tế đây là thông tin SLA hữu ích và câu từ nhã nhặn. **Sửa rubric:** Cho phép thông báo khung thời gian đối soát tiền về tiêu chuẩn của ngân hàng (3-5 ngày). |
| 5 | H-051 | FAIL | PASS | Người đúng | Reply: *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* Judge gắn nhãn FAIL do xuất hiện từ tiếng Anh ("refund"). Dù nên ưu tiên từ thuần Việt "hoàn tiền", câu văn vẫn giữ được phép lịch sự căn bản. **Sửa rubric:** Tách tiêu chí ngôn ngữ (thuần Việt) ra khỏi tiêu chí thái độ/tone để tránh false positive. |
| 6 | HL-016 (Helpful) | PASS | FAIL | Người đúng | Reply: *"Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này và rất trân trọng sự kiên nhẫn của..."* Agent nói lời xin lỗi hoa mỹ nhưng hoàn toàn không giải quyết được vấn đề đơn hàng. Judge helpful bị đánh lừa bởi độ trôi chảy (fluency). **Sửa rubric:** Bắt buộc judge kiểm tra tính hành động (actionability) thay vì chỉ đánh giá độ lịch thiệp. |

## Kết luận
- **Tiêu chí bị loại:** `judge_helpful` bị loại hoàn toàn khỏi pipeline release gate. Ma trận nhầm lẫn của `judge_helpful` cho thấy tỷ lệ bỏ lọt lỗi nghiêm trọng: `tp=9, fn=21` (Recall chỉ đạt 30.0%, Cohen's Kappa chỉ đạt 0.375). Judge này bị "bỏ bùa" bởi các câu trả lời dài, xin lỗi bóng bẩy nhưng sáo rỗng, khiến 70% các phản hồi vô dụng lọt qua mắt judge.
- **Hiệu chỉnh Rogan-Gladen:** Với `judge_tone`, tỷ lệ lỗi quan sát được là 28.0%, sau khi hiệu chỉnh theo độ nhạy (Recall 0.8) và độ đặc hiệu (Specificity 66/70 ≈ 0.943), tỷ lệ lỗi thực tế xấp xỉ 30.0%.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ được hiệu chuẩn trên tập replay tĩnh đã công khai (`judge_*_public.csv`). Khi đưa vào thực tế cần bổ sung tập dữ liệu thẩm định mới (held-out blind test) với nhãn gán độc lập từ chuyên viên QA con người để tránh overfit rubric.
