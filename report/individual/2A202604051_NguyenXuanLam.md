# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202604051
- Ngày / CVAT local: 11–12/11/2026 / CVAT local (Docker, localhost:8080)
- Công cụ đã dùng: CVAT local — Polygon, Brush, Join. Easy và Medium vẽ tay hoàn toàn. Hard: dùng YOLO11x-seg + SAM 2.1 (Ultralytics, chạy offline trên laptop) để tạo gợi ý mask cho các vật (car, truck, bus, person, motorcycle, traffic light), import vào CVAT dạng COCO 1.0 rồi tự kiểm, xóa gợi ý sai và sửa; 5 class stuff (road, sidewalk, building, vegetation, sky) vẽ tay bằng Polygon. Có thử SAM 3 trên CVAT Online nhưng công cụ báo lỗi "n.shapes is not iterable" nên không dùng.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | chưa có | 0 / 1 | 3 |
| cp2_slice | chưa có | 0 / 1 | 3 |
| cp5_occlusion | chưa có | 0 / 1 | 3 |
| cp3_thin | chưa có | 0 / 1 | 3 |
| cp4_curb | chưa có | 0 / 1 | 3 |
| cp6_coverage | chưa có | 0 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Ba ZIP đã chạy `python scripts/inspect_submissions.py --dir submissions` và đều báo OK. Sáu checkpoint chưa làm kịp do ưu tiên thời gian cho ba tier chính.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: ảnh `000000373353.jpg`, chiếc taxi vàng ở giữa ảnh, phía trước có người mặc áo xám đứng che một phần thân xe.
- Class và quy tắc em dùng để chọn biên: `car`. Em lấy viền ngoài của thân xe, gồm cả kính, gương và bánh xe; không lấy bóng đổ trên mặt đường; không lấy phần thân người đang đứng che phía trước, vì chỉ vẽ phần xe nhìn thấy được.
- Nếu dùng gợi ý sau đó: ở Medium em **không dùng** gợi ý, vẽ tay toàn bộ bằng Polygon và Brush. Gợi ý chỉ dùng ở Hard. Tại đó, YOLO tạo một polygon `truck` rất lớn ôm nhầm tòa nhà vàng và cửa hàng lốp bên trái ảnh `000000460147.jpg` — đó là nhà, không phải xe, nên em xóa. Các mask xe trên đường bám khá sát thân xe nên em giữ, chỉ xóa vài gợi ý xe li ti ở xa mà em không chắc. Gợi ý cũng bỏ sót người đạp xe và chiếc xe đạp ở giữa làn phải ảnh `000000350023.jpg`; em chưa kịp vẽ bổ sung hai vật này.

## 3. Một lỗi em tìm thấy và sửa

- Task/ảnh/vùng: `hard_panoptic`, ảnh `000000460147.jpg`, toàn bộ mặt đường của hai làn xe.
- Lỗi thuộc loại: sai lớp.
- Bằng chứng em nhìn thấy: khi tô màu bài theo label, mặt đường hiện màu của `sky`; ảnh này có 3 object `sky` nhưng không có object `road` nào. Nguyên nhân là lúc vẽ polygon mặt đường, ô Label vẫn đang chọn `sky`.
- Quy tắc và hành động sửa: mặt đường xe chạy là `road`. Em đổi label của 2 polygon mặt đường từ `sky` sang `road`, chỉ giữ 1 polygon `sky` cho phần trời phía trên ảnh.
- Sau sửa đã Save và export lại chưa? Đã Save, export lại COCO 1.0 và ghi đè `hard_panoptic.zip`; kiểm lại thì ảnh có 1 `sky` và 2 `road`.

Kết quả tự đánh giá trên GitHub Actions: chưa có điểm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `000000460147.jpg` (Hard), các ô tô đang nằm trên xe chở ô tô ở giữa ảnh | Là một phần của object `truck` / là từng `car` riêng | Nhìn thấy rõ hình dạng, kính và bánh của từng chiếc xe hơi | Vẽ xe chở là `truck`, mỗi ô tô trên đó là một `car` riêng. Câu hỏi cho coach: đáp án quy ước trường hợp này thế nào? |
| 2. `000000350023.jpg` (Hard), xe màu trắng-đỏ ở góc dưới bên phải | `bus` / loại xe khác (xe điện, xe du lịch) | Thân dài, nhiều cửa sổ, chở khách; không có class xe điện trong danh sách | Gán `bus` |
| 3. `7ee6d192-89e2408b.jpg` (Easy), đồi cỏ khô màu nâu và dải bê tông chắn bên trái đường | Đồi cỏ là `vegetation`, dải bê tông là `building` / để trống | Đồi cỏ khô không phải cây xanh; dải bê tông chắn không phải nhà; cả hai không thuộc rõ class nào trong 5 class của task | Để trống cả hai vùng |
