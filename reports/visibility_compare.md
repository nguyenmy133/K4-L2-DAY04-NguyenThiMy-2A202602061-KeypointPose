# Visibility report

- Thư mục nhãn: `dataset\labels\train`
- 20 ảnh, 29 skeleton, trung bình 15.79 khớp có v > 0 mỗi người
- Tổng: v=2 327 | v=1 131 | v=0 35

So sánh với `gold\labels\train` (29 skeleton).
Cột **lệch** là hiệu số phần trăm v=1 - chỗ nào lệch nhiều nhất là chỗ guideline chưa nói rõ.

| # | Khớp | %v=1 (bạn) | %v=1 (đối chiếu) | lệch |
| ---: | --- | ---: | ---: | ---: |
| 3 | left_ear | 66% | 3% | 62 |
| 4 | right_ear | 48% | 14% | 34 |
| 2 | right_eye | 28% | 0% | 28 |
| 0 | nose | 24% | 3% | 21 |
| 1 | left_eye | 24% | 3% | 21 |
| 14 | right_knee | 31% | 14% | 17 |
| 9 | left_wrist | 21% | 7% | 14 |
| 8 | right_elbow | 14% | 3% | 10 |
| 12 | right_hip | 31% | 21% | 10 |
| 11 | left_hip | 34% | 28% | 7 |
| 5 | left_shoulder | 14% | 7% | 7 |
| 7 | left_elbow | 14% | 7% | 7 |
| 10 | right_wrist | 28% | 21% | 7 |
| 15 | left_ankle | 24% | 17% | 7 |
| 16 | right_ankle | 28% | 24% | 3 |
| 6 | right_shoulder | 3% | 3% | 0 |
| 13 | left_knee | 21% | 21% | 0 |

## Đọc bảng này thế nào

1. Khớp nào có **%v=1 cao**: khớp hay bị che. Cổ tay và hông thường là hai vị trí cần xem lại guideline trước khi kết luận.
2. Khớp nào có **v=0 cao bất thường**: mọi người đang dùng Outside ở chỗ đáng lẽ là Occluded. Đó là lỗi số 3 của slide 46, và nó xoá thẳng khớp đó khỏi bảng điểm OKS.
3. Khi so hai người: **lệch lớn = bất đồng về guideline**, không phải về bức ảnh. Sửa guideline trước, sửa nhãn sau.
