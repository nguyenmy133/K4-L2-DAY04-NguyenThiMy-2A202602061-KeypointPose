# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Nguyễn Thị My   |   Nhóm: Team AI   |   Ngày: 16/09/2026

---

## 1. Nhãn của tôi

<!-- Lấy số từ reports/visibility_report.md hoặc outputs/visibility_report.json sau Chặng 4. -->

| Chỉ số | Giá trị |
| :--- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 29 |
| v=2 / v=1 / v=0 | 327 / 131 / 35 |
| Thời gian trung bình mỗi ảnh | ~4.5 phút / ảnh |

**Ba khớp có `%v=1` cao nhất (lấy từ `reports/visibility_report.md`):**

1. `left_ear` (66% - 19/29 người bị che)
2. `right_ear` (48% - 14/29 người bị che)
3. `left_hip` (34% - 10/29 người bị che) & `right_hip` / `right_knee` (31%)

**Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích:**
- **Không hoàn toàn trùng nhau.** Cần phân biệt rõ giữa *“khớp hay bị che khuất”* và *“khớp khó xác định vị trí giải phẫu”*:
  - **Khớp tai (`left_ear`, `right_ear`):** Rất hay bị tóc dài, nón bảo hiểm hoặc góc nghiêng đầu che khuất ($\%v=1$ cao nhất), nhưng về mặt giải phẫu lại rất **dễ định vị** vị trí do đối xứng với tai còn lại và mắt.
  - **Khớp hông (`left_hip`, `right_hip`):** Mới thực sự là khớp **khó gán nhất** vì trên thực tế không có người nào mặc quần áo mà lộ bề mặt khớp hông. Người gán phải dựa vào kinh nghiệm giải phẫu và các mốc tham chiếu phụ (đường thắt lưng, nếp gấp đùi) để ước lượng tâm xoay mấu chuyển lớn xương đùi.

---

## 2. Chấm với gold

<!-- Lấy từ outputs/eval_vs_gold.json -->

| Chỉ số | Trước rework | Sau rework |
| :--- | ---: | ---: |
| OKS trung bình | 0.876 | 0.912 |
| OKS@0.50 | 1.000 | 1.000 |
| OKS@0.75 | 0.931 | 0.965 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 0 | 0 |
| Lỗi `xoa_khop_bi_che` | 2 | 0 |

**Tôi đã sửa gì giữa hai lần chạy:**
- `train_10.jpg` + người thứ 1 + `left_hip` & `right_hip`: Đổi từ `v = 0` sang `v = 1` và đặt chấm ước lượng tại vị trí khớp hông sau vạt áo.
- `train_18.jpg` + người thứ 1 + `right_eye` & `left_ear`: Căn chỉnh lại toạ độ chấm mắt phải và tai trái vào đúng mốc đối xứng của hộp sọ khi đầu nghiêng.
- `train_15.jpg` + người thứ 1 + `left_ankle` & `right_ankle`: Tinh chỉnh toạ độ khớp mắt cá chân sát hơn với trục cẳng chân khi vận động thể thao.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?**
- **Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh.** Nhờ tuân thủ nguyên tắc kiểm tra hướng mắt trước khi gán và soi lại trên `tools/visualize_pose.py` (màu xanh bên trái, màu cam bên phải không bị cắt chéo).

---

## 3. Kiểm chéo

Bạn cùng nhóm / Dữ liệu đối chiếu: **COCO Gold Reference & Partner Audit**

**Khớp lệch `%v=1` nhiều nhất giữa hai bảng đếm:**

| Khớp | Bạn (%v=1) | Đối chiếu Gold (%v=1) | Lệch (%) | Nguyên nhân (guideline hay gán sai?) |
| :--- | ---: | ---: | ---: | :--- |
| `left_ear` | 66% | 3% | 62% | **Guideline khác nhau:** COCO Gold dùng `v=0` cho ca không nhìn thấy; Guideline lớp yêu cầu giữ `v=1` và ước lượng vị trí. |
| `right_ear` | 48% | 14% | 34% | **Guideline khác nhau:** Tai bị tóc/góc quay che khuất một phần. |
| `right_eye` | 28% | 0% | 28% | **Guideline khác nhau:** Mắt nghiêng góc khuất được lớp quy ước `v=1`. |
| `right_knee` | 31% | 14% | 17% | **Guideline chưa rõ:** Chân gập sau người khi ngồi. |

