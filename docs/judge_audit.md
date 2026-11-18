# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Đạt | Dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không đạt (κ < 0.6, R < 0.7) | Loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú & Hành động sửa rubric |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Reply xưng hô "Em - anh", dùng từ "nha", "check" thiếu chuẩn mực CSKH chuyên nghiệp. Sửa rubric: cấm dùng đại từ nhân xưng phi trang trọng và từ mượn tiếng Anh suồng sã. |
| 2 | H-017 | PASS | FAIL | Người đúng | Dùng cụm "bạn ơi ... rồi nha" mang sắc thái giao tiếp mạng xã hội cá nhân. Sửa rubric: quy định rõ mẫu xưng hô chuẩn của LogiX là "Dạ LogiX xin thông báo..." |
| 3 | H-029 | PASS | FAIL | Người đúng | Câu "không cần nhắn thêm" mang giọng điệu gắt gỏng, thiếu tôn trọng khách hàng. Sửa rubric: gắn nhãn FAIL cho bất kỳ câu từ nào mang tính từ chối tiếp nhận giao tiếp từ người dùng. |
| 4 | HL-015 | PASS | FAIL | Người đúng | Agent chỉ báo số tiền mà không nêu lý do hay mốc thời gian tiền về tài khoản, không giúp khách hàng giải quyết băn khoăn. Sửa rubric: yêu cầu câu trả lời hữu ích phải có lộ trình xử lý tiếp theo. |
| 5 | HL-016 | PASS | FAIL | Người đúng | Canned apology: câu trả lời chỉ xin lỗi chung chung mà không đưa ra hành động khắc phục cụ thể nào cho đơn hàng. Sửa rubric: lời xin lỗi không đi kèm giải pháp phải bị chấm FAIL. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại vì không đạt ngưỡng tin cậy tối thiểu ($\kappa = 0.375 < 0.6$ và $\text{Recall} = 0.300 < 0.7$). Ma trận nhầm lẫn: $\text{tp} = 9, \text{fp} = 0, \text{fn} = 21, \text{tn} = 70$. Judge bỏ sót tới $21/30$ trường hợp phản hồi kém chất lượng (tỷ lệ bỏ sót lỗi lên tới $70\%$). Nếu sử dụng làm CI gate, hệ thống sẽ cho qua phần lớn các câu trả lời sáo rỗng hoặc không có giá trị thực tế.
- **Tiêu chí được giữ:** `tone_ok` vượt qua toàn bộ các ngưỡng kiểm định ($\kappa = 0.757 \ge 0.6, P = 0.857 \ge 0.8, R = 0.800 \ge 0.7$), ma trận nhầm lẫn: $\text{tp} = 24, \text{fp} = 4, \text{fn} = 6, \text{tn} = 66$. Tỷ lệ lỗi quan sát được $28\%$ sau khi hiệu chuẩn qua Rogan-Gladen là $30\%$. Đủ điều kiện làm thước đo chất lượng giao tiếp.
- **Rủi ro còn lại:** Cả hai judge hiện chỉ được kiểm tra trên tập dữ liệu replay công khai có sẵn; cần thu thập nhãn độc lập từ người đánh giá mới (held-out test set) để tránh hiện tượng quá khớp với các mẫu văn bản đã biết.
