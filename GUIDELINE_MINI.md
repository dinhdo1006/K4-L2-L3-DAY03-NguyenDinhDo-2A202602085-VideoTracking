# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Đình Độ — MSSV 2A202602085 (làm cá nhân)`
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

Bổ sung của nhóm (nếu có): `Không phân loại loại xe. Xe đang đỗ vẫn gán và giữ track suốt thời gian còn trong khung. Không gán xe máy dù đi gần cụm xe bốn bánh.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | dưới ngưỡng này vẫn cùng một xe; đổi ID sẽ làm AssA/IDF1 tụt |
| Xe bị che lâu hơn ngưỡng trên | mở track mới | mất continuity quá lâu; gán lại ID cũ dễ nhầm xe khác |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | đã ra khỏi khung là kết thúc track theo luật lab |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe một bbox riêng, giữ nguyên ID từng xe; không Merge hai xe khác nhau | tránh ID switch khi crossing |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `nhìn rõ thân xe (không chỉ một vệt sáng), ước lượng chiều rộng ≳ 15–20 px` |
| Xe đang đỗ, không di chuyển | vẫn gán `vehicle` và giữ cùng ID suốt thời gian xe nằm trong khung |
| Keyframe đặt dày ở đâu | khi xe rẽ, phanh, bị che, gần rìa ảnh, hoặc sắp ra/vào khung; đoạn đi thẳng đều thì thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 1–190 / ID 3`
- Tình huống: `Xe đỗ / gần như không di chuyển suốt clip`
- Quyết định: `Giữ một track ID 3 từ đầu đến cuối clip`
- Lý do: `Luật lab: xe đỗ vẫn là vehicle và cần track liên tục khi còn trong khung`

### Ca 2
- Clip / frame / ID: `clip_01 / frame ~149–151 / ID 4`
- Tình huống: `Xe ID 4 sắp / vừa rời khung; dễ quên Outside nên bbox treo thêm vài frame`
- Quyết định: `Kết thúc track đúng frame xe không còn nhìn thấy; Outside ngay khi rời khung`
- Lý do: `Eval vs gold báo bbox treo ID 4 ở frame 149–151 — đây là FP sau exit`

### Ca 3
- Clip / frame / ID: `clip_02 / frame 1–3 / ID 2`
- Tình huống: `Xe rất nhỏ / sát rìa, chỉ rõ vài frame rồi biến mất`
- Quyết định: `Vẫn mở track từ frame xác định được là xe bốn bánh; track ngắn 3 frame được chấp nhận nếu không phải xe máy`
- Lý do: `Ưu tiên không bỏ sót entry; không kéo dài bbox sau khi đã ra khỏi khung`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Nhắc mạnh Outside: sau khi xe rời khung phải Outside ngay; ID 4 treo 3 frame là lỗi thao tác, không phải luật ID.`
- `Thêm keyframe quanh đoạn bbox trôi (clip_01 ID 5, frame 96–97, IoU ~0.59) — đoạn giữa hai keyframe xa cần tua kiểm giữa.`
- `Track cực ngắn ở rìa (clip_02 ID 2) phải ghi rõ trong guideline để peer không hiểu nhầm là FP.`
