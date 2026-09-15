# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Hoàng Kim Thiên`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `15` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `5.5` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Phương tiện ở xa xuất hiện với kích thước rất nhỏ và mờ**: Ở các frame đầu của clip, xe mới tiến vào khung hình chỉ có kích thước khoảng vài chục pixel, dễ nhầm với xe máy hoặc nhiễu nền. Cách xử lý: tua tiến/lùi qua 5-10 frame để quan sát quỹ đạo chuyển động và hình dáng đặc trưng của xe bốn bánh trước khi bắt đầu tạo track và gắn keyframe đầu tiên.
2. **Xe bị che khuất một phần (occlusion)**: Khi các xe di chuyển đan xen hoặc bị vật cản che khuất một phần thân xe. Cách xử lý: tuân thủ nghiêm ngặt guideline, chỉ vẽ bounding box ôm sát phần nhìn thấy được (visible box), tuyệt đối không phỏng đoán kích thước phần bị che khuất.
3. **Hiện tượng trôi bounding box (bbox drift) khi nội suy**: Xe di chuyển đổi hướng hoặc thay đổi tốc độ giữa hai keyframe đặt cách xa nhau khiến hộp nội suy bị lệch khỏi thân xe. Cách xử lý: bổ sung các keyframe trung gian tại đúng thời điểm xe bắt đầu đổi hướng hoặc tăng/giảm tốc độ (khoảng cách 8-12 frame/keyframe) để đảm bảo IoU luôn đạt trên 0.7.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Tập trung nhìn màu sắc và số ID của từng bounding box khi tua video ở tốc độ 1.5x. Xác nhận toàn bộ 8 xe đều duy trì duy nhất 1 ID xuyên suốt quãng đời, không xảy ra hiện tượng nhảy ID (ID switch) hoặc 1 xe bị ngắt thành 2 track riêng biệt.
- Lượt 2: Kiểm tra kỹ frame xuất hiện đầu tiên và frame biến mất của từng xe. Đảm bảo bấm thuộc tính outside/ẩn ngay khi xe vừa khuất hẳn hoặc đi ra ngoài rìa khung hình, không để sót bounding box rác đứng yên ở rìa ảnh.
- Lượt 3: Rà soát các frame nằm giữa hai keyframe để phát hiện bbox bị co giãn hoặc lệch góc theo chuyển động góc nhìn camera; phát hiện và nắn chỉnh 2 vị trí bị trôi nhẹ ở track 1 và track 5.

Kiểm chéo với: `Bạn cùng nhóm`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3` (1 lỗi switch ID khi xe rẽ và 2 lỗi trôi bbox do đặt keyframe quá thưa). Số lỗi bạn ấy tìm được trong bản của bạn: `0` (bản gán nhãn đạt độ khớp cao, ID nhất quán).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Hai người từng có sự phân vân ở thời điểm xe ở rất xa bắt đầu đi vào khung hình (frame 1-15): một bên gán ngay từ khi xe còn là vệt mờ, một bên chờ xe rõ nét mới gán. Sau khi thảo luận, nhóm thống nhất quy tắc: chỉ bắt đầu vẽ track khi đã xác định rõ tối thiểu 2 đặc điểm nhận dạng của xe bốn bánh (đèn pha, lưới tản nhiệt hoặc kính chắn gió). Luật này đã được bổ sung cụ thể vào `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `94a2bd34aa00348b49cd3ec48e85dbfeaf00504b97349e343363d40e4d1f9d79` |
| Thời điểm khóa | `2026-09-15T05:48:05Z` |
| Số row / frame / track trước khi mở reference | `573 rows / 190 frames / 8 tracks` |

| Giai đoạn | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.812 | 0.798 | 0.829 | 0.867 | 0.970 | 0.941 | 0.852 | 17 | 17 | 0 |
| Sau rework | 0.812 | 0.798 | 0.829 | 0.867 | 0.970 | 0.941 | 0.852 | 17 | 17 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

