# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Lê Ngọc Nam — làm cá nhân`
Ngày: `15/09/2026`

> Báo cáo này dùng chỉ số frame MOT 1.1 (frame đầu là 1) và chỉ ghi các dữ kiện
> kiểm chứng được từ annotation, validator, các file JSON đánh giá và cấu hình
> model hiện có trong repo.

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT, export định dạng MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | Không có dữ liệu thời gian được ghi nhận |
| Thời gian gán `clip_01` | Không có dữ liệu thời gian được ghi nhận |
| Số track đã vẽ trong `clip_01` | 8 track, ID 1–8 |
| Số keyframe trung bình mỗi track | Không thể suy ra từ file MOT 1.1 vì định dạng này không lưu cờ keyframe |

Ba tình huống cần rà soát kỹ nhất theo annotation và kết quả đánh giá:

1. ID 3 gần như đứng yên ở frame 1–15 nhưng vẫn là xe hợp lệ; cần giữ track khi xe còn nhìn thấy, không dùng `Outside` chỉ vì xe không di chuyển.
2. Vùng frame 73–106 có nhiều xe chồng lấn. Annotation giữ riêng ID 4, 5 và 6 nên không có ID switch hay fragmentation, nhưng biên bắt đầu của các track và bbox nội suy còn rộng.
3. ID 8 đi vào từ mép dưới rồi ra mép phải ở khoảng frame 134–169; cần đặt bbox chạm đúng rìa, không đoán phần ngoài ảnh và kiểm frame-by-frame tại entry/exit.

## 2. Tự kiểm và kiểm chéo

Phần này được bỏ qua theo yêu cầu. Repo hiện không có
`reports/review_partner.md`, vì vậy báo cáo không tạo dữ liệu reviewer, finding
hoặc closure giả.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| Kết quả được cung cấp | `ban_vs_gold` trong `outputs/eval_vs_gold.json` |
| Bản annotation được chấm | 611 row / 190 frame / 8 track |
| SHA-256 và thời điểm khóa pre-gold | Không có `evidence/pre-gold/clip_01/manifest.json` để xác minh |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `ban_vs_gold` | 0.799 | 0.778 | 0.823 | 0.864 | 0.956 | 0.909 | 0.850 | 45 | 7 | 0 |

**Cổng annotation: ĐẠT** — `{'IDF1': 0.956, 'MOTA': 0.909, 'MOTP': 0.850}`.

Bảng được cung cấp xác nhận chất lượng của bản annotation hiện tại khi so với
gold. Vì không có snapshot/manifest pre-gold và không có một hàng metric
`pre_gold_vs_gold` riêng, báo cáo không thể tính mức thay đổi trước–sau rework.
Bản hiện tại vẫn có các finding sau và cần sửa trong CVAT rồi export lại nếu
tiếp tục rework:

