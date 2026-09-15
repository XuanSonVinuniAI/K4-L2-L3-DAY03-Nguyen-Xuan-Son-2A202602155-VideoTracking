# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Xuân Sơn`
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

Bổ sung của nhóm (nếu có): `Không có`

Quy tắc bổ sung: Chỉ gán đối tượng là xe bốn bánh thực sự xuất hiện trong cảnh chính. Không gán các hình ảnh/phản chiếu của xe không thuộc cảnh thực như ảnh quảng cáo, hình trong gương hoặc phản chiếu dưới bóng nước.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Đây vẫn được xem là cùng một đối tượng và cần duy trì identity liên tục. |
| Xe bị che lâu hơn ngưỡng trên | Tạo ID mới khi xe được xác định lại | Sau thời gian che dài, không đủ chắc chắn để khẳng định identity vẫn là cùng track. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Khi xe rời khỏi frame, lần xuất hiện lại được xem là một lần xuất hiện mới. |
| Hai xe cắt nhau / chồng lên nhau | Giữ nguyên ID của từng xe, không đổi ID chỉ vì hai bbox chồng/cắt nhau | Identity dựa trên đối tượng thực tế, không dựa trên vị trí bbox; cần theo dõi đặc điểm hình dáng, vị trí và chuyển động để tránh ID switch. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: không đặt ngưỡng kích thước cố định (pixel), ưu tiên khả năng nhận diện chắc chắn. |
| Xe đang đỗ, không di chuyển | Vẫn gán và duy trì track nếu xe thuộc phạm vi annotation và còn xuất hiện trong cảnh; trạng thái đứng yên không phải lý do để bỏ track. |
| Keyframe đặt dày ở đâu | Đặt keyframe dày hơn tại các đoạn có chuyển động nhanh, xe đổi hướng, che khuất, hai xe giao nhau/chồng lấn hoặc bbox thay đổi mạnh; các đoạn chuyển động ổn định có thể đặt thưa hơn. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: Clip 1 / từ frame 59 đến 80 / ID 4,5,6
- Tình huống: Xe buýt ID 4 che một phần xe con ID 5 và ID 6
- Quyết định: Giữ nguyên ID 4, 5, 6; bbox của xe con chỉ ôm phần nhìn thấy được
- Lý do: Các xe vẫn có thể nhận diện riêng dù bị che một phần; không đổi ID chỉ vì bbox bị chồng/che

### Ca 2
- Clip / frame / ID: Clip 2 / từ frame 6 đến 17 / ID 1
- Tình huống: Xe con ID 1 xuất hiện trong gương/ảnh phản chiếu của xe buýt ID 6
- Quyết định: Không gán / xóa track ID 1
- Lý do: Theo phạm vi annotation, không gán xe xuất hiện trong gương hoặc ảnh phản chiếu; chỉ gán xe thực sự xuất hiện trong cảnh chính.

### Ca 3
- Clip / frame / ID: Clip 1 / frame 102 / ID 7
- Tình huống: Xe mới xuất hiện ở mép ảnh, ban đầu chỉ nhìn thấy một phần rất nhỏ/đèn xe (đèn xe) 
- Quyết định: Chưa gán ở frame này; bắt đầu track ở frame đầu tiên xác định chắc chắn là xe bốn bánh
- Lý do: Phần hình ảnh hiện tại chưa đủ thông tin để xác định loại đối tượng, tránh tạo false positive

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Thời điểm bắt đầu track:** Không chỉ yêu cầu “xác định chắc chắn là xe”, mà cần quy định rõ: bắt đầu track tại **frame đầu tiên mà xe bốn bánh đã đủ rõ để xác định**, không gán các frame trước đó khi chỉ thấy một phần quá nhỏ hoặc chưa thể phân biệt chắc chắn.

- **Thời điểm kết thúc track:** Khi xe **rời khỏi khung hình hoàn toàn**, phải kết thúc track tại frame cuối cùng còn nhìn thấy xe; không tiếp tục giữ bbox thêm các frame sau khi xe đã rời khỏi cảnh.

- **Xe xuất hiện ở mép ảnh:** Nếu chỉ thấy một phần rất nhỏ và chưa đủ thông tin để xác định là xe bốn bánh thì **chưa tạo ID**. Khi đến frame đầu tiên có thể xác định chắc chắn, bắt đầu track từ frame đó và bbox chạm đúng rìa ảnh nếu xe bị cắt bởi mép ảnh.

- **Bbox khi xe bị che khuất:** Bbox chỉ bao quanh **phần xe thực sự nhìn thấy**, không đoán phần bị che. Khi mức che khuất thay đổi, cần chỉnh keyframe để bbox tiếp tục bám sát phần nhìn thấy.

- **Kiểm tra sau khi gán:** Bổ sung bước kiểm tra riêng các frame **ngay trước/sau thời điểm bắt đầu và kết thúc track**, vì đây là nơi dễ tạo bbox dư hoặc thiếu. Sau khi rework cần chạy lại evaluator để xác nhận metric thay đổi đúng hướng.
