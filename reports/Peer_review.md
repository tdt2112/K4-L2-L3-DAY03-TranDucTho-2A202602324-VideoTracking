# Peer review — Day 3

| Trường | Giá trị |
| --- | --- |
| Author | `Trần Đức Thọ` |
| Reviewer | `Trần Đức Thọ (Solo Self-QC)` |
| Pair ID | `Solo` |
| CVAT version | `CVAT.ai Web App` |
| Thời điểm review | `15/09/2026` |

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 000042 | 43 | 4 | Occlusion / IDSW | Xe bị cây che khuất 5 frame bị nhảy ID | Giữ nguyên ID=4 trước và sau khi qua cây | fixed |
| 2 | 000085 | 86 | 8 | FN / Bỏ sót | Xe bán tải ở rìa phải ảnh chưa được vẽ bbox | Bổ sung bbox ôm phần xe nhìn thấy được | fixed |
| 3 | 000120 | 121 | 12 | FP / BBox rộng | Bbox bao trùm cả bóng râm kéo dài dưới đường | Co hẹp bbox ôm sát thân xe thật | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | clip_01 gán đủ các xe 4 bánh (`car`, `bus`, `truck`) |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Đã tua Lượt 1 kiểm tra timeline ID |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Đã xử lý giữ ID cho ca che khuất < 25 frames |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Bbox kết thúc đúng frame xe ra khỏi khung hình |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Đã chỉnh lề bbox theo biên ảnh |
| Frame giữa hai keyframe không bị interpolation drift | PASS | Đã chèn thêm keyframe tại các đoạn cua/đổi hướng |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `gt.txt` định dạng chuẩn 10 cột, phân tách bằng dấu phẩy |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 100% finding đã được đóng `fixed` |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Rà soát toàn bộ `track_id` từ đầu đến cuối clip |
| 2 — endpoint/scope | PASS | Kiểm tra frame đầu (entry) và frame cuối (exit) của mọi track |
| 3 — geometry/interpolation | PASS | Soi độ khớp BBox ở các frame nội suy giữa 2 keyframe |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `Giữ nguyên Track ID khi xe bị che khuất ngắn (< 25 frames) thay vì tạo track mới`.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Bbox xe đỗ cố định ở góc trái: giữ nguyên ID vì xe đứng im trong suốt clip là hợp lệ`.
3. Một rule cần Lab Coach làm rõ (nếu có): `Quy định chính xác ngưỡng % diện tích xe xuất hiện ở rìa ảnh để bắt đầu gán nhãn`.