**Luật mới đã bổ sung vào `GUIDELINE_MINI.md` sau khi thống nhất:**
- Khớp đầu/mặt (mắt, mũi, tai) hoặc khớp thân (vai, hông) bị che một phần bởi tóc, quần áo hay góc nghiêng cơ thể nhưng còn nằm trong giới hạn khung hình thì **bắt buộc đánh dấu `v = 1` và đặt chấm toạ độ ước lượng**, không được xoá thành `v = 0`.

---

## 4. Model & Phân tích huấn luyện

<!-- Kết quả thực tế từ file outputs/eval_model.json sau khi chạy Colab Notebook -->

| Chỉ số | yolo26n-pose gốc | Sau fine-tune (20 ảnh) | Chênh |
| :--- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook:

1. **`pose_mAP50-95` thay đổi bao nhiêu? Nếu nó thay đổi, hãy giải thích:**
   - Sau fine-tune trên 20 ảnh gán chất lượng cao (OKS 0.876), `pose_mAP50-95` **tăng từ 0.6853 lên 0.6908 (+0.0055, tương đương +0.55%)**, đồng thời độ chính xác `pose_precision` **tăng từ 0.9734 lên 0.9792 (+0.58%)**.
   - Điều này chứng minh: Dù chỉ với 20 ảnh huấn luyện, nhãn gán đúng chuẩn (không bị đảo trái/phải, cờ `v=1` chuẩn xác) đã giúp mô hình học thêm được các đặc trưng tư thế phức tạp và định vị khớp chính xác hơn so với mô hình gốc.
2. **`box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm *khớp* dễ hơn? Vì sao?**
   - `box_mAP50-95` (0.8041) cao hơn đáng kể so với `pose_mAP50-95` (0.6908), chênh lệch hơn 0.11 (11%).
   - Model tìm **người (bounding box) dễ hơn nhiều so với tìm khớp (keypoints)** vì bounding box bao quát toàn bộ thể tích đối tượng với nhiều đặc trưng bề mặt, trong khi keypoint là các điểm đơn lẻ cục bộ, rất dễ bị che khuất hoặc biến dạng lớn khi đổi tư thế.
3. **Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43:**
   - Ở ảnh `test_06.jpg` (người ngồi với tư thế chân bị khuất), mô hình gặp lỗi **"Lệch nhẹ" (Slight offset)** ở khớp cổ chân do vật cản phía trước.
4. **Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?**
   - Ảnh `train_15.jpg` có OKS thấp nhất giữa nhãn gán và model dự đoán. **Người gán đúng hơn model** vì người đang vận động thể thao mạnh, mô hình bị trượt khớp sang vị trí lân cận, trong khi người gán xác định đúng trục liên kết giải phẫu của xương.
5. **Ảnh bạn gán tệ nhất có cũng là ảnh model đoán tệ nhất không? Nếu có, điều đó nói gì về bức ảnh đó?**
   - Đúng, cả người gán và model đều có độ tự tin/OKS thấp nhất ở `train_15.jpg` và `train_10.jpg`. Điều này chứng tỏ đây là các **ảnh khó thực sự (hard samples)** với nhiều điểm che khuất và tư thế góc gập hẹp.

---

## 5. Một rule evidence bạn đã dùng

- **Ảnh & Người & Khớp:** `train_10.jpg`, người thứ 1, khớp `left_hip` (hông trái).
- **Căn cứ thị giác (Visual Evidence):** Người ngồi trên ghế lái, áo khoác phủ xuống qua thắt lưng che khuất hoàn toàn hông, nhưng phần thân trên (cột sống, vai) và đùi trên vẫn nhìn thấy rõ phương hướng.
- **Lý do quyết định trạng thái (`v = 1` thay vì `v = 0`):** Toàn bộ cơ thể người nằm gọn bên trong khung ảnh (không hề bị cắt mép). Khớp hông chỉ bị che bởi vạt áo nên bắt buộc phải chọn **`v = 1` (Occluded)** và đặt chấm toạ độ ước lượng tại giao điểm giữa trục thân trên và trục xương đùi, thay vì để `v = 0` (Outside).
