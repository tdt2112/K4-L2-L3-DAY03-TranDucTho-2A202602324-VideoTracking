# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Trần Đức Thọ`
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

Bổ sung của nhóm: Lọc theo class COCO `[2, 5, 7]` (`car`, `bus`, `truck`), tuyệt đối bỏ qua `motorcycle` (class 3) và người để tránh lỗi False Positive (FP) so với Gold.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (25 frame = 2 giây @ 12.5 fps) | Mặc định của lab; xe bị che tạm thời bởi chướng ngại vật ngắn vẫn là cùng một đối tượng |
| Xe bị che lâu hơn ngưỡng trên | Cấp `track_id` mới | Tránh gán nhầm ID khi đối tượng đã di chuyển quá xa vị trí Kalman dự đoán |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Đảm bảo đúng nguyên tắc không reuse ID cho xe khác hoặc xe đã rời cảnh |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID riêng của từng xe dựa vào quỹ đạo và đặc trưng ngoại dạng (appearance) | Tránh hiện tượng tráo ID (IDSW) khi hai bbox đè lên nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng tự tin xem xét `conf >= 0.25` |
| Xe đang đỗ, không di chuyển | Gán nhãn và giữ nguyên 01 `track_id` duy nhất xuyên suốt tất cả các frame xe đứng im |
| Keyframe đặt dày ở đâu | Đặt dày tại frame xe bắt đầu bị che khuất, frame đổi hướng đột ngột, hoặc khi hai xe cắt qua nhau |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01` / `WORST_FRAME (Cell 5)` / `T# (Model ReID)`
- Tình huống: Model nhận nhầm vật thể tĩnh (quầy hàng/bốt điện/xe in trên bạt) thành xe và giữ ID kéo dài.
- Quyết định: **Không gán nhãn**.
- Lý do: Đây là nhiễu tĩnh/False Positive của model, không phải xe bốn bánh thật.

### Ca 2
- Clip / frame / ID: `clip_01` / `Rìa khung hình` / `B# (Nhãn gán)`
- Tình huống: Xe vừa xuất hiện ở rìa ảnh, mới lộ 20-30% góc đầu xe.
- Quyết định: **Gán nhãn BBox ôm phần nhìn thấy**.
- Lý do: Đảm bảo không bỏ sót xe (giảm FN) ngay từ frame đầu tiên nhận diện được.

### Ca 3
- Clip / frame / ID: `clip_01` / `Khu vực occlusion` / `Track cắt nhau`
- Tình huống: Hai xe đè lên nhau khiến detector tụt tự tin (`conf < 0.25`).
- Quyết định: **Duy trì ID cũ cho cả 2 xe**.
- Lý do: Áp dụng cơ chế ByteTrack vòng 2 (giữ bbox điểm thấp) để không làm đứt gãy track.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Bổ sung quy định rõ về xử lý vật thể tĩnh đứng im hàng chục frame ở rìa đường để không bị nhầm với xe đỗ.
- Quy định rõ ranh giới BBox chỉ ôm phần nhìn thấy, không vẽ trùm lên bóng râm (shadow) dưới mặt đường.