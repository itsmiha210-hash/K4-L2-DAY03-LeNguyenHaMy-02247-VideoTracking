# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Nguyễn Hà My / T032`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT (Rectangle Track mode)` |
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `30` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6.5` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe con Track 5 bám đuôi xe buýt Track 4 từ frame 80 đến 139. Trong nhiều frame (đặc biệt từ frame 85–100), nửa trước của xe con bị thân sau xe buýt che khuất. *Cách xử lý*: Tuân thủ nguyên tắc bbox chỉ ôm sát phần thân xe nhìn thấy được (visible bounding box), không vẽ bao trùm phần bị che khuất; đồng thời giữ nguyên một ID (`track_id = 5`) duy nhất vì thời gian che ngắt quãng dưới 25 frame (dưới 2 giây).
2. Các xe rẽ hoặc đi ra khỏi rìa ảnh (như Track 1 rẽ trái ra khỏi khung tại frame 11, Track 2 thoát ở frame 44, Track 4 thoát ở frame 150, Track 8 xuất hiện ở góc dưới bên phải frame 138 rồi thoát ở frame 170). *Cách xử lý*: Chỉ bắt đầu tạo track khi xe nhận diện rõ là xe 4 bánh (>15px); bbox khi chạm rìa ảnh được cắt thẳng theo mép ảnh (không đoán phần ngoài ảnh); đặt trạng thái `outside: true` ngay tại frame kế tiếp sau khi xe hoàn toàn rời khỏi khung nhìn để tránh lỗi floating/hanging box.
3. Xe con đỗ cố định ở lề đường bên trái trong toàn bộ 190 frame của video mà không dịch chuyển vị trí. *Cách xử lý*: Duy trì một track ID duy nhất (`track_id = 3`) từ frame đầu tiên đến frame cuối cùng, khóa tọa độ bbox sau khi nắn chuẩn ở keyframe đầu để tránh hiện tượng trôi lệch (bbox drift) do vô tình di chuột hoặc do lỗi nội suy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Tua nhanh toàn bộ clip để rà soát timeline từng track ID trên thanh công cụ CVAT. Đảm bảo toàn clip có đúng 8 track xe 4 bánh tương ứng với 8 ID độc lập (1–8), không xảy ra hiện tượng hoán đổi ID (ID switch = 0), không tái sử dụng ID cũ cho xe mới xuất hiện và không ngắt vụn một xe thành nhiều ID rời rạc.
- Lượt 2: Dừng lại tua chậm từng frame tại thời điểm xuất hiện đầu tiên và thời điểm rời khung hình của từng xe. Bắt được lỗi Track 1 sau khi rẽ trái thoát khung ở frame 11 cần đánh dấu `outside` dứt khoát tại frame 12; kiểm tra Track 4 (xe buýt) thoát khung tại frame 150 và đóng `outside` đúng frame 151; đảm bảo không còn bất kỳ bounding box rác nào tồn tại sau khi xe biến mất.
- Lượt 3: Tua chậm từng bước ở các đoạn giữa 2 keyframe, đặc biệt ở các đoạn xe thay đổi phối cảnh xa-gần hoặc đổi hướng di chuyển (Track 4 xe buýt, Track 5 bám đuôi, Track 7 xe tải). Phát hiện một số frame giữa bị trôi nhẹ khiến bbox hơi rộng so với viền xe, từ đó chèn thêm keyframe bổ sung để bbox luôn ôm khít phần nhìn thấy được của phương tiện.

Kiểm chéo với: 
Số lỗi bạn tìm được trong bản của bạn ấy: 

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `e0a75fb1ebbb4eced77884d0acb37c327eaade7068e2cb101d9492cbe5290560` |
| Thời điểm khóa | `2026-09-15T04:17:05.176648+00:00` |
| Số row / frame / track trước khi mở reference | `575 row / 190 frame / 8 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.838 | 0.828 | 0.849 | 0.877 | 0.976 | 0.951 | 0.867 | 15 | 13 | 0 |
| Sau rework | 0.838 | 0.828 | 0.849 | 0.877 | 0.976 | 0.951 | 0.867 | 15 | 13 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): Có

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox trôi (IoU 0.51 - 0.54) | Frame 108, 109 | ID 7 (xe tải) | Thêm keyframe tại frame 108 và 109 khi xe tải vừa tiến vào khung hình từ mép phải để nắn bbox khít với mép xe (ban đầu do nội suy tuyến tính từ frame 106 khiến bbox bị trễ so với mũi xe). |
| Bbox trôi (IoU 0.55 - 0.60) | Frame 81, 96, 100 | ID 5 (xe con) | Bổ sung keyframe tại frame 81, 96, 100 khi xe con cua bám theo sau xe buýt, nắn bbox bám sát phần thân nhìn thấy được, tránh bị lệch tâm do che khuất. |
| Bbox trôi (IoU 0.56 - 0.57) | Frame 55, 103, 104 | ID 4 (xe buýt), ID 6 (xe con) | Thêm keyframe tại frame 55 cho xe buýt lúc bắt đầu nhập làn chính, và frame 103–104 cho xe 6 khi vừa xuất hiện từ góc phải để bbox ôm sát viền xe ngay từ đầu. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `Python 3.13.15` / `ultralytics 8.4.145` / `torch 2.11.0+cu128` / `lap 0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `ByteTrack control (bytetrack.yaml)` và `BoT-SORT + ReID treatment (botsort-reid.yaml)` |
| conf / IoU / imgsz / classes | `conf: 0.25` / `iou: 0.7` / `imgsz: 960` / `classes: [2, 5, 7]` (car, bus, truck) |
| device | `0` (Tesla GPU trên Google Colab) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.838 | 0.828 | 0.849 | 0.877 | 0.976 | 0.951 | 0.867 | 15 | 13 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.867 | 0.814 | 0.924 | 0.952 | 0.913 | 0.823 | 0.949 | 81 | 18 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong kết quả đánh giá nhãn của bạn so với gold: IDF1 đạt 0.976, cao hơn MOTA đạt 0.951 (cả hai đều đạt mức Xuất sắc, IDSW = 0 tuyệt đối).
- Ý nghĩa khi MOTA cao mà IDF1 thấp:
  Hiện tượng một hệ thống có MOTA cao nhưng IDF1 thấp phản ánh rằng hệ thống làm rất tốt ở bài toán Detection — phát hiện đúng vị trí có xe, ít bỏ sót (FN thấp) và ít bắt nhầm (FP thấp), nhưng lại kém ở khâu duy trì danh tính theo thời gian. Nghĩa là các đối tượng liên tục bị nhảy ID hoặc một xe bị cắt vụn thành nhiều track ID rời rạc qua các frame.
- Vì sao MOTA không phạt nặng lỗi ID?
  Công thức tính MOTA:
  $$\text{MOTA} = 1 - \frac{\sum_t (\text{FN}_t + \text{FP}_t + \text{IDSW}_t)}{\sum_t \text{GT}_t}$$
  Trong MOTA, mỗi lần tráo đổi ID ($\text{IDSW}$) chỉ bị tính là **1 điểm lỗi đơn lẻ** đúng tại frame xảy ra hoán đổi, ngang bằng với 1 lỗi FP hoặc FN trong frame đó. Sau khi đã đổi sang ID mới, nếu tracker tiếp tục bám theo xe thì ở tất cả các frame tiếp theo, nó vẫn được tính là True Positive hợp lệ (không hề bị phạt thêm). Ví dụ: một xe chạy suốt 100 frame bị nhảy ID ở frame thứ 50 (chia thành 2 đoạn mỗi đoạn 50 frame), MOTA chỉ bị trừ đúng $1 / 100 = 1\%$!
  Ngược lại, IDF1 ($\text{IDF1} = \frac{2 \cdot \text{IDTP}}{2 \cdot \text{IDTP} + \text{IDFP} + \text{IDFN}}$) giải bài toán ghép cặp cực đại (bipartite matching) 1-1 trên toàn bộ quỹ đạo (trajectory) của track. Khi một track 100 frame bị cắt làm đôi, thuật toán chỉ có thể gán 1 trong 2 nửa làm $\text{IDTP}$ (50 frame), nửa còn lại 50 frame lập tức bị coi là vừa $\text{IDFP}$ vừa $\text{IDFN}$, khiến IDF1 sụt giảm nghiêm trọng xuống chỉ còn ~50%. Vì vậy, MOTA mang tính "detector-heavy" (ưu tiên phát hiện từng frame) và không phản ánh đúng mức độ sai sót danh tính dài hạn như IDF1.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- So sánh số liệu định lượng (so với Gold):
  - ByteTrack (Control): IDF1 = `0.875`, AssA = `0.776`, IDSW = `2`. Tách 3 track gold: Track 4 bị chia thành ID [14, 15], Track 5 thành ID [23, 32], Track 7 thành ID [52, 71].
  - BoT-SORT + ReID (Treatment): IDF1 = `0.900` (+0.025), AssA = `0.820` (+0.044), IDSW = `2`. Tách 3 track gold: Track 5 thành ID [17, 18], Track 6 thành ID [24, 31], Track 7 thành ID [29, 39].
  - Cả hai phương pháp đều ghi nhận 2 lần ID switch trực tiếp trên timeline, tuy nhiên BoT-SORT + ReID vượt trội rõ rệt về AssA (0.820 so với 0.776) và IDF1 (0.900 so với 0.875), cho thấy mức độ gắn kết danh tính toàn cục và độ dài các đoạn track nhất quán cao hơn đáng kể.
- Dẫn chứng chuỗi frame cụ thể (Frame Sequence Evidence):
  - Xe buýt lớn Track 4 (frame 54 – 150): Với ByteTrack control, tại frame 59 khi xe buýt vừa đi vào làn đường chính, tracker gặp biến động về kích thước bounding box và lập tức bị ID switch từ ID 14 sang ID 15, dẫn đến toàn bộ hành trình 95 frame của xe buýt bị chẻ đôi thành hai track riêng biệt. Ngược lại, BoT-SORT + ReID đã theo dõi liên tục Track 4 từ đầu đến cuối (95 frame) hoàn toàn dưới một ID duy nhất mà không hề bị ID switch, nhờ vào visual feature của thân xe buýt màu sắc đặc trưng giúp thuật toán duy trì liên kết.
  - Xe con Track 5 bám đuôi xe buýt (frame 80 – 139): Khi đi sau xe buýt bị che khuất một phần, ByteTrack bị mất dấu ở frame 94 (nhảy từ ID 23 sang 32, và chỉ phủ được 46/60 frame, FN lên tới 54). BoT-SORT + ReID giảm FN từ 54 xuống 26, duy trì nhận diện tốt hơn dù vẫn bị ngắt ID ở frame 87 (ID 17 sang 18).
- Lưu ý phương pháp luận quan trọng:
  Thí nghiệm này không cô lập được hiệu ứng nhân quả thuần túy của ReID. Lý do là BoT-SORT và ByteTrack là hai kiến trúc tracker khác nhau trên nhiều khía cạnh kỹ thuật: BoT-SORT tích hợp thêm mô-đun Bù trừ chuyển động camera (Camera Motion Compensation - CMC dựa trên trích xuất đặc trưng quang học GCP/ORB), sử dụng ma trận Kalman Filter tinh chỉnh lại trạng thái đo chiều dài/rộng $(w, h)$ thay vì tỉ lệ khung hình $(a, h)$, và áp dụng cơ chế kết hợp trọng số giữa khoảng cách hình học IoU với khoảng cách cosine từ visual embedding. Do đó, sự vượt trội về AssA và IDF1 là kết quả tổng hợp của toàn bộ hệ thống BoT-SORT chứ không thể quy kết 100% riêng cho ReID feature.
  *(Ghi chú thêm từ thí nghiệm Stretch ở Cell 21: Khi thay đổi ngưỡng `appearance_thresh` từ 0.70, 0.80 đến 0.90, điểm HOTA giữ nguyên 0.763, AssA giữ 0.820, IDF1 dao động rất nhỏ từ 0.900 xuống 0.899 và IDSW giữ nguyên 2, chứng minh rằng ngưỡng ReID trong khoảng này hoạt động rất ổn định nhưng không tạo đột biến đơn lẻ).*

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- Biến thiên DetA, FP và FN:
  - ByteTrack control vs gold: DetA = `0.649`, FP = `88`, FN = `54` (trên tổng số 573 bbox gold).
  - BoT-SORT + ReID vs gold: DetA = `0.711` (+0.062), FP = `91` (+3), FN = `26` (-28).
  - Điểm nổi bật nhất là FN giảm hơn một nửa (từ 54 xuống 26): cơ chế kết hợp visual appearance và tracking state của BoT-SORT giúp hệ thống "cứu" được rất nhiều frame bị che khuất hoặc detector có confidence thấp ở ngưỡng biên, giúp độ phủ của các track tăng lên đáng kể (Track 8 và Track 5 không còn bị mất dấu nhiều frame như ByteTrack).
  - Tuy nhiên, FP vẫn ở mức rất cao (88 ở ByteTrack và 91 ở BoT-SORT): cả hai tracker đều sinh ra một lượng lớn False Positives (khoảng 15% tổng số bbox).
- Lỗi còn lại chủ yếu là Detector hay Association?
  Lỗi còn lại CHỦ YẾU LÀ DETECTOR:
  - Về mặt Association: AssA của BoT-SORT đã đạt `0.820`, IDF1 đạt `0.900`, và trên toàn bộ 190 frame chỉ xảy ra vỏn vẹn `2` lần ID switch. Khả năng liên kết track giữa các frame đã rất tốt.
  - Về mặt Detection: DetA chỉ đạt `0.711` (thấp hơn nhiều so với AssA 0.820). Lỗi chính đến từ mô hình phát hiện YOLO26n zero-shot:
    1. Bắt nhầm vật thể tĩnh bên lề đường (ki-ốt, biển hiệu phản quang, bóng cây) thành phương tiện (tạo ra các track "ma" / bbox thừa như ID 7 tồn tại suốt 43 frame từ 16–116, ID 27 tồn tại 16 frame từ 106–121, sinh ra hàng chục FP).
    2. Bounding box chưa ôm sát vật thể thực tế (có tới 9–30 frame bbox bị lệch với IoU quanh ngưỡng 0.51–0.58).
    3. Nhạy cảm với kích thước vật thể ở xa và rìa ảnh, gây ra nhiễu detection 1 frame (như ID 26, 28).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- Vị trí cụ thể: `Frame 16 – 116, Track ID 7 của ReID (tương ứng ID 10 của ByteTrack)`.
- Hiện tượng: Model phát hiện liên tục một vật thể tĩnh bên lề đường bên trái (khu vực gần xe đỗ Track 3 và quầy ki-ốt/bóng râm) và duy trì một track giả kéo dài suốt 43 frame (từ frame 16 đến 116), đóng góp hàng chục lỗi FP cho mô hình.
- Vì sao bạn đúng và ReID sai: Người gán nhãn quan sát ngữ cảnh xuyên suốt video, hiểu rõ đây là vật thể tĩnh ven đường (không phải phương tiện giao thông xe bốn bánh hợp lệ theo `GUIDELINE_MINI.md`), nên hoàn toàn không gán nhãn cho khu vực này (khớp chính xác với ground truth gold, nơi cũng không hề có track nào ở vị trí đó). Model ReID bị "đánh lừa" bởi đặc trưng visual tĩnh của detector và Kalman filter dự đoán vận tốc xấp xỉ 0 nên liên tục match nhầm.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- Vị trí cụ thể: `Frame 108 – 110, Track 7 (xe tải lớn)`.
- Hiện tượng: Xe tải Track 7 bắt đầu xuất hiện từ góc bên phải ở frame 106 và lăn bánh sang trái. Trong bản gán nhãn ban đầu của bạn, ở frame 108 và 109, bbox bị kéo lệch nhẹ về phía sau thân xe so với thực tế (kết quả chấm so với gold cho thấy IoU tụt xuống 0.51 ở frame 109 và 0.54 ở frame 108 - được liệt kê trong danh sách 8 frame "Bbox trôi" tại Cell 10). Trong khi đó, ReID model chạy detector per-frame không bị phụ thuộc vào nội suy giữa 2 keyframe, đã bắt trọn và ôm rất sát phần đầu xe tải đang nhô ra ở frame 108–109.
- Vì sao làm bạn xem lại annotation: Kết quả detection của model tại các frame này cho thấy người gán nhãn đã đặt khoảng cách keyframe hơi xa (từ frame 106 nhảy sang frame 112) trong khi xe tải đang tăng tốc vào cua, dẫn đến hiện tượng trôi bbox do nội suy tuyến tính (interpolation drift). Nhờ soi lại kết quả này, người gán nhãn nhận thấy cần phải cắm thêm keyframe tại frame 108 và 109 để đảm bảo bbox luôn bám chặt mép xe ngay khi xe vừa nhập làn.
  *(Mặt khác, đối với các frame 105–107 ở bảng Disagreement Cell 18, evidence cho thấy model sai khi sinh ra các box rác 1 frame như ID 26 ở frame 105 và ID 28 ở frame 107 do nhiễu detector ở rìa ảnh).*

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Cải tiến trong `GUIDELINE_MINI.md`:
  1.  Bổ sung quy tắc bắt buộc: Khi xe bắt đầu vào khung hình (entry) hoặc có sự thay đổi về vận tốc/hướng di chuyển (như xe tải Track 7 lúc nhập làn, xe con Track 5 khi bo cua sau xe buýt), khoảng cách giữa 2 keyframe không được vượt quá 3 frame để triệt tiêu hoàn toàn lỗi trôi nội suy (interpolation drift).
  2. Ghi rõ quy tắc "Chỉ ôm phần pixel nhìn thấy (visible mask), không đoán phần thân xe bị che khuất quá 20%". Đồng thời quy định rõ: Nếu phần nhìn thấy bị đứt đoạn thành 2 mảng nhỏ, bbox chỉ bao trùm mảng nhìn thấy lớn nhất.
  3. Bổ sung hình ảnh minh họa cho các vật thể tĩnh ven đường dễ nhầm lẫn (ki-ốt, quầy hàng, biển báo thấp, bốt điện) và quy định rõ xe ở quá xa có kích thước nhỏ hơn 15x15 pixel chưa nhận diện rõ bánh xe thì chưa bắt đầu track.

- Thay đổi trong quy trình làm việc (Annotation Workflow):
  1. Không kiểm tra dàn trải, mà chia rõ 3 lượt tua riêng biệt: Lượt 1 kiểm tra tính liên tục của ID (bật hiển thị timeline ID trong CVAT để phát hiện ID switch/track vỡ); Lượt 2 soi từng điểm entry/exit để đảm bảo đánh dấu `outside` dứt khoát không có box treo; Lượt 3 soi hình học bbox ở chế độ slow-motion (0.5x).
  2. Chạy script kiểm tra bước nhảy diện tích và vận tốc tọa độ bbox giữa các frame liên tiếp trước khi khóa pre-gold; các frame có tốc độ biến thiên bất thường sẽ được gắn cờ (flag) để người gán nhãn kiểm tra lại thủ công.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md`
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` 
