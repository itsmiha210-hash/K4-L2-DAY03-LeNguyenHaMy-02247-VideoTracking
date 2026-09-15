# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Nguyễn Hà My - MSSV: 02247`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Không gán các vật thể tĩnh ven đường như quầy hàng, ki-ốt, trạm biến áp, xe phế liệu/xe mô hình trưng bày không tham gia giao thông.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Trong giao thông đô thị, xe bị che khuất thoáng qua sau xe khác hoặc cột đèn rồi hiện lại thì vẫn là cùng một thực thể phương tiện; giữ ID giúp tracking model học đúng tính liên tục danh tính (identity continuity). |
| Xe bị che lâu hơn ngưỡng trên | Ngắt track và gán track mới với ID mới khi xuất hiện lại | Nếu bị che quá 25 frame (>2 giây), độ bất định về vị trí và khả năng đổi hướng hoặc rẽ nhánh rất cao; gộp chung ID dễ gây sai lệch lớn về quỹ đạo di chuyển (trajectory). |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Khi xe đã hoàn toàn rời khỏi trường nhìn của camera (FOV), không có bằng chứng liên tục để khẳng định danh tính nếu không có hệ thống ReID toàn cảnh. |
| Hai xe cắt nhau / chồng lên nhau | Duy trì độc lập 2 ID cho 2 xe, tuyệt đối không hoán đổi ID | Xe ở phía trước giữ bbox bình thường; xe ở phía sau chỉ vẽ bbox phần nhìn thấy được (visible mask), giữ nguyên ID của từng xe qua suốt giao cắt. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, tuyệt đối không vẽ ước lượng phần thân xe ngoài khung hình. |
| Xe bị xe khác che một phần | Bbox chỉ ôm phần **nhìn thấy được** (visible bounding box), không vẽ trùm lên phần bị che. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: cạnh tối thiểu > 15 pixel và nhận diện được kết cấu bánh xe/đèn xe. |
| Xe đang đỗ, không di chuyển | Gán 1 track ID duy nhất từ frame 1 đến hết clip; cố định kích thước và tọa độ box sau khi nắn chuẩn ở keyframe đầu. |
| Keyframe đặt dày ở đâu | Đặt dày (cách nhau 2–3 frame) ở: (1) Frame xe bắt đầu xuất hiện hoặc chuẩn bị ra khỏi ảnh; (2) Đoạn xe đổi hướng, vào cua hoặc thay đổi vận tốc đột ngột; (3) Đoạn xảy ra che khuất (occlusion). |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 85–100 / ID 5`
- Tình huống: Xe con Track 5 bám đuôi xe buýt Track 4, nửa trước của xe 5 bị đuôi xe buýt che khuất trong nhiều frame liên tiếp.
- Quyết định: Tiếp tục giữ nguyên ID 5, chỉ vẽ bbox ôm phần đuôi và thân xe nhìn thấy được (visible mask).
- Lý do: Xe vẫn xuất hiện liên tục và di chuyển cùng chiều với dòng xe; việc giữ nguyên ID giúp mô hình đánh giá đúng khả năng tracking khi có occlusion nhẹ.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 106–110 / ID 7`
- Tình huống: Xe tải lớn Track 7 bắt đầu xuất hiện từ góc bên phải, chỉ nhô ra phần đầu xe và bánh trước.
- Quyết định: Bắt đầu tạo track ID 7 ngay từ frame 106 khi mũi xe vừa chạm rìa ảnh (>15px), bbox chạm sát mép ảnh và đặt keyframe dày ở frame 106, 108, 109.
- Lý do: Đã xác định rõ hình khối xe tải, việc đặt keyframe dày ở những frame đầu giúp tránh lỗi nội suy tuyến tính làm trôi lệch tâm bbox khi xe đang tăng tốc vào làn.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 16–116 / (vật thể tĩnh bên lề)`
- Tình huống: Một vật thể tĩnh bên lề đường bên trái (gần khu vực xe đỗ Track 3) có khối hình chữ nhật và phản quang giống đuôi xe.
- Quyết định: Không gán nhãn cho vật thể này (mặc dù tracker AI ByteTrack và ReID đều bắt nhầm thành ID 10 và ID 7 suốt hàng chục frame).
- Lý do: Quan sát toàn bộ video thấy đây là chướng ngại vật/quầy ki-ốt cố định ven đường, không phải phương tiện xe 4 bánh theo phạm vi của bài lab.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Bắt buộc đặt keyframe cách nhau tối đa 2–3 frame ngay khi xe bắt đầu ló dạng vào khung hình (entry) hoặc sắp rời khung hình (exit) để triệt tiêu hoàn toàn lỗi trôi nội suy (interpolation drift, IoU < 0.60).
- Chỉ ôm phần pixel nhìn thấy, không đoán phần thân xe bị che khuất quá 20%. Đồng thời quy định rõ: Nếu phần nhìn thấy bị đứt đoạn thành 2 mảng nhỏ, bbox chỉ bao trùm mảng nhìn thấy lớn nhất.
