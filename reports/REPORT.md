# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Xuân Sơn`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `...` |
| Thời gian gán `clip_02` (warm-up) | 35 phút |
| Thời gian gán `clip_01` | 75 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | **76,75 keyframe/track**|

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe buýt ID 4 che một phần xe con ID 5 và ID 6:** Giữ nguyên ID 4, 5, 6; mỗi xe vẫn có bbox riêng và bbox của xe bị che chỉ ôm phần nhìn thấy được. Không đổi ID chỉ vì các xe bị chồng lấn.

2. **Xe con ID 1 xuất hiện trong gương của xe buýt ID 6:** Không gán xe này vì đây là hình ảnh trong gương, không phải xe thực sự xuất hiện trong cảnh chính. Nếu đã tạo ID 1 cho trường hợp này thì xóa/sửa lại track.

3. **Xe mới xuất hiện ở mép ảnh, ban đầu chỉ thấy một phần rất nhỏ/đèn xe:** Chưa gán ngay. Chỉ bắt đầu track tại frame đầu tiên có thể xác định chắc chắn đó là xe bốn bánh, tránh tạo nhãn sai khi đối tượng còn quá mơ hồ.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: **Kiểm tra ID xuyên suốt clip, phát hiện và sửa trường hợp xe trong gương bị gán nhầm ID 1; kiểm tra các xe bị che/chồng lấn vẫn giữ đúng ID.**

- Lượt 2: **Kiểm tra frame bắt đầu/kết thúc của từng track; bảo đảm xe chỉ được tạo ID khi đủ rõ để nhận diện và kết thúc track đúng khi xe rời khỏi cảnh.**

- Lượt 3: **Kiểm tra các đoạn giữa track, đặc biệt các đoạn xe bị che khuất, giao nhau hoặc bbox thay đổi mạnh; chỉnh bbox để bám sát phần xe nhìn thấy và tránh ID switch.**

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | aaaf41870fdc48402c8308b9e7bd7d285034608daeafccaad069de249d3c861c |
| Thời điểm khóa | 2026-09-15T05:37:19.143110+00:00 |
| Số row / frame / track trước khi mở reference | 614 rows / 190 frame / 8 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8114 | 0.7960 | 0.8291 | 0.8749 | 0.9621 | 0.9215 | 0.8623 | 43 | 2 | 0 |
| Sau rework | 0.820 | 0.805 | 0.837 | 0.875 | 0.966 | 0.930 | 0.863 | 37 | 3 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Track xuất hiện quá sớm | 76–78; 79–100 | 5; 6 | Cắt các bbox trước thời điểm xe thực sự xuất hiện theo reference. |
| Track kéo dài quá thời điểm kết thúc | 149–151; 169–171 | 4; 8 | Xóa bbox sau khi xe đã rời khỏi khung hình. |
| Bbox chưa sát xe | 81, 82, 83, 91, 92, 96 | 5 | Chỉnh lại bbox ID 5 sát phần xe nhìn thấy, giảm vùng thừa. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | yolo26n.pt / bytetrack.yaml / botsort-reid.yaml  |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / 2, 5, 7 |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.820 | 0.805 | 0.837 | 0.875 | 0.966 | 0.930 | 0.863 | 37 | 3 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.745 | 0.689 | 0.809 | 0.877 | 0.880 | 0.759 | 0.866 | 85 | 61 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi thấp hơn IDF1: sau rework MOTA = **0.930**, còn IDF1 = **0.966**. MOTA phản ánh tổng hợp lỗi FP, FN và IDSW nên có thể vẫn cao khi detection và localization tốt. Nếu MOTA cao nhưng IDF1 thấp, điều đó cho thấy số lượng bbox đúng có thể tốt nhưng việc duy trì đúng identity giữa các frame còn kém. MOTA không phạt nặng lỗi ID vì mỗi ID switch chỉ đóng góp một lỗi trong công thức MOTA, trong khi IDF1 đánh giá trực tiếp mức độ duy trì đúng identity qua toàn bộ track.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với gold, ByteTrack có **IDF1 = 0.875, AssA = 0.776, IDSW = 2**, còn BoT-SORT + ReID có **IDF1 = 0.900, AssA = 0.820, IDSW = 2**. Như vậy ReID treatment tốt hơn ở IDF1 và AssA, cho thấy khả năng association/duy trì identity tốt hơn; tuy nhiên IDSW không giảm, vẫn là 2.

