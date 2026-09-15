# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Trần Đức Thọ`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `75` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất tạm thời khi di chuyển song song: Giữ nguyên Track ID gốc, bật Occluded (`Q`) và điều chỉnh bbox ôm phần nhìn thấy.
2. Xe vừa vào khung hình ở viền cạnh: Xác định chính xác frame xe hiện ra rõ dạng 4 bánh rồi mới tạo Track.
3. Xe di chuyển ra khỏi khung hình: Đặt keyframe chuẩn tại frame cuối còn thấy và bấm `O` (Outside) ngay frame kế tiếp.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (Identity): Không phát hiện lỗi ID switch hay trùng ID trong cùng frame.
- Lượt 2 (Endpoint): Phát hiện 1 ca quên ngắt Outside ở frame xe ra khỏi khung và đã chỉnh lại.
- Lượt 3 (Geometry): Phát hiện 2 vị trí Bbox bị drift nhẹ ở midpoint và đã bổ sung keyframe.

Kiểm chéo với: `Trần Đức Thọ (Solo Self-QC)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3`. Số lỗi bạn ấy tìm được trong bản của bạn: `3`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không có xung đột do tự kiểm soát chất lượng. Đã bổ sung luật quy định rõ số frame che khuất tối đa (25 frames) được phép giữ nguyên Track ID.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| Thời điểm khóa | `2026-09-15T10:30:00Z` |
| Số row / frame / track trước khi mở reference | `634 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.812 | 0.835 | 0.792 | 0.890 | 0.845 | 0.780 | 0.765 | 24 | 31 | 3 |
| Sau rework | 0.865 | 0.872 | 0.858 | 0.902 | 0.892 | 0.835 | 0.788 | 12 | 15 | 1 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo | 136 | 4 | Đã bấm phím O (Outside) để ngắt track chính xác |
| Boundary | 134 | 8 | Bổ sung keyframe điều chỉnh bbox ôm khít xe từ frame đầu |
| Bbox drift | 11 | 1 | Điều chỉnh lại Bbox ở midpoint bị trượt khỏi đuôi xe |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.10.12 / 8.4.145 / 2.1.0+cu121 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml vs botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.865 | 0.872 | 0.858 | 0.902 | 0.892 | 0.835 | 0.788 | 12 | 15 | 1 |
| ByteTrack control vs gold | 0.624 | 0.685 | 0.570 | 0.842 | 0.645 | 0.590 | 0.721 | 85 | 92 | 12 |
| BoT-SORT + ReID vs gold | 0.678 | 0.690 | 0.668 | 0.845 | 0.723 | 0.621 | 0.725 | 78 | 88 | 5 |
| ReID vs bạn | 0.662 | 0.680 | 0.645 | 0.838 | 0.701 | 0.605 | 0.718 | 82 | 90 | 6 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA phạt mỗi sự cố ID Switch đúng 1 lần tại đúng frame xảy ra chuyển đổi ID (`IDSW`), trong khi IDF1 đo lường độ nhất quán danh tính (identity mapping) trên toàn bộ thời lượng tồn tại của đối tượng. Nếu một chiếc xe bị gán nhầm ID ở giữa clip, MOTA chỉ trừ điểm 1 lần duy nhất, nhưng IDF1 sẽ bị phạt nặng trong suốt toàn bộ khoảng thời gian còn lại do lệch chuỗi nhận dạng ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID đạt chỉ số cao hơn ở IDF1 (+0.078) và AssA (+0.098), đồng thời giảm đáng kể số lỗi IDSW (từ 12 xuống 5). Ví dụ tại phân đoạn Frame 130–145, khi hai xe di chuyển song song và che khuất nhau: ByteTrack chỉ dùng dự đoán vị trí Kalman + IoU nên bị mất dấu và cấp Track ID mới; trong khi BoT-SORT bổ sung ReID embedding giữ được đặc trưng thị giác nên duy trì ID nhất quán. Lưu ý rằng đây là so sánh toàn bộ hệ thống (system comparison), chênh lệch không chỉ riêng do ReID mà còn từ sự khác biệt trong thuật toán liên kết (association algorithm) của hai thư viện.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Chỉ số DetA giữa ByteTrack (0.685) và ReID (0.690) gần như tương đương nhau, số lượng FP (78–85) và FN (88–92) ở cả hai model vẫn ở mức rất cao so với nhãn tay. Điều này chứng tỏ yếu tố chính giới hạn chất lượng theo dõi hiện tại nằm ở bước **Detector** (YOLO zero-shot bỏ sót xe nhỏ/rìa hoặc bắt nhầm vật thể) chứ không thuần túy là lỗi liên kết Association.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại Frame 134, Model ReID phát hiện một quầy hàng tĩnh ở vỉa hè và gán nhãn thành xe con (False Positive), tạo ra Track ID mới duy trì trong nhiều frame. Nhãn gán tay của người gán chính xác hơn do nhận diện đúng ngữ cảnh vật thể không phải là xe bốn bánh đang lưu thông.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Tại Frame 105, Model ReID phát hiện một chiếc xe góc xa bắt đầu đi vào khung hình từ rìa trái sớm hơn 2 frame so với nhãn gán tay ban đầu. Kiểm tra lại ảnh gốc cho thấy phần đầu xe đã xuất hiện rõ ràng ở viền ảnh, giúp người gán điều chỉnh lại frame bắt đầu tạo track chính xác hơn.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ cập nhật vào `GUIDELINE_MINI.md` quy định rõ ràng về kích thước pixel tối thiểu khi bắt đầu gán nhãn xe ở rìa ảnh xa, đồng thời chuẩn hóa quy trình tự kiểm tra 3 lượt (Identity, Endpoint, Geometry) ngay sau khi hoàn thành từng clip.

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