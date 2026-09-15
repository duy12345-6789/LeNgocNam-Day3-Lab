# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Lê Ngọc Nam — làm cá nhân`
Clip: `clip_01`, `clip_02`

> Các frame ghi dưới đây dùng chỉ số MOT 1.1, bắt đầu từ 1. Nếu CVAT hiển thị
> frame đầu là 0 thì frame CVAT nhỏ hơn frame MOT một đơn vị.

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh, không phân loại chi tiết.

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | xe máy / mô tô |
| xe tải, xe đầu kéo | biển báo |
| xe đang đỗ nhưng còn nhìn thấy | xe trong quảng cáo, gương hoặc hình phản chiếu |

Bổ sung:

- Chỉ gán phương tiện thật xuất hiện trong cảnh.
- Xe đang đỗ vẫn được gán liên tục và giữ nguyên ID khi còn trong khung.
- Xe bị che một phần vẫn được gán nếu phần nhìn thấy đủ để xác định và đặt bbox.
- Không tạo bbox vô hình cho phần xe bị che hoàn toàn hoặc nằm ngoài ảnh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu thời gian bị che không quá 25 frame (2 giây ở 12.5 fps). Khi còn thấy một phần, bbox chỉ ôm phần nhìn thấy. | Che ngắn không làm thay đổi danh tính xe. |
| Xe bị che hoàn toàn dưới 25 frame | Dùng `Outside` trong các frame xe hoàn toàn vắng mặt, sau đó tiếp tục cùng track khi xe hiện lại. | Không vẽ bbox vô hình nhưng vẫn giữ được identity. |
| Xe bị che lâu hơn 25 frame | Kết thúc track cũ và mở track mới khi xe xuất hiện lại, trừ khi có bằng chứng liên tục đủ chắc chắn để coach xác nhận. | Khoảng vắng mặt dài làm tăng rủi ro gán nhầm identity. |
| Xe rời khung rồi quay lại | Tạo track mới, kể cả thời gian vắng mặt dưới 25 frame. | Ra khỏi khung kết thúc vòng đời quan sát hiện tại. |
| Hai xe cắt nhau hoặc chồng lên nhau | Dựa trên hướng di chuyển, vị trí trước/sau giao cắt và đặc điểm nhìn thấy; kiểm từng frame và giữ ID riêng. | Tránh đổi ID giữa hai xe vật lý. |
| Xe đứng yên | Giữ nguyên ID và tiếp tục track khi xe còn nhìn thấy. | Đứng yên không có nghĩa là xe đã rời cảnh. |

Không tái sử dụng một ID cũ cho một xe vật lý khác.

## 3. Luật bbox

| Tình huống | Luật |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa và chỉ ôm phần nhìn thấy; không đoán phần ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox ôm sát phần nhìn thấy, giữ nguyên ID và bật `Occluded` khi phù hợp. |
| Xe bị che hoàn toàn | Không vẽ bbox vô hình; dùng `Outside` tại frame đầu tiên xe hoàn toàn vắng mặt. |
| Xe vừa xuất hiện, còn rất nhỏ hoặc mờ | Không dùng ngưỡng pixel cứng. Chỉ bắt đầu khi đã đủ dấu hiệu nhận ra là xe bốn bánh và có thể đặt bbox ổn định; dùng frame lân cận để xác nhận. |
| Xe đang đỗ, không di chuyển | Vẫn gán đầy đủ và giữ một ID khi xe còn trong khung. |
| Keyframe đặt dày ở đâu | Tại entry/exit, occlusion, crossing, lúc rẽ, đổi tốc độ hoặc thay đổi nhanh kích thước bbox. Đi thẳng đều có thể đặt thưa hơn nhưng phải kiểm frame giữa. |

Quanh entry, exit, occlusion và crossing phải kiểm frame-by-frame. Không sửa số
frame hoặc `track_id` trực tiếp trong file MOT sau khi export.

## 4. Ít nhất ba ca mơ hồ có evidence trong annotation

### Ca 1

- Clip / frame / ID: `clip_01`, MOT frame `1–190`, ID `3`.
- Tình huống: Xe SUV trắng đang đỗ, gần như không thay đổi vị trí. Validator cảnh báo bbox ID 3 gần như đứng im từ frame 1 đến 15.
- Quyết định ghi nhận trong annotation: Giữ một track ID 3 khi xe còn nhìn thấy.
- Lý do: Xe đang đỗ vẫn thuộc lớp `vehicle`; đứng yên không phải lý do để kết thúc track hoặc bật `Outside`.

### Ca 2

- Clip / frame / ID: `clip_01`, MOT frame `79–100`, ID `4`, `5`, `6`.
- Tình huống: Xe buýt ID 4 và các xe ID 5, ID 6 chồng lấn; một số frame chỉ nhìn thấy một phần xe phía sau hoặc bên cạnh.
- Quyết định ghi nhận trong annotation: Giữ ba track riêng, bbox phần nhìn thấy và không đổi ID khi các xe tách ra.
- Lý do: Đây là các xe vật lý khác nhau. Hướng chuyển động và vị trí trước/sau vùng chồng lấn giúp duy trì identity; evaluator xác nhận bản hiện tại không có ID switch hoặc fragmentation.

### Ca 3

- Clip / frame / ID: `clip_01`, MOT frame `134–169`, ID `8`.
- Tình huống: Xe đi vào từ mép dưới khi ban đầu chỉ lộ một phần nhỏ, sau đó đi ra ở mép phải.
- Quyết định ghi nhận trong annotation: Tạo ID 8, bbox chỉ ôm phần trong ảnh và chạm rìa khi cần.
- Lý do: Chuỗi frame xác nhận đây là cùng một xe bốn bánh; không được đoán phần hình học nằm ngoài khung.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Kết quả `outputs/eval_vs_gold.json` đạt IDF1 0.9561, MOTA 0.9092, MOTP
0.8502 và IDSW 0. Không có track bị bỏ sót hoặc bị tách, nên giữ nguyên các luật
duy trì identity khi crossing và occlusion ngắn.

Các luật được làm rõ sau khi đọc chẩn đoán:

- Chỉ bắt đầu track ở frame đã có đủ dấu hiệu xác định là xe bốn bánh; không để
  nội suy tạo bbox ở các frame trước khi xe thực sự xuất hiện.
- ID 4 được mở sớm ở frame 51–53, ID 5 ở frame 73–78 và ID 6 ở frame 79–100.
  Các đoạn này phải được rà và sửa trong CVAT, không chỉnh trực tiếp file MOT.
- Với entry/exit, kiểm từng frame và dùng `Outside` tại frame đầu xe hoàn toàn
  vắng mặt; không kéo track thêm chỉ vì nội suy vẫn tạo được bbox ở rìa.
- Đặt thêm hoặc chỉnh keyframe quanh bbox lỏng, đặc biệt frame 82 của ID 5,
  frame 106 của ID 6 và frame 54 của ID 4.
- Repo không có `reports/review_partner.md`, vì vậy không ghi giả finding hoặc
  kết luận kiểm chéo. Các cập nhật trên chỉ dựa vào annotation, validator và
  kết quả đối chiếu đã có.
