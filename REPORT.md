# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: 2A202602100
- Họ và tên: Bùi Công Mạnh
- Ngày / CVAT local: 17/09/2026 / http://localhost:8080
- Công cụ đã dùng: Brush, Polygon, AI Detection/Segment hỗ trợ và Eraser tinh chỉnh thủ công

## 1. Bài đã nộp

Tất cả 9/9 task đã được hoàn thành, lưu (Save) trên CVAT và Export đúng định dạng quy định vào thư mục `submissions/`. Toàn bộ các file ZIP đều đạt chuẩn kiểm tra cấu trúc của bài lab.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: Ảnh `000000181542.jpg`, đối tượng `person` đứng ở khu vực trung tâm nửa trên ảnh (tọa độ bbox khoảng x=426..559, y=119..317).
- Class và quy tắc tôi dùng để chọn biên: Class `person`. Quy tắc biên: Chỉ vẽ theo đường viền cơ thể và trang phục nhìn thấy thực tế (visible boundaries); phóng to để bám sát viền vai, chân và nếp áo; dừng mask ở mép tiếp xúc với mặt đất/vật cản, tuyệt đối không vẽ lấn sang vùng bóng đổ (`shadow`) và không tự suy đoán biên phần bị che khuất.
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: Sau khi tự vẽ đối tượng đầu tiên, tôi bật gợi ý tự động cho các đối tượng xe ô tô xung quanh. Gợi ý tự động nhận diện đúng vị trí nhưng bị lem biên sang vùng bóng đen dưới gầm xe và dính chùm các xe đỗ sát nhau. Tôi đã dùng Eraser/Brush xóa bỏ phần bóng đổ và tách rời biên giữa các xe, vì bóng đổ thuộc về mặt đường (`road`), không thuộc cấu trúc vật thể xe.
- Nếu không dùng gợi ý: Không áp dụng (đã nêu chi tiết quá trình tự vẽ đối tượng đầu tiên và tinh chỉnh gợi ý tự động ở trên).

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: Task `cp2_slice`, ảnh `000000017627.jpg`, khu vực hai xe ô tô (`car`) đỗ sát cạnh nhau.
- Lỗi thuộc loại: gộp-tách (Merge error — gộp nhầm 2 xe thành 1 instance).
- Bằng chứng tôi nhìn thấy: Do hai xe cùng màu và đỗ tiếp giáp sát nhau, mask ban đầu vô tình gộp chung cả hai chiếc xe thành một mask lớn duy nhất.
- Quy tắc và hành động sửa: Theo quy tắc Instance Segmentation và tiêu chí kiểm tra của `cp2_slice`, mỗi cá thể xe dù cùng class và chạm nhau vẫn bắt buộc phải là một instance riêng biệt. Tôi đã phóng to (zoom in), lần theo đường rãnh phân cách giữa hai thân xe để cắt đôi mask, tạo thêm một object mới với ID độc lập cho chiếc xe thứ hai.
- Sau sửa đã Save và export lại chưa? Đã Save đầy đủ trên CVAT và xuất lại file `cp2_slice.zip` hợp lệ vào thư mục `submissions/`.
- Kết quả tự đánh giá / chạy script scoring: Sau khi chạy công cụ chấm đối chiếu với bộ tham chiếu `tiers_gt`, kết quả đạt được:
  - `easy_semantic`: mIoU **0.840** (điểm: **19.6 / 20**; coverage: 99.2%)
  - `medium_instance`: Mean Matched IoU **0.789**, Recall@0.5 **0.82** (điểm: **17.4 / 32**)
  - `hard_panoptic`: Panoptic Quality (PQ) **0.359**, SQ 0.583, RQ 0.454 (điểm: **10.6 / 30**)
  - **Tổng điểm 3 tier**: **47.6 / 82** trên `scorecard.py`, toàn bộ kết quả không có cờ cảnh báo bất thường (`no review flags`).

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp4_curb`<br>(ảnh `7d83710e-4697c3b2.jpg`, gờ bó vỉa hè) | (1) Gán vào `road` vì bề mặt bê tông/nhựa đường cùng màu xám tối;<br>(2) Gán vào `sidewalk` vì là cấu trúc bó vỉa ngăn cách và phân tầng cao hơn. | Quy tắc bài lab nêu rõ ranh giới giữa `road` và `sidewalk` phân chia theo công năng sử dụng và cao độ gờ bó vỉa, không thuần túy theo màu sắc điểm ảnh. | Quyết định gộp gờ bó vỉa vào lớp `sidewalk` để thể hiện đúng ranh giới an toàn cho người đi bộ. |
| `cp1_holes`<br>(ảnh `000000144300.jpg`, cửa kính ô tô trong suốt) | (1) Khoét lỗ rỗng phần kính vì nhìn xuyên thấu cảnh vật/cây xanh phía sau;<br>(2) Tô phủ kín cả kính xe vào mask của ô tô. | Tiêu chí `cp1_holes` quy định kính/khe hở nằm trong chu vi vật thể vẫn giữ nguyên bên trong mask, không được khoét rỗng tùy tiện. | Quyết định vẽ trùm liên tục toàn bộ khung kính vào đối tượng `car`, không khoét rỗng. |
| `cp5_occlusion`<br>(ảnh `000000336232.jpg`, thân xe bị cột cản che cắt đôi) | (1) Tách thành 2 object `car` riêng vì vùng pixel nhìn thấy bị chia làm 2 phần rời rạc;<br>(2) Giữ nguyên 1 instance `car` với đa giác phức (multi-polygon). | Quy tắc Instance Segmentation cho đối tượng bị che khuất: cùng một thực thể vật lý bị ngắt quãng vẫn chỉ tính là một đối tượng duy nhất (1 instance). | Quyết định tạo 1 annotation duy nhất với 2 polygon cho phần đầu và đuôi xe nhìn thấy, không tạo thêm ID mới. |


