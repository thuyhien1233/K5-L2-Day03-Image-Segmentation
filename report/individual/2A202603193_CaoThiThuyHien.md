# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: `2A202603193`
- Ngày / CVAT local: 11/11/2026 – 12/11/2026 / `http://localhost:8080`
- Công cụ đã dùng: CVAT local (Brush/Mask và công cụ gợi ý tự động — thể hiện qua định dạng mask RLE trong các export `COCO 1.0`, thuộc tính `source="auto"` trong export XML của `cp5_occlusion`, và PNG mask trong `Segmentation mask 1.1`)

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `submissions/cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `submissions/cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `submissions/cp5_occlusion.zip` *(hiện xuất nhầm định dạng CVAT XML `annotations.xml`, cần xuất lại `COCO 1.0`)* | 1 / 1 | 3 |
| cp3_thin | `submissions/cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `submissions/cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `submissions/cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Ghi chú trạng thái export: 8/9 file ZIP đạt kiểm tra cấu trúc (`easy_semantic.zip`, `medium_instance.zip`, `hard_panoptic.zip`, `cp1_holes.zip`, `cp2_slice.zip`, `cp3_thin.zip`, `cp4_curb.zip`, `cp6_coverage.zip`). Riêng `cp5_occlusion.zip` (Job 10 trên CVAT local, ảnh `000000336232.jpg`) đã Save đầy đủ 22 mask nhưng khi export đang xuất dưới dạng CVAT XML (`annotations.xml`) thay vì `COCO 1.0` (`annotations/*.json`), cần vào lại CVAT export đúng định dạng `COCO 1.0` (không tự sửa file bên trong ZIP).

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg` (`medium_instance`), vùng rìa trái khung hình (`bbox ≈ [34, 163, 71, 99]`, annotation `id=1` trong file `instances_default.json`): người đội mũ ngồi yên sau xe máy ngoài cùng bên trái, ngay phía sau người lái xe máy (`ann#3`). *(Ghi chú: trích theo annotation đầu tiên `id=1` trong export `medium_instance.zip`; học viên kiểm tra lại nếu thứ tự vẽ thực tế trước khi bật gợi ý là object khác.)*
- Class và quy tắc tôi dùng để chọn biên: Chọn class `person` theo đúng `data/tiers/medium_instance/classes.json`. Áp dụng quy tắc vẽ theo phần nhìn thấy và tách từng cá thể: chỉ tô phần thân, tay và mũ nhìn thấy của người ngồi sau (`ann#1`), dừng biên ngay tại ranh tiếp giáp với lưng/vai của người lái phía trước (`ann#3`) và không gộp chung với thân ô tô phía sau (`ann#2`).
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: Phần lớn các đối tượng tách biệt rõ ở 3 ảnh đều bám sát đường biên nhìn thấy nên được giữ lại (như `#15 person` mặc áo dài ở `000000181542.jpg`, `#35 bus` hai tầng ở `000000373353.jpg`, `#47 person` đi ván trượt ở `000000458325.jpg`). Tuy nhiên khi rà soát lại file export hiện tại vẫn còn một số lỗi từ đề xuất tự động chưa xử lý triệt để trên CVAT: ở `000000181542.jpg` còn gán trùng `#10 bicycle` và `#11 motorcycle` lên cùng bánh xe góc dưới trái (IoU ≈ 0.958), trùng lặp `#8` và `#13 motorcycle` (IoU ≈ 0.913), và xe máy sau người áo dài bị tách làm `#18` và `#19`; ở `000000373353.jpg` taxi vàng bị người đi bộ `#27 person` che giữa bị tách thành `#29` và `#30 car`.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp5_occlusion` / `000000336232.jpg` (toàn bộ file export và vùng xe máy phía trước bên trái `left=11..268, top=194..330`, cùng hai ô tô sát nhau ở góc dưới phải `left=372, top=202`).
- Lỗi thuộc loại: sai định dạng export kết hợp lỗi gộp-tách vật bị che và chồng trùng mask.
- Bằng chứng tôi nhìn thấy: Khi kiểm tra `submissions/cp5_occlusion.zip`, bên trong là `annotations.xml` (CVAT XML gồm 22 mask đều mang `source="auto"`) thay vì `annotations/instances_default.json` của `COCO 1.0`. Khi chiếu các mask lên ảnh `000000336232.jpg`, chiếc xe máy phía trước bên trái bị người lái (`#6 person`) che phần giữa đang bị tách thành 2 mask riêng (`#5` phần đầu và `#4` phần đuôi); hai ô tô sát nhau ở góc dưới phải bị gộp thành 1 mask `#2 car`; và `#12 car` chồng trùng lên `#22 car` (IoU ≈ 0.576). *(Ngoài ra khi rà soát chéo các task khác cũng ghi nhận lỗi gán trùng 2 class lên cùng 1 xe ở `hard_panoptic` và tô nhầm phần lớn vỉa hè phải thành `road` ở `cp6_coverage`.)*
- Quy tắc và hành động sửa: Theo `CHECKPOINTS.md` và `docs/SUBMISSION.md`, cần gộp hai phần nhìn thấy của cùng một xe máy bị che thành **1 instance `motorcycle` duy nhất** (không nối qua thân người lái), tách hai ô tô sát nhau góc dưới phải thành 2 instance `car` riêng, xóa mask chồng trùng, rồi Save và export lại đúng định dạng **`COCO 1.0`**.
- Sau sửa đã Save và export lại chưa? **Chưa sửa mask và chưa export lại sang `COCO 1.0`** tại thời điểm lập báo cáo (đã Save 22 mask hiện tại ở Job 10 trên CVAT local; cần mở lại Job 10 trên CVAT để chỉnh và xuất lại `COCO 1.0` trước khi nộp, không tự chỉnh tay file bên trong ZIP).

Kết quả tự kiểm tra cấu trúc tại máy (đối chiếu logic `scripts/inspect_submissions.py`): 8/9 ZIP đạt cấu trúc `OK`; riêng `cp5_occlusion.zip` báo `LỖI` do chứa `annotations.xml` thay vì COCO JSON. Hiện **chưa có điểm** tự đánh giá vì repo không chứa ground truth và chưa chạy scorer ba tier.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `cp4_curb` / `7d83710e-4697c3b2.jpg` (dải đất gốc cây và sân đỗ xe SUV trắng bên phải) | (1) Gộp cả dải đất/bó vỉa quanh gốc cây và phần sân bê tông đỗ xe bên phải vào `sidewalk`; (2) Chỉ gán `sidewalk` cho dải lối đi lát bê tông dành cho người đi bộ. | Task chỉ có 2 class `road` và `sidewalk`. Dải đất quanh gốc cây không phải mặt đường đi bộ, còn phần sân bên phải có khung chắn kim loại làm chỗ đỗ xe riêng. | Đã dừng `road` sát mép rãnh bó vỉa (14.71% ảnh) và tô `sidewalk` cho phần lối đi bộ lát bê tông (13.08% ảnh), để dải đất và bãi đỗ xe làm nền. Xin hỏi coach: lối xe ra vào cắt ngang hoặc dải đất cây xanh có cần gán `sidewalk` không? |
| 2. `easy_semantic` / `81ae7cbb-6bc63a4a.jpg` (tán cây mùa đông trụi lá góc trái trên) | (1) Tô bao trùm cả cụm cành thưa vào `vegetation`; (2) Chỉ tô phần thân/cành dày và giữ khoảng hở giữa các cành là `sky`. | Cây mùa đông không có lá, mạng cành nhỏ đan xen dày trên nền trời xanh (`sky` 33.80%, `vegetation` 12.12%); tách từng khe nhỏ dễ tạo viền nhiễu. | Đã chọn tô bao theo cụm tán cành chính bên trái là `vegetation`, còn một số cành mảnh thưa ở xa giữ theo nền trời/nền chưa tô (`background` 8.61%). Khi chấm mIoU, khoảng trời lọt giữa tán cây trụi lá tính theo `sky` hay gộp vào tán `vegetation`? |
| 3. `medium_instance` / `000000181542.jpg` (hành khách sau cửa kính xe buýt góc trên trái `#16 person` & `#20 bus`) | (1) Tách riêng một instance `person` cho phần đầu/vai hành khách nhìn thấy qua kính xe buýt; (2) Không gán `person` vì hành khách nằm bên trong khung `bus`. | Theo quy tắc `medium_instance`, cần đánh dấu từng `person` nhìn thấy và không khoét lỗ kính xe (`#20 bus` phủ nguyên khung xe buýt, còn `#16 person` có `bbox=[57, 104, 21, 23]`, diện tích 397 px nằm đè lên vùng cửa kính). | Hiện đã giữ `#20 bus` liền mảng và tách thêm `#16 person` cho phần đầu/vai hành khách nhìn rõ qua kính. Xin hỏi coach: các trường hợp người ngồi trong xe/sau kính có kích thước nhỏ như vậy có bắt buộc tách instance riêng không? |