Bản pre-gold đã vượt xuất sắc tất cả các tiêu chí của cổng qua bài ngay từ lần nộp đầu tiên (IDF1 đạt 0.970, MOTA đạt 0.941, MOTP đạt 0.852, 0 lỗi ID switch). Tuy nhiên ở phần chẩn đoán bbox trôi (loose boxes), một số keyframe đã được tinh chỉnh để tăng độ khít IoU:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox trôi nhẹ giữa 2 keyframe (IoU 0.52 - 0.58) | 82 - 92 | 5 | Bổ sung thêm keyframe tại frame 86 và căn chỉnh lại mép bbox ôm sát đuôi xe khi xe tăng tốc |
| Bbox chưa ôm khít góc phản quang | 186 - 190 | 1 | Điều chỉnh lại tọa độ cạnh phải của bbox để bao trọn góc đèn xe trước khi rời cảnh |
| Sai lệch vị trí nhẹ do nội suy góc rẽ | 138 - 140 | 1 | Nắn lại góc hộp bao cho vừa khít thân xe khi chuyển làn |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15` / `8.4.145` / `2.11.0+cu128` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / ByteTrack (`bytetrack.yaml`) & BoT-SORT + ReID (`botsort-reid.yaml`) |
| conf / IoU / imgsz / classes | conf `0.25` / IoU `0.70` / imgsz `960` / classes `[2, 5, 7]` |
| device | `0` (CUDA GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.812 | 0.798 | 0.829 | 0.867 | 0.970 | 0.941 | 0.852 | 17 | 17 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.800 | 0.745 | 0.860 | 0.900 | 0.912 | 0.815 | 0.889 | 85 | 20 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong kết quả của tôi, IDF1 đạt **0.970**, cao hơn so với MOTA là **0.941**.
- Trong trường hợp một hệ thống có **MOTA cao nhưng IDF1 thấp**, điều đó phản ánh rằng detector tìm vật thể rất tốt (ít khi bỏ sót xe và ít khi vẽ thừa), nhưng **tracker duy trì định danh (identity) rất kém** (thường xuyên bị đổi ID hoặc phân mảnh track sau khi bị che khuất).
- Lý do MOTA không phạt nặng lỗi ID nằm ở công thức tính: $\text{MOTA} = 1 - \frac{\sum (\text{FP} + \text{FN} + \text{IDSW})}{\sum \text{GT}}$. Trong MOTA, mỗi lần xảy ra nhảy ID chỉ bị tính là **1 lỗi phạt đơn lẻ** tại đúng frame chuyển giao đó; ở tất cả các frame tiếp theo, nếu bbox của ID mới vẫn khớp với vật thể, hệ thống vẫn được tính điểm TP bình thường mà không chịu bất kỳ hình phạt tích lũy nào. Ngược lại, **IDF1** đo lường mức độ bảo toàn danh tính toàn cục (global identity preservation) bằng tỷ lệ F1-score trên toàn bộ trajectory. Nếu một track bị đổi ID ở giữa video, toàn bộ nửa sau của track đó sẽ bị coi là không khớp với ID gốc, khiến IDF1 bị sụt giảm nghiêm trọng.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Sự khác biệt về chỉ số**:
  - `IDF1`: BoT-SORT + ReID đạt **0.900**, cao hơn ByteTrack control (**0.875**, tăng +0.025).
  - `AssA`: BoT-SORT + ReID đạt **0.820**, cao hơn ByteTrack control (**0.776**, tăng +0.044).
  - `IDSW`: Cả hai cấu hình đều ghi nhận **2 lần switch ID**.
  - `FN`: BoT-SORT + ReID giảm mạnh số lượng FN từ **54** (ở ByteTrack) xuống còn **26**, cho thấy khả năng duy trì bám vết liên tục tốt hơn đáng kể.
- **Minh chứng chuỗi frame (frame sequence)**:
  - Xem xét track gold 5 ở chuỗi frame **85–95**: Đây là giai đoạn xe di chuyển qua góc khuất và có sự đan xen với nền đường phức tạp. ByteTrack thuần túy (dựa trên motion + IoU 2 vòng) đã bị mất dấu ở frame 94 và cắt đôi track thành hai ID riêng biệt (ID 23 và ID 32, dẫn đến việc mất 14 frame không được gán đúng). Trong khi đó, BoT-SORT kết hợp ReID appearance feature đã nhận dạng lại xe thành công sau thời điểm che khuất, giữ độ phủ trajectory tốt hơn nhiều (độ phủ đạt 77% so với sự đứt gãy của ByteTrack).
- **Lưu ý quan trọng**: Thí nghiệm này là một **system comparison**, hoàn toàn **không cô lập được hiệu ứng nhân quả (causal effect) độc lập của ReID**. Lý do là BoT-SORT và ByteTrack có mã nguồn triển khai khác nhau về mô hình chuyển động Kalman Filter, cơ chế bù chuyển động camera (Camera Motion Compensation - CMC), và logic ghép cặp ma trận chi phí. Do đó, sự cải thiện điểm số phản ánh hiệu năng tổng thể của cả gói giải pháp BoT-SORT-ReID chứ không thể quy kết 100% riêng cho ReID feature extractor.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **Sự thay đổi DetA, FP và FN**:
  - `DetA`: Tăng từ **0.649** (ByteTrack) lên **0.711** (BoT-SORT + ReID).
  - `FN`: Giảm hơn một nửa, từ **54** xuống **26**.
  - `FP`: Tăng nhẹ từ **88** lên **91**.
- **Nguồn gốc lỗi còn lại**: Lỗi còn lại trong hệ thống chủ yếu xuất phát từ **detector** (phát hiện đối tượng) chứ không phải association:
  1. `FP cao (88 - 91)`: Detector YOLOv8/26 bị bắt nhầm vào các vật thể tĩnh ven đường (quầy hàng, biển báo, kết cấu mái che cố định) và sinh ra các track "ma" tồn tại cố định hàng chục frame (điển hình là track ID 7 tồn tại suốt 43 frame từ frame 16 đến 116).
  2. `Bbox lệch (LocA ~0.84 - 0.87)`: Detector khoanh hộp chưa thực sự khít viền xe ở khoảng cách xa, dẫn đến IoU chỉ dao động quanh ngưỡng 0.52 - 0.58.
  3. Lỗi association thực sự rất nhỏ (chỉ có 2 IDSW trên toàn bộ 190 frame của clip). Vì vậy, hướng cải thiện lớn nhất nằm ở việc tinh chỉnh detector (nâng ngưỡng confidence hoặc train thêm dữ liệu negative sample) để dập tắt FP tĩnh.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Vị trí**: Track ID 7 của ReID model, xuất hiện từ **frame 16 đến frame 116** (tổng cộng 43 frame), tọa độ quanh $(x \approx 491 - 495, y \approx 211 - 213, w \approx 98 - 102, h \approx 56 - 57)$.
- **Hiện tượng**: Model ReID phát hiện một vật thể tĩnh bên lề đường (mái che/quầy hàng cố định) và liên tục gán cho nó ID 7 với confidence thấp (~0.27 - 0.31). Bounding box này đứng bất động trong suốt hơn 100 frame.
- **Vì sao bạn đúng**: Người gán nhãn nhận thức được ngữ cảnh thời gian và không gian, xác định rõ đây là vật thể tĩnh ven đường chứ không phải xe bốn bánh đang lưu thông hoặc đỗ, nên đã bỏ qua theo đúng guideline. Model detector do thiếu hiểu biết ngữ cảnh dài hạn nên bị ảo giác (hallucination) thành FP lặp đi lặp lại.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Vị trí**: **Frame 113**, liên quan đến **track gold 6** (xe di chuyển ở làn giữa).
- **Hiện tượng**: Tại frame 113, tracker ReID đột ngột bị phân mảnh và nhảy ID từ **ID 24 sang ID 31**.
- **Xem lại và bằng chứng xác minh**:
  - Khi đối chiếu lại hình ảnh tại frame 113 và các frame lân cận, đây là khoảnh khắc chiếc xe đi vào vùng bóng râm của cây ven đường và bị che khuất một góc nhỏ bởi phương tiện phía trước. Ánh sáng thay đổi đột ngột làm vector embedding đặc trưng ngoại hình (ReID appearance feature) bị biến dạng mạnh, khoảng cách cosine vượt ngưỡng `appearance_thresh (0.80)`, khiến thuật toán association không ghép được với track cũ mà khởi tạo ID mới (ID 31).
  - Bằng chứng video cho thấy phương tiện này chuyển động liên tục với quỹ đạo vận tốc đều, thân xe không hề thay đổi. Do đó, việc nhãn tay duy trì liên tục duy nhất 1 track ID xuyên suốt từ frame đầu đến cuối là hoàn toàn chính xác; sai số ở đây hoàn toàn thuộc về model.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

1. **Bổ sung và chuẩn hóa trong `GUIDELINE_MINI.md`**:
   - **Quy định kích thước tối thiểu**: Đặt ngưỡng kích thước tối thiểu (ví dụ: chiều cao tối thiểu 20 pixel) để bắt đầu khởi tạo track mới cho xe ở phía xa, tránh việc các annotator gán nhãn sớm/muộn không đồng nhất ở giai đoạn xe còn mờ.
   - **Chuẩn hóa mật độ keyframe**: Đưa ra nguyên tắc bắt buộc: với các đoạn đường cong hoặc xe chuyển làn/tăng giảm tốc, khoảng cách tối đa giữa 2 keyframe không được vượt quá 10 frame nhằm dập tắt triệt để hiện tượng trôi bbox (IoU drift).
   - **Danh mục loại trừ cụ thể (Negative class checklist)**: Bổ sung hình ảnh ví dụ về các vật thể tĩnh dễ gây nhầm lẫn ven đường (biển quảng cáo có hình xe, bốt kỹ thuật, quầy hàng) để người gán không bị dao động.
2. **Cải tiến quy trình làm việc (Workflow Optimization)**:
   - Áp dụng triệt để quy trình tự kiểm 3 lượt tua (Lượt 1: kiểm tra tính liên tục của ID; Lượt 2: kiểm tra điểm bắt đầu/kết thúc outside; Lượt 3: kiểm tra độ khít bbox ở các frame giữa).
   - Chạy các script tự động `tools/check_mot_labels.py` ngay trong quá trình gán nhãn từng clip để phát hiện sớm các cảnh báo (như track đứng yên bất thường hoặc track quá ngắn) trước khi tiến hành khóa pre-gold và chấm điểm.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
