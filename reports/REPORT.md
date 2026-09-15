# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Đình Độ — MSSV 2A202602085 (cá nhân)`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (`app.cvat.ai`), label `vehicle`, chế độ **Track** |
| Thời gian gán `clip_02` (warm-up) | `~30` phút |
| Thời gian gán `clip_01` | `~90` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `~5–8` (dày hơn khi che / rẽ / gần rìa) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe đỗ dài (clip_01 ID 3, gần như cả 190 frame): dễ quên cập nhật bbox khi góc nhìn đổi nhẹ — giữ một ID, thỉnh thoảng thêm keyframe để bbox không trôi.`
2. `Xe vào/ra rìa và bị cắt khung (nhiều ID gần mép): bbox chạm rìa, không đoán phần ngoài; Outside ngay khi xe biến mất.`
3. `Nhiều xe cùng lúc (khoảng frame 109+ có ID 5/6/7/8): làm xong từng xe một track, tránh nhảy qua lại để giảm ID switch.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Không thấy ID nhấp nháy / đổi số giữa chừng; 8 ID ổn định.`
- Lượt 2: `Phát hiện nguy cơ bbox treo cuối track (đặc biệt ID 4 gần lúc rời khung); track ngắn clip_02 ID 2 (frame 1–3) vẫn hợp lệ vì xe chỉ lộ ngắn.`
- Lượt 3: `Một số đoạn giữa keyframe của xe chuyển động (vùng ID 5 quanh frame 96–97) bbox hơi lỏng — cần thêm keyframe.`

Kiểm chéo với: `làm cá nhân — không làm peer review với bạn cùng nhóm`.
Số lỗi bạn tìm được trong bản của bạn ấy: `N/A`. Số lỗi bạn ấy tìm được trong bản của bạn: `N/A`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`N/A (không peer). Sau khi đối chiếu gold, bổ sung rõ luật Outside (tránh bbox treo) và luật keyframe dày quanh đoạn IoU tụt.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `6c20a82a2b8eb18f0fd70696635e177dd1b2924ec8853aa226e5fa32435a4ef8` |
| Thời điểm khóa | `2026-09-15T08:11:34.644235+00:00` (UTC) |
| Số row / frame / track trước khi mở reference | `563 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.844 | 0.825 | 0.871 | 0.889 | 0.972 | 0.944 | 0.878 | 11 | 21 | 0 |
| Sau rework | 0.844 | 0.825 | 0.871 | 0.889 | 0.972 | 0.944 | 0.878 | 11 | 21 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

> Ghi chú: bản `annotations/clip_01/gt.txt` hiện trùng snapshot pre-gold và **đã qua cổng**. Các lỗi còn lại ở dưới là việc nên sửa trên CVAT nếu còn thời gian; chưa export bản rework mới nên metric hai hàng giống nhau. Nguồn: `outputs/eval_pre_gold.json`, `outputs/eval_vs_gold.json`.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo / thừa | 149–151 | 4 | `Đã ghi nhận; cần Outside sớm hơn trên CVAT (xe đã rời khung) rồi export lại` |
| Bbox trôi | 96–97 | 5 | `Đã ghi nhận; thêm keyframe quanh frame 96–97 vì IoU ~0.59` |
| Bỏ sót track gold | — | — | `Không có (missed_gt_tracks rỗng; cùng 8 track với gold)` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt` · ByteTrack (`bytetrack.yaml`) vs BoT-SORT+ReID (`botsort-reid.yaml`) |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / COCO [2,5,7]` (car, bus, truck) |
| device | `cpu` (`persist=true`, 190 frame) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.844 | 0.825 | 0.871 | 0.889 | 0.972 | 0.944 | 0.878 | 11 | 21 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.784 | 0.729 | 0.847 | 0.884 | 0.904 | 0.797 | 0.874 | 94 | 19 | 1 |

> Model: ByteTrack 607 bbox / 16 track; ReID 638 bbox / 16 track. Sweep `appearance_thresh` 0.7 / 0.8 / 0.9 gần như không đổi (HOTA≈0.763, IDF1≈0.900, IDSW=2) → không kết luận được ReID “một mình” là nguyên nhân; đây là so sánh hai hệ tracker khác implementation.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA (0.944) thấp hơn một chút so với IDF1 (0.972); cả hai đều cao và IDSW = 0. Nếu MOTA cao mà IDF1 thấp thì thường là lỗi identity (cắt track / đổi ID): MOTA chỉ cộng ~1 lần mỗi ID switch, còn IDF1/AssA phạt theo phần đời track bị gán sai. Bài này không rơi vào pattern đó.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ReID tốt hơn control ở IDF1 (0.900 vs 0.875) và AssA (0.820 vs 0.776); IDSW cùng = 2 nhưng vị trí khác (ByteTrack: gold4@f59, gold5@f94; ReID: gold5@f87, gold6@f113). FN giảm mạnh 54→26 (ít mất dấu hơn), trong khi FP vẫn cao (~88–91). Overlay frame 109–111: xe chính (bus/SUV) hai bên khớp, nhưng ReID còn bbox thừa nền (T7, T27) — treatment cải thiện association/phủ track hơn là “sạch FP”. Đây là system comparison ByteTrack vs BoT-SORT, không isolate nhân quả ReID; sweep appearance 0.7–0.9 gần như phẳng củng cố điều đó.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng 0.649→0.711 chủ yếu nhờ FN giảm (54→26); FP gần như không giảm (88→91). IDSW vẫn 2 và vẫn có tách track (gold 5/6/7) → association chưa hết lỗi, nhưng phần “còn lại” lớn hơn là detector: nhiều bbox thừa/ghost (ReID ID 7, 27, 38…) và bbox lệch (IoU thấp).`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 109–111: ReID có T7 (xe xa nền) và T27 (sedan gần bus) trong khi nhãn tay không gán — khớp finding “bbox chỉ model có”. Theo schema lab (chỉ xe bốn bánh rõ, không đoán quá mờ/xa), đây nghiêng về FP của model/detector, không phải bạn bỏ sót bắt buộc.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`ReID vs bạn báo track tay ID 5 bị tách phía model (ID 17→18 @ frame 87) và nhiều frame “chỉ model có”. Overlay 109–111 khiến mình soi lại xe nền/sedan cạnh bus: nếu nhìn rõ là xe bốn bánh thì có thể bổ sung; còn quá nhỏ/mờ thì giữ nguyên và ghi model sai (FP). Riêng lỗi annotation đã biết từ gold (ID 4 treo 149–151; ID 5 IoU thấp 96–97) vẫn cần sửa trên CVAT dù model không phải đáp án.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Siết checklist Outside trước khi Save/export; bắt buộc tua giữa hai keyframe xa nhất mỗi track; ghi mọi ca track ngắn (<5 frame) vào guideline. Quy trình: một xe một mạch → Save bằng nút đĩa → reload xác nhận → mới sang xe khác; warm-up clip ngắn trước mỗi batch clip dài.`

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
- [ ] `reports/review_partner.md` (không nộp — làm cá nhân)
- [x] `reports/REPORT.md` (file này)