Ở sequence **frame 87 và frame 112**, kết quả ReID vẫn ghi nhận ID switch: tại frame 87, GT track 5 đổi từ prediction ID 17 sang 18; tại frame 112, GT track 6 đổi từ prediction ID 28 sang 31. Vì vậy ReID cải thiện association tổng thể nhưng không loại bỏ hoàn toàn lỗi identity.

Kết quả này **không chứng minh causal effect riêng của ReID**, vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau, không chỉ khác mỗi thành phần ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Sau rework, annotation của tôi có **DetA tăng từ 0.796 lên 0.805**, FP giảm từ **43 xuống 37**, nhưng FN tăng nhẹ từ **2 lên 3**. IDSW vẫn bằng **0** và AssA tăng từ 0.829 lên 0.837. Vì vậy lỗi còn lại của annotation chủ yếu nằm ở **bbox/coverage và thời điểm xuất hiện/kết thúc track**, không phải lỗi association identity. Với model, các lỗi FP/FN và DetA thấp hơn cho thấy detection/coverage vẫn là một nguồn sai lệch đáng kể.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Frame 87, ID 5:** annotation của tôi giữ identity của xe theo đúng track tham chiếu, trong khi ReID bị ID switch, đổi prediction từ **ID 17 sang ID 18**. Đây là lỗi association của ReID vì cùng một GT track 5 bị tách thành hai prediction track.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Frame 112, ID 6:** ReID đổi từ prediction **ID 28 sang ID 31** đối với GT track 6. Đây là điểm cần xem lại annotation vì model báo dấu hiệu association khó. Tuy nhiên evidence cho thấy annotation của tôi vẫn giữ **ID 6 liên tục**, trong khi pre-gold có **IDSW = 0** và sau rework vẫn **IDSW = 0**. Vì vậy chưa có bằng chứng để đổi ID annotation chỉ dựa trên prediction của ReID.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

  - Tôi sẽ sửa `GUIDELINE_MINI.md` theo hướng cụ thể hóa hơn các ca dễ gây sai lệch: (1) xe bị che khuất và quy tắc giữ ID, (2) xe rời khung rồi quay lại, (3) xe xuất hiện ở mép ảnh nhưng chưa đủ rõ, (4) xe trong gương/phản chiếu/quảng cáo, và (5) thời điểm bắt đầu/kết thúc track. Tôi cũng sẽ bổ sung ví dụ frame thực tế cho các ca này để người gán có cùng cách xử lý.

  Trong quy trình, tôi sẽ luôn làm 3 lượt self-check: lượt 1 kiểm tra ID xuyên suốt track, lượt 2 kiểm tra frame đầu/cuối, lượt 3 kiểm tra các đoạn giữa có che khuất hoặc chuyển động mạnh. Sau đó khóa pre-gold trước khi xem gold/reference. Khi có rework, tôi sẽ ghi lại frame + ID + loại lỗi + cách sửa và chạy lại evaluator để kiểm tra metric trước/sau thay vì chỉ sửa theo cảm tính.

  Tôi cũng sẽ ưu tiên kiểm tra các track có bbox thay đổi mạnh, các điểm xe bắt đầu/kết thúc xuất hiện và các vùng có nhiều xe chồng lấn, vì đây là những vị trí dễ tạo FP/FN hoặc ID switch nhất.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
