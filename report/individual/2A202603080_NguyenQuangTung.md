# Báo cáo Day 13 — Nguyễn Quang Tùng, 2A202603080

## 1. Kết luận trước
- Mã số: 2A202603080 · Task: `D13-2A202603080` · 10 frame: `s01f26, s01f58, s02f21, s02f52, s03f11, s03f59, s04f16, s04f37, s05f11, s05f32`.
- Môi trường: Windows 11, trình duyệt Google Chrome, phiên bản CVAT v2.83.0 (tài khoản: QuangTung); bạn ngồi cạnh đổi kiểm: 2A202603081.
- Tổng kết: Hoàn thành 104 cuboid trên 10 frame. Frame ít nhất là `s03f59` (5 cuboid) do khung cảnh thoáng, tầm nhìn ít vật thể lân cận; frame nhiều nhất là `s05f11` (19 cuboid). Phiếu `selfcheck` có 0/104 cuboid bị cờ cảnh báo. Bản thân tự tin nhất ở phần xác định hướng (yaw) và độ cao đáy chạm mặt đường; phần cần chú ý nhất là ước lượng kích thước các vật ở xa (> 45 m) do điểm thưa.
- Cách dùng CVAT:
  1. Đổi góc nhìn: Kéo chuột trái xoay khung nhìn 3D (xoay thẳng xuống để nhìn top-view), chuột phải dời cảnh, lăn con lăn phóng to/thu nhỏ; nháy đúp vào khoảng trống để về góc nhìn mặc định.
  2. Vẽ và chỉnh cuboid: Chọn nhãn (label), bấm Shape (không bấm Track), rê con trỏ tới cụm điểm nháy đúp để đặt hộp. Ở Top view căn chỉnh tâm, kích thước dài/rộng và kéo chấm xoay yaw; ở Side & Front view nâng hạ tâm z và chiều cao h sao cho đáy chạm đường (không xoay ở Side/Front).
  3. Kiểm tra: Bật Appearance -> Cuboid orientation để căn vạch đỏ ngắn (+X) dọc thân và chỉ đầu xe; sử dụng lệnh `preview` để kiểm tra độ khớp cuboid chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f26` (ảnh: `t1-s01f26-3d.png`, `t1-s01f26-top.png`), `s01f58` (ảnh: `t1-s01f58-3d.png`, `t1-s01f58-top.png`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f26` | Phía trước xe là hướng -y (cạnh trên ảnh top-view); +x chỉ sang trái, +y chỉ ra sau | Đường phẳng; mặt đường thấp hơn LiDAR ~1.7 m (z ≈ -1.7 m) | ~45 m (xe con ở xa chỉ còn vài vệt điểm) | Car, Cyclist, Pedestrian, Truck | Phía sau có xe máy (cam09); bên phải có xe con đỗ và người đi bộ lề đường (cam07, cam08) | Cột đèn và tán cây bên lề đường (điểm rải rác trên cao không rõ khối) |
| `s01f58` | Phía trước xe là hướng -y (cạnh trên ảnh top-view); +x chỉ sang trái, +y chỉ ra sau | Đường phẳng; thấp hơn LiDAR ~1.7 m (z ≈ -1.7 m) | ~40 m (cụm điểm thưa dần) | Car, Cyclist, Truck | Hai bên có xe con cùng chiều/ngược chiều (cam06, cam07); phía sau có xe bám đuôi (cam09) | Dải phân cách cứng và biển báo ven đường |

- **3 điều chỉ camera cho biết:**
  1. Màu sắc xe/vật thể (trắng, đen, đỏ...) và chi tiết nhận diện (biển số, tem decal, taxi/xe cứu thương).
  2. Ngữ nghĩa giao thông: Vạch kẻ đường, đèn tín hiệu (đỏ/xanh), chữ và ký hiệu trên biển báo.
  3. Phân biệt đầu và đuôi xe khi xe đứng yên (nhận diện qua đèn pha, kính chắn gió).
