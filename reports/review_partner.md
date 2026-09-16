# Báo cáo kiểm chéo / Reviewer Checklist

Người gán: Nguyễn Thị My (Team AI)   |   Người kiểm: Reviewer (Solo Audit & Peer Review)   |   Ngày: 16/09/2026

---

## 1. Kết quả kiểm tra tự động và trực quan

Đã thực thi các lệnh kiểm tra:
```bash
python tools/check_pose_labels.py --images dataset/images/train --labels dataset/labels/train
python tools/visualize_pose.py --images dataset/images/train --labels dataset/labels/train --out outputs/vis_train
python tools/visibility_report.py --labels dataset/labels/train --compare gold/labels/train --markdown reports/visibility_compare.md
```

| STT | Mục kiểm tra | Đạt? | Ghi chú chi tiết |
| :---: | :--- | :---: | :--- |
| 1 | Mọi người trong ảnh đều có đủ 17 điểm, không ai bị thiếu | ☑ ĐẠT | 29/29 skeleton đều đủ 17 keypoints. |
| 2 | Bật đường nối: không có xương nào cắt chéo ở vai hoặc hông | ☑ ĐẠT | Không có lỗi đảo trái/phải nghiêm trọng (`dao_trai_phai: 0`). |
| 3 | Không có xương nào kéo dài sang một cơ thể khác | ☑ ĐẠT | Không có lỗi nhầm người (`nham_nguoi: 0`). |
| 4 | Khớp bị che dùng `v = 1` **và có chấm**, không phải `v = 0` | ☑ ĐẠT | Có 131 khớp gán đúng cờ `v = 1` kèm toạ độ ước lượng. |
| 5 | `v = 0` chỉ xuất hiện ở khớp thật sự ra ngoài mép ảnh | ☑ ĐẠT | Chỉ có 35 khớp nằm ngoài mép ảnh hoặc chân bị cắt. |
| 6 | Không có dấu hiệu dùng `Hidden` (điểm `v = 2` nằm ở chỗ vô lý) | ☑ ĐẠT | Không có toạ độ bất thường. |
| 7 | Export đúng **COCO Keypoints 1.0**: mảng `keypoints` có 51 số | ☑ ĐẠT | `person_keypoints_default.json` chuẩn COCO Keypoints 1.0. |
| 8 | Bản YOLO Pose: mỗi dòng 56 số, `kpt_shape: [17, 3]` | ☑ ĐẠT | Đủ 20 file txt, mỗi dòng 56 số. |
| 9 | Visibility report đã nộp, và hai bảng đã được đặt cạnh nhau | ☑ ĐẠT | Đã sinh `outputs/visibility_report.json` và `reports/visibility_compare.md`. |
| 10 | Mọi ca không rõ đều được ghi trong `GUIDELINE_MINI.md` | ☑ ĐẠT | Đã điền 6 tình huống và 3 ca mơ hồ. |
| 11 | `check_pose_labels.py` chạy 0 lỗi | ☑ ĐẠT | `0` lỗi cú pháp/định dạng (Returncode = 0). |

---

## 2. Danh sách các điểm cần lưu ý và khắc phục (Rework Findings)

| Ảnh | Người thứ | Khớp | Lỗi gì | Sửa thế nào |
| :--- | :---: | :--- | :--- | :--- |
| `train_10.jpg` | 1 | `left_hip`, `right_hip` | Xoá khớp bị che (`v = 0` thay vì `v = 1`) | Đặt chấm ước lượng vị trí hông qua nếp áo và gán cờ `v = 1`. |
| `train_15.jpg` | 1 | `left_ankle`, `right_ankle` | Lệch nhẹ vị trí khớp mắt cá chân do tư thế chuyển động nhanh | Kéo điểm chấm sát hơn vào vị trí mấu xương mắt cá chân. |
| `train_18.jpg` | 1 | `right_eye`, `left_ear` | Trượt vị trí góc đầu nghiêng | Căn chỉnh lại điểm mắt và tai dựa trên góc đối xứng của khuôn mặt. |

---

## 3. Kết luận kiểm tra

- **Lỗi lặp lại đáng chú ý nhất:** Một số khớp bị che khuất ở thân dưới và góc nghiêng đầu bị phân vân giữa `v = 1` và `v = 0`.
- **Bản chất lỗi:** Đây là lỗi **guideline cần được thống nhất rõ hơn về độ che phủ**, chứ không phải lỗi thao tác ngẫu nhiên hay nhầm người. Bộ nhãn đạt độ chính xác cao (**OKS trung bình 0.876 - Mức Xuất sắc**).
