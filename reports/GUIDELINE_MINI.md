# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Trần Đức Thọ
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che dưới 25 frame (2 giây @ 12.5 fps) | Duy trì tính liên tục của trajectory đối tượng |
| Xe bị che lâu hơn ngưỡng trên | `Tạo Track ID mới | Tránh nhầm lẫn identity do khoảng cách không gian lớn |
| Xe rời khung hình rồi quay lại | Track mới | Đã ra khỏi khung coi như kết thúc một track |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID gốc của từng xe, điều chỉnh bbox ôm phần nhìn thấy | Dựa vào chuyển động tuyến tính để không swap ID |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần nhìn thấy được |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: 15x15 px |
| Xe đang đỗ, không di chuyển | Giữ nguyên bbox và ID xuyên suốt thời gian có mặt trong ảnh |
| Keyframe đặt dày ở đâu | Đặt keyframe tại các điểm đổi hướng, đổi tốc độ hoặc bắt đầu/kết thúc bị che |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / Frame 134 / Track ID 8
- Tình huống: Xe mới bắt đầu đi vào từ góc dưới bên phải khung hình.
- Quyết định: Vẽ bbox ngay từ frame 134 khi phần đầu xe xuất hiện rõ nét.
- Lý do: Xác định chắc chắn là xe bốn bánh, đủ diện tích nhận diện.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame 128 / Track ID 4
- Tình huống: Xe di chuyển ra khỏi rìa trái khung hình.
- Quyết định: Bấm Outside (`O`) tại frame 136 khi xe khuất hẳn.
- Lý do: Không để bbox treo lơ lửng ở khoảng trống.
### Ca 3
- Clip / frame / ID: `clip_02` / Frame 1 / Track ID 3
- Tình huống: Xe đang đỗ bên đường không di chuyển suốt từ frame 1 đến 60.
- Quyết định: Giữ nguyên Track ID 3 suốt 60 frame.
- Lý do: Xe đỗ tĩnh vẫn là object bốn bánh nằm trong schema gán nhãn.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Rà soát chính xác các frame chuyển tiếp khi xe đi ra khỏi khung hình để bật thuộc tính `Outside` đúng frame.
- Đặt keyframe dày hơn ở các phân đoạn xe bắt đầu gia tốc hoặc chuyển hướng rẽ.