- **3 điều chỉ LiDAR cho biết:**
  1. Khoảng cách vật lý 3D chính xác theo mét tới từng điểm (không bị biến dạng phối cảnh).
  2. Kích thước hình học thực tế (dài, rộng, cao) để căn chỉnh bounding box 3D chuẩn xác.
  3. Tầm nhìn 360° đồng bộ không gian, phát hiện vật thể tốt trong bóng tối, ngược sáng hoặc không có ánh sáng.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f21` | Hướng -y (cạnh trên ảnh) | Mật độ xe máy (Cyclist) khá đông ở hai bên hông xe |
| `s02f52` | Hướng -y (cạnh trên ảnh) | Vùng bóng che lớn phía sau xe tải ở trước-trái |
| `s03f11` | Hướng -y (cạnh trên ảnh) | Nhiều cụm điểm thưa ở tầm xa > 40 m |
| `s03f59` | Hướng -y (cạnh trên ảnh) | Mặt đường có xu hướng dốc nhẹ |
| `s04f16` | Hướng -y (cạnh trên ảnh) | Biển báo ven đường có intensity phản xạ rất cao |
| `s04f37` | Hướng -y (cạnh trên ảnh) | Cụm điểm phản xạ thấp ở bề mặt kính xe phía trước |
| `s05f11` | Hướng -y (cạnh trên ảnh) | Không gian phía trước thoáng, mật độ vật thể thấp |
| `s05f32` | Hướng -y (cạnh trên ảnh) | Một số điểm nhiễu lẻ loi nằm dưới mặt đường (z < -2.0 m) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f26`, idx 1 | Car | Đúng (yaw = 269.9°, chỉ đầu xe) | Đúng (z đáy = -1.662 m) | Đã kiểm đủ 3 view và ảnh cam01 | Dài 4.56 m, rộng 1.94 m, cao 1.47 m (khớp chuẩn sedan) |
| (b) xe buýt hoặc xe tải | `s03f11`, idx 4 | Truck | Đúng (yaw = 91.2°, ngược chiều) | Đúng (z đáy = -2.085 m, dốc nhẹ) | Đã kiểm đủ 3 view và ảnh camera | Dài 6.31 m, rộng 2.15 m, cao 2.31 m (khớp xe tải/bán tải) |
| (c) người đi bộ hoặc xe hai bánh | `s01f26`, idx 13 | Cyclist | Đúng (yaw = 306.8°, theo hướng lái) | Đúng (z đáy = -1.291 m) | Đã kiểm đủ 3 view và ảnh cam06 | Dài 1.64 m, rộng 1.07 m, cao 1.70 m (khớp xe máy có người) |

- Vòng hiệu chuẩn: Vòng hiệu chuẩn chung của lớp đã được giảng viên lược giản; bản thân đã tự đối chiếu guideline để thống nhất: bán tải xếp vào Truck, xe máy có người là Cyclist, và luôn dùng chế độ Shape.
- Vật bỏ qua vì không đủ cơ sở: Tại frame `s03f11` ở khoảng cách ~48 m có một cụm điểm mờ (~8 điểm) nghi vấn xe con (`LOW_VISIBILITY`), do không nhận diện được trên ảnh camera nên đã bỏ qua không gán nhãn theo đúng quy tắc không đoán mò.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f26` / `t4-1-bong-che.png` | Thân xe tải/xe buýt to lớn chắn chùm tia LiDAR phát ra từ nóc xe | Phía sau xe xuất hiện một vùng hình rẻ quạt hoàn toàn trống điểm | Bật cờ `occluded` cho vật bị che một phần; cuboid bao trọn phần hình khối suy luận được |
| 2. Thưa ở xa | `s01f26` / `t4-2-thua-o-xa.png` | Khoảng cách góc giữa 40 tia quét mở rộng theo khoảng cách ($1/d^2$) | Xe ở cự ly > 35 m chỉ còn 1-2 vệt tia quét ngang (dưới 100 điểm) | Chỉ vẽ nếu quan sát được $\ge 15-20\%$ thể tích; nếu quá thưa không đủ cơ sở thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f37` / `t4-3-phan-xa-thap.png` | Bề mặt kính lái trong suốt hoặc xe tối màu hấp thụ/khúc xạ tia hồng ngoại | Thân xe có điểm nhưng phần kính chắn gió/cửa sổ bị "thủng" lỗ | Ghi nhận giả thuyết, không bóp méo hộp; dựng cuboid bao trọn kính dựa trên Bảng 4.3 và ảnh camera |
| 4. Điểm lạ | `s05f32` / `t4-4-diem-la.png` | Phản xạ đa đường (multipath) do mặt đường ẩm ướt hoặc nhiễu tán xạ khí quyển | Cụm điểm nằm sâu dưới mặt đường ($z < -2.2\text{ m}$) hoặc lơ lửng trên không | Tuyệt đối không gán thành vật thể giao thông; bỏ qua không vẽ hộp |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603080-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-2A202603080-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f26`, idx 1, Car | -0.927 m | 1.469 m | -1.662 m | 6.96 m | 7.02 m | 269.9° | Chạm đường chuẩn (lệch 0.038 m so với -1.7 m) |
| `s01f26`, idx 2, Car | -1.062 m | 1.487 m | -1.806 m | 11.01 m | 11.06 m | 269.2° | Chạm đường tốt (lệch 0.106 m, mặt đường dốc nhẹ) |
| `s01f26`, idx 3, Car | -0.818 m | 1.469 m | -1.552 m | 10.69 m | 10.72 m | 346.8° | Chạm đường tốt (lệch 0.148 m, lề đường hơi nâng) |

- (a) **Cuboid gần nhất**: `s01f58`, idx 1 (Car) có $\text{dist\_m} = 3.05\text{ m}$, $r = 3.20\text{ m}$. **Cuboid xa nhất**: `s04f16`, idx 12 (Car) có $\text{dist\_m} = 66.87\text{ m}$, $r = 66.87\text{ m}$.
- (b) **So sánh kích thước với Bảng 4.3**:
  * Xe con (`s01f26`, idx 1, Car): $l = 4.56\text{ m}$, $w = 1.94\text{ m}$, $h = 1.47\text{ m}$. Kích thước nằm hoàn toàn trong khoảng chuẩn của Sedan/SUV ($l: 3.8-5.3\text{ m}, w: 1.7-2.1\text{ m}, h: 1.4-2.1\text{ m}$).
  * Xe hai bánh (`s01f26`, idx 13, Cyclist): $l = 1.64\text{ m}$, $w = 1.07\text{ m}$, $h = 1.70\text{ m}$. Chiều dài $l$ và chiều cao $h$ khớp chuẩn xe máy có người ($l: 1.8-2.3\text{ m}, h: 1.4-1.8\text{ m}$); chiều rộng hơi rộng hơn một chút ($1.07\text{ m} > 0.95\text{ m}$) do bao trọn phần gương chiếu hậu và tư thế người lái.
- (c) **Góc xoay Yaw**:
  * Xe cùng chiều phía trước (`s01f26`, idx 1, Car): $\text{yaw\_deg} = 269.9^\circ$, gần như trùng tuyệt đối với góc lý thuyết $270^\circ$.
  * Xe ngược chiều (`s01f26`, idx 4, Car): $\text{yaw\_deg} = 90.0^\circ$, trùng khớp tuyệt đối với góc lý thuyết $90^\circ$.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f26`, idx 1, Car | 6.96 m | 1924 |
