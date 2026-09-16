# Mini guideline - nhóm: Team AI  |  người gán: Nguyễn Thị My  |  ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn (đã thống nhất)

| Tình huống | Luật nhóm bạn chọn | Vì sao |
| --- | --- | --- |
| **Hông của người mặc quần áo dài** | Ước lượng vị trí khớp chậu/mấu chuyển lớn của xương đùi dựa trên nếp gấp hông hoặc thắt lưng; gắn cờ `v = 1`. | Khớp hông không có bề mặt nhìn thấy trực tiếp khi mặc đồ, sigma của COCO cho hông khá lớn (0.107) nên chấp nhận ước lượng giải phẫu. |
| **Tai bị tóc hoặc mũ bảo hiểm che một phần** | Nếu vẫn định vị được gốc tai/vị trí giải phẫu $\rightarrow$ đặt chấm và gán `v = 1` (Occluded). Chỉ để `v = 0` nếu đầu bị cắt khỏi khung hình. | Giữ nguyên cờ `v = 1` giúp mô hình học được mối tương quan giữa mắt - tai - vai kể cả khi có vật che nhẹ. |
| **Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên)** | Các khớp dưới (gối, cổ chân) nằm ngoài mép ảnh hoàn toàn $\rightarrow$ gán `v = 0` (Outside), toạ độ để `(0, 0)`. | Theo chuẩn COCO và YOLO Pose, khớp ra ngoài rìa khung ảnh không thể ước lượng chính xác nên phải loại khỏi loss. |
| **Cổ tay nằm sau tay lái / sau thân mình** | Dựa vào hướng của cẳng tay và bàn tay để nội suy vị trí cổ tay; đặt chấm và gán `v = 1`. | Cẳng tay định hướng rõ ràng vị trí khớp cổ tay, giúp mô hình duy trì tính liên tục của chuỗi xương cánh tay. |
| **Hai người chồng lên nhau** | Gán trọn vẹn từng người một; khớp của người sau bị người trước che $\rightarrow$ đặt chấm ước lượng và gán `v = 1`. Tuyệt đối không kéo điểm sang cơ thể người trước. | Tránh lỗi nghiêm trọng "nhầm người" (`nham_nguoi`) làm sai lệch liên kết pose giữa 2 cá thể liền kề. |
| **Người nhỏ đến mức nào thì không gán nữa** | Gán toàn bộ người nhìn rõ cấu trúc thân (chiều cao box $\ge 40$ pixel). Những người nền quá mờ/nhỏ ở hậu cảnh xa không có cấu trúc xương thì bỏ qua. | Đảm bảo tập dữ liệu tập trung vào các đối tượng đủ độ phân giải để mô hình học keypoint chính xác. |

---

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_10.jpg`, người thứ `1`, khớp `left_hip` và `right_hip`

- **Mơ hồ ở chỗ nào:** Người ngồi trên phương tiện/ghế, phần hông và đùi bị che khuất bởi áo khoác dài và vật cản phía trước.
- **Bạn quyết thế nào:** Ước lượng vị trí hai bên khớp hông dựa trên trục cột sống và hướng thân trên, đánh dấu `v = 1` (Occluded).
- **Vì sao:** Người vẫn nằm trọn trong khung hình, việc gán `v = 1` đúng với quy tắc của lớp (bị che nhưng còn trong khung) thay vì xóa điểm (`v = 0`).
- **Nếu người khác quyết ngược lại (để `v = 0`):** Mô hình sẽ học sai rằng người này bị cụt thân dưới hoặc mất liên kết giữa thân trên và chi dưới khi bị che khuất.

### Ca 2 - ảnh `train_18.jpg`, người thứ `1`, khớp `left_ear` và `right_eye`

- **Mơ hồ ở chỗ nào:** Người đứng nghiêng góc 3/4 quay lưng lại máy ảnh, tai trái và mắt phải bị che khuất một phần bởi góc nghiêng đầu và mái tóc.
- **Bạn quyết thế nào:** Định vị góc đối xứng của đầu dựa trên mắt trái và mũi để ước lượng vị trí mắt phải và tai trái, đặt chấm và gán `v = 1`.
- **Vì sao:** Cấu trúc khuôn mặt có tính đối xứng cao, việc ước lượng điểm che giúp mô hình học góc xoay đầu (head pose).
- **Nếu người khác quyết ngược lại (chấm ẩu không cờ `v = 2` hoặc xóa `v = 0`):** Nếu để `v = 2` mô hình sẽ bị phạt vì đoán sai đặc trưng nhìn thấy; nếu để `v = 0` mô hình mất thông tin về cấu trúc hộp sọ 3D.

### Ca 3 - ảnh `train_15.jpg`, người thứ `1`, khớp `left_knee` và `right_ankle`

- **Mơ hồ ở chỗ nào:** Người đang vận động thể thao với tư thế uốn người gập chân nhanh, chân chuyển động tạo góc khuất sau thân mình.
- **Bạn quyết thế nào:** Dựa trên hướng của đùi và cẳng chân để xác định tâm xoay của khớp gối và cổ chân, đánh dấu `v = 1`.
- **Vì sao:** Toàn bộ cơ thể người nằm trong ảnh, các khớp bị thân che cục bộ phải duy trì `v = 1` và có chấm toạ độ.
- **Nếu người khác quyết ngược lại:** Dẫn đến đứt đoạn khung xương (broken skeleton), mô hình không bắt được các tư thế thể thao phức tạp.

---

## 4. Sau khi so visibility report với bộ Gold / Bạn cùng nhóm

- **Khớp lệch `%v=1` nhiều nhất:** `left_ear` (bài của bạn `66%` / đối chiếu gold `3%`, lệch `62%`) và `right_ear` (bài của bạn `48%` / đối chiếu gold `14%`, lệch `34%`).
- **Nguyên nhân là guideline chưa rõ hay một trong hai bên gán sai:** 
  - Đây là sự khác biệt có chủ đích giữa **Guideline của lớp** và **COCO Gold gốc**. 
  - COCO sử dụng `v = 0` cho cả trường hợp "không gán" và "ngoài khung", trong khi lớp yêu cầu chặt chẽ: tai bị tóc/nón che khuất nhưng còn trong ảnh phải gán `v = 1` và ước lượng vị trí.
- **Luật mới bổ sung vào mục 2 sau khi thống nhất:**
  - Khẳng định rõ: Bất kỳ khớp đầu/mặt nào (mắt, tai, mũi) bị tóc hoặc góc quay che khuất một phần nhưng đầu còn trong ảnh thì **100% gán `v = 1` kèm chấm toạ độ ước lượng**, không được để `v = 0`.
