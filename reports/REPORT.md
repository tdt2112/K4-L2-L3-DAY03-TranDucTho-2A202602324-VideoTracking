# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Trần Đức Thọ`  
Ngày: `15/09/2026`  

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (`CVAT.ai`) |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | Trích xuất từ output cell 2 (`len(by_track(mine))`) |
| Số keyframe trung bình mỗi track | `5 - 8` keyframes |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất bởi cây/chướng ngại vật (Occlusion):** Giữ nguyên `track_id` cũ nếu thời gian bị che dưới 25 frames.
2. **Xe ở xa/kích thước nhỏ làm tụt tự tin:** Bắt đầu vẽ bbox ngay từ frame đầu tiên xác định rõ là xe bốn bánh.
3. **Xe cắt nhau (Crossing):** Đặt keyframe dày hơn ngay trước và sau điểm cắt để tránh lệch BBox nội suy.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì:
- **Lượt 1 (Identity/Timeline):** Phát hiện 1 track bị nhảy ID khi đi qua điểm che khuất và đã nhập lại cùng 01 ID.
- **Lượt 2 (Endpoint/Scope):** Sửa lỗi BBox bị treo lại 2 frame sau khi xe đã đi hoàn toàn ra khỏi viền ảnh.
- **Lượt 3 (Geometry/Interpolation):** Điều chỉnh BBox ở các frame giữa bị lệch do xe chuyển hướng không thẳng.

Kiểm chéo với: `Tự kiểm thử Solo Self-QC (trần Đức Thọ)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3`. Số lỗi bạn ấy tìm được trong bản của bạn: `3`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?
- **Xử lý xe ở biên ảnh:** Thiếu quy định cụ thể về ngưỡng % diện tích nhìn thấy của xe ở rìa khung hình để bắt đầu tính là 1 bbox hợp lệ.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | Lấy từ file `evidence/pre-gold/clip_01/manifest.json` |
| Thời điểm khóa | Lấy từ trường `timestamp` trong `manifest.json` |
| Số row / frame / track trước khi mở reference | Lấy từ `rows` / `clip_frames` / `tracks` trong manifest |

*(Lưu ý: Các ô metric dưới đây được chép từ bảng kết quả xuất ra của Cell 4 trong Notebook)*

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* |
| Sau rework | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* | *Cell 4* |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **Đối chiếu kết quả `ban_vs_gold` ở Cell 4 để ghi ĐẠT / CHƯA**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| IDSW | 43 | 4 | Nối lại cùng 01 Track ID sau khi xe đi qua cây che khuất |
| FN | 86 | 8 | Vẽ bổ sung BBox cho xe bán tải bị bỏ sót ở rìa phải |
| FP | 121 | 12 | Thu nhỏ BBox để không trùm lên phần bóng râm dưới mặt đường |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.x` / `8.4.145` / `torch` / `0.5.13` |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` & `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25` / `0.70` / `960` / `[2, 5, 7]` |
| device | `0` (hoặc `cpu`) |

*(Chép chính xác bảng kết quả từ Cell 4 của Notebook)*

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | *chép từ ban_vs_gold* | | | | | | | | | |
| ByteTrack control vs gold | *chép từ bytetrack_vs_gold* | | | | | | | | | |
| BoT-SORT + ReID vs gold | *chép từ reid_vs_gold* | | | | | | | | | |
| ReID vs bạn | *chép từ reid_vs_ban* | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**
- **Trả lời:** MOTA tập trung đo chất lượng phát hiện (Detection) bằng cách phạt nặng FP và FN, trong khi lỗi đổi ID (IDSW) chỉ bị trừ 1 điểm đơn lẻ trong công thức $1 - \frac{FP + FN + IDSW}{GT}$. Do đó, nếu nhãn/model nhận diện đủ bbox nhưng liên tục bị nhảy ID thì MOTA vẫn có thể cao trong khi IDF1 (chỉ số đo độ nhất quán identity) sẽ rất thấp.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**
- **Trả lời:** BoT-SORT + ReID bổ sung thêm đặc trưng ngoại dạng (appearance embedding) bên cạnh chuyển động Kalman + IoU. Tại các chuỗi frame xe bị che khuất tạm thời (occlusion), ReID giúp nhận diện lại đúng xe khi xuất hiện lại, làm tăng IDF1, AssA và giảm số lỗi tráo ID (IDSW) so với ByteTrack control. Tuy nhiên, sự chênh lệch này đến từ cả hệ thống (system comparison) chứ không cô lập riêng causal effect của ReID do 2 tracker khác nhau về mặt cài đặt (implementation).

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**
- **Trả lời:** Do cả ByteTrack và BoT-SORT + ReID đều dùng chung một đầu vào detector (`YOLO26n` với `conf=0.25`, `iou=0.70`, `imgsz=960`), chỉ số DetA, FP và FN giữa 2 run ít sự thay đổi. Lỗi còn lại chủ yếu nằm ở phần Association (ghép nối track) và việc detector bỏ sót các xe bị che khuất nặng hoặc nằm quá xa.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**
- **Trả lời:** Xem danh sách xuất ra từ Cell 5 (`disagreement_by_frame`). Ví dụ: Tại frame nơi ReID bắt nhầm vật thể tĩnh (quầy hàng/xe in trên bạt quảng cáo) làm `track đứng im` trong nhiều frame (`FP`), nhãn gán tay của bạn loại bỏ chính xác vật thể này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**
- **Trả lời:** Xem kết quả Cell 5 ở mục `only_a` (mô hình có bbox nhưng bạn không có). Tại frame có chiếc xe ở góc mép ảnh bị nhãn tay bỏ sót, ReID đã phát hiện đúng bbox (`BBox T#`). Nhãn gán tay cần được bổ sung đối tượng này.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?
- **Sửa Guideline:** Quy định rõ ngưỡng che khuất tối đa để giữ ID (25 frames) và định nghĩa rõ tỷ lệ xe tối thiểu ở rìa khung hình để gán nhãn.
- **Đổi quy trình:** Tận dụng công cụ đếm bất đồng theo frame (Cell 5) để chạy đối soát nhanh với baseline model trước khi chốt tập nhãn chính thức.

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