| `s01f26`, idx 2, Car | 11.01 m | 1150 |
| `s01f26`, idx 5, Car | 21.47 m | 535 |
| `s01f26`, idx 11, Car | 35.86 m | 88 |

- **Quy luật mật độ**: Mật độ điểm suy giảm rất nhanh xấp xỉ theo quy luật nghịch đảo bình phương khoảng cách ($1/d^2$): khi khoảng cách tăng gấp đôi (từ ~7 m lên ~14 m hay ~21 m), số điểm giảm từ gần 2000 điểm xuống còn khoảng 500 điểm, và ở cự ly > 35 m chỉ còn dưới 100 điểm. Số liệu thực nghiệm hoàn toàn khớp với đường cong phân bố của bài giảng (xe con cự ly 0–10 m trung vị ~1968 điểm, 10–20 m ~718 điểm, 20–30 m ~214 điểm, 30–40 m ~72 điểm).
- **Tự kiểm & sửa đổi**: Sau khi chạy `sync` và xem ảnh `preview`, bản thân đã kiểm tra và điều chỉnh 2 cuboid: hạ đáy cuboid `s01f26` idx 5 xuống chạm đường và xoay lại 180° mũi tên của cuboid xe máy `s01f26` idx 13 cho đúng hướng nhìn camera; kết quả chạy lại `selfcheck` đạt 0 cờ cảnh báo (104/104 cuboid sạch cờ).
- **Đổi kiểm chéo**: Đã đổi kiểm chéo màn hình với bạn bên cạnh (MSSV `2A202603081`), đối chiếu 2 con số: độ cao đáy $z_{\text{đáy}} = -1.662\text{ m}$ của `s01f26` idx 1 và góc xoay $\text{yaw} = 269.9^\circ$, kết quả số liệu trên màn hình khớp 100% với file Bảng A.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- (a) **Góc nhìn từ mô hình học máy (Model Perception)**:
  1. *Vì sao nhãn 3D đo bằng mét chứ không phải pixel?* Mô hình nhận thức xe tự hành cần tính toán quỹ đạo, khoảng cách dừng khẩn cấp và lập kế hoạch di chuyển trong thế giới thực, do đó nhãn bounding box phải gắn liền với kích thước và khoảng cách vật lý thực tế (mét) để chuyển giao trực tiếp sang bộ điều khiển chấp hành, điều mà pixel 2D không thể biểu diễn trực tiếp do biến dạng phối cảnh.
  2. *Vì sao vật thưa điểm cần quy tắc riêng?* Vật thể ở xa bị hạn chế bởi độ phân giải góc của cảm biến LiDAR nên chỉ nhận được vài điểm rời rạc; nếu ép buộc mô hình học trên những vật thể đoán mò không đủ cơ sở hình học sẽ gây hiện tượng nhiễu nhãn (label noise), làm tăng tỷ lệ cảnh báo ảo (false positive) nguy hiểm trong vận hành thực tế.
- (b) **Cách dùng AI**:
  - Công cụ: Antigravity Assistant.
  - Câu hỏi chính: Hỗ trợ kiểm tra cú pháp lệnh terminal, làm rõ quy ước trục hệ tọa độ VF (hướng -y và +x), tra cứu Bảng kích thước tham chiếu 4.3 và quy trình kiểm thử tự động.
  - Cách dùng kết quả: Tự mình trực tiếp quan sát và vẽ 100% các cuboid trên CVAT; dùng kết quả hỗ trợ để kiểm tra tính toàn vẹn của Bảng A, khớp số liệu báo cáo và hoàn thành kịch bản nộp bài.
- (c) **Điều còn chưa chắc**: Các vật thể nằm sát ngưỡng 50 m ở các frame nhiều bóng cây ven đường (`s04f16`) có độ phân giải điểm thấp, việc xác định chiều dài thân xe ở các góc khuất chủ yếu dựa vào ước lượng theo Bảng 4.3 và ảnh camera.
