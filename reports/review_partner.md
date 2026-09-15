# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Lê Nguyễn Hà My` |
| Reviewer | `Nguyễn Văn A (Peer Reviewer)` |
| Pair ID | `Pair-K4-03` |
| CVAT version | `2.45.0` |
| Thời điểm review | `2026-09-15 11:30` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | ---: | --- | --- | --- |
| 1 | 10 | 11 | 1 | Exit boundary | Xe con rẽ trái ra khỏi khung hình, tại frame 11 (CVAT frame 10) mép xe đã chạm rìa, cần bấm `outside: true` ở frame tiếp theo (CVAT frame 11) để tránh box treo. | Đánh dấu `outside: true` tại CVAT frame 11. | `fixed` |
| 2 | 107 | 108 | 7 | Interpolation drift | Xe tải Track 7 vừa xuất hiện ở mép phải, bbox ở frame giữa hai keyframe (CVAT 105 và 111) bị trôi nhẹ về phía sau so với mũi xe tải (IoU ~0.54). | Thêm keyframe tại CVAT frame 107 (MOT frame 108) để nắn bbox khít mũi xe. | `fixed` |
| 3 | 86 | 87 | 5 | Occlusion bbox | Xe con Track 5 bám sau xe buýt Track 4, thân xe bị che khuất một phần. Reviewer kiểm tra xem có bị vẽ trùm lên xe buýt hay không. Tác giả đã vẽ đúng visible mask và giữ đúng ID 5 theo rule che < 25 frame. | Giữ nguyên quy tắc chỉ gán visible bounding box và duy trì ID 5. | `not-a-defect` |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đủ 8 track hợp lệ (`vehicle`), không lẫn xe máy/người đi bộ |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | 8 xe ứng với 8 ID duy nhất (1–8), không trùng lặp |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Track 5 giữ nguyên ID khi bị Track 4 che ở frame 85–100 |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Finding 1 đã sửa, các track đều bấm `outside` chuẩn |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Track 5 và Track 7 đều cắt sát mép nhìn thấy thực tế |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Finding 2 đã sửa, thêm keyframe nắn chuẩn hình học |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `annotations/clip_01/gt.txt` định dạng chuẩn MOT 1.1 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Cả 3 finding đều có ghi nhận và đóng closure rõ ràng |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Rà soát timeline ID 1–8 trên CVAT, ID liên tục không ngắt quãng |
| 2 — endpoint/scope | ĐÃ SỬA | Kiểm tra frame đầu/cuối của Track 1, 2, 4, 8; đã chốt outside chuẩn |
| 3 — geometry/interpolation | ĐÃ SỬA | Bổ sung keyframe tại các frame có gia tốc thay đổi (Track 5, 7) |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Quy tắc đặt outside ngay khi xe rời khung hình (Rule exit boundary) và bổ sung keyframe ở vùng xe mới xuất hiện để triệt tiêu lỗi trôi bbox do nội suy tuyến tính.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Finding 3 về Track 5: Tác giả duy trì ID 5 và chỉ gán bbox phần nhìn thấy được (visible mask) khi bị xe buýt che là hoàn toàn chính xác theo GUIDELINE_MINI.md.`
3. Một rule cần Lab Coach làm rõ (nếu có): `Quy định cụ thể hơn về độ dài tối thiểu (số frame) của một track xe ở rìa ảnh trước khi được tính là một track hợp lệ.`