| Loại lỗi | Frame MOT | ID của bản hiện tại | Trạng thái / hành động cần làm |
| --- | --- | --- | --- |
| Track bắt đầu sớm | 51–53 | 4 | Chưa có bằng chứng đã sửa; rà lại entry và bắt đầu từ frame đầu xác định chắc chắn là xe |
| Track bắt đầu sớm | 73–78 | 5 | Chưa có bằng chứng đã sửa; rà lại entry trước vùng chồng lấn |
| Track bắt đầu sớm | 79–100 | 6 | Chưa có bằng chứng đã sửa; đối chiếu từng frame và bỏ bbox trước lúc xe thực sự xuất hiện |
| Bbox lỏng / trôi | 82–84, 91–93, 106 | 5 | Thêm hoặc chỉnh keyframe quanh đoạn đổi hình dạng/che khuất |
| Bbox lỏng / trôi | 102, 105–107, 110–112 | 6 | Thêm hoặc chỉnh keyframe, chỉ ôm phần nhìn thấy |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` / `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / COCO `[2, 5, 7]` |
| device | CUDA device `0`; `persist=true`; 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.799 | 0.778 | 0.823 | 0.864 | 0.956 | 0.909 | 0.850 | 45 | 7 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.738 | 0.685 | 0.798 | 0.870 | 0.884 | 0.768 | 0.857 | 83 | 56 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của bản annotation là 0.909, thấp hơn IDF1 0.956 khoảng 0.047. Bản này
không có tình huống MOTA cao nhưng IDF1 thấp. Nói chung, nếu xuất hiện khoảng
cách đó thì detection tổng thể có thể vẫn tốt nhưng liên kết identity dài hạn
kém. MOTA cộng FP, FN và mỗi lần ID switch như các lỗi rời rạc; một track bị đổi
ID kéo dài nhiều frame có thể chỉ tạo một ID switch. IDF1 so identity trên toàn
bộ vòng đời nên phản ánh mạnh hơn hậu quả kéo dài của việc gán sai ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, treatment tăng IDF1 từ 0.875 lên 0.900 (+0.025) và AssA từ
0.776 lên 0.820 (+0.044), nhưng IDSW vẫn bằng 2. Ở gold track 4 trong chuỗi
frame 57–60, ByteTrack dùng ID 14 ở frame 57, mất liên kết ở frame 58 rồi xuất
hiện lại bằng ID 15 từ frame 59; evaluator ghi một switch tại frame 59. Treatment
giữ ID 9 xuyên chuỗi frame 57–60, phù hợp hơn với identity liên tục. Tuy vậy,
treatment vẫn switch gold track 5 tại frame 87 và gold track 6 tại frame 113.
Đây không phải causal ablation chỉ riêng ReID vì ByteTrack và BoT-SORT khác cả
implementation; kết quả chỉ mô tả hai hệ thống với cùng detector input.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Treatment tăng DetA từ 0.649 lên 0.711. FP tăng nhẹ từ 88 lên 91, trong khi FN
giảm mạnh từ 54 xuống 26; vì vậy cải thiện detection chủ yếu đến từ việc phủ được
nhiều bbox thật hơn. Lỗi còn lại thuộc cả hai nhóm: 91 FP và 26 FN cho thấy vẫn
có lỗi detector/coverage; 2 IDSW cùng fragmentation ở gold track 5, 6 và 7 cho
thấy association vẫn chưa ổn định.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Quanh frame 86–88, bản annotation giữ liên tục ID 5 và gold cũng giữ cùng một
track. ReID mất liên kết ngắn rồi đổi từ ID 17 sang ID 18 tại frame 87. Vì đối
tượng vật lý vẫn là cùng xe trước và sau đoạn che nên việc giữ ID liên tục của
annotation phù hợp hơn trong chuỗi này.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

So sánh ReID với annotation cho thấy ID 6 của bản gán dài 78 frame nhưng model
chỉ phủ khớp 49/78 frame và còn đổi 24 → 28 → 31 quanh frame 107–110. Điều này
buộc phải rà lại toàn bộ biên của ID 6. Kết quả so với gold xác nhận annotation
bắt đầu ID 6 quá sớm ở frame 79–100 (22 bbox thừa) và có bbox lỏng quanh frame
102–112. Vì vậy bất đồng với model đã chỉ đúng một vùng cần kiểm lại, dù các lần
đổi ID của chính model ở đoạn 107–113 vẫn là lỗi association của treatment.

## 6. Nếu phải gán thêm 10 clip nữa

Đề xuất quy trình: ghi rõ tiêu chí bắt đầu/kết thúc track trước khi annotate;
hoàn tất từng xe rồi mới sang xe khác; kiểm frame-by-frame quanh entry, exit,
occlusion và crossing; đặt keyframe dày tại chỗ đổi hướng hoặc bị che; chạy
validator và xem visualization ngay sau mỗi lần export. `GUIDELINE_MINI.md` cần
giữ quy tắc không dùng ngưỡng pixel cứng cho xe quá nhỏ, không tái sử dụng ID,
và không để nội suy sinh bbox trước khi xe thật sự đủ rõ để nhận dạng.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` — chưa có
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` — bỏ qua theo yêu cầu, chưa có
- [x] `reports/REPORT.md` (file này